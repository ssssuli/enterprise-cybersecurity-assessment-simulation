# Assessment Environment

## 1. Environment Overview

The cybersecurity assessment was conducted within a controlled and isolated virtual lab designed to simulate a small enterprise server environment for the fictional organization **Asteria Solutions Sdn. Bhd.**

The lab consisted of:

- A Kali Linux security assessment workstation
- A Metasploitable 2 vulnerable Linux server
- An isolated VMware host-only network connecting the systems

The environment was intentionally vulnerable and used exclusively for authorized cybersecurity testing, education, and portfolio development.

No production infrastructure, third-party systems, or public internet targets were included in the assessment.

---

## 2. Environment Purpose

The lab was designed to support a structured security assessment covering:

- Host discovery
- Network reconnaissance
- TCP port scanning
- Service and version enumeration
- File-sharing service assessment
- Web and administrative-interface enumeration
- Authentication testing
- Manual vulnerability validation
- Controlled proof-of-impact testing
- Risk assessment
- Attack-path analysis
- Remediation planning
- Security reporting

The environment was intentionally kept small so that selected findings could be investigated and documented in greater depth rather than relying only on automated vulnerability scanning.

---

## 3. Network Architecture

The assessment systems were connected through a **VMware host-only network**.

```text
+-------------------------------+
| Security Assessment Workstation |
| Kali Linux                    |
| 192.168.247.128               |
+---------------+---------------+
                |
                | VMware Host-Only Network
                |
+---------------+---------------+
| Simulated Enterprise Server   |
| Metasploitable 2              |
| 192.168.247.129               |
+-------------------------------+
```

The host-only configuration was selected to isolate the deliberately vulnerable target from external networks while still allowing direct communication between the assessment workstation and the target.

The assessment target was not intentionally exposed to the public internet.

---

## 4. Assessment Workstation

| Attribute | Details |
|---|---|
| Asset ID | AS-01 |
| System | Security Assessment Workstation |
| Operating System | Kali Linux |
| IP Address | `192.168.247.128` |
| Role | Security testing and analysis |
| Assessment Target | No |
| Virtualization | VMware Workstation |

The Kali Linux workstation was used to conduct all authorized testing against the simulated enterprise server.

### Tools Used

Tools and utilities used during the assessment included:

- Nmap
- `smbclient`
- NFS client utilities
- FTP client
- `curl`
- Web browser
- Standard Linux command-line utilities

Automated output was used to support investigation but was not treated as confirmation of a vulnerability without further analysis or validation.

---

## 5. Assessment Target

### TG-01 — Simulated Enterprise Server

| Attribute | Details |
|---|---|
| Asset ID | TG-01 |
| System | Metasploitable 2 |
| Hostname | `metasploitable.localdomain` |
| IP Address | `192.168.247.129` |
| Operating System | Ubuntu Linux-based intentionally vulnerable system |
| Role | Simulated Asteria Solutions enterprise server |
| Scope | In Scope |

The Metasploitable 2 system represented the primary server operated by the fictional organization.

The target hosted multiple network, file-sharing, database, remote-access, and web application services.

Initial reconnaissance identified **23 open TCP ports**, providing the attack surface used for subsequent service prioritization and vulnerability assessment.

---

## 6. Exposed Services

Service enumeration identified the following major network-accessible services:

| Port | Service / Technology | Assessment Relevance |
|---|---|---|
| 21 | FTP / vsFTPd | Anonymous authentication assessment |
| 22 | SSH / OpenSSH | Remote administration exposure |
| 23 | Telnet | Legacy remote-access service |
| 25 | SMTP / Postfix | Mail service exposure |
| 53 | DNS / BIND | Infrastructure service |
| 80 | Apache HTTP Server | Web application exposure |
| 111 | RPCBind | RPC service discovery |
| 139 | NetBIOS / SMB | File-sharing exposure |
| 445 | SMB / Samba | File-sharing and access-control assessment |
| 512–514 | Legacy Unix remote services | Legacy remote-access exposure |
| 1099 | Java RMI Registry | Application service exposure |
| 1524 | Legacy service / bind shell | High-risk service identified during enumeration |
| 2049 | NFS | Filesystem exposure and permission assessment |
| 2121 | ProFTPD | Additional FTP service |
| 3306 | MySQL | Database service exposure |
| 5432 | PostgreSQL | Database service exposure |
| 5900 | VNC | Remote graphical-access service |
| 6000 | X11 | Remote display service |
| 6667 | IRC / UnrealIRCd | Application service exposure |
| 8009 | AJP | Apache application connector |
| 8180 | Apache Tomcat | Web application and management interface |

The presence of an open service was treated as an **attack-surface observation**, not automatically as a vulnerability.

Deeper testing was prioritized based on the service's accessibility, permissions, authentication controls, and potential security impact.

---

## 7. Prioritized Assessment Areas

Following reconnaissance and service enumeration, four service areas were selected for deeper assessment:

### NFS

NFS was prioritized because the service exposed filesystem resources and therefore had the potential to affect sensitive operating-system data and permissions.

Testing ultimately resulted in:

**F-001 — Unrestricted NFS Root Filesystem Export with Root Privilege Preservation**

---

### SMB / Samba

SMB was prioritized to determine whether network shares could be enumerated or accessed without valid credentials and whether exposed shares permitted modification.

Testing resulted in:

**F-002 — Anonymous Read/Write Access to SMB Temporary Share**

and:

**O-002 — Legacy SMBv1 Protocol Enabled**

---

### Web / Apache Tomcat

Web services were assessed to identify exposed applications and administrative interfaces.

Apache Tomcat on TCP/8180 exposed a Manager interface that was subsequently tested for authentication weaknesses.

Testing resulted in:

**F-003 — Weak Credentials Permit Unauthorized Access to Apache Tomcat Manager**

---

### FTP

FTP was assessed after automated enumeration identified anonymous authentication.

Manual testing confirmed anonymous login but found no meaningful exposed content and denied anonymous file uploads.

The condition was therefore recorded as:

**O-001 — Anonymous FTP Access with Plaintext Transport**

---

## 8. Asset Inventory

| Asset ID | Asset | IP Address | Function | Scope | Status |
|---|---|---|---|---|---|
| AS-01 | Kali Linux | `192.168.247.128` | Security Assessment Workstation | Not a Target | Active |
| TG-01 | Metasploitable 2 | `192.168.247.129` | Simulated Enterprise Server | In Scope | Active |

No separate second web application target was used.

Web services, including Apache HTTP and Apache Tomcat, were hosted on **TG-01** and assessed as services belonging to the same target system.

---

## 9. Environment Isolation and Safety

The deliberately vulnerable target was operated within a VMware host-only network to reduce the possibility of assessment traffic reaching unauthorized systems.

Testing intentionally excluded:

- The host operating system
- Physical network infrastructure
- Public internet systems
- Third-party services
- Production environments
- Real organizations or customers

The assessment also excluded denial-of-service activity and destructive modification of the target.

---

## 10. Validation Controls

Manual validation was limited to the minimum activity required to demonstrate the existence and impact of identified weaknesses.

Examples included:

- Mounting the NFS export and inspecting accessible filesystem locations
- Creating a harmless temporary file to validate NFS write permissions
- Removing the NFS validation artifact after testing
- Connecting anonymously to the SMB `tmp` share
- Uploading a harmless SMB validation file
- Removing the SMB validation file following testing
- Authenticating anonymously to FTP and testing whether uploads were permitted
- Comparing unauthenticated and authenticated responses from the Tomcat Manager interface
- Accessing the Tomcat Manager interface without deploying applications or executing server-side commands

No persistence mechanisms were created.

No reverse shells were established.

No sensitive operating-system files were intentionally modified.

---

## 11. Environment Changes During Testing

The following temporary changes were made as part of controlled validation:

| Validation Activity | Temporary Change | Cleanup |
|---|---|---|
| NFS write validation | Temporary file created within the target `/tmp` directory | File removed after validation |
| SMB write validation | Temporary test file uploaded to the SMB `tmp` share | File deleted after validation |
| FTP write validation | Upload attempted | Server denied file creation; no remote artifact created |
| Tomcat validation | Authenticated administrative interface accessed | No application or configuration changes made |

These actions were limited to demonstrating access or permissions and did not intentionally alter the operational configuration of the target.

---

## 12. Supporting Evidence

Environment and reconnaissance evidence is retained within:

```text
evidence/reconnaissance/
```

Relevant evidence includes:

- Host discovery results
- Initial TCP port scan
- Service/version enumeration
- Assessment workstation network configuration
- Target network configuration

Detailed service-specific evidence is retained under:

```text
evidence/enumeration/
evidence/validation/
```

---

## 13. Environment Limitations

This project represents a **controlled single-target simulation** rather than a full production enterprise network.

As a result:

- Lateral movement between multiple enterprise hosts was not assessed.
- Active Directory was not included.
- Production identity infrastructure was not included.
- Internet-facing perimeter controls were not simulated.
- Business-critical production data was not present.
- Availability and denial-of-service testing were excluded.

These limitations were considered when interpreting the potential impact of identified findings.

The purpose of the environment was to demonstrate a structured vulnerability-assessment process rather than reproduce every component of a production enterprise network.
