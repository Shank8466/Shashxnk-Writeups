
---
# What are business logic vulnerabilities?

```
Business logic vulnerabilities are flaws in the design and implementation of an application that allow an attacker to elicit unintended behavior. This potentially enables attackers to manipulate legitimate functionality to achieve a malicious goal. These flaws are generally the result of failing to anticipate unusual application states that may occur and, consequently, failing to handle them safely.
Note

In this context, the term "business logic" simply refers to the set of rules that define how the application operates. As these rules aren't always directly related to a business, the associated vulnerabilities are also known as "application logic vulnerabilities" or simply "logic flaws".

Logic flaws are often invisible to people who aren't explicitly looking for them as they typically won't be exposed by normal use of the application. However, an attacker may be able to exploit behavioral quirks by interacting with the application in ways that developers never intended.

One of the main purposes of business logic is to enforce the rules and constraints that were defined when designing the application or functionality. Broadly speaking, the business rules dictate how the application should react when a given scenario occurs. This includes preventing users from doing things that will have a negative impact on the business or that simply don't make sense.

Flaws in the logic can allow attackers to circumvent these rules. For example, they might be able to complete a transaction without going through the intended purchase workflow. In other cases, broken or non-existent validation of user-supplied data might allow users to make arbitrary changes to transaction-critical values or submit nonsensical input. By passing unexpected values into server-side logic, an attacker can potentially induce the application to do something that it isn't supposed to.

Logic-based vulnerabilities can be extremely diverse and are often unique to the application and its specific functionality. Identifying them often requires a certain amount of human knowledge, such as an understanding of the business domain or what goals an attacker might have in a given context. This makes them difficult to detect using automated vulnerability scanners. As a result, logic flaws are a great target for bug bounty hunters and manual testers in general.

### Business Logic Vulnerabilities — Short Explanation

**Business logic vulnerabilities are flaws in an application's rules that allow an attacker to make the application behave in an unintended way.**

Think of it as:

> **Application rules → Attacker finds a way to bypass/manipulate those rules → Unexpected behavior**

### Example

An online store says:

> "You must pay before downloading a product."

If an attacker discovers a way to directly access the download endpoint **without paying**, that's a business logic flaw.

Another example:

> Maximum quantity = 5

If the attacker sends `quantity=1000` and the server accepts it, the application's business rule was not properly enforced.

### Why they're important for bug bounty

- Often **unique to each application**
    
- Usually require **manual testing and understanding the workflow**
    
- Automated scanners often **can't detect them**
    
- You need to think: **"What happens if I do this in an unexpected order or manipulate this value?"**
    

**Core mindset:** 🧠

> Don't just test whether each feature works — test whether you can **break the application's intended rules by using legitimate functionality in unintended ways.**

```

---
# How do business logic vulnerabilities arise?

```
 Business logic vulnerabilities often arise because the design and development teams make flawed assumptions about how users will interact with the application. These bad assumptions can lead to inadequate validation of user input. For example, if the developers assume that users will pass data exclusively via a web browser, the application may rely entirely on weak client-side controls to validate input. These are easily bypassed by an attacker using an intercepting proxy.

Ultimately, this means that when an attacker deviates from the expected user behavior, the application fails to take appropriate steps to prevent this and, subsequently, fails to handle the situation safely.

Logic flaws are particularly common in overly complicated systems that even the development team themselves do not fully understand. To avoid logic flaws, developers need to understand the application as a whole. This includes being aware of how different functions can be combined in unexpected ways. Developers working on large code bases may not have an intimate understanding of how all areas of the application work. Someone working on one component could make flawed assumptions about how another component works and, as a result, inadvertently introduce serious logic flaws. If the developers do not explicitly document any assumptions that are being made, it is easy for these kinds of vulnerabilities to creep into an application. 

In short:

**Business logic vulnerabilities happen when developers make wrong assumptions about how users will use an application.**

For example:

- Developer assumes users will only use the website normally.
    
- They put important validation only on the **client side**.
    
- An attacker intercepts the request and changes the values.
    
- The server doesn't properly validate the modified request → **logic flaw**.
    

### Simple example

A shopping site says:

> “Users can buy a maximum of 5 items.”

But the server only checks this limit in JavaScript. An attacker intercepts the request and changes:

`quantity=5` → `quantity=500`

If the server accepts it, that's a **business logic vulnerability**.

**Key idea:**

> The application works correctly for the expected user behavior, but breaks when someone does something the developers didn't anticipate.

```

---
#  The impact of business logic vulnerabilities
 
