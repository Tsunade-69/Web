## Introduction

#### **Cross-Origin Resource Sharing (CORS)**

CORS is a browser security feature that allows a website to safely access resources from a different website or domain.

#### **Same-Origin Policy (SOP)**

The SOP is a security rule that blocks a website from accessing data/resources from another website.

An Origin is identified by its scheme (HTTP/HTTPS), domain, and port.

It allows a domain to issue requests to other domain, but not to access the responses.

---

## Understanding SOP

#### Same Origin Policy

The same-origin policy restricts scripts on one origin from accessing data from another origin. 

**What is an Origin**:

An **origin** is made up of:

- **Scheme/Protocol (HTTP/HTTPS)**
- **Domain**
- **Port**

**Example**:

```
   https://example.com:443
      │            │                │
      │            │                └── Port
      │            └──────────── Domain
      └──────────────────── Scheme/Protocol
```

All three must match to be considered the **same origin**.

**Practical Examples**

Suppose the current website is:

```
 https://example.com
```

| URL | Same Origin? | Reason |
| --- | --- | --- |
| https://example.com/home | ✅ Yes | Same protocol, domain, port |
| https://example.com/profile | ✅ Yes | Same origin |
| http://example.com | ❌ No | Different protocol |
| https://api.example.com | ❌ No | Different subdomain |
| https://google.com | ❌ No | Different domain |
| https://example.com:8080 | ❌ No | Different port |

**Note:** Modern browsers treat the **protocol, domain, and port** as part of the origin, so a different port is considered a different origin and access is blocked by SOP.  **Older versions of Internet Explorer** ignored the port number in some cases, but this legacy behavior is no longer relevant since IE is retired.

#### Why is SOP Needed?

Imagine you're logged into:

```
https://gmail.com
```

Your browser stores Gmail session cookies.

Now you visit:

```powershell
https://evil.com
```

Without SOP, JavaScript on **evil.com** could do:

```powershell
fetch("https://gmail.com")
```

and read:

- Emails
- Contacts
- Attachments
- Private data

That would be disastrous.

**SOP blocks the malicious website from reading Gmail’s response.**

**What Does SOP Allow?**

Browsers still allow loading resources from other websites.

For example:

```
<img src="https://example.com/logo.png">

<script src="https://cdn.example.com/jquery.js"></script>

<video src="movie.mp4"></video>
```

These resources **can be loaded**, but JavaScript **cannot read their contents** unless permission is granted (e.g., via CORS).

#### Cookies and Same-Origin Policy (SOP)

SOP applies to JavaScript, not directly to cookies.

Cookies follow their own rules based on the **`Domain`** and **`Path`** attributes.

**Cookie Sharing Across Subdomains**

If a cookie is set for the parent domain:

```
Set-Cookie: session=abc123; Domain=example.com
```

The browser automatically sends it to:

- `app.example.com`
- `admin.example.com`
- `api.example.com`

Even though these are **different origins** under SOP.

**Note:**

SOP restricts JavaScript from accessing cross-origin data, but cookies are governed by their own `Domain` and `Path` attributes. A cookie set for `example.com` can be shared with its subdomains. Using the `HttpOnly` flag prevents JavaScript from reading the cookie, helping mitigate XSS attacks.

---

## Understanding CORS

#### What is CORS?

CORS (Cross-Origin Resource Sharing) is a browser security mechanism that allows a server to relax the Same-Origin Policy (SOP) and permit a website to access resources from another origin.

CORS is enforced by the browser, not the server.

The server always processes the request**.**

The browser decides whether JavaScript can read the response based on the CORS headers.

#### How CORS Works

Example:

```powershell
Frontend:
https://app.example.com

API:
https://api.example.com
```

JavaScript sends:

```powershell
fetch("https://api.example.com/user")
```

Browser adds:

```powershell
Origin: https://app.example.com
```

Server replies:

```powershell
Access-Control-Allow-Origin: https://app.example.com
```

Browser checks:

- ✅ Origin allowed → JavaScript can read the response.
- ❌ Origin not allowed → Browser blocks JavaScript from reading the response.

