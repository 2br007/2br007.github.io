---
layout: post
title: "Broken Access Control"
date: 2025-07-09 00:00:00 +0100
categories: [portswigger, hacking]
tags: [labs, theory, broken access control, idor, roles, privilege escalation, web]
---

![bac](pics/bac.png)

### Intro

**What Is Broken Access Control?**
- Broken Access Control occurs when restrictions on what authenticated users are allowed to do are not properly enforced. This allows attackers to act outside of their intended permissions.

**Key Concepts:**
- Access Control: Determines what actions users can perform and what resources they can access.
- Broken Access Control: Happens when these rules are improperly implemented, allowing unauthorized access to data or functions.

**Common Vulnerabilities**
- Insecure Direct Object References (IDOR): Users can access data by modifying a URL or parameter (e.g., changing user_id=123 to user_id=124).
- Privilege Escalation:
  - Horizontal: Accessing another user’s data at the same privilege level.
  - Vertical: Gaining admin-level access from a regular user account.
- Missing Function-Level Access Control: APIs or endpoints lack proper checks, allowing unauthorized actions.
- Forced Browsing: Accessing hidden or unauthorized pages by guessing URLs.
- JWT or Cookie Tampering: Modifying tokens to elevate privileges.

**Broken access control is rooted in failures of:**
- Authorization Logic: The system fails to verify if a user is allowed to perform an action.
- Least Privilege Principle: Users should only have the minimum access necessary - but this is often violated.
- State Management: Improper session handling can allow attackers to hijack or replay sessions.


### Vertical privilege escalation (Apprentice)

#### Lab: Unprotected admin functionality

- A website might host sensitive functionality at the following URL: `https://insecure-website.com/admin`. This might be accessible by any user, not only administrative users who have a link to the functionality in their user interface. In some cases, the administrative URL might be disclosed in other locations, such as the `robots.txt` file, so check for `/robots.txt` first ;)

#### Lab: Unprotected admin functionality with unpredictable URL

- In some cases, sensitive functionality is concealed by giving it a less predictable URL. This is an example of so-called "security by obscurity". However, hiding sensitive functionality does not provide effective access control because users might discover the obfuscated URL in a number of ways.
- Imagine an application that hosts administrative functions at the following URL: `https://insecure-website.com/administrator-panel-yb556`
- This might not be directly guessable by an attacker. However, the application might still leak the URL to users. The URL might be disclosed in JavaScript that constructs the user interface based on the user's role:

<script>
	var isAdmin = false;
	if (isAdmin) {
		...
		var adminPanelTag = document.createElement('a');
		adminPanelTag.setAttribute('href', 'https://insecure-website.com/administrator-panel-yb556');
		adminPanelTag.innerText = 'Admin panel';
		...
	}
</script>

- So the solution  somewhere in the JS - just open any product and inspect the page!

#### Lab: User role controlled by request parameter

- Parameter-based access control methods:
Some applications determine the user's access rights or role at login, and then store this information in a user-controllable location. This could be:

- A hidden field.
- A cookie.
- A preset query string parameter.
- The application makes access control decisions based on the submitted value. For example:

`https://insecure-website.com/login/home.jsp?admin=true`
`https://insecure-website.com/login/home.jsp?role=1`

- So the solution - checkout cookies ...

#### Lab: User role can be modified in user profile

- Very carefully check all the functionality you have (use Burp to check all the requests and responses)!

### Vertical privilege escalation (Practitioner)

#### Lab: URL-based access control can be circumvented

**Broken access control resulting from platform misconfiguration**
Some applications enforce access controls at the platform layer. they do this by restricting access to specific URLs and HTTP methods based on the user's role. For example, an application might configure a rule as follows:

`DENY: POST, /admin/deleteUser`, managers
This rule denies access to the POST method on the URL `/admin/deleteUser`, for users in the managers group. Various things can go wrong in this situation, leading to access control bypasses.

Some application frameworks support various non-standard HTTP headers that can be used to override the URL in the original request, such as X-Original-URL and X-Rewrite-URL. If a website uses rigorous front-end controls to restrict access based on the URL, but the application allows the URL to be overridden via a request header, then it might be possible to bypass the access controls using a request like the following:

```html
POST / HTTP/1.1
X-Original-URL: /admin/deleteUser
...
```

- Let's add `X-Original-URL` header to the request to home page:

```html
GET / HTTP/2
Host: 0a5600af0361a790811bbb76004e0029.web-security-academy.net
Cookie: session=ZXVcX03D2Ro7YjEaVZxfbpGJv0bBqpoG
Sec-Ch-Ua: "Not)A;Brand";v="8", "Chromium";v="138"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Linux"
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://0a5600af0361a790811bbb76004e0029.web-security-academy.net/
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
X-Original-Url: /admin
```

- Ok, we are able to see admin panel, cool (you could check delete links),  now we could try to update our initial request(pay attention that params could not be inside of the header but we could add them to a path which will be appended to `/admin/delete` part of path):

```html
GET /?username=carlos HTTP/2
Host: 0a5600af0361a790811bbb76004e0029.web-security-academy.net
Cookie: session=ZXVcX03D2Ro7YjEaVZxfbpGJv0bBqpoG
Sec-Ch-Ua: "Not)A;Brand";v="8", "Chromium";v="138"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Linux"
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://0a5600af0361a790811bbb76004e0029.web-security-academy.net
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
X-Original-Url: /admin/delete
Content-Length: 2
```

#### Lab: Method-based access control can be circumvented

- An alternative attack relates to the HTTP method used in the request. The front-end controls described in the previous sections restrict access based on the URL and HTTP method. Some websites tolerate different HTTP request methods when performing an action. If an attacker can use the GET (or another) method to perform actions on a restricted URL, they can bypass the access control that is implemented at the platform layer.

- You got admin account creds, so you can login as admin, try all the things admin can, logout and login as a user, copy admins request to promote user, update session cookies and try to send a request, try then to update method

### Horizontal privilege escalation (Apprentice)

#### Lab: User ID controlled by request parameter

- Update `id=wiener` -> `id=carlos`

#### Lab: User ID controlled by request parameter, with unpredictable user IDs

- Pay attention to a post authors

#### Lab: User ID controlled by request parameter with data leakage in redirect

- Try to update `id=wiener` -> `id=carlos` and check requests - there will be a redirect to a login page with leakage

#### Lab: User ID controlled by request parameter with password disclosure

- Try to update `id=wiener` -> `id=administrator` on password/email changing page - there is a hidden password field with prefilled password

#### Lab: Insecure direct object references

- Using Burp take a look for a downloading chat and use repeater to update id.

### Access control vulnerabilities in multi-step processes (Practitioner)

#### Lab: Multi-step process with no access control on one step

- Using admin account we could upgrade any user, to have admin previleges, then downgrade it
- Next login as wiener and copy session id
- Put account upgrade to a repeater and change admin session to wiener session

#### Lab: Referer-based access control

- I was able to solve this one in the same manner like previous one

### How to prevent access control vulnerabilities
Access control vulnerabilities can be prevented by taking a defense-in-depth approach and applying the following principles:

- Never rely on obfuscation alone for access control.
- Unless a resource is intended to be publicly accessible, deny access by default.
- Wherever possible, use a single application-wide mechanism for enforcing access controls.
- At the code level, make it mandatory for developers to declare the access that is allowed for each resource, and deny access by default.
- Thoroughly audit and test access controls to ensure they work as designed.