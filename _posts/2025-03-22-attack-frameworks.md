---
layout: post
title: "Cyber Kill Chain vs Mitre ATT&CK vs Unified Kill Chain"
date: 2025-03-22 00:00:00 +0100
categories: [hacking, theory]
tags: [cyber kill chain, unified kill chain, mitre att&ck]
---

![frameworks](pics/3frameworks.png)

### Intro
Cybersecurity professionals use structured frameworks to understand, detect, and defend against cyber threats. So in this article we will talk about 3 main frameworks:
- The *Cyber Kill Chain* outlines a step-by-step attack process, from reconnaissance to data exfiltration, helping organizations identify attack stages and implement defenses.
- *MITRE ATT&CK* provides a detailed database of real-world attack techniques, offering deeper insights into adversary tactics used in penetration testing and red teaming.
- *The Unified Kill Chain (UKC)* expands on these models by adding 18 interconnected attack steps, covering advanced threats, social engineering, and insider attacks.

Together, these frameworks enhance threat detection, response strategies, and adversary simulation, making them essential for both blue team defense and offensive security testing. Understanding these models will help you to become a better expert, predict, prevent, and respond to modern cyber threats effectively.

-------
### The Cyber Kill Chain
It is a framework developed by Lockheed Martin that describes the different stages of a cyber attack, from reconnaissance to data exfiltration. It helps organizations understand and detect cyber threats at different phases. The Kill Chain consists of seven stages, each representing a step in an attack.

#### Reconnaissance
- Gathering information about the target, such as IP addresses, domains, employee details, and vulnerabilities.

```
Example:
Perform Google dorking to find sensitive files on a company’s website or scans their network using Nmap.
```

#### Weaponization
- Creating a malicious payload (e.g., malware, exploit, or phishing document) to deliver to the target.

```
Example:
Crafts a malicious Word document with a macro that drops a reverse shell.
```

#### Delivery
- Delivering the malicious payload to the victim via phishing emails, USB drops, or drive-by downloads.

```
Example:
A hacker sends a phishing email with a malicious attachment to an employee.
```

#### Exploitation
- The payload executes, exploiting a vulnerability on the target system.

```
Example:
The malicious macro in the Word document executes PowerShell commands, giving the attacker remote access.
```

#### Installation
- Installing malware or a backdoor on the compromised system for persistence.

```
Example:
The attacker installs Cobalt Strike or a Trojan to maintain remote access.
```

#### Command and Control (C2)
- The compromised machine connects to the attacker's server for further instructions.

```
Example:
The infected machine communicates with a C2 server over an HTTPS tunnel, allowing the attacker to control it remotely.
```

#### Actions on Objectives (Exfiltration or Impact)
- The attacker achieves their goal, such as stealing data, encrypting files, or disrupting services.

```
Example:
The attacker exfiltrates customer data from a database or deploys ransomware to encrypt files.
```

#### Defensive Measures for Each Stage


| Kill Chain Stage | Defensive Measures |
|------------------|--------------------|
| Reconnaissance   | Use threat intelligence and monitor logs for suspicious scans. |
| Weaponization | Implement email filtering and sandboxing for attachments. |
| Delivery | Use phishing awareness training and block malicious URLs. |
| Exploitation | Keep software patched and enable endpoint protection. |
| Installation | Implement application whitelisting and behavior monitoring. |
| C2 | Block suspicious outbound traffic and use network segmentation. |
| Actions on Objectives | Use data loss prevention (DLP) and real-time monitoring to detect anomalies. |


### MITRE ATT&CK
- It is one of the best resources for penetration testers and red teamers because it maps real-world attack techniques used by adversaries. Below is a breakdown of the most relevant tactics and techniques for penetration testing.

#### Initial Access (TA0001) – Gaining Entry

These techniques help attackers gain an initial foothold in a system.

***Phishing (T1566)*** – Sending malicious emails with payloads.

***Valid Accounts (T1078)*** – Using stolen credentials.

***Exploit Public-Facing Application (T1190)*** – Exploiting web applications (e.g., SQLi, RCE).

***Drive-By Compromise (T1189)*** – Exploiting users visiting malicious sites.

```
Example:
Test for SQL Injection, XSS, LFI, and RCE in web applications.
Conduct phishing simulations.
Attempt credential stuffing and brute-force attacks.
```

#### Execution (TA0002) – Running Malicious Code

After initial access, attackers execute malicious code.

***Command and Scripting Interpreter (T1059)*** – Using PowerShell, Bash, Python, etc.

***Scheduled Task/Job (T1053)*** – Running malicious code via scheduled tasks.

***User Execution (T1204)*** – Tricking users into running malicious files.

```
Example:
Use PowerShell Empire, Metasploit, and Python payloads.
Try macro-based attacks in documents.
Exploit misconfigured scheduled tasks.
```

#### Persistence (TA0003) – Maintaining Access

Once inside, attackers ensure they can return later.

***Create or Modify System Process (T1543)*** – Creating new services.

***Registry Run Keys/Startup Folder (T1547)*** – Persistence via Windows registry.

***Web Shell (T1505.003)*** – Deploying a malicious PHP, ASP, or JSP shell.

```
Example:
Deploy web shells after gaining access to a web server.
Use meterpreter persistence modules in Metasploit.
```

#### Privilege Escalation (TA0004) – Becoming Admin/Root

Attackers try to gain higher privileges to access more sensitive data.

***Exploitation for Privilege Escalation (T1068)*** – Exploiting kernel vulnerabilities.

***Access Token Manipulation (T1134)*** – Stealing admin tokens.

***Sudo and Sudoers File Modification (T1548.003)*** – Abusing sudo misconfigurations.

```
Example:
Use LinPEAS, WinPEAS for privilege escalation checks.
Exploit Linux misconfigurations (e.g., sudo -l).
Look for Windows token impersonation attacks.
```

#### Defense Evasion (TA0005) – Hiding from Security

Attackers try to bypass antivirus, firewalls, and logging.

***Obfuscated Files or Information (T1027)*** – Encrypting or encoding payloads.

***Disable Security Tools (T1562)*** – Turning off Windows Defender.

***Masquerading (T1036)*** – Hiding malware as legitimate software.

```
Example:
Use Veil, MSFVenom, or obfuscation tools to bypass AV.
Modify logs and clear event history to test detection.
```

#### Credential Access (TA0006) – Stealing Passwords

Attackers attempt to grab passwords and hashes.

***Brute Force (T1110)*** – Trying username/password combinations.

***OS Credential Dumping (T1003)*** – Dumping credentials from memory (mimikatz).

***Unsecured Credentials (T1552)*** – Searching for stored passwords.

```
Example:
Use Mimikatz or LaZagne to extract passwords.
Search for exposed credentials in config files.
Perform password spraying and brute-force tests.
```

#### Discovery (TA0007) – Learning the Target Environment

Attackers gather more information to move laterally.

***Network Service Scanning (T1046)*** – Identifying open ports/services.

***System Information Discovery (T1082)*** – Checking OS, users, and processes.

***File and Directory Discovery (T1083)*** – Finding sensitive files.

```
Example:
Use nmap, netstat, and SharpHound for network recon.
Enumerate users and groups using built-in Windows/Linux commands.
```

#### Lateral Movement (TA0008) – Spreading Across the Network

Attackers move to other machines after compromising one.

***Remote Services (T1021)*** – Exploiting RDP, SMB, SSH.

***Pass the Hash (T1550.002)*** – Using stolen NTLM hashes.

***Windows Admin Shares (T1077)*** – Moving via C$ or ADMIN$ shares.

```
Example:
Use psexec, RDP hijacking, or SSH pivoting.
Perform Pass-the-Hash attacks using impacket tools.
```

#### Collection (TA0009) – Gathering Sensitive Data

Attackers search for important files, emails, and credentials.

***Screen Capture (T1113)*** – Taking screenshots of the victim’s system.

***Email Collection (T1114)*** – Accessing email inboxes.

***Input Capture (T1056)*** – Keylogging user input.

```
Example:
Deploy keyloggers or memory dumping tools.
Search for sensitive documents in C:\Users\Documents.
```

#### Exfiltration (TA0010) – Stealing Data

Attackers steal sensitive data for financial or espionage purposes.

***Automated Exfiltration (T1020)*** – Using scripts to send data out.

***Exfiltration Over Web (T1567)*** – Sending stolen data via HTTP.

***Exfiltration Over C2 Channel (T1041)*** – Using C2 servers to exfiltrate data.

```
Example:
Use exfiltration over DNS tunnels.
Test data exfiltration through HTTP/HTTPS.
```

#### How to Use MITRE ATT&CK for Pentesting?
- Map your attack path – Align your pentesting steps with MITRE techniques.
- Use ATT&CK Navigator – A visual tool to track attack progress.
- Simulate real-world attacks – Red team exercises should use ATT&CK techniques.
- Develop better defense strategies – Blue teams can use ATT&CK to detect attacks.

### Unified Kill Chain

The 18 steps are grouped into three attack phases:

- Initial Foothold (Steps 1–8) – Gaining access
- Network Propagation (Steps 9–13) – Expanding control
- Action on Objectives (Steps 14–18) – Achieving attack goals

#### Initial Foothold (Steps 1–8)

This phase focuses on getting initial access to the target network.

**Reconnaissance** - Gathering information (OSINT, network scanning, social engineering).

**Resource Development** - Acquiring infrastructure (e.g., C2 servers, domains, malware).

**Weaponization** - Creating malware or exploits (Trojanized software, exploit kits).

**Delivery** - Sending payloads (phishing emails, drive-by downloads, infected USBs).

**Social Engineering** - Tricking users into running malicious code.

**Exploitation** - Taking advantage of software, network, or human vulnerabilities.

**Persistence** - Maintaining access (creating backdoors, modifying registry keys).

**Defense Evasion** - Avoiding detection (code obfuscation, disabling antivirus, rootkits).

```
Example:
Attacker sends a phishing email with a malicious macro document.
User opens the document, triggering PowerShell execution.
Meterpreter payload is executed, giving the attacker access.
```

#### Network Propagation (Steps 9–13)

Once inside, the attacker spreads across the network.

**Credential Access** – Stealing credentials (Mimikatz, keyloggers, Pass-the-Hash).

**Privilege Escalation** – Gaining higher access (exploiting vulnerabilities, abusing sudo).

**Discovery** – Mapping the environment (checking user accounts, services, file shares).

**Lateral Movement** – Spreading to other systems (RDP, SMB, SSH, Pass-the-Ticket).

**Execution** – Running malicious payloads remotely (PowerShell scripts, scheduled tasks).

```
Example:
Attacker dumps NTLM hashes using Mimikatz.
Uses Pass-the-Hash attack to move laterally to an Active Directory domain controller.
Deploys remote PowerShell execution to gain access to critical servers.
```

#### Action on Objectives (Steps 14–18)

The final phase, where the attacker achieves their goal.

**Collection** – Gathering sensitive data (file searches, keylogging, database dumping).

**Command and Control (C2)** – Maintaining long-term control (C2 beacons, DNS tunneling).

**Exfiltration** – Stealing data (FTP, cloud storage, covert channels).

**Impact** – Disrupting operations (ransomware, DoS, wiper malware).

**Sustainment** – Staying undetected (cleaning logs, hiding backdoors, insider threats).

```
Example:
Attacker steals financial data and uploads it to a cloud server.
Deploys ransomware to encrypt files and demand Bitcoin payment.
Leaves a hidden backdoor for future attacks.
```

#### Why Use Unified Kill Chain?

The UKC improves on Cyber Kill Chain by:
- Covering advanced attack techniques (e.g., social engineering, insider threats).
- Mapping closely to real-world attacks (e.g., APTs, ransomware).
- Integrating MITRE ATT&CK techniques for a more granular approach.
- Helping blue teams detect multi-stage attacks.
- Supporting red team operations for full adversary emulation.

#### For Penetration Testers & Red Teamers:

 - The UKC can be used to plan complex attack chains during red team exercises.

 - Many MITRE ATT&CK techniques map directly to UKC steps.

 - Helps simulate real-world adversary behavior in a structured way.

### Final Thoughts

| Framework	| Best For	| Focus |
|-----------|-----------|-------|
| Cyber Kill Chain	| Basic security teams, traditional defense	| External attacks, early-stage detection |
| MITRE ATT&CK | Advanced SOC teams, red teaming | TTPs of real-world attackers |
| Unified Kill Chain | Advanced defenders, holistic security | Full attack lifecycle, APTs |

-If you need a simple, easy-to-understand approach, use Cyber Kill Chain.

-If you want deep insights into attacker techniques, use MITRE ATT&CK.

-If you want a full lifecycle approach with both defensive and offensive views, go for Unified Kill Chain.

For the best security posture, a combination of MITRE ATT&CK + Unified Kill Chain is often the most effective strategy.