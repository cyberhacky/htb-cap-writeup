# Hack The Box – Cap

## Professional Penetration Testing Report

**Machine:** Cap

**Platform:** Hack The Box

**Difficulty:** Easy

**Operating System:** Linux

**Assessment Type:** Black Box Penetration Test

**Author:** Cornelius Donkor

**GitHub:** https://github.com/cyberhacky

**Date:** August 2026

---

# Executive Summary

A black-box penetration test was performed against the Hack The Box **Cap** Linux machine to simulate the activities of an external attacker with no prior knowledge of the target environment.

The assessment focused on identifying exposed network services, enumerating accessible resources, validating discovered vulnerabilities, and evaluating the impact of successful exploitation.

Testing confirmed multiple weaknesses that could be chained to achieve full system compromise. The attack path began with reconnaissance and web application enumeration, leading to the discovery of a Broken Access Control vulnerability that exposed another user's packet capture. Analysis of the capture revealed plaintext credentials, which enabled authenticated SSH access. Local enumeration then identified an improperly assigned Linux capability on the Python interpreter, allowing privilege escalation to the root account.

The engagement concluded with successful administrative access, demonstrating how several individually manageable weaknesses can combine to produce a complete system compromise when layered security controls are absent.

All findings documented in this report were manually validated through direct observation and command execution within the authorized laboratory environment.

# Scope

| Item | Value |
|------|-------|
| Target | Hack The Box – Cap |
| Platform | Hack The Box |
| Operating System | Linux |
| Assessment Type | Black-Box Penetration Test |
| Difficulty | Easy |
| Tester | Cornelius Donkor |
| Assessment Date | August 2026 |

# Assessment Objectives

The objectives of this engagement were to:

- Identify exposed network services.
- Enumerate publicly accessible resources.
- Discover and validate security weaknesses.
- Demonstrate practical exploitation of confirmed vulnerabilities.
- Obtain user-level access.
- Escalate privileges to the root account.
- Document evidence, findings, and remediation recommendations.

  # Rules of Engagement

This assessment was conducted within the Hack The Box laboratory environment.

Activities were limited to the assigned target machine and performed under the platform's authorization for educational purposes.

No attacks were directed toward systems outside the defined scope, and no automated exploitation tools were used without first validating findings through manual enumeration and analysis.

# Methodology

The assessment followed a structured penetration testing methodology consisting of six sequential phases:

1. Reconnaissance
2. Enumeration
3. Vulnerability Analysis
4. Exploitation
5. Privilege Escalation
6. Reporting

Information collected during each phase informed the activities performed in the next phase. Every identified weakness was manually validated before exploitation, ensuring that conclusions were based on observed evidence rather than assumptions or automated tool output alone.


    

> **Disclaimer**
>
> This report documents a penetration test performed against the Hack The Box "Cap" machine within an authorized laboratory environment. All activities were conducted for educational purposes on infrastructure designed for security training. No testing was performed against unauthorized systems.

## Table of Contents

1. Executive Summary
2. Scope
3. Assessment Objectives
4. Rules of Engagement
5. Methodology
6. Attack Surface Overview
7. Reconnaissance
8. Enumeration
9. Vulnerability Analysis
10. Exploitation
11. Privilege Escalation
12. Security Findings
13. Risk Assessment
14. MITRE ATT&CK Mapping
15. Remediation Recommendations
16. Lessons Learned
17. References

     # Attack Surface Overview

## Objective

The objective of this phase was to identify the externally exposed attack surface of the target system and determine which network services and applications were available for further assessment.

Understanding the attack surface provides the foundation for all subsequent enumeration and vulnerability analysis activities by identifying potential entry points and prioritizing areas of interest.

## Methodology

The assessment began with active network reconnaissance using Nmap to identify open TCP ports, detect running services, and determine application versions where possible.

Service detection was performed to establish the technologies exposed by the target and to guide subsequent enumeration efforts.

Each discovered service was analyzed to determine its potential role within the attack surface before moving to deeper application-specific testing.

## Target Information

| Property | Value |
|----------|-------|
| Target Platform | Hack The Box |
| Machine | Cap |
| Target IP | 10.129.79.148 |
| Operating System | Linux |

## Initial Network Reconnaissance

The first stage of the assessment focused on identifying reachable TCP services.

Nmap was selected because it provides reliable host discovery, service detection, version identification, and scripting capabilities that support structured penetration testing workflows.

The following command was executed:
```bash
nmap -Pn -sC -sV -oA cap_initial 10.129.79.148
```

| Option | Purpose                                          |
| ------ | ------------------------------------------------ |
| `-Pn`  | Skip host discovery and treat the host as online |
| `-sC`  | Run the default NSE script set                   |
| `-sV`  | Detect service versions                          |
| `-oA`  | Save output in normal, XML, and grepable formats |

**Figure 1.** Initial Nmap service discovery scan.
![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/htblab.png?raw=true)

| Port | Service | Version       | Purpose         | Priority |
| ---: | ------- | ------------- | --------------- | -------- |
|   21 | FTP     | vsftpd 3.0.3  | File Transfer   | Medium   |
|   22 | SSH     | OpenSSH 8.2p1 | Remote Access   | High     |
|   80 | HTTP    | Gunicorn      | Web Application | Critical |


External Attacker
        │
        ▼
   10.129.79.148
        │
 ┌──────┼───────────┐
 │      │           │
FTP    SSH       HTTP
21      22         80



## Analysis

Three externally accessible services were identified during reconnaissance.

