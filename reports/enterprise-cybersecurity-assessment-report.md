# Enterprise Cybersecurity Assessment Report

**Client:** Asteria Solutions Sdn. Bhd.  
**Engagement Type:** Simulated Enterprise Cybersecurity Assessment  
**Assessment Environment:** Controlled VMware Lab  
**Assessment Period:** September 2026  
**Prepared by:** Sulaiman Khan  
**Classification:** Portfolio / Educational Simulation

---

# 1. Executive Summary

A cybersecurity assessment was conducted against a simulated enterprise server environment representing **Asteria Solutions Sdn. Bhd.**

The objective of the engagement was to identify exposed network services, investigate security weaknesses, manually validate significant issues, assess their potential impact, and provide practical remediation recommendations.

Testing was performed from a Kali Linux security assessment workstation against a single intentionally vulnerable Linux server operating within an isolated VMware host-only network.

Initial reconnaissance identified **23 open TCP ports**, representing a broad network attack surface that included file-sharing services, web applications, remote administration services, databases, and several legacy protocols.

Rather than attempting to exploit every exposed service, testing prioritized areas where insecure access controls, excessive permissions, or administrative functionality could produce meaningful security impact.

The assessment identified:

- **3 confirmed vulnerability findings**
- **2 security observations**

The confirmed findings consisted of:

| ID | Finding | Severity |
|---|---|---:|
| F-001 | Unrestricted NFS Root Filesystem Export with Root Privilege Preservation | **Critical** |
| F-003 | Weak Credentials Permit Unauthorized Access to Apache Tomcat Manager | **High** |
| F-002 | Anonymous Read/Write Access to SMB Temporary Share | **Medium** |

Supporting security observations were:

| ID | Observation | Priority |
|---|---|---:|
| O-002 | Legacy SMBv1 Protocol Enabled | **Medium** |
| O-001 | Anonymous FTP Access with Plaintext Transport | **Low** |

The most significant weakness involved the NFS service.

The target exported its entire root filesystem with read/write permissions and `no_root_squash` enabled. Manual validation demonstrated that a remote privileged client could create a **root-owned file on the target filesystem**.

This confirmed a direct privileged filesystem modification capability and represents a potential path toward full system compromise if abused further.

A second significant weakness affected the Apache Tomcat Manager interface. Weak credentials successfully authenticated to the administrative console, providing confirmed unauthorized administrative application access.

The SMB service also permitted anonymous access to a writable temporary share. Testing confirmed that an unauthenticated user could remotely create and remove files within the share.

Testing followed a minimum-impact approach. Once sufficient evidence was obtained to validate each vulnerability, unnecessary post-compromise activity was deliberately avoided.

The primary remediation priorities are therefore:

1. Remove the unrestricted NFS root filesystem export and restore appropriate root-squashing controls.
2. Replace weak Tomcat administrative credentials and restrict access to the Manager interface.
3. Remove anonymous SMB write access and enforce authenticated least-privilege permissions.
4. Disable SMBv1 where operationally unnecessary.
5. Disable unnecessary anonymous FTP access and migrate from plaintext FTP where possible.

---

# 2. Engagement Background

Asteria Solutions Sdn. Bhd. is a **fictional Malaysian small-to-medium enterprise** created solely to provide realistic business context for this simulated security assessment.

The organization is assumed to operate a small technology environment supporting:

- File-sharing services
- Web applications
- Application management
- Remote administration
- File-transfer services
- Database services
- Supporting network infrastructure

Management requested an assessment to identify weaknesses that could contribute to:

- Unauthorized access
- Unauthorized modification of server resources
- Exposure of sensitive configuration or information
- Application compromise
- Service disruption
- Potential compromise of the affected server

The assessment was designed to simulate the workflow and documentation expected during a structured vulnerability assessment or penetration-testing engagement.

---

# 3. Scope

## 3.1 In-Scope Target

The authorized assessment target was:

| Asset ID | Asset | IP Address | Function |
|---|---|---|---|
| TG-01 | Metasploitable 2 | `192.168.247.129` | Simulated Enterprise Server |

All network and application services hosted by TG-01 were eligible for reconnaissance and security assessment within the defined Rules of Engagement.

---

## 3.2 Assessment Workstation

Testing was conducted from:

| Asset ID | Asset | IP Address | Function |
|---|---|---|---|
| AS-01 | Kali Linux | `192.168.247.128` | Security Assessment Workstation |

