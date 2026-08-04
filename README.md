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

## Analysis

Three externally accessible services were identified during reconnaissance.

While FTP and SSH represent common administrative services, the HTTP service exposed a web application that presented the broadest attack surface due to its interactive functionality and multiple accessible endpoints.

Based on the initial reconnaissance results, the web application was selected as the primary focus for further enumeration because web applications commonly expose authentication workflows, administrative functionality, and object references that may be susceptible to access control weaknesses.

FTP and SSH were retained as secondary attack vectors pending the discovery of valid credentials or additional supporting evidence.
