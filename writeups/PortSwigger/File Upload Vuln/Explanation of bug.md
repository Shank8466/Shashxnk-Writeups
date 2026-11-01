
---

# What are file upload vulnerabilities?

```
File upload vulnerabilities are when a web server allows users to upload files to its filesystem without sufficiently validating things like their name, type, contents, or size. Failing to properly enforce restrictions on these could mean that even a basic image upload function can be used to upload arbitrary and potentially dangerous files instead. This could even include server-side script files that enable remote code execution.

In some cases, the act of uploading the file is in itself enough to cause damage. Other attacks may involve a follow-up HTTP request for the file, typically to trigger its execution by the server. 
```

---
# What is the impact of file upload vulnerabilities?

```
The impact of file upload vulnerabilities generally depends on two key factors:

    Which aspect of the file the website fails to validate properly, whether that be its size, type, contents, and so on.
    What restrictions are imposed on the file once it has been successfully uploaded.

In the worst case scenario, the file's type isn't validated properly, and the server configuration allows certain types of file (such as .php and .jsp) to be executed as code. In this case, an attacker could potentially upload a server-side code file that functions as a web shell, effectively granting them full control over the server.

If the filename isn't validated properly, this could allow an attacker to overwrite critical files simply by uploading a file with the same name. If the server is also vulnerable to directory traversal, this could mean attackers are even able to upload files to unanticipated locations.

Failing to make sure that the size of the file falls within expected thresholds could also enable a form of denial-of-service (DoS) attack, whereby the attacker fills the available disk space. 
```

---
# How do file upload vulnerabilities arise?

```
 Given the fairly obvious dangers, it's rare for websites in the wild to have no restrictions whatsoever on which files users are allowed to upload. More commonly, developers implement what they believe to be robust validation that is either inherently flawed or can be easily bypassed.

For example, they may attempt to blacklist dangerous file types, but fail to account for parsing discrepancies when checking the file extensions. As with any blacklist, it's also easy to accidentally omit more obscure file types that may still be dangerous.

In other cases, the website may attempt to check the file type by verifying properties that can be easily manipulated by an attacker using tools like Burp Proxy or Repeater.

Ultimately, even robust validation measures may be applied inconsistently across the network of hosts and directories that form the website, resulting in discrepancies that can be exploited. 
```

---
# Exploiting flawed validation of file uploads

```
In the wild, it's unlikely that you'll find a website that has no protection against file upload attacks like we saw in the previous lab. But just because defenses are in place, that doesn't mean that they're robust. You can sometimes still exploit flaws in these mechanisms to obtain a web shell for remote code execution.
Flawed file type validation

When submitting HTML forms, the browser typically sends the provided data in a POST request with the content type application/x-www-form-urlencoded. This is fine for sending simple text like your name or address. However, it isn't suitable for sending large amounts of binary data, such as an entire image file or a PDF document. In this case, the content type multipart/form-data is preferred.

Consider a form containing fields for uploading an image, providing a description of it, and entering your username. Submitting such a form might result in a request that looks something like this:
POST /images HTTP/1.1
    Host: normal-website.com
    Content-Length: 12345
    Content-Type: multipart/form-data; boundary=---------------------------012345678901234567890123456

    ---------------------------012345678901234567890123456
    Content-Disposition: form-data; name="image"; filename="example.jpg"
    Content-Type: image/jpeg

    [...binary content of example.jpg...]

    ---------------------------012345678901234567890123456
    Content-Disposition: form-data; name="description"

    This is an interesting description of my image.

    ---------------------------012345678901234567890123456
    Content-Disposition: form-data; name="username"

    wiener
    ---------------------------012345678901234567890123456--

As you can see, the message body is split into separate parts for each of the form's inputs. Each part contains a Content-Disposition header, which provides some basic information about the input field it relates to. These individual parts may also contain their own Content-Type header, which tells the server the MIME type of the data that was submitted using this input.

One way that websites may attempt to validate file uploads is to check that this input-specific Content-Type header matches an expected MIME type. If the server is only expecting image files, for example, it may only allow types like image/jpeg and image/png. Problems can arise when the value of this header is implicitly trusted by the server. If no further validation is performed to check whether the contents of the file actually match the supposed MIME type, this defense can be easily bypassed using tools like Burp Repeater. 

In short:

- **Uploading a `.php`/script file doesn't automatically mean it will execute.**
- Servers are usually configured so that only certain directories can **execute scripts**.
- An upload directory like `/uploads/` may be configured to **store files only**, so `exploit.php` is returned as text instead of executed.
- However, if an attacker can somehow make the script get saved in a **different directory where script execution is enabled**, it may execute.
- The `filename` parameter in a `multipart/form-data` upload can influence **what filename/path the server uses when saving the file**.

**Core idea:**

> Uploading a malicious script is only dangerous if you can get it into a location where the server will actually execute that script.
```