The assessment workstation was not considered a target.

---

## 3.3 Network Boundary

Both systems operated within an isolated:

**VMware host-only network**

The deliberately vulnerable server was not intentionally exposed to public infrastructure.

Testing outside the controlled virtual environment was prohibited.

---

# 4. Rules of Engagement

The engagement permitted activities including:

- Host discovery
- TCP port scanning
- Service enumeration
- Service/version identification
- File-sharing enumeration
- Web-service enumeration
- Authentication testing
- Anonymous-access testing
- Permission validation
- Controlled proof-of-impact testing
- Risk assessment
- Attack-path analysis

The following activities were excluded:

- Denial-of-service testing
- Destructive testing
- Malware deployment
- Persistence
- Modification of critical authentication files
- Unnecessary post-compromise activity
- Social engineering
- Testing against production systems
- Testing against third-party systems
- Testing outside the authorized target

The guiding validation principle was:

> **Perform the minimum activity required to demonstrate the security weakness and its potential impact.**

---

# 5. Assessment Environment

The lab consisted of:

```text
Kali Linux
192.168.247.128
        |
        | VMware Host-Only Network
        |
Metasploitable 2
192.168.247.129
```

The Metasploitable 2 system represented the simulated enterprise server.

The environment was intentionally vulnerable and existed solely for authorized cybersecurity testing.

Detailed architecture is available in:

[`../diagrams/network-architecture.md`](../diagrams/network-architecture.md)

---

# 6. Assessment Methodology

The engagement followed the process:

```text
Scoping
   ↓
Reconnaissance
   ↓
Service Enumeration
   ↓
Vulnerability Analysis
   ↓
Manual Validation
   ↓
Risk Assessment
   ↓
Attack-Path Analysis
   ↓
Remediation
   ↓
Reporting
```

The methodology emphasized analyst validation rather than treating automated output as proof of vulnerability.

Specifically:

- An open port was not automatically considered a vulnerability.
- A software version was not automatically considered exploitable.
- Scanner warnings were reviewed before classification.
- Significant weaknesses were manually validated where appropriate.
- Demonstrated impact was separated from theoretical downstream impact.
- Testing stopped when sufficient evidence had been obtained.

The detailed methodology is documented in:

[`../documentation/methodology.md`](../documentation/methodology.md)

---

# 7. Reconnaissance Summary

Initial Nmap reconnaissance identified **23 open TCP ports**.

The exposed attack surface included:

| Category | Examples |
|---|---|
| Remote Access | SSH, Telnet, VNC, legacy Unix remote services |
| File Sharing | SMB / Samba, NFS |
| Web / Application | Apache HTTP, Apache Tomcat, AJP |
| Databases | MySQL, PostgreSQL |
| File Transfer | FTP, ProFTPD |
| Infrastructure | SMTP, DNS, RPC |
| Additional Services | Java RMI, IRC and legacy services |

The environment presented considerably more exposed services than were necessary for the purposes of the assessment.

Rather than attempting to exploit every identified service, deeper testing focused on four areas:

1. NFS
2. SMB / Samba
3. Apache Tomcat
4. FTP

These services were selected because of their accessibility, permissions, authentication characteristics, administrative functionality, and potential impact.

Detailed reconnaissance analysis is available in:

[`../documentation/reconnaissance.md`](../documentation/reconnaissance.md)

---

# 8. Risk Assessment Methodology

Confirmed vulnerabilities were rated using a qualitative:

```text
Likelihood × Impact
```

model.

Likelihood considered:

- Network accessibility
- Authentication requirements
- Exploitation complexity
- Required attacker access
- Reproducibility of the weakness

Impact considered:

- Confidentiality
- Integrity
- Availability
- Privileges obtained
- Administrative functionality
- Potential effect on the affected system

Severity was based primarily on **demonstrated evidence**, rather than the most severe hypothetical consequence imaginable.

This approach was particularly important for F-002, where anonymous write access was confirmed but code execution or privileged compromise was not.

The complete methodology is documented in:

[`../documentation/risk-methodology.md`](../documentation/risk-methodology.md)

---

# 9. Findings Summary

