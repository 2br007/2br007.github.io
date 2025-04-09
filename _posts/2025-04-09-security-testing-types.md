---
layout: post
title: "Security Testing Types"
date: 2025-04-09 00:00:00 +0100
categories: [hacking, theory]
tags: [security testing types]
---

![sec-types](pics/sec-types.png)


### Vulnerability Scanning
Automated process that identifies known security weaknesses in systems, networks, and applications.
Typically performed regularly to detect outdated software, misconfigurations, and exposed services.
- Tools:

  - Nessus

  - OpenVAS

  - Qualys

- Use Cases:

  - Monthly security checks on infrastructure

  - Preparing for compliance audits

  - Asset discovery and risk baseline

### Penetration Testing (Pentesting)
Ethical hacking method where testers simulate real attacks to exploit vulnerabilities.
Involves manual and automated techniques; can be black-box, white-box, or gray-box.
- Tools:

  - Metasploit

  - Burp Suite

  - Nmap

- Use Cases:

  - Test web applications before deployment

  - Assess internal network security

  - Evaluate response of security controls

### Security Auditing
Formal review of system configurations, policies, and practices against security standards.
Focuses on compliance, documentation, and procedural integrity.
- Tools:

  - Lynis (for Linux)

  - CIS-CAT

  - Manual policy checks

- Use Cases:

  - Compliance with HIPAA, PCI-DSS, ISO 27001

  - Internal IT governance reviews

  - Third-party security evaluations

### Risk Assessment
Identifying potential threats, vulnerabilities, and the impact of risks on assets.
Not a technical test, but a strategic analysis and prioritization process.
- Tools:

  - OCTAVE

  - FAIR

  - Microsoft Threat Modeling Tool

- Use Cases:

  - Prioritize investment in controls

  - Create business continuity plans

  - Understand organization-wide threat exposure

### Security Posture Assessment
Holistic analysis of an organization’s current security strength.
Combines elements of audits, pentests, risk assessments, and social engineering.
- Tools:

  - Custom framework using multiple tools (SIEM, vulnerability scanners, pentesting kits)

- Use Cases:

  - Executive-level reporting

  - Benchmarking security improvements

  - Mergers and acquisitions security checks

### Static Application Security Testing (SAST)
Analyzes source code or binaries for vulnerabilities without executing the application.
Integrated into the Software Development Lifecycle (SDLC); ideal for early bug detection.
- Tools:

  - SonarQube

  - Checkmarx

  - Fortify

- Use Cases:

  - CI/CD pipeline integration

  - Secure coding practices

  - Early detection of logic flaws or insecure code

### Dynamic Application Security Testing (DAST)
Tests applications in real-time to find security issues while the app is running.
Black-box approach that doesn’t require source code access.
- Tools:

  - OWASP ZAP

  - Burp Suite (Pro)

  - Acunetix

- Use Cases:

  - Live website/app testing for OWASP Top 10 flaws

  - QA phase security verification

  - Cloud-based app scanning

### Fuzz Testing (Fuzzing)
Bombards software with random or malformed data to find crashes, bugs, and unexpected behavior.
Often used to test parsers, protocols, and input handling in software.
- Tools:

  - AFL (American Fuzzy Lop)

  - Peach Fuzzer

  - Boofuzz

- Use Cases:

  - Discover memory corruption vulnerabilities

  - Test custom protocols or file parsers

  - Secure embedded or IoT devices