While FTP and SSH represent common administrative services, the HTTP service exposed a web application that presented the broadest attack surface due to its interactive functionality and multiple accessible endpoints.

Based on the initial reconnaissance results, the web application was selected as the primary focus for further enumeration because web applications commonly expose authentication workflows, administrative functionality, and object references that may be susceptible to access control weaknesses.

FTP and SSH were retained as secondary attack vectors pending the discovery of valid credentials or additional supporting evidence.

### Key Findings

- Three externally accessible TCP services were identified.
- The HTTP service presented the largest attack surface.
- No anonymous FTP access was permitted.
- SSH required authentication.
- The web application was selected as the primary enumeration target.

  > **My Observation**
>
> Although FTP and SSH were exposed, both required authentication before any meaningful interaction was possible. The HTTP service, by contrast, exposed multiple application endpoints and represented the most promising avenue for further investigation.

## Reconnaissance Summary

Enumeration of the exposed services revealed that:

- FTP required valid credentials.
- SSH required authentication and exposed no immediately actionable weaknesses.
- The HTTP service exposed a feature-rich administrative dashboard with multiple accessible endpoints that warranted deeper investigation.

Based on these findings, subsequent assessment efforts focused on the web application due to its significantly larger attack surface and exposed functionality.

# HTTP Enumeration

## Objective

The objective of this phase was to enumerate the web application to identify exposed functionality, technologies, administrative interfaces, and potential attack vectors that could lead to unauthorized access.

## Initial Web Fingerprinting

Before interacting with the application, passive fingerprinting techniques were used to identify the technologies powering the web server and to gather information that could guide subsequent enumeration.

Fingerprinting helps determine the underlying web server, client-side frameworks, and other technologies that may influence both the application's functionality and its potential attack surface.

```bash
whatweb http://10.129.79.148
```

![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/htblab1.png?raw=true)

The WhatWeb scan identified several technologies and frameworks associated with the application, including:

- Gunicorn HTTP Server
- Bootstrap
- jQuery 2.2.4
- Modernizr 2.8.3
- HTML5

The application returned an HTTP 200 response and presented a page titled **Security Dashboard**. :contentReference[oaicite:2]{index=2}

The presence of Gunicorn indicates that the application is likely implemented using a Python-based web framework.

Client-side libraries including Bootstrap and jQuery suggest a dynamic administrative interface, while the Security Dashboard title indicates that the application provides operational monitoring functionality rather than serving static content.

These findings justified performing deeper application enumeration.

```bash
curl -I http://10.129.79.148
```
![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/htblab2.png?raw=true)

![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/htblab3.png?raw=true)

While I was reviewing the HTML source revealed multiple administrative functions exposed through the application's navigation menu.

The available functionality included:

- Dashboard
- Security Snapshot
- IP Configuration
- Network Status

Among these, the Security Snapshot functionality appeared particularly interesting because it suggested that the application generated packet captures for later review.

Features responsible for collecting, storing, or displaying sensitive network data frequently warrant closer inspection during web application assessments because they may expose information through insecure access controls or object references.

> **My Observation**
>
> At this stage of the assessment, the Security Snapshot feature appeared to represent the largest potential attack surface because it involved creating and retrieving network capture files. Functionality that references stored objects often becomes a candidate for authorization testing during subsequent vulnerability analysis.

curl http://10.129.79.148/ip
![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/htblab4.png?raw=true)

![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/htblab5.png?raw=true)

The IP Configuration page displayed the network configuration of the underlying Linux host directly within the web application.

Information disclosed included:

- Interface names
- IPv4 addressing
- IPv6 addresses
- MAC address
- Packet statistics
- Interface status

Although this functionality does not immediately provide code execution or authentication bypass, it exposes detailed host networking information that would normally be unavailable to unauthenticated users.

This information can assist attackers during later stages of an assessment by revealing:

- Internal addressing
- Network interfaces
- Host configuration
- Active network statistics

The exposure represents unnecessary information disclosure that could assist reconnaissance activities.

curl http://10.129.79.148/netstat

![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/htblab6.png?raw=true)
![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/htblab7.png?raw=true)

The Network Status page returned the output of the Linux netstat utility directly through the web interface.

The page disclosed:

- Listening services
- Active network connections
- Established sessions
- Local ports
- UNIX domain sockets

Publishing live netstat output significantly increases the information available to an attacker.

While these services had already been identified through external reconnaissance, the endpoint confirmed their operational state from the host itself and exposed additional runtime information unavailable through ordinary port scanning.

Such functionality should normally be restricted to authenticated administrators because it provides valuable intelligence regarding internal system activity.

![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/htblab8.png?raw=true)

# My Observation
Requesting the Security Snapshot endpoint resulted in an HTTP redirect rather than immediately displaying content.

The application redirected the request to:

/data/1

# Analysis

The redirect exposed an object identifier within the application's URL structure.

Applications that reference stored resources using sequential numeric identifiers frequently warrant authorization testing to determine whether access controls are correctly enforced.

At this stage no conclusions were made regarding vulnerability; however, the predictable resource identifier was identified as a candidate for further assessment during the Vulnerability Analysis phase.

## HTTP Enumeration Summary

Manual inspection of the exposed application identified multiple administrative endpoints that disclosed operational information.

The application exposed:

- Dashboard
- Security Snapshot
- IP Configuration
- Network Status

While the IP Configuration and Network Status pages revealed useful system information, the Security Snapshot feature appeared to reference stored resources using a predictable numeric identifier.

This observation established the basis for focused authorization testing during the Vulnerability Analysis phase.