**Note:** The server still sends the response. Only the browser blocks access to it.

#### Common CORS Headers

| Header | Purpose |
| --- | --- |
| **Access-Control-Allow-Origin** | Specifies which origin(s) can access the resource. ⭐ |
| **Access-Control-Allow-Credentials** | Allows cookies/credentials in cross-origin requests. ⭐ |
| **Access-Control-Allow-Methods** | Allowed HTTP methods (GET, POST, PUT, DELETE, etc.). |
| **Access-Control-Allow-Headers** | Allowed custom request headers. |
| **Access-Control-Max-Age** | Caches the preflight response. |

#### Access-Control-Allow-Origin

Allow only one domain:

```
Access-Control-Allow-Origin: https://app.example.com
```

Allow everyone:

```
**Access-Control-Allow-Origin: ***
```

⚠️ **Never use `*` with credentials.**

#### Access-Control-Allow-Credentials

```html
Access-Control-Allow-Credentials: true
```

Allows the browser to include:

- Cookies
- HTTP Authentication
- Client Certificates

**Important Rule:**

❌ Invalid

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

Browsers reject this because it is insecure.

Must be:

```
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Credentials: true
```

#### Simple Request vs. Preflight Request

**Simple Request**

A request is **Simple** if:

- GET
- POST
- HEAD

AND

Uses only simple Content-Types:

- `application/x-www-form-urlencoded`
- `multipart/form-data`
- `text/plain`

No custom headers.

Browser sends the request **directly**.

**Preflight Request**

If the request is **not simple**, the browser first sends:

```
OPTIONS /api/user
```

This is called a **Preflight Request**.

Browser asks:

> "Can I send this request?"

Server replies:

```
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: PUT
Access-Control-Allow-Headers: Authorization
```

If allowed:

➡️ Browser sends the real request.

Otherwise:

❌ Browser blocks it.

#### Typical CORS Flow

1. The browser first sends an HTTP request to the server.
2. The server then checks the Origin header against its list of allowed origins.
3. If the origin is allowed, the server responds with the appropriate `Access-Control-Allow-Origin` header.
4. The browser will block the cross-origin request if the origin is not allowed.

#### Common Uses of CORS

- APIs
- Single Page Applications (SPA)
- Microservices
- OAuth / SSO
- Third-party widgets
- CDN resources
- Web fonts

---

## Access-Control-Allow-Origin (ACAO) In Depth

#### ACAO Configurations

**1] Single Origin**  

```
Access-Control-Allow-Origin: https://app.example.com
```

- Only one trusted origin is allowed.
- ✅ Most secure configuration.

**2] Multiple Origin**

The server maintains a whitelist and dynamically returns the requesting origin if it is trusted.

**Example**:

```
Allowed Origins:
- https://app.example.com
- https://admin.example.com
```

If the request comes from:

```
Origin: https://admin.example.com
```

**Response**:

```
Access-Control-Allow-Origin: https://admin.example.com
```

- More flexible than a single origin.
- Ensure only trusted origins are included.

**3] Wildcard Origin**

```
Access-Control-Allow-Origin: *
```

- Allows **any website** to access the resource.
- Suitable only for **public resources** that do not contain sensitive data.
- ❌ Should never be used for authenticated APIs.

**4]**  **With Credentials**

```
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Credentials: true
```

Allows cross-origin requests with:

- Cookies
- HTTP Authentication
- Client Certificates

**Important Rule**

❌ Invalid configuration

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

Browsers reject this combination because it is insecure.

#### Why You Should Never Use `*` with Credentials

**Scenario**

Suppose you're logged into:

```
https://bank.com
```

Your browser stores a session cookie:

```
Cookie: session=abc123
```

Now you visit a malicious website:

```
https://evil.com
```

The attacker runs:

```
fetch("https://bank.com/account", {
    credentials:"include"
})
.then(res =>res.text())
.then(data =>console.log(data));
```

This tells the browser to include your **bank.com** cookies with the request.

