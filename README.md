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
![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab.png?raw=true)

| Port | Service | Version       | Purpose         | Priority |
| ---: | ------- | ------------- | --------------- | -------- |
|   21 | FTP     | vsftpd 3.0.3  | File Transfer   | Medium   |
|   22 | SSH     | OpenSSH 8.2p1 | Remote Access   | High     |
|   80 | HTTP    | Gunicorn      | Web Application | Critical |




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

![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab1.png?raw=true)

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
![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab2.png?raw=true)

![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab3.png?raw=true)

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
![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab4.png?raw=true)

![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab5.png?raw=true)

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

![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab6.png?raw=true)
![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab7.png?raw=true)

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

![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab8.png?raw=true)

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

# Vulnerability Analysis

## Objective

The objective of this phase was to validate whether the functionality identified during enumeration could be abused to access information or resources beyond the intended authorization boundaries.

Particular attention was given to the **Security Snapshot** feature after observing that captured data was referenced using sequential numeric identifiers.

curl http://10.129.79.148/data/1

![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab9.png?raw=true)
![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab10.png?raw=true)

The request successfully returned the contents of the first stored capture.

The response contained:

- Packet statistics
- IP packet count
- TCP packet count
- UDP packet count
- Download functionality

  curl -L http://10.129.79.148/capture

![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab11.png?raw=true)
![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab12.png?raw=true)

Following the redirect automatically caused the application to generate a new capture.

The resulting page referenced a different resource identifier.

Previously:

/download/1

After requesting another capture:

/download/2

# Analysis

The assessment established that the application stores captures using sequential numeric object identifiers.

Each invocation of the capture functionality generated a new identifier while exposing the corresponding download endpoint.

Predictable object identifiers frequently indicate resources that should be protected by server-side authorization checks.

At this stage, no authorization bypass had yet been confirmed. However, the identifier pattern justified further testing to determine whether previously generated captures belonging to other users could be accessed.

# Security Assessment

The observations indicated several characteristics commonly associated with object reference vulnerabilities:

- Predictable numeric identifiers
- Direct object references in URLs
- Download functionality tied to object IDs
- No visible evidence of user-specific identifiers within the URL

These characteristics warranted authorization testing against alternate object identifiers.

The next phase focused on determining whether modifying the object identifier would allow unauthorized access to packet capture files belonging to other users.

curl http://10.129.79.148/data/0

![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab13.png?raw=true)
![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab14.png?raw=true)

The application successfully returned packet capture statistics for object identifier 0.

The response included:

- Number of packets
- IP packet count
- TCP packet count
- Download button

Download endpoint:

/download/0

curl -I http://10.129.79.148/download/0

![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab16.png?raw=true)

The server responded with HTTP 200 OK.

Response headers confirmed that a packet capture file could be downloaded.

Content-Disposition:
attachment; filename=0.pcap

Content-Type:
application/vnd.tcpdump.pcap

## Finding 1 – Insecure Direct Object Reference (IDOR)

**Severity:** High

**OWASP Top 10:** A01:2021 – Broken Access Control

**CWE:** CWE-639 – Authorization Bypass Through User-Controlled Key

### Description

The Security Snapshot functionality stores packet capture files using sequential numeric identifiers.

Rather than using unpredictable object references or enforcing ownership validation, the application exposes capture data through URLs of the form:

/data/1

/download/0

This implementation allows users to request resources directly by modifying the numeric identifier.

### Objective

I have to determine whether packet capture objects belonging to other users could be accessed by modifying the numeric identifier within the application URLs.

### Testing Methodology

After identifying that packet captures were referenced using sequential identifiers, multiple object IDs were requested manually to evaluate whether server-side authorization controls restricted access to individual resources.

curl http://10.129.79.148/data/3

![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab15.png?raw=true)

The application redirected the request back to the homepage instead of returning packet capture data.

This behavior indicates that object identifier 3 did not exist rather than demonstrating an authorization failure.

### Analysis

Testing demonstrated that packet capture objects were directly accessible using predictable numeric identifiers.

Object identifier 0 returned valid capture statistics and exposed a downloadable packet capture file.

No authentication or authorization challenge was observed before the application disclosed access to the stored capture.

Because access to the object depended solely on knowledge of the identifier, the application failed to enforce object-level authorization.

### Security Impact