```
 The impact of business logic vulnerabilities can, at times, be fairly trivial. It is a broad category and the impact is highly variable. However, any unintended behavior can potentially lead to high-severity attacks if an attacker is able to manipulate the application in the right way. For this reason, quirky logic should ideally be fixed even if you can't work out how to exploit it yourself. There is always a risk that someone else will be able to.

Fundamentally, the impact of any logic flaw depends on what functionality it is related to. If the flaw is in the authentication mechanism, for example, this could have a serious impact on your overall security. Attackers could potentially exploit this for privilege escalation, or to bypass authentication entirely, gaining access to sensitive data and functionality. This also exposes an increased attack surface for other exploits.

Flawed logic in financial transactions can obviously lead to massive losses for the business through stolen funds, fraud, and so on.

You should also note that even though logic flaws may not allow an attacker to benefit directly, they could still allow a malicious party to damage the business in some way

In Short

### Impact of Business Logic Vulnerabilities — Short Explanation

The **impact depends on what part of the application has the logic flaw**.

- **Authentication flaw** → attacker may bypass login or escalate privileges.
    
- **Financial/payment flaw** → could cause money theft, fraud, or financial loss.
    
- **Authorization flaw** → attacker may access functionality/data they shouldn't.
    
- **Other logic flaws** → may allow abuse or damage to the business even without direct personal gain.
    

### Key idea

> **A small-looking logic flaw can sometimes become a serious vulnerability when combined with other functionality.**

That's why unusual or "quirky" application behavior should be investigated rather than ignored.

```

---
# Failing to handle unconventional input

```
 One aim of the application logic is to restrict user input to values that adhere to the business rules. For example, the application may be designed to accept arbitrary values of a certain data type, but the logic determines whether or not this value is acceptable from the perspective of the business. Many applications incorporate numeric limits into their logic. This might include limits designed to manage inventory, apply budgetary restrictions, trigger phases of the supply chain, and so on.

Let's take the simple example of an online shop. When ordering products, users typically specify the quantity that they want to order. Although any integer is theoretically a valid input, the business logic might prevent users from ordering more units than are currently in stock, for example.

To implement rules like this, developers need to anticipate all possible scenarios and incorporate ways to handle them into the application logic. In other words, they need to tell the application whether it should allow a given input and how it should react based on various conditions. If there is no explicit logic for handling a given case, this can lead to unexpected and potentially exploitable behavior.

For example, a numeric data type might accept negative values. Depending on the related functionality, it may not make sense for the business logic to allow this. However, if the application doesn't perform adequate server-side validation and reject this input, an attacker may be able to pass in a negative value and induce unwanted behavior.

Consider a funds transfer between two bank accounts. This functionality will almost certainly check whether the sender has sufficient funds before completing the transfer:
$transferAmount = $_POST['amount'];
$currentBalance = $user->getBalance();

if ($transferAmount <= $currentBalance) {
    // Complete the transfer
} else {
    // Block the transfer: insufficient funds
}

But if the logic doesn't sufficiently prevent users from supplying a negative value in the amount parameter, this could be exploited by an attacker to both bypass the balance check and transfer funds in the "wrong" direction. If the attacker sent -$1000 to the victim's account, this might result in them receiving $1000 from the victim instead. The logic would always evaluate that -1000 is less than the current balance and approve the transfer.

Simple logic flaws like this can be devastating if they occur in the right functionality. They are also easy to miss during both development and testing, especially given that such inputs may be blocked by client-side controls on the web interface.

When auditing an application, you should use tools such as Burp Proxy and Repeater to try submitting unconventional values. In particular, try input in ranges that legitimate users are unlikely to ever enter. This includes exceptionally high or exceptionally low numeric inputs and abnormally long strings for text-based fields. You can even try unexpected data types. By observing the application's response, you should try and answer the following questions:

    Are there any limits that are imposed on the data?
    What happens when you reach those limits?
    Is any transformation or normalization being performed on your input?

This may expose weak input validation that allows you to manipulate the application in unusual ways. Keep in mind that if you find one form on the target website that fails to safely handle unconventional input, it's likely that other forms will have the same issues. 

In Short


```

---
# Making flawed assumptions about user behavior

```
One of the most common root causes of logic vulnerabilities is making flawed assumptions about user behavior. This can lead to a wide range of issues where developers have not considered potentially dangerous scenarios that violate these assumptions. In this section, we'll provide some cautionary examples of common assumptions that should be avoided and demonstrate how they can lead to dangerous logic flaws.
Trusted users won't always remain trustworthy

Applications may appear to be secure because they implement seemingly robust measures to enforce the business rules. Unfortunately, some applications make the mistake of assuming that, having passed these strict controls initially, the user and their data can be trusted indefinitely. This can result in relatively lax enforcement of the same controls from that point on.

If business rules and security measures are not applied consistently throughout the application, this can lead to potentially dangerous loopholes that may be exploited by an attacker. 
```

---
# Users won't always supply mandatory input

