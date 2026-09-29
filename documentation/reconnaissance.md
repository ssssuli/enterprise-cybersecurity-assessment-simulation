# Reconnaissance & Attack Surface Analysis

## 1. Overview

Reconnaissance was performed against the authorized target:

```text
192.168.247.129
```

The assessment workstation was:

```text
Kali Linux — 192.168.247.128
```

The purpose of this phase was to identify the target's exposed network attack surface, determine which services warranted deeper investigation, and establish an evidence-based path from initial discovery to manual vulnerability validation.

The assessment followed the process:

```text
Host Discovery
      ↓
Port Discovery
      ↓
Service Identification
      ↓
Security Relevance Analysis
      ↓
Targeted Enumeration
      ↓
Potential Weakness
      ↓
Manual Validation
      ↓
Risk Assessment
      ↓
Finding / Observation
```

An exposed service was not automatically considered a vulnerability.

---

# 2. Host Discovery

Initial host discovery confirmed that the target system was reachable from the assessment workstation within the isolated VMware host-only network.

Target:

```text
192.168.247.129
```

Supporting evidence is retained at:

```text
evidence/reconnaissance/host-discovery.txt
```

Network configuration evidence is also retained within:

```text
evidence/reconnaissance/
```

---

# 3. Initial TCP Port Discovery

An initial Nmap scan was performed against the target:

```bash
nmap 192.168.247.129
```

The scan identified **23 open TCP ports**.

| Category | Ports | Observed Services |
|---|---|---|
| Remote Access | 21, 22, 23, 512, 513, 514 | FTP, SSH, Telnet and legacy Unix remote services |
| Web / Application | 80, 8009, 8180 | HTTP, AJP and Apache Tomcat |
| File Sharing | 139, 445, 2049 | SMB / Samba and NFS |
| Databases | 3306, 5432 | MySQL and PostgreSQL |
| Remote Display | 5900, 6000 | VNC and X11 |
| Infrastructure | 25, 53, 111 | SMTP, DNS and RPC |
| Additional Services | 1099, 1524, 2121, 6667 | Java RMI, legacy service, additional FTP and IRC |

The complete initial scan output is retained at:

```text
evidence/reconnaissance/initial-port-scan.txt
```

---

# 4. Initial Attack Surface Assessment

The target exposed a broad range of network-accessible services from the assessment network.

Several characteristics increased the security relevance of the discovered attack surface:

- Multiple remote-access services were enabled.
- File-sharing protocols were directly accessible.
- Administrative and web application services were exposed.
- Multiple legacy protocols were present.
- Database services were reachable.
- Several services provided identifiable product and version information.

The number of exposed services alone did not establish that the target was vulnerable.

Further enumeration was required to determine:

- What software was actually running
- Whether authentication was required
- Whether anonymous access was available
- What permissions were granted
- Whether administrative interfaces were exposed
- Whether an identified condition could be manually reproduced

---

# 5. Service Version Enumeration

Service/version enumeration was performed against the identified ports using:

```bash
nmap -sV -p 21,22,23,25,53,80,111,139,445,512,513,514,1099,1524,2049,2121,3306,5432,5900,6000,6667,8009,8180 192.168.247.129
```

The scan identified multiple network services and product versions.

Selected results included:

| Port | Service | Identified Technology |
|---|---|---|
| 21 | FTP | vsFTPd 2.3.4 |
| 22 | SSH | OpenSSH 4.7p1 |
| 23 | Telnet | Linux telnetd |
| 25 | SMTP | Postfix |
| 53 | DNS | ISC BIND 9.4.2 |
| 80 | HTTP | Apache HTTP Server 2.2.8 |
| 139 / 445 | SMB | Samba |
| 2049 | NFS | NFS |
| 2121 | FTP | ProFTPD 1.3.1 |
| 3306 | MySQL | MySQL 5.0.51a |
| 5432 | PostgreSQL | PostgreSQL 8.3.x |
| 5900 | VNC | VNC protocol 3.3 |
| 6667 | IRC | UnrealIRCd |
| 8009 | AJP | Apache JServ Protocol 1.3 |
| 8180 | HTTP / Application | Apache Tomcat / Coyote |

The full service/version evidence is retained at:

```text
evidence/reconnaissance/service-version-scan.txt
```

---

# 6. Interpretation of Version Information

Service-version information was treated as **reconnaissance evidence**, not confirmation of vulnerability.

For example:

```text
Product version identified
        ≠
Confirmed exploitable vulnerability
```

A version string may provide useful information for further investigation, but exploitability may depend on:

- Exact software build
- Patch state
- Operating-system packaging
- Configuration
- Authentication requirements
- Network controls
- Whether the suspected weakness is actually reachable

The assessment therefore avoided reporting vulnerabilities solely because an old or potentially vulnerable product version was identified.

This approach reduced the likelihood of false-positive or overstated findings.

---

# 7. Service Prioritization

Rather than attempting to exploit all 23 exposed TCP services, deeper testing was prioritized according to:

1. Accessibility from the assessment network
2. Potential access to sensitive resources
3. Authentication requirements
4. Observed permissions
5. Administrative functionality
6. Potential impact on confidentiality, integrity, or availability
7. Ability to validate the condition safely

Four primary areas were selected:

- NFS
- SMB / Samba
- Web / Apache Tomcat
- FTP

---

# 8. NFS Prioritization

NFS on TCP/2049 was prioritized because file-sharing services can expose significant server-side filesystem resources.

The primary questions were:

- What directories were exported?
- Which systems were permitted to access the export?
- Was authentication required?
- Was the export read-only or writable?
- Were remote root privileges restricted?

Enumeration identified:

```text
/ *
```

Further validation identified the server-side configuration:

```text
/ *(rw,sync,no_root_squash,no_subtree_check)
```

Manual testing ultimately confirmed remote creation of a root-owned file.

### Assessment Result

**F-001 — Unrestricted NFS Root Filesystem Export with Root Privilege Preservation**

**Severity: Critical**

Supporting evidence:

```text
evidence/enumeration/nfs-exports.txt
evidence/validation/nfs-root-listing.txt
evidence/validation/nfs-etc-listing.txt
evidence/validation/nfs-home-listing.txt
evidence/validation/nfs-export-config.txt
evidence/validation/nfs-write-validation.txt
```

See:

[`../findings/F001-unrestricted-nfs-root-filesystem-export.md`](../findings/F001-unrestricted-nfs-root-filesystem-export.md)

---

# 9. SMB / Samba Prioritization

SMB services on TCP/139 and TCP/445 were prioritized because unauthenticated or excessively permissive network shares can provide direct access to server-side resources.

The assessment investigated:

- Whether shares could be enumerated anonymously
- Which shares were accessible
- Whether authentication was required
- Whether anonymous users had read access
- Whether anonymous users had write access
- Which SMB protocol versions were accepted

Anonymous enumeration identified several shares, including:

```text
print$
tmp
opt
IPC$
ADMIN$
```

Further testing confirmed anonymous access to the `tmp` share.

A controlled test file was successfully uploaded, confirmed on the remote share, and removed after validation.

Protocol enumeration also confirmed SMBv1 support.

### Assessment Results

**F-002 — Anonymous Read/Write Access to SMB Temporary Share**

**Severity: Medium**

and:

**O-002 — Legacy SMBv1 Protocol Enabled**

**Priority: Medium**

Supporting evidence:

```text
evidence/enumeration/smbclient-share-list.txt
evidence/enumeration/smb-enumeration.txt
evidence/validation/smb-anonymous-access.txt
evidence/validation/smb-write-validation-session.txt
```

See:

[`../findings/F002-anonymous-smb-read-write-access.md`](../findings/F002-anonymous-smb-read-write-access.md)

[`../observations/O002-legacy-smbv1-protocol-enabled.md`](../observations/O002-legacy-smbv1-protocol-enabled.md)

---

# 10. Web and Apache Tomcat Prioritization

Web enumeration was performed against TCP/80 and TCP/8180.

Enumeration identified:

- Apache HTTP Server on TCP/80
- Apache Tomcat on TCP/8180
- An exposed Tomcat Manager administrative interface

The Manager interface was prioritized because compromise of administrative application functionality could provide substantially greater impact than normal web-content exposure.

Testing first established the expected unauthenticated behavior.

An unauthenticated request returned:

```text
HTTP 401 Unauthorized
```

Authentication testing then demonstrated that the weak credential pair:

```text
tomcat:tomcat
```

was accepted.

The authenticated response returned:

```text
HTTP 200 OK
```

Browser validation confirmed access to the Tomcat Web Application Manager.

Testing stopped at confirmed administrative access.

No application deployment, reverse shell, operating-system command execution, or persistence was performed.

### Assessment Result

**F-003 — Weak Credentials Permit Unauthorized Access to Apache Tomcat Manager**