Successful exploitation allows an attacker to access packet capture files that should be restricted to their respective owners.

Packet captures may contain:

- Authentication credentials
- Session cookies
- HTTP requests
- Network metadata
- Sensitive application traffic

Disclosure of this information can facilitate additional attacks, including credential compromise and unauthorized system access.

### Root Cause

The application uses predictable sequential identifiers to reference stored objects without validating whether the requesting user is authorized to access the selected resource.

Authorization decisions are therefore based solely on user-supplied object identifiers rather than ownership validation.

### Evidence Summary

| Test          | Result                                      |
| ------------- | ------------------------------------------- |
| `/capture`    | Created a new packet capture                |
| `/data/1`     | Displayed capture statistics                |
| `/data/0`     | Displayed a different capture object        |
| `/download/0` | Returned HTTP 200 and a downloadable PCAP   |
| `/data/3`     | Redirected because the object did not exist |

### Conclusion

The assessment confirmed the presence of an Insecure Direct Object Reference (IDOR) vulnerability.

The application permitted direct access to packet capture resources using predictable object identifiers without enforcing appropriate authorization controls.

This weakness enabled unauthorized retrieval of packet capture files, ultimately exposing sensitive information that could be leveraged to obtain valid user credentials during the subsequent exploitation phase.

# Exploitation

## Objective

The objective of this phase was to determine whether the unauthorized packet capture identified during the Vulnerability Analysis phase contained information that could facilitate authenticated access to the target system.

Rather than attempting password guessing or brute-force attacks, the assessment focused on analyzing the exposed packet capture for sensitive information that had been unintentionally disclosed.

### Downloading the Packet Capture

Since the response headers confirmed that a packet capture file could be downloaded, I tried to download and analyze it.
```bash
wget http://10.129.79.148/download/0
mv 0 0.pcap
```
![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab17.png?raw=true)

The application successfully returned the packet capture file associated with object identifier `0`.

The file was downloaded without authentication or authorization checks and saved locally for forensic analysis.

### Packet Capture Analysis

```bash
tshark -r 0.pcap
```
![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab18.png?raw=true)
![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab19.png?raw=true)

Initial inspection of the packet capture revealed both HTTP and FTP traffic.

The FTP session was of particular interest because authentication exchanges are transmitted in plaintext when FTP is used without encryption.

### Extracting Credentials

```bash
strings 0.pcap
```
![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab20.png?raw=true)
![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab21.png?raw=true)

The packet capture contained plaintext FTP authentication credentials.

Because FTP does not encrypt authentication traffic, both the username and password were directly recoverable from the capture without requiring any cryptographic attacks or protocol manipulation.

The successful login response further confirmed that the credentials were valid at the time the traffic was captured.

USER nathan
PASS Buck3tH4TF0RM3!
230 Login successful.

### Protocol Validation

```bash
tshark -r 0.pcap -Y http
```
![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab22.png?raw=true)

Filtering the capture for HTTP traffic confirmed normal web requests associated with the application.

This helped separate routine web activity from the FTP authentication exchange, allowing the analysis to focus on the credentials exposed over FTP.

### Initial Access

```bash
ssh nathan@10.129.79.148
```
![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab23.png?raw=true)

The recovered credentials were successfully reused to authenticate to the SSH service.

Authentication succeeded without requiring any additional exploitation, providing an interactive shell as the user `nathan`.

# Local Enumeration

## Objective

Following successful authentication to the target via SSH, the objective of this phase was to identify opportunities for privilege escalation.

Enumeration focused on understanding the operating environment, user permissions, operating system configuration, and common Linux privilege escalation vectors before attempting any exploitation.

A structured enumeration methodology was used to minimize assumptions and ensure that all findings were supported by observable evidence.

### Initial Access Verification

# Purpose

Before performing system enumeration, access to the host was verified and the current execution context established.

```bash
whoami
id
hostname
pwd
```

![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab24.png?raw=true)

### Analysis

The compromised account was identified as `nathan`, a standard non-privileged Linux user.

The output confirmed that the session did not possess administrative privileges and therefore required privilege escalation to obtain full control of the system.

Establishing the current execution context is an essential first step because it determines the privileges available to the attacker and guides subsequent enumeration activities.

```bash
uname -a
cat /etc/os-release
```
![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab25.png?raw=true)

The target system was identified as:

- Ubuntu 20.04.2 LTS (Focal Fossa)
- Linux Kernel 5.4.0-80-generic
- x86_64 architecture

  ### Analysis

Identifying the operating system and kernel version is a critical enumeration step because many privilege escalation techniques depend on the underlying platform and software versions.

At this stage of the assessment, the operating system information was recorded to support subsequent privilege escalation research and to identify host-specific attack vectors.

### Sudo Privilege Assessment

```bash
sudo -i
```
![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab26.png?raw=true)

The user nathan is not in the sudoers file.

### Analysis

The assessment confirmed that the compromised user was not permitted to execute commands with sudo.

This eliminated straightforward administrative access through sudo misconfigurations and indicated that alternative privilege escalation vectors would need to be investigated.
> **My Observation**
>
> Verifying sudo permissions early in the enumeration process helps quickly determine whether administrative access is already available or whether additional investigation is required. Since the user lacked sudo privileges, subsequent efforts focused on identifying other privilege escalation mechanisms.

## Enumeration Summary

Initial host enumeration established the following:

- Authenticated access was obtained as the user `nathan`.
- The target was running Ubuntu 20.04.2 LTS.
- The Linux kernel version was 5.4.0-80-generic.
- The compromised account did not possess sudo privileges.

These findings indicated that privilege escalation would require identifying alternative mechanisms beyond standard administrative delegation.

# Privilege Escalation Enumeration

## Objective

The objective of this phase was to identify local privilege escalation opportunities available to the compromised user account.

Rather than relying on automated exploitation, common Linux privilege escalation vectors were systematically evaluated, including Linux capabilities, SUID binaries, scheduled tasks, and filesystem permissions.

This structured approach ensures that privilege escalation attempts are evidence-driven and minimizes the likelihood of overlooking viable attack paths.

## Linux Capability Enumeration

Linux capabilities provide fine-grained privilege assignments that allow executables to perform specific privileged operations without requiring full root privileges.

Misconfigured capabilities assigned to general-purpose executables can introduce significant security risks and therefore represent an important privilege escalation vector during Linux assessments.

```bash
getcap -r / 2>/dev/null
```
![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab27.png?raw=true)

| Binary                         | Capability                              | Assessment                         |
| ------------------------------ | --------------------------------------- | ---------------------------------- |
| `/usr/bin/python3.8`           | `cap_setuid`, `cap_net_bind_service`    | **Requires Investigation**         |
| `/usr/bin/ping`                | `cap_net_raw`                           | Expected                           |
| `/usr/bin/mtr-packet`          | `cap_net_raw`                           | Expected                           |
| `/usr/bin/traceroute6.iputils` | `cap_net_raw`                           | Expected                           |
| `gst-ptp-helper`               | `cap_net_bind_service`, `cap_net_admin` | Expected for service functionality |

### Analysis

The capability enumeration identified several binaries with elevated Linux capabilities.

Most observed capabilities were consistent with their intended functionality and did not present an immediate privilege escalation opportunity.

However, the Python 3.8 interpreter was assigned the `cap_setuid` capability, which is unusual for a general-purpose scripting interpreter.

Because `cap_setuid` permits a process to change its effective user ID, this finding was prioritized for further investigation as a potential privilege escalation vector.

> **My Observation**
>
> During Linux privilege escalation assessments, general-purpose interpreters such as Python, Perl, or Ruby should not normally possess capabilities that allow privilege modification. Their presence warrants immediate investigation because they may permit arbitrary code execution with elevated privileges.

```bash
crontab -l
ls -la /etc/cron*
```
![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab28.png?raw=true)

The assessment identified standard system cron directories and scheduled maintenance tasks.

No user-specific cron jobs or writable scheduled tasks were identified.

### Analysis

Scheduled task enumeration did not reveal any misconfigurations or user-controlled scripts that could be leveraged for privilege escalation.

The observed cron entries corresponded to standard Ubuntu maintenance activities.

### SUID Enumeration

```bash
find / -perm -4000 -type f 2>/dev/null
```
![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab29.png?raw=true)

| Binary      | Status   |
| ----------- | -------- |
| sudo        | Standard |
| su          | Standard |
| passwd      | Standard |
| mount       | Standard |
| umount      | Standard |
| pkexec      | Present  |
| ssh-keysign | Standard |

Enumeration identified multiple SUID binaries commonly present on Ubuntu systems.

No immediately exploitable custom binaries or application-specific SUID executables were observed during manual review.