```
One misconception is that users will always supply values for mandatory input fields. Browsers may prevent ordinary users from submitting a form without a required input, but as we know, attackers can tamper with parameters in transit. This even extends to removing parameters entirely.

This is a particular issue in cases where multiple functions are implemented within the same server-side script. In this case, the presence or absence of a particular parameter may determine which code is executed. Removing parameter values may allow an attacker to access code paths that are supposed to be out of reach.

When probing for logic flaws, you should try removing each parameter in turn and observing what effect this has on the response. You should make sure to:

    Only remove one parameter at a time to ensure all relevant code paths are reached.
    Try deleting the name of the parameter as well as the value. The server will typically handle both cases differently.
    Follow multi-stage processes through to completion. Sometimes tampering with a parameter in one step will have an effect on another step further along in the workflow.

This applies to both URL and POST parameters, but don't forget to check the cookies too. This simple process can reveal some bizarre application behavior that may be exploitable. 
```

---

```
 Making assumptions about the sequence of events can lead to a wide range of issues even within the same workflow or functionality. Using tools like Burp Proxy and Repeater, once an attacker has seen a request, they can replay it at will and use forced browsing to perform any interactions with the server in any order they want. This allows them to complete different actions while the application is in an unexpected state.

To identify these kinds of flaws, you should use forced browsing to submit requests in an unintended sequence. For example, you might skip certain steps, access a single step more than once, return to earlier steps, and so on. Take note of how different steps are accessed. Although you often just submit a GET or POST request to a specific URL, sometimes you can access steps by submitting different sets of parameters to the same URL. As with all logic flaws, try to identify what assumptions the developers have made and where the attack surface lies. You can then look for ways of violating these assumptions.

Note that this kind of testing will often cause exceptions because expected variables have null or uninitialized values. Arriving at a location in a partly defined or inconsistent state is also likely to cause the application to complain. In this case, be sure to pay close attention to any error messages or debug information that you encounter. These can be a valuable source of information disclosure, which can help you fine-tune your attack and understand key details about the back-end behavior. 
```

---
# Domain-specific flaws

```
In many cases, you will encounter logic flaws that are specific to the business domain or the purpose of the site.

The discounting functionality of online shops is a classic attack surface when hunting for logic flaws. This can be a potential gold mine for an attacker, with all kinds of basic logic flaws occurring in the way discounts are applied.

For example, consider an online shop that offers a 10% discount on orders over $1000. This could be vulnerable to abuse if the business logic fails to check whether the order was changed after the discount is applied. In this case, an attacker could simply add items to their cart until they hit the $1000 threshold, then remove the items they don't want before placing the order. They would then receive the discount on their order even though it no longer satisfies the intended criteria.

You should pay particular attention to any situation where prices or other sensitive values are adjusted based on criteria determined by user actions. Try to understand what algorithms the application uses to make these adjustments and at what point these adjustments are made. This often involves manipulating the application so that it is in a state where the applied adjustments do not correspond to the original criteria intended by the developers.

To identify these vulnerabilities, you need to think carefully about what objectives an attacker might have and try to find different ways of achieving this using the provided functionality. This may require a certain level of domain-specific knowledge in order to understand what might be advantageous in a given context. To use a simple example, you need to understand social media to understand the benefits of forcing a large number of users to follow you.

Without this knowledge of the domain, you may dismiss dangerous behavior because you simply aren't aware of its potential knock-on effects. Likewise, you may struggle to join the dots and notice how two functions can be combined in a harmful way. For simplicity, the examples used in this topic are specific to a domain that all users will already be familiar with, namely an online shop. However, whether you're bug bounty hunting, pentesting, or even just a developer trying to write more secure code, you may at some point encounter applications from less familiar domains. In this case, you should read as much documentation as possible and, where available, talk to subject-matter experts from the domain to get their insight. This may sound like a lot of work, but the more obscure the domain is, the more likely other testers will have missed plenty of bugs. 
```

---
# Providing an encryption oracle

```
Dangerous scenarios can occur when user-controllable input is encrypted and the resulting ciphertext is then made available to the user in some way. This kind of input is sometimes known as an "encryption oracle". An attacker can use this input to encrypt arbitrary data using the correct algorithm and asymmetric key.

This becomes dangerous when there are other user-controllable inputs in the application that expect data encrypted with the same algorithm. In this case, an attacker could potentially use the encryption oracle to generate valid, encrypted input and then pass it into other sensitive functions.

This issue can be compounded if there is another user-controllable input on the site that provides the reverse function. This would enable the attacker to decrypt other data to identify the expected structure. This saves them some of the work involved in creating their malicious data but is not necessarily required to craft a successful exploit.

The severity of an encryption oracle depends on what functionality also uses the same algorithm as the oracle. 
```

---