**If the Bank Server Responded With:**

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

The browser would allow **any website** to read your authenticated response.

The attacker could access sensitive information such as:

- Account balance
- Transaction history
- Personal information

This would completely bypass the protection provided by the Same-Origin Policy.

---

## Common CORS Misconfigurations

#### What is CORS Misconfigurations?

A **CORS misconfiguration** occurs when a server incorrectly trusts origins, allowing unauthorized websites to read sensitive responses.

#### 1] Null Origin Misconfiguration

**What is a Null Origin?**

Normally, browsers send an **Origin** header like:

```
Origin: https://app.example.com
```

Sometimes, the browser sends:

```
Origin: null
```

instead.

This happens when requests originate from:

- `file://` URLs (local HTML files)
- `data:` URLs
- Sandboxed iframes (`<iframe sandbox>`)
- Some browser extensions

**Vulnerable Configuration**

Server response:

```
Access-Control-Allow-Origin: null
Access-Control-Allow-Credentials: true
```

The server trusts the **null** origin.

**Practical Attack Example**

**Step 1**

Attacker creates:

```
steal.html
```

containing:

```
fetch("https://bank.com/account", {
    credentials:"include"
})
.then(r =>r.text())
.then(console.log);
```

**Step 2**

Attacker emails it to the victim.

Example:

> "Please open the attached invoice."

Victim downloads:

```
file:///C:/Users/Alice/Desktop/steal.html
```

Since the page is opened locally:

```
Origin: null
```

**Step 3**

The vulnerable server replies:

```
Access-Control-Allow-Origin: null
Access-Control-Allow-Credentials: true
```

Browser thinks:

> "The server trusts the null origin."

JavaScript now reads:

- Account details
- Personal information
- API responses

**Secure Configuration**

Never trust:

```
Access-Control-Allow-Origin: null
```

Only allow trusted domains.

#### 2] Bad Regex in Origin Validation

**What Happens?**

Instead of comparing exact domains, developers use regex.

Example:

```
/example.com$/
```

They think it matches:

```
https://example.com
```

Unfortunately it also matches:

```
https://badexample.com
```

because the string ends with:

```
example.com
```

**Practical Example**

Whitelist intended:

```
example.com
```

Attacker registers:

```
badexample.com
```

Browser sends:

```
Origin: https://badexample.com
```

Regex matches.

Server replies:

```
Access-Control-Allow-Origin: https://badexample.com
```

Attacker now bypasses SOP.

**Another Regex Mistake**

Developer wants:

```
*.example.com
```

Instead checks:

```
Starts with example.com
```

Attacker buys:

```
example.com.attacker.com
```

Still passes the regex.

**Correct Validation**

Always compare against an exact allowlist.

Good:

```
https://app.example.com
https://admin.example.com
```

Avoid loose regex.

#### 3] Trusting Arbitrary Origin (Origin Reflection)

**What Happens?**

The browser sends:

```
Origin: https://evil.com
```

Server blindly copies it:

```
Access-Control-Allow-Origin: https://evil.com
```

instead of checking whether it's trusted.

This is called **Origin Reflection**.

**Vulnerable Server Logic**

**Pseudo-code:**

```
Access-Control-Allow-Origin = $_SERVER["Origin"];
```

No validation.

Whatever the browser sends becomes trusted.

**Practical Example**

Victim is logged into:

```
https://bank.com
```

Attacker hosts:

```
https://evil.com
```

JavaScript:

```
fetch("https://bank.com/profile", {
    credentials:"include"
})
.then(r =>r.text())
.then(console.log);
```

Browser sends:

```
Origin: https://evil.com
```

Server reflects it:

```
Access-Control-Allow-Origin: https://evil.com
Access-Control-Allow-Credentials: true
```

Browser allows JavaScript to read:

- Name
- Address
- Email
- Balance

Attacker steals user data.

**Correct Solution**

Instead of reflecting:

```
Any Origin
```

Maintain an allowlist:

```
https://app.example.com
https://admin.example.com
```

Only return ACAO if the origin exists in this list.

---