Although `pkexec` was present, exploitation was not attempted because the assessment had already identified a higher-confidence privilege escalation vector through Linux capabilities.

### Capability Validation

```bash
getcap /usr/bin/python3.8
```
![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab30.png?raw=true)

/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip

### Analysis

The capability assignment confirmed that the Python 3.8 interpreter possessed the `cap_setuid` capability.

Unlike standard Linux capability assignments that are typically limited to specialized system utilities, granting `cap_setuid` to a general-purpose scripting interpreter significantly increases the risk of privilege escalation because arbitrary code executed through the interpreter can request a change to the effective user ID.

### Exploitation
```bash
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```
![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab31.png?raw=true)

The Python interpreter successfully invoked the `setuid(0)` system call and spawned a new shell with an effective user ID of 0.

Verification commands confirmed that the assessment obtained administrative privileges on the target host.

This demonstrated that the capability assignment was sufficient to bypass normal privilege separation and provide unrestricted access to the operating system.

### Root Verification

![Image Alt](https://github.com/cyberhacky/htb-cap-writeup/blob/main/screenshots/htblab34.png?raw=true)

Access to the `/root` directory confirmed complete administrative control of the target system.

At this stage, the assessment objectives had been achieved, demonstrating a successful privilege escalation from a standard user account to the root account.

## Security Impact

Successful exploitation of the capability assignment resulted in complete compromise of the target host.

An attacker obtaining a shell as a non-privileged user could:

- Execute commands with root privileges.
- Access all files and directories.
- Modify system configuration.
- Install persistent mechanisms.
- Create privileged accounts.
- Disable security controls.
- Exfiltrate sensitive data.

This finding represents a complete loss of confidentiality, integrity, and availability for the affected system.

## Root Cause

The Python interpreter was assigned the `cap_setuid` Linux capability.

General-purpose interpreters should not normally possess capabilities that allow modification of process privileges because they enable arbitrary user-supplied code to execute privileged system calls.

The misconfiguration violated the principle of least privilege and directly enabled local privilege escalation.

### MITRE ATT&CK Mapping

| Tactic               | Technique      | Description                            |
| -------------------- | -------------- | -------------------------------------- |
| Discovery            | T1082          | System Information Discovery           |
| Discovery            | T1046          | Network Service Discovery              |
| Credential Access    | T1552          | Unsecured Credentials                  |
| Initial Access       | Valid Accounts | SSH access using recovered credentials |
| Privilege Escalation | T1548          | Abuse Elevation Control Mechanism      |

### Findings Summary

| ID   | Finding                               | Severity | Status    |
| ---- | ------------------------------------- | -------- | --------- |
| F-01 | Broken Access Control (IDOR)          | High     | Confirmed |
| F-02 | Sensitive Information Disclosure      | High     | Confirmed |
| F-03 | Credential Exposure in FTP Traffic    | High     | Confirmed |
| F-04 | Excessive Linux Capability Assignment | Critical | Confirmed |

### Remediation Recommendations

## F-01 – Broken Access Control

Enforce object-level authorization checks on every request.
Avoid predictable sequential object identifiers.
Validate ownership before serving resources.

## F-02 – Sensitive Information Disclosure
Restrict access to packet capture files.
Encrypt sensitive network captures at rest.
Limit retention of diagnostic captures.

## F-03 – Credential Exposure
Replace FTP with encrypted alternatives such as SFTP or FTPS.
Disable plaintext authentication protocols.
Rotate exposed credentials immediately.

## F-04 – Linux Capability Misconfiguration
Remove the cap_setuid capability from /usr/bin/python3.8.
Assign capabilities only to binaries that require them for their intended function.
Periodically audit Linux capabilities as part of system hardening.

# Lessons Learned

This assessment demonstrated how multiple individually manageable weaknesses can be chained to achieve complete system compromise.

Key observations include:

- Thorough enumeration is essential for identifying meaningful attack paths.
- Predictable object references should always be tested for authorization enforcement.
- Diagnostic data such as packet captures may contain highly sensitive information and require appropriate access controls.
- Plaintext protocols such as FTP expose credentials to anyone with access to captured traffic.
- Local privilege escalation should be preceded by systematic enumeration rather than immediate exploitation attempts.
- Linux capabilities should be audited alongside SUID binaries during privilege escalation assessments because inappropriate capability assignments can provide direct paths to administrative access.