| ID | Finding | Likelihood | Impact | Severity |
|---|---|---:|---:|---:|
| F-001 | Unrestricted NFS Root Filesystem Export with Root Privilege Preservation | High | Critical | **Critical** |
| F-003 | Weak Credentials Permit Unauthorized Access to Apache Tomcat Manager | High | High | **High** |
| F-002 | Anonymous Read/Write Access to SMB Temporary Share | High | Medium | **Medium** |

---

# 10. F-001 — Unrestricted NFS Root Filesystem Export with Root Privilege Preservation

## Severity

**Critical**

## Affected Service

```text
192.168.247.129:2049/TCP
Network File System (NFS)
```

---

## Description

NFS enumeration identified the server's root filesystem as an accessible network export:

```text
/ *
```

Further inspection identified the export configuration:

```text
/ *(rw,sync,no_root_squash,no_subtree_check)
```

The configuration combined several dangerous conditions:

- The entire root filesystem was exported.
- Access was broadly permitted using `*`.
- The export was writable.
- `no_root_squash` preserved remote root privileges.

This meant a privileged user on a network client could interact with the exported filesystem while retaining root identity on the target.

---

## Validation

The filesystem was initially mounted read-only to inspect accessible resources.

The assessment confirmed access to operating-system directories including:

```text
/etc
/home
/root
/usr
/var
```

A controlled write test was then performed within `/tmp`.

A temporary validation file was created through the NFS mount.

The resulting remote file had:

```text
root root
```

ownership.

This confirmed that remote root identity was preserved and that privileged filesystem modification was possible.

The temporary file was removed following validation.

---

## Impact

The directly demonstrated impact was:

**Remote root-level filesystem modification**

The confirmed access could potentially enable:

- Modification of authentication files
- Addition of SSH authorized keys
- Modification of startup configuration
- Scheduled-task manipulation
- Application tampering
- Service-configuration modification
- Credential exposure
- Persistence
- Full host compromise

These downstream actions were deliberately not performed.

---

## Risk

**Likelihood: High**

The export was directly reachable and required no application-level authentication.

**Impact: Critical**

The assessment demonstrated privileged modification of the target filesystem.

**Overall Severity: Critical**

---

## Recommendation

Priority remediation should include:

- Remove the root filesystem from NFS exports.
- Export only specifically required directories.
- Replace `*` with explicitly authorized clients.
- Remove `no_root_squash`.
- Restore root-squashing behavior.
- Use read-only exports wherever possible.
- Restrict NFS network exposure.
- Review all current exports for excessive permissions.
- Monitor sensitive exported locations for unexpected changes.

Detailed finding:

[`../findings/F001-unrestricted-nfs-root-filesystem-export.md`](../findings/F001-unrestricted-nfs-root-filesystem-export.md)

---

# 11. F-003 — Weak Credentials Permit Unauthorized Access to Apache Tomcat Manager

## Severity

**High**

## Affected Service

```text
192.168.247.129:8180/TCP
Apache Tomcat Manager
```

---

## Description

The target exposed the Apache Tomcat Web Application Manager interface.

An unauthenticated request returned:

```text
HTTP 401 Unauthorized
```

demonstrating that the interface required authentication.

However, the credential pair:

```text
tomcat:tomcat
```

was accepted.

The authenticated request returned:

```text
HTTP 200 OK
```

and browser testing confirmed successful access to the Tomcat Web Application Manager.

---

## Validation

Validation established:

```text
Manager interface reachable
        ↓
Unauthenticated request rejected
        ↓
Weak credentials supplied
        ↓
Authentication successful
        ↓
Administrative interface access confirmed
```

Testing stopped after administrative access had been confirmed.

The assessment did not:

- Deploy a WAR application
- Execute operating-system commands
- Establish a reverse shell
- Create persistence
- Modify deployed applications

---

## Impact

The confirmed impact was:

**Unauthorized administrative application access**

Depending on assigned privileges, Tomcat Manager functionality may allow:

- Application deployment
- Application removal
- Application state management
- Modification of hosted applications
- Service disruption
- Potential server-side code execution

Server-side execution was not validated during this engagement.

---

## Risk

**Likelihood: High**

The management interface was reachable and accepted an easily guessable password identical to the username.

**Impact: High**

Administrative application-management functionality was exposed.

**Overall Severity: High**

---

## Recommendation

Recommended remediation includes:

