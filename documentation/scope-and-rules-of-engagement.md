# Security Assessment Scope & Rules of Engagement

## 1. Engagement Overview

This document defines the scope, authorization boundaries, testing limitations, and Rules of Engagement for the simulated cybersecurity assessment conducted for **Asteria Solutions Sdn. Bhd.**

Asteria Solutions Sdn. Bhd. is a fictional organization created solely to provide realistic business context for this portfolio assessment.

The engagement was conducted entirely within a controlled VMware lab containing an intentionally vulnerable Linux server.

The purpose of the assessment was to:

- Identify the target's exposed attack surface
- Enumerate accessible network and application services
- Identify potential security weaknesses
- Manually validate significant findings
- Evaluate likelihood and potential impact
- Recommend appropriate remediation
- Produce professional technical and executive-level security documentation

No production infrastructure or unauthorized third-party systems were included.

---

## 2. Authorization

Testing was authorized only against systems deliberately deployed within the controlled lab environment for this project.

The assessment workstation and target system were owned or controlled for educational cybersecurity testing.

Authorization applied exclusively to:

```text
Target: 192.168.247.129
System: Metasploitable 2
Role: Simulated Asteria Solutions enterprise server
```

Testing outside this defined target was not authorized.

---

## 3. In-Scope Assets

| Asset ID | Asset | IP Address | Function | Scope |
|---|---|---|---|---|
| TG-01 | Metasploitable 2 | `192.168.247.129` | Simulated enterprise server | In Scope |

All network and application services hosted by TG-01 were considered eligible for reconnaissance and security assessment, subject to the restrictions defined in this document.

Examples of identified services included:

- FTP
- SSH
- Telnet
- SMTP
- DNS
- HTTP
- RPC
- SMB / Samba
- NFS
- MySQL
- PostgreSQL
- VNC
- Apache Tomcat
- Additional legacy services

The presence of a service within scope did not require that every service be exploited.

Testing was prioritized according to security relevance and potential impact.

---

## 4. Assessment Workstation

| Asset ID | Asset | IP Address | Function | Target Status |
|---|---|---|---|---|
| AS-01 | Kali Linux | `192.168.247.128` | Security assessment workstation | Not a Target |

The Kali Linux system was used solely to conduct authorized assessment activity.

It was not considered an assessment target.

---

## 5. Network Scope

Testing was conducted across an isolated **VMware host-only network** connecting the assessment workstation and intentionally vulnerable target.

The assessment was designed so that testing traffic remained within the controlled virtual environment.

The following were explicitly outside the network scope:

- Public internet infrastructure
- The physical local network
- The host operating system
- Other virtual machines not explicitly included
- Third-party infrastructure
- Production systems

---

## 6. Permitted Activities

The following activities were authorized against the in-scope target:

- Host discovery
- TCP port scanning
- Service enumeration
- Service/version identification
- Network attack-surface analysis
- File-sharing enumeration
- Web-service enumeration
- Administrative-interface discovery
- Authentication testing
- Anonymous-access testing
- Permission testing
- Manual vulnerability validation
- Controlled proof-of-impact testing
- Limited file creation where required to validate write permissions
- HTTP request and response analysis
- Risk assessment
- Attack-path analysis
- Remediation analysis

Testing was limited to activity necessary to establish whether an identified security weakness was present and demonstrate sufficient evidence of its potential impact.

---

## 7. Controlled Validation

Manual validation was permitted where additional testing was necessary to distinguish a potential vulnerability from a confirmed finding.

The guiding principle was:

> **Perform the minimum level of interaction required to prove the security condition without causing unnecessary impact.**

Examples of permitted validation included:

- Mounting an exposed NFS filesystem
- Inspecting accessible directories and configuration information
- Creating a harmless temporary file to validate NFS write permissions
- Connecting anonymously to an SMB share
- Uploading a harmless test file to validate SMB write capability
- Attempting a harmless FTP upload to determine available permissions
- Testing authentication against an exposed administrative interface
- Comparing authenticated and unauthenticated HTTP responses
- Manually confirming access through a browser

Temporary assessment artifacts were removed whenever the tested service permitted cleanup.

---

## 8. Prohibited and Out-of-Scope Activities

The following activities were excluded from the engagement:

- Denial-of-service or resource-exhaustion attacks
- Destructive testing
- Intentional corruption of system data
- Modification of critical authentication files
- Modification of SSH keys for persistence
- Creation of persistent accounts
- Scheduled-task or startup persistence
- Malware deployment
- Destructive database modification
- Social engineering
- Testing against real individuals
- Testing of external systems
- Testing of production infrastructure
- Attacks against the physical network
- Attacks against the virtualization host
- Unnecessary post-compromise activity

The assessment also avoided actions whose only purpose would have been to demonstrate additional control after a vulnerability had already been sufficiently validated.

---

## 9. Exploitation Boundary

Limited exploitation was permitted only where it was necessary to validate the existence or practical impact of a security weakness.

Successful validation did not automatically justify further exploitation.

For example, once access to the Apache Tomcat Manager interface was confirmed using weak credentials, the engagement did **not** proceed to:

- Deploy a malicious WAR file
- Execute operating-system commands
- Establish a reverse shell
- Create persistence
- Alter existing applications

Similarly, NFS validation stopped after controlled root-owned file creation demonstrated the consequences of the export configuration.

This boundary was intended to preserve a minimally invasive assessment approach.

---

## 10. Credential Testing

Authentication testing was permitted where an exposed service presented an authentication interface relevant to the assessment.

The engagement did not rely on large-scale brute-force attacks, uncontrolled password spraying, or denial-of-service-style authentication attempts.

Where a weak or predictable credential was tested successfully, validation stopped once the resulting access level could be established.

Credentials used exclusively within the intentionally vulnerable lab may be retained as technical evidence where necessary to explain a finding.

No real personal, organizational, or third-party credentials were used.

---

## 11. Vulnerability Classification

An open port, identified software version, or automated tool result was not automatically classified as a confirmed vulnerability.

Issues were categorized based on available evidence.

### Confirmed Finding

A security weakness was classified as a confirmed finding when manual testing demonstrated the insecure condition and provided sufficient evidence of practical security impact.

### Security Observation

A condition was recorded as a security observation where:

- An insecure configuration or legacy technology was confirmed, but
- The available evidence did not justify the same level of impact as a primary vulnerability finding.

This distinction was used to reduce false positives and avoid overstating assessment results.

---

## 12. Evidence Handling

Assessment evidence included:

- Nmap output
- Service enumeration results
- Command-line session output
- HTTP responses
- Directory listings
- Configuration evidence
- Controlled write-validation results
- Screenshots
- Manual validation records

Evidence was organized into:

```text
evidence/reconnaissance/
evidence/enumeration/
evidence/validation/
```

Only information relevant to the simulated assessment was retained in the public portfolio.

Real credentials, authentication tokens, unrelated personal information, or third-party sensitive data were not intentionally collected or published.

---

## 13. Cleanup Requirements

Where validation created temporary artifacts, those artifacts were removed following testing where technically possible.

Confirmed cleanup activities included:

- Removal of the temporary NFS validation file
- Removal of the temporary SMB validation file

The FTP write attempt failed and therefore did not create a remote artifact.

Tomcat validation did not modify server-side applications or configuration.

---

## 14. Assessment Limitations

The engagement was performed against a deliberately vulnerable **single-target lab environment**.

The assessment therefore did not represent:

- A complete production enterprise penetration test
- An Active Directory assessment
- A multi-host lateral-movement exercise
- A social-engineering assessment
- A wireless security assessment
- A cloud-security assessment
- A denial-of-service assessment
- A complete source-code review

Not every open service was tested to exploitation.

Services were prioritized according to exposure, permissions, authentication controls, and expected security relevance.

These limitations should be considered when interpreting the assessment results.

---

## 15. Deliverables

The completed engagement produces the following portfolio deliverables:

- Client scenario
- Scope and Rules of Engagement
- Assessment environment documentation
- Assessment methodology
- Risk methodology
- Reconnaissance and attack-surface analysis
- Confirmed vulnerability findings
- Security observations
- Supporting technical evidence
- Network architecture diagram
- Attack-path analysis
- Remediation recommendations
- Executive summary
- Final cybersecurity assessment report

---

## 16. Rules of Engagement Summary

| Requirement | Rule |
|---|---|
| Authorized Target | `192.168.247.129` only |
| Assessment Network | Isolated VMware host-only network |
| Production Testing | Prohibited |
| Third-Party Testing | Prohibited |
| Denial of Service | Prohibited |
| Destructive Testing | Prohibited |
| Manual Validation | Permitted |
| Controlled File Creation | Permitted when required for validation |
| Persistence | Prohibited |
| Evidence Collection | Permitted |
| Temporary Artifact Cleanup | Required where possible |
| Exploitation Depth | Minimum necessary to validate security impact |

---

## 17. Engagement Principle

The assessment followed a simple operating principle:

**Identify → Investigate → Validate → Assess Impact → Document → Remediate**

The objective was not to exploit every service available on the target.

The objective was to demonstrate a controlled and defensible security-assessment process in which conclusions were supported by evidence and testing remained within clearly defined authorization boundaries.