**Severity: High**

Supporting evidence:

```text
evidence/enumeration/web-enumeration.txt
evidence/enumeration/tomcat-manager-unauthenticated.txt
evidence/validation/tomcat-manager-authenticated.txt
evidence/validation/tomcat-manager-authenticated-access.png
```

See:

[`../findings/F003-tomcat-manager-weak-credentials.md`](../findings/F003-tomcat-manager-weak-credentials.md)

---

# 11. FTP Prioritization

FTP on TCP/21 was investigated after enumeration indicated that anonymous authentication was permitted.

Manual testing confirmed:

```text
230 Login successful.
```

The anonymous account could access its FTP root directory.

However:

- No meaningful files were exposed.
- No useful subdirectories were identified.
- A controlled file-upload attempt was rejected.

The upload attempt returned:

```text
553 Could not create file.
```

Because anonymous authentication was confirmed but meaningful data exposure or modification was not demonstrated, the condition was not elevated to a primary vulnerability finding.

### Assessment Result

**O-001 — Anonymous FTP Access with Plaintext Transport**

**Priority: Low**

Supporting evidence:

```text
evidence/enumeration/ftp-enumeration.txt
evidence/validation/ftp-anonymous-access.txt
evidence/validation/ftp-write-validation.txt
```

See:

[`../observations/O001-anonymous-ftp-access.md`](../observations/O001-anonymous-ftp-access.md)

---

# 12. Services Not Selected for Deeper Validation

Several additional services were identified during reconnaissance but were not selected for full exploitation.

These included services such as:

- SSH
- Telnet
- SMTP
- DNS
- Database services
- VNC
- Java RMI
- IRC
- AJP
- Additional legacy remote-access services

Their presence contributed to the overall attack-surface assessment, but deeper testing was not necessary to satisfy the objectives of this portfolio engagement.

This was a deliberate prioritization decision.

The objective was not to demonstrate every possible Metasploitable 2 vulnerability.

The objective was to demonstrate the ability to:

```text
Discover
   ↓
Prioritize
   ↓
Investigate
   ↓
Validate
   ↓
Assess Risk
   ↓
Document
   ↓
Recommend Remediation
```

Once three meaningful confirmed findings and supporting observations had been established, further exploitation would have provided diminishing portfolio value.

---

# 13. Reconnaissance-to-Finding Mapping

| Assessment Area | Initial Observation | Validation Outcome | Final Classification |
|---|---|---|---|
| NFS | TCP/2049 accessible | Root filesystem exposed with remote root-owned write | F-001 — Critical |
| SMB | TCP/139 and 445 accessible | Anonymous `tmp` share read/write confirmed | F-002 — Medium |
| SMB Protocol | SMBv1 accepted | Legacy protocol confirmed; no CVE exploitation attempted | O-002 — Medium |
| Tomcat | TCP/8180 administrative service exposed | Weak credentials provided Manager access | F-003 — High |
| FTP | Anonymous authentication available | Login confirmed; no meaningful content; write denied | O-001 — Low |

---

# 14. Assessment Outcome

Reconnaissance successfully reduced a broad attack surface of **23 exposed TCP services** into a smaller set of prioritized investigation areas.

The resulting assessment produced:

- **1 Critical confirmed finding**
- **1 High confirmed finding**
- **1 Medium confirmed finding**
- **1 Medium security observation**
- **1 Low security observation**

The most significant issue was the NFS configuration because manual testing demonstrated privileged filesystem modification through an unauthenticated network-accessible export.

The Tomcat Manager authentication weakness provided confirmed administrative application access.

The SMB configuration provided confirmed anonymous remote file modification, but no evidence established privileged execution or host compromise through the writable share.

FTP and SMBv1 conditions were retained as observations because the available evidence did not justify treating them as equivalent to the primary confirmed vulnerabilities.

---

# 15. Evidence Index

Reconnaissance evidence:

```text
evidence/reconnaissance/host-discovery.txt
evidence/reconnaissance/initial-port-scan.txt
evidence/reconnaissance/service-version-scan.txt
```

Targeted enumeration evidence:

```text
evidence/enumeration/
```

Manual validation evidence:

```text
evidence/validation/
```

Risk-rating criteria are documented in:

[`risk-methodology.md`](risk-methodology.md)

Assessment scope and authorization boundaries are documented in:

[`scope-and-rules-of-engagement.md`](scope-and-rules-of-engagement.md)