```
 You should also note that even though you may send all of your requests to the same domain name, this often points to a reverse proxy server of some kind, such as a load balancer. Your requests will often be handled by additional servers behind the scenes, which may also be configured differently.
Insufficient blacklisting of dangerous file types

One of the more obvious ways of preventing users from uploading malicious scripts is to blacklist potentially dangerous file extensions like .php. The practice of blacklisting is inherently flawed as it's difficult to explicitly block every possible file extension that could be used to execute code. Such blacklists can sometimes be bypassed by using lesser known, alternative file extensions that may still be executable, such as .php5, .shtml, and so on.
Overriding the server configuration

As we discussed in the previous section, servers typically won't execute files unless they have been configured to do so. For example, before an Apache server will execute PHP files requested by a client, developers might have to add the following directives to their /etc/apache2/apache2.conf file:
LoadModule php_module /usr/lib/apache2/modules/libphp.so
    AddType application/x-httpd-php .php

Many servers also allow developers to create special configuration files within individual directories in order to override or add to one or more of the global settings. Apache servers, for example, will load a directory-specific configuration from a file called .htaccess if one is present.

Similarly, developers can make directory-specific configuration on IIS servers using a web.config file. This might include directives such as the following, which in this case allows JSON files to be served to users:
<staticContent>
    <mimeMap fileExtension=".json" mimeType="application/json" />
    </staticContent>

Web servers use these kinds of configuration files when present, but you're not normally allowed to access them using HTTP requests. However, you may occasionally find servers that fail to stop you from uploading your own malicious configuration file. In this case, even if the file extension you need is blacklisted, you may be able to trick the server into mapping an arbitrary, custom file extension to an executable MIME type. 

### Short explanation

This section describes **ways file-upload restrictions can fail**:

1. **Reverse proxy / multiple servers**
    
    - You may see one domain, but requests can pass through a load balancer/reverse proxy to different backend servers.
        
    - Those backend servers may have **different configurations**, so behavior can vary.
        
2. **Weak extension blacklist**
    
    - Blocking only `.php` isn't enough.
        
    - Other extensions such as `.php5` or `.shtml` might also be interpreted as executable by some server configurations.
        
    - **Idea:** blacklists are incomplete because they try to list every dangerous extension.
        
3. **Configuration-file upload**
    
    - Some servers allow directory-specific configuration files:
        
        - Apache → `.htaccess`
            
        - IIS → `web.config`
            
    - If an upload function lets an attacker upload/overwrite such a configuration file, they may be able to **change how the server handles certain file extensions**.
        
    - This can potentially turn an otherwise harmless-looking uploaded file into something the server treats as executable.
        

### The main concept

Think of it as:

**Upload restriction → filename extension check → server configuration → actual execution**

A weakness at any stage can potentially undermine the protection.
```

---
```
Obfuscating file extensions

Even the most exhaustive blacklists can potentially be bypassed using classic obfuscation techniques. Let's say the validation code is case sensitive and fails to recognize that exploit.pHp is in fact a .php file. If the code that subsequently maps the file extension to a MIME type is not case sensitive, this discrepancy allows you to sneak malicious PHP files past validation that may eventually be executed by the server.

You can also achieve similar results using the following techniques:

    Provide multiple extensions. Depending on the algorithm used to parse the filename, the following file may be interpreted as either a PHP file or JPG image: exploit.php.jpg
    Add trailing characters. Some components will strip or ignore trailing whitespaces, dots, and suchlike: exploit.php.
    Try using the URL encoding (or double URL encoding) for dots, forward slashes, and backward slashes. If the value isn't decoded when validating the file extension, but is later decoded server-side, this can also allow you to upload malicious files that would otherwise be blocked: exploit%2Ephp
    Add semicolons or URL-encoded null byte characters before the file extension. If validation is written in a high-level language like PHP or Java, but the server processes the file using lower-level functions in C/C++, for example, this can cause discrepancies in what is treated as the end of the filename: exploit.asp;.jpg or exploit.asp%00.jpg
    Try using multibyte unicode characters, which may be converted to null bytes and dots after unicode conversion or normalization. Sequences like xC0 x2E, xC4 xAE or xC0 xAE may be translated to x2E if the filename parsed as a UTF-8 string, but then converted to ASCII characters before being used in a path.

Other defenses involve stripping or replacing dangerous extensions to prevent the file from being executed. If this transformation isn't applied recursively, you can position the prohibited string in such a way that removing it still leaves behind a valid file extension. For example, consider what happens if you strip .php from the following filename:
exploit.p.phphp

This is just a small selection of the many ways it's possible to obfuscate file extensions. 

### Short explanation

The main idea is **validation and server parsing may interpret the filename differently**.

For example:

- **Case differences:** `exploit.pHp`  
    Validation may reject only lowercase `.php`, while the server may still recognize `.pHp` as PHP.
    
- **Multiple extensions:** `exploit.php.jpg`  
    One component may see `.jpg`, while another may process `.php`.
    
- **Trailing characters:** `exploit.php.`  
    Some systems may remove the trailing `.` and end up with `exploit.php`.
    
- **Encoding:** `exploit%2Ephp`  
    If validation checks the encoded version but the server later decodes it, the server may see `exploit.php`.
    
- **Different parsers:** Different programming languages/server components can disagree about where the filename ends, creating inconsistencies.
    
- **Unicode normalization:** Different components may normalize characters differently, potentially changing what extension is ultimately interpreted.
    

### Important concept

This is essentially a **parser/validation mismatch**:

> **Validator sees:** harmless filename  
> **Server sees:** executable filename

Also, simply **removing** a dangerous string can be unsafe if the filtering isn't done carefully. The example `exploit.p.phphp` demonstrates how a single-pass removal of `.php` can potentially leave another `.php` behind.

These techniques are commonly studied in **file-upload security labs** because they show why robust upload validation should use an allowlist and consistent filename/path handling.
```

---
```
 Instead of implicitly trusting the Content-Type specified in a request, more secure servers try to verify that the contents of the file actually match what is expected.

In the case of an image upload function, the server might try to verify certain intrinsic properties of an image, such as its dimensions. If you try uploading a PHP script, for example, it won't have any dimensions at all. Therefore, the server can deduce that it can't possibly be an image, and reject the upload accordingly.

Similarly, certain file types may always contain a specific sequence of bytes in their header or footer. These can be used like a fingerprint or signature to determine whether the contents match the expected type. For example, JPEG files always begin with the bytes FF D8 FF.

This is a much more robust way of validating the file type, but even this isn't foolproof. Using special tools, such as ExifTool, it can be trivial to create a polyglot JPEG file containing malicious code within its metadata. 
```