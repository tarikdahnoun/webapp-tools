# Cross Site Scripting (XSS)

Cheat sheet: <https://portswigger.net/web-security/cross-site-scripting/cheat-sheet>

Three main types:

* Reflected XSS, where the malicious script comes from the current HTTP request.
* Stored XSS, where the malicious script comes from the website's database.
* DOM-based XSS, where the vulnerability exists in client-side code rather than server-side code.

An attacker who exploits a cross-site scripting vulnerability is typically able to:

* Impersonate or masquerade as the victim user.
* Carry out any action that the user is able to perform.
* Read any data that the user is able to access.
* Capture the user's login credentials.
* Perform virtual defacement of the web site.
* Inject trojan functionality into the web site.

What is the difference between XSS and CSRF? XSS involves causing a web site to return malicious JavaScript, while CSRF vinvolves inducing a victim user to perform actions they do not intend to do.

What is the difference between XSS and SQL injection? XSS is a client-side vulnerability that targets other application users, while SQL injection is a server-side vulnerability that targets the application's database.

## Common test methdods

The most common test is trying an input like:

``` html
<script>alert(1)</script>
```

For DOM-based XSS, the previous test wont usually work since methods like innerHTML and document.write dont execute JS code in a `<script>` tag. A SVG element, however, will execute at onload:

``` html
<svg onload=alert(1)>
```

or, to trigger on error:

``` html
<img src=1 onerror=alert(1)>
```

A common exploit with jQuery is using `location.hash` to inject XSS into the `$()` selector sink. This is usually patched by preventing input that begins with `#` but it still possible to find.

To exploit, need to trigger an iframe
``` html
<iframe src="https://vulnerable-website.com#" onload="this.src+='<img src=1 onerror=alert(1)>'">
```