- Immediately replace weak administrative credentials.
- Use strong unique passwords.
- Remove unnecessary Manager accounts.
- Restrict the Manager interface to authorized administrative systems.
- Place management functionality on a restricted network.
- Review Tomcat administrative roles.
- Disable the Manager application where unnecessary.
- Monitor administrative authentication activity.
- Upgrade obsolete Tomcat deployments and maintain security patches.

Detailed finding:

[`../findings/F003-tomcat-manager-weak-credentials.md`](../findings/F003-tomcat-manager-weak-credentials.md)

---

# 12. F-002 — Anonymous Read/Write Access to SMB Temporary Share

## Severity

**Medium**

## Affected Service

```text
192.168.247.129:139/TCP
192.168.247.129:445/TCP
SMB / Samba
Share: tmp
```

---

## Description

SMB enumeration identified several exposed shares.

The `tmp` share permitted anonymous access.

Manual testing confirmed that an unauthenticated network user could:

- Connect to the share
- Enumerate directory contents
- Upload a file
- Verify the file remotely
- Remove the assessment-created artifact

No valid SMB credentials were required.

---

## Validation

A harmless validation file was uploaded to the anonymous share.

The server accepted the upload and the file appeared in the remote directory listing.

The file was subsequently removed.

Testing did not demonstrate:

- Code execution
- Privilege escalation
- Modification of privileged system files
- Automatic application processing of uploaded content
- Host compromise
- Lateral movement

---

## Impact

The confirmed impact was:

**Unauthorized remote file placement within the writable share**

Potential production consequences could include:

- Shared-data tampering
- Storage of unauthorized files
- Introduction of malicious content
- Interaction with attacker-controlled files by users or applications

These downstream conditions were not demonstrated during the assessment.

---

## Risk

**Likelihood: High**

Anonymous access required no valid credentials and remote file creation was straightforward.

**Impact: Medium**

Remote file modification was demonstrated, but no privileged execution or system compromise was established.

**Overall Severity: Medium**

---

## Recommendation

Recommended remediation includes:

- Disable anonymous or guest write access.
- Require authenticated access to writable shares.
- Apply least-privilege file permissions.
- Restrict SMB access to trusted networks.
- Review Samba share configurations.
- Separate temporary writable storage from privileged application resources.
- Monitor network shares for unauthorized file creation or modification.

Detailed finding:

[`../findings/F002-anonymous-smb-read-write-access.md`](../findings/F002-anonymous-smb-read-write-access.md)

---

# 13. Security Observations

## O-002 — Legacy SMBv1 Protocol Enabled

**Priority: Medium**

Protocol enumeration confirmed:

```text
NT LM 0.12 (SMBv1)
```

The assessment confirmed that SMBv1 was enabled but did not validate exploitation of a specific SMBv1 vulnerability or CVE.

The condition increases attack surface and represents unnecessary reliance on legacy protocol functionality.

### Recommendation

- Disable SMBv1 where operationally unnecessary.
- Require supported modern SMB protocol versions.
- Identify and upgrade legacy systems that depend on SMBv1.
- Restrict SMB access to trusted networks.

Detailed observation:

[`../observations/O002-legacy-smbv1-protocol-enabled.md`](../observations/O002-legacy-smbv1-protocol-enabled.md)

---

## O-001 — Anonymous FTP Access with Plaintext Transport

**Priority: Low**

Anonymous FTP authentication was successfully confirmed.

The service also used plaintext control and data connections.

However:

- No meaningful files were exposed.
- No sensitive information was identified.
- Anonymous uploads were rejected.
- No remote write capability was demonstrated.

The condition was therefore retained as a Low-priority observation.

### Recommendation

- Disable anonymous FTP where unnecessary.
- Require authenticated access.
- Replace plaintext FTP with SFTP or FTPS where appropriate.
- Restrict file-transfer services to approved network locations.

Detailed observation:

[`../observations/O001-anonymous-ftp-access.md`](../observations/O001-anonymous-ftp-access.md)

---

# 14. Attack-Path Analysis

The assessment identified three independent network-accessible attack avenues.

They were **not treated as a chained attack** because no lateral movement or multi-stage exploitation was demonstrated.

---

## 14.1 NFS Attack Avenue

```text
Network Access
      ↓
NFS :2049
      ↓
Root filesystem exported
      ↓
Read/Write enabled
      ↓
no_root_squash
      ↓
Root-owned remote write CONFIRMED
      ↓
Potential privileged system modification
```

This represented the most significant path identified during the assessment.

---

## 14.2 Tomcat Attack Avenue

```text
Network Access
      ↓
Tomcat :8180
      ↓
Manager interface exposed
      ↓
Weak credentials accepted
      ↓
Administrative access CONFIRMED
      ↓
Potential application/server compromise
```

No server-side execution was attempted.

---

## 14.3 SMB Attack Avenue

```text
Network Access
      ↓
SMB :445
      ↓
Anonymous tmp access
      ↓
Read/Write permissions
      ↓
Remote file upload CONFIRMED
      ↓
Unauthorized file placement
```

No privileged execution or host compromise was established.

The full analysis is available in:

[`../diagrams/attack-path.md`](../diagrams/attack-path.md)

---

# 15. Remediation Priorities

## Priority 1 — Immediate

### F-001 — NFS Root Filesystem Export

Actions:

- Remove the `/` export.
- Remove `no_root_squash`.
- Restrict allowed NFS clients.
- Reduce exports to required directories only.
- Remove unnecessary write permissions.
- Apply network restrictions to TCP/2049.

This issue should receive the highest remediation priority because privileged remote filesystem modification was directly demonstrated.

---

## Priority 2 — High

### F-003 — Tomcat Manager Weak Credentials

Actions:

- Replace `tomcat:tomcat`.
- Audit all configured management accounts.
- Restrict access to TCP/8180.
- Disable unnecessary administrative functionality.
- Apply appropriate management-network segmentation.
- Review Tomcat patch/support status.

---

## Priority 3 — Medium

### F-002 — Anonymous SMB Write Access

Actions:

- Disable guest write access.
- Require authentication.
- Review share-level and filesystem permissions.
- Restrict SMB exposure.
- Monitor writable shares.

### O-002 — SMBv1

Actions:

- Disable SMBv1.
- Migrate clients to modern SMB versions.
- Upgrade incompatible legacy systems.

---

## Priority 4 — Lower

### O-001 — Anonymous FTP

Actions:

- Disable unnecessary anonymous authentication.
- Replace plaintext FTP.
- Restrict file-transfer service exposure.

---

# 16. Strategic Remediation Themes

Several broader security themes were identified across the findings.

## 16.1 Reduce Unnecessary Service Exposure

The target exposed 23 TCP services.

Production systems should expose only services required for legitimate business functionality.

Unused services should be:

- Disabled
- Removed
- Firewalled
- Segmented from untrusted networks

---

## 16.2 Enforce Authentication

Anonymous access contributed directly to both the SMB finding and FTP observation.

Sensitive resources should generally require authenticated access with appropriate authorization.

---

## 16.3 Apply Least Privilege

The NFS and SMB findings demonstrate excessive permission assignment.

Permissions should be restricted to the minimum necessary for legitimate operations.

---

## 16.4 Protect Administrative Interfaces

Administrative interfaces such as Tomcat Manager should not be broadly accessible.

Recommended controls include:

- Dedicated management networks
- Firewall allowlists
- Strong authentication
- Restricted administrative roles
- Monitoring of management activity

---

## 16.5 Remove Legacy Protocols

Legacy protocols increase attack surface and complicate system hardening.

SMBv1 should be retired where possible, and plaintext FTP should be replaced with an encrypted alternative.

---

# 17. Assessment Limitations

This engagement was performed against a deliberately vulnerable **single-target lab environment**.

The assessment did not represent a complete enterprise penetration test.

The following were not included:

- Active Directory
- Multi-host lateral movement
- Production endpoints
- Cloud infrastructure
- Wireless infrastructure
- Social engineering
- Physical security
- Denial-of-service testing
- Source-code review
- Full exploitation of every exposed service

Not every open port was investigated to the same depth.

Services were prioritized based on security relevance and expected assessment value.

The findings therefore represent conditions confirmed during the defined engagement and should not be interpreted as an exhaustive list of every weakness present within Metasploitable 2.

---

# 18. Validation and Evidence Philosophy

A central objective of the assessment was to avoid overstating results.

The following distinctions were maintained:

```text
Open port
≠
Confirmed vulnerability
```

```text
Old software version
≠
Confirmed exploitable CVE
```

```text
Potential impact
≠
Impact actually performed
```

For example:

- SMBv1 was documented as an observation rather than claiming an SMB exploit.
- Anonymous FTP remained Low priority because uploads were denied and no sensitive content was identified.
- Anonymous SMB write access was rated Medium because no code execution was demonstrated.
- Tomcat Manager access was rated High without claiming host compromise.
- NFS was rated Critical because privileged remote write access was directly demonstrated.

This approach was intended to ensure that assessment conclusions remained evidence-based and defensible.

---

# 19. Evidence Index

## Reconnaissance

```text
evidence/reconnaissance/
```

Key evidence includes:

- Host discovery
- Initial port scan
- Service/version scan
- Network configuration evidence

---

## Enumeration

```text
evidence/enumeration/
```

Key evidence includes:

- NFS export enumeration
- SMB share enumeration
- SMB protocol enumeration
- FTP enumeration
- Web/Tomcat enumeration
- Tomcat unauthenticated response

---

## Validation

```text
evidence/validation/
```

Key evidence includes:

- NFS filesystem listings
- NFS export configuration
- NFS privileged write validation
- SMB anonymous access
- SMB write validation
- FTP anonymous login
- FTP upload denial
- Tomcat authenticated HTTP response
- Tomcat Manager browser screenshot

Detailed evidence references are maintained within each individual finding and observation.

---

# 20. Conclusion

The simulated assessment identified multiple meaningful security weaknesses within the Asteria Solutions environment.

The most significant issue was the unrestricted NFS root filesystem export with `no_root_squash`, which allowed a remote privileged user to create root-owned content on the target filesystem.

This provided direct evidence of privileged filesystem modification and justified a **Critical severity** classification.

Weak credentials on the Apache Tomcat Manager interface also resulted in confirmed unauthorized administrative access and were classified as **High severity**.

Anonymous write access to the SMB `tmp` share demonstrated unauthorized server-side file modification. Because no privileged execution or host compromise was established, the issue was conservatively classified as **Medium severity**.

Two additional hardening issues—SMBv1 support and anonymous plaintext FTP—were retained as security observations rather than overstated as confirmed exploitation findings.

Overall, the engagement demonstrated that effective security assessment requires more than identifying open ports or running vulnerability scanners.

The assessment process required:

```text
Discovery
   ↓
Technical Analysis
   ↓
Prioritization
   ↓
Manual Validation
   ↓
Evidence Collection
   ↓
Risk Assessment
   ↓
Remediation Planning
   ↓
Professional Reporting
```

Remediation should prioritize the NFS configuration first, followed by securing administrative Tomcat access and removing anonymous SMB write permissions.

Once these issues have been addressed, validation should be repeated to confirm that the identified attack avenues are no longer available.

---

# Appendix A — Findings and Observations

| ID | Type | Title | Rating |
|---|---|---|---:|
| F-001 | Confirmed Finding | Unrestricted NFS Root Filesystem Export with Root Privilege Preservation | Critical |
| F-003 | Confirmed Finding | Weak Credentials Permit Unauthorized Access to Apache Tomcat Manager | High |
| F-002 | Confirmed Finding | Anonymous Read/Write Access to SMB Temporary Share | Medium |
| O-002 | Security Observation | Legacy SMBv1 Protocol Enabled | Medium |
| O-001 | Security Observation | Anonymous FTP Access with Plaintext Transport | Low |

---

# Appendix B — Supporting Documentation

- [`../documentation/client-scenario.md`](../documentation/client-scenario.md)
- [`../documentation/scope-and-rules-of-engagement.md`](../documentation/scope-and-rules-of-engagement.md)
- [`../documentation/environment.md`](../documentation/environment.md)
- [`../documentation/methodology.md`](../documentation/methodology.md)
- [`../documentation/risk-methodology.md`](../documentation/risk-methodology.md)
- [`../documentation/reconnaissance.md`](../documentation/reconnaissance.md)
- [`../diagrams/network-architecture.md`](../diagrams/network-architecture.md)
- [`../diagrams/attack-path.md`](../diagrams/attack-path.md)

---

# Disclaimer

This report documents a cybersecurity assessment conducted entirely within a controlled, isolated, and intentionally vulnerable laboratory environment.

**Asteria Solutions Sdn. Bhd. is a fictional organization.**

No production systems, real client infrastructure, public targets, or unauthorized third-party systems were tested.

The assessment was conducted solely for cybersecurity education, professional development, and portfolio demonstration.
