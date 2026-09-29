# Enterprise Cybersecurity Assessment & Penetration Testing

A simulated cybersecurity assessment of an isolated enterprise-style Linux environment, covering reconnaissance, service enumeration, manual vulnerability validation, risk analysis, remediation planning, and security reporting.

The engagement was conducted for the fictional organization **Asteria Solutions Sdn. Bhd.** within a controlled VMware lab environment. Testing focused on identifying exposed services, validating meaningful security weaknesses, assessing their potential impact, and documenting findings using a consulting-style assessment process.

---

## Project Overview

The objective of this project was to simulate a structured cybersecurity assessment rather than perform isolated vulnerability-scanning exercises.

The assessment followed the workflow:

`Scoping → Reconnaissance → Service Enumeration → Vulnerability Analysis → Manual Validation → Risk Assessment → Remediation → Reporting`

Testing identified **three confirmed security findings** and **two supporting security observations**.

Validation was intentionally limited to the minimum activity required to demonstrate each issue. Destructive testing, persistence, unnecessary privilege escalation, and denial-of-service activity were excluded from the engagement.

---

## Assessment Environment

| Asset | IP Address | Role |
|---|---|---|
| Kali Linux | `192.168.247.128` | Security Assessment Workstation |
| Metasploitable 2 | `192.168.247.129` | Simulated Enterprise Server |

The systems were connected through an isolated **VMware host-only network**, preventing deliberately vulnerable services from being exposed to external networks.

The target server hosted multiple network and application services including:

- NFS
- SMB / Samba
- FTP
- SSH
- Telnet
- Apache HTTP
- Apache Tomcat
- MySQL
- PostgreSQL
- RPC services
- VNC
- Additional legacy services

Initial reconnaissance identified **23 open TCP ports**. Services were then prioritized for deeper investigation based on exposure, authentication requirements, permissions, and potential security impact.

---

## Key Findings

| ID | Finding | Severity | Status |
|---|---|---:|---|
| [F-001](findings/F001-unrestricted-nfs-root-filesystem-export.md) | Unrestricted NFS Root Filesystem Export with Root Privilege Preservation | **Critical** | Confirmed |
| [F-002](findings/F002-anonymous-smb-read-write-access.md) | Anonymous Read/Write Access to SMB Temporary Share | **Medium** | Confirmed |
| [F-003](findings/F003-tomcat-manager-weak-credentials.md) | Weak Credentials Permit Unauthorized Access to Apache Tomcat Manager | **High** | Confirmed |

### F-001 — Unrestricted NFS Root Filesystem Export

The target exported its entire root filesystem through NFS using:

```text
/ *(rw,sync,no_root_squash,no_subtree_check)
```

Testing confirmed that an unauthenticated client could mount the exported filesystem and access operating-system directories.

A controlled write test also confirmed that a privileged remote client could create a **root-owned file on the target filesystem**, demonstrating the impact of the `no_root_squash` configuration.

No sensitive system files were modified.

---

### F-002 — Anonymous SMB Read/Write Access

The SMB service exposed a `tmp` share that could be accessed without authentication.

Manual testing confirmed that an anonymous user could:

- Connect to the share
- Enumerate directory contents
- Upload a controlled test file
- Confirm the file existed remotely
- Delete the test artifact following validation

No code execution or privilege escalation through the share was claimed or attempted.

---

### F-003 — Weak Tomcat Manager Credentials

The Apache Tomcat Manager interface was exposed on TCP/8180.

An unauthenticated request returned:

```text
HTTP 401 Unauthorized
```

Authentication using the weak credential pair:

```text
tomcat:tomcat
```

returned:

```text
HTTP 200 OK
```

Browser validation confirmed successful access to the **Tomcat Web Application Manager** administrative interface.

Testing stopped at confirmed administrative access. No WAR deployment, reverse shell, operating-system command execution, or persistence activity was performed.

---

## Security Observations

In addition to the primary findings, two lower-priority security conditions were documented.

| ID | Observation | Priority | Status |
|---|---|---:|---|
| [O-001](observations/O001-anonymous-ftp-access.md) | Anonymous FTP Access with Plaintext Transport | Low | Confirmed |
| [O-002](observations/O002-legacy-smbv1-protocol-enabled.md) | Legacy SMBv1 Protocol Enabled | Medium | Confirmed |

### Anonymous FTP

Anonymous FTP authentication was successfully validated.

The exposed FTP directory contained no meaningful files, and a controlled upload attempt was rejected by the server. The issue was therefore retained as a lower-priority observation rather than elevated to a primary vulnerability finding.

### SMBv1

Protocol enumeration confirmed support for the legacy SMBv1 `NT LM 0.12` dialect.

No SMBv1-specific vulnerability or CVE was claimed because exploitability was not independently validated.

---

## Attack Surface Prioritization

Following reconnaissance, deeper testing focused primarily on:

| Service | Reason for Prioritization | Result |
|---|---|---|
| NFS | Filesystem exposure and potential privilege implications | Critical finding |
| SMB | Anonymous share access and permissions | Medium finding + SMBv1 observation |
| Apache Tomcat | Exposed administrative interface | High finding |
| FTP | Anonymous authentication and plaintext communication | Low observation |

This approach prioritized meaningful validation rather than attempting to exploit every exposed service.

---

## Validation Approach

Potential vulnerabilities were not treated as confirmed findings based solely on:

- Open ports
- Service-version strings
- Automated scanner output
- Known vulnerabilities associated with a product version

Where appropriate, findings were manually validated to determine whether the identified condition was actually present and what level of access or impact could be demonstrated.

Validation followed a **minimum-impact approach**.

Examples include:

- Creating and removing a harmless file during NFS write validation
- Uploading and deleting a controlled file through the anonymous SMB share
- Attempting a harmless FTP upload to determine whether anonymous write access was permitted
- Comparing unauthenticated and authenticated HTTP responses for Tomcat Manager
- Stopping Tomcat testing once administrative access was demonstrated rather than pursuing unnecessary code execution

---

## Attack Path Summary

The assessment demonstrated several independent paths through which an attacker with network access to the environment could interact with insecurely configured services.

```text
Network-accessible attacker
        |
        +-- NFS :2049
        |      |
        |      +-- Root filesystem exported
        |      +-- Read/write enabled
        |      +-- no_root_squash enabled
        |      |
        |      +--> Root-owned remote filesystem write CONFIRMED
        |             |
        |             +--> Potential host compromise
        |
        +-- Tomcat :8180
        |      |
        |      +-- Manager interface exposed
        |      +-- Weak credentials accepted
        |      |
        |      +--> Administrative Manager access CONFIRMED
        |             |
        |             +--> Potential application/server compromise
        |
        +-- SMB :445
               |
               +-- Anonymous tmp share access
               +-- Read/write permissions
               |
               +--> Unauthorized file placement CONFIRMED
```

Potential downstream impact is distinguished from actions directly validated during testing.

See:

- [`diagrams/network-architecture.md`](diagrams/network-architecture.md)
- [`diagrams/attack-path.md`](diagrams/attack-path.md)

---

## Tools Used

The assessment used a combination of standard security and operating-system utilities, including:

- **Nmap** — host discovery, port scanning, service/version detection and protocol enumeration
- **smbclient** — SMB share enumeration and access validation
- **NFS utilities** — export discovery, filesystem mounting and permission validation
- **FTP client** — anonymous authentication and permission validation
- **curl** — HTTP authentication and response validation
- **Web browser** — manual web and administrative-interface validation
- **Kali Linux** — security assessment workstation
- **VMware Workstation** — isolated virtualization environment

Tools were used to support investigation and validation. Automated output alone was not considered proof of vulnerability.

---

## Repository Structure

```text
enterprise-cybersecurity-assessment-simulation/
|
├── README.md
|
├── documentation/
│   ├── client-scenario.md
│   ├── scope-and-rules-of-engagement.md
│   ├── environment.md
│   ├── methodology.md
│   ├── risk-methodology.md
│   └── reconnaissance.md
|
├── findings/
│   ├── F001-unrestricted-nfs-root-filesystem-export.md
│   ├── F002-anonymous-smb-read-write-access.md
│   └── F003-tomcat-manager-weak-credentials.md
|
├── observations/
│   ├── O001-anonymous-ftp-access.md
│   └── O002-legacy-smbv1-protocol-enabled.md
|
├── evidence/
│   ├── reconnaissance/
│   ├── enumeration/
│   └── validation/
|
├── diagrams/
│   ├── network-architecture.md
│   └── attack-path.md
|
└── report/
    ├── enterprise-cybersecurity-assessment-report.md
    └── enterprise-cybersecurity-assessment-report.pdf
```

---

## Documentation

Supporting engagement documentation includes:

- [Client Scenario](documentation/client-scenario.md)
- [Scope & Rules of Engagement](documentation/scope-and-rules-of-engagement.md)
- [Assessment Environment](documentation/environment.md)
- [Assessment Methodology](documentation/methodology.md)
- [Risk Methodology](documentation/risk-methodology.md)
- [Reconnaissance & Attack Surface Analysis](documentation/reconnaissance.md)

The completed assessment report is available under:

- [Final Cybersecurity Assessment Report](report/enterprise-cybersecurity-assessment-report.md)

---

## Skills Demonstrated

This project demonstrates practical experience in:

**Security Assessment**
- Assessment scoping
- Rules of Engagement
- Attack-surface analysis
- Vulnerability assessment
- Manual vulnerability validation
- Risk analysis
- Remediation planning

**Technical Security**
- Network reconnaissance
- Service enumeration
- NFS security assessment
- SMB/Samba assessment
- Web-service enumeration
- Authentication testing
- Linux security analysis
- Controlled proof-of-impact testing

**Security Consulting**
- Evidence collection
- Technical finding documentation
- Severity classification
- Business-impact analysis
- Remediation recommendations
- Attack-path analysis
- Executive and technical reporting

---

## Assessment Principles

Several principles were followed throughout the project:

1. **Open services are not automatically vulnerabilities.**
2. **Version information alone does not confirm exploitability.**
3. **Automated scanner results require analyst validation.**
4. **Testing should demonstrate sufficient impact without causing unnecessary damage.**
5. **Confirmed evidence must be separated from potential impact.**
6. **Remediation should address the underlying weakness rather than only the test performed.**

---

## Disclaimer

This project was conducted entirely within a **controlled, isolated and intentionally vulnerable lab environment** for cybersecurity education and professional portfolio development.

No production systems, real client infrastructure, public targets or unauthorized third-party systems were tested.

**Asteria Solutions Sdn. Bhd. is a fictional organization created solely for this simulated assessment.**
