Spookifier — quick summary
I found server-side template evaluation in a “spooky text” generator. After confirming it was Jinja2, I used safe, incremental probes (math and filters) to fingerprint the engine, inspected what objects were reachable, and then used a small context gadget to get command output. All steps below use sanitized placeholders and are intended only for authorized labs.

Environment / prep

Attacker: Kali (VM) or any pentest box.

Tools: browser, Burp (or developer tools), curl, a text editor.

Important: do this only on HTB/THM boxes or systems you own/have permission for.

Step-by-step 

Spot the sink (where input is rendered)

What I did: I typed a small template expression into the input field, for example ${{7*7}}, and submitted.

What I saw: the page returned 49. That proved user input was being evaluated by the server, not literally echoed.

Confirm the engine (fingerprinting)

What I tried next: a couple of short, harmless probes: ${{'abc'|upper}} and ${{7*'7'}}.

What the behavior told me: filters and string-repetition behavior matched Jinja2 (common in Flask apps). Error text (if any) also hinted Jinja2.

Find the least-filtered sink

Why: some fields are sanitized, others are not.

What I did: I tested other inputs (title, preview fields, theme selector) and watched raw responses in Burp to find the best field for payloads.

Safe object inspection (don’t crash the app)

Goal: learn what’s reachable without causing a timeout or app crash.

Probes used: things like ${{ ''.__class__.__mro__ }} and small indexed checks of subclasses (but avoid printing the entire subclass dump — it’s huge and noisy).

Result: confirmed Python objects were accessible and the template context wasn’t fully sandboxed.

Search for short pivots (globals/context)

Idea: some built-in objects expose a __globals__ dict (via small functions or modules). That dict sometimes contains modules like os or subprocess.

Test payload (conceptual): access some_object.__init__.__globals__ and look for an os entry. This is a short, low-noise pivot if available.

Controlled command execution (small test commands)

Principle: run minimal commands first — whoami or id — to confirm execution without causing side effects.

Example conceptual payload: use the detected globals to call os.popen('whoami').read() and check the response. If you get the username back, you’ve confirmed safe RCE in the lab.

Fallback: subclass gadget route (if globals aren’t exposed)

If __globals__ isn’t present, you can locate classes like subprocess.Popen via __subclasses__() and then call them.

Do this carefully: enumerate names or small slices first, find the right index for the class you need, then run a short command. Don’t print huge lists.

Locate flags / sensitive files (lab-only)

After command execution works, probe likely paths: /app, /home, /var/www or run ls on small directories. Then cat the specific files (e.g., cat /flag.txt) only in lab contexts.

Cleanup & notes

Save your commands and outputs to your personal notes (sanitized).

Don’t leave artifacts on the target.

When publishing, replace IPs and flags with <target> / <flag> placeholders.

Key payload examples (sanitized placeholders — use only in authorized labs)

Engine probe: {{7*7}} → expect 49

Filter probe: {{'abc'|upper}} → expect ABC

MRO probe: {{''.__class__.__mro__}} → view type chain

Globals pivot (concept): {{ cycler.__init__.__globals__.os.popen('whoami').read() }}

Subclass gadget (concept): find index X, then {{ ''.__class__.__mro__.__subclasses__()[X].communicate() }}

Common problems & quick fixes

Braces or quotes stripped: try other sinks, URL-encoding, or build strings from slices of existing strings.

Output HTML-escaped: view the raw response (Burp / DevTools) or use a sink that returns raw text.

Large subclass lists overload the response: print only .__name__ or small slices to find the needed class.

App times out on heavy payloads: use tiny, targeted probes.
