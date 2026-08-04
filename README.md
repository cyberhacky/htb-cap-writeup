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
