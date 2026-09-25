# Assessment Environment

## 1. Environment Overview

The cybersecurity assessment is conducted within a controlled and isolated lab environment designed to simulate a small enterprise network. (Sulaiman Solutions)

The environment contains intentionally vulnerable systems used to demonstrate common security assessment and penetration testing techniques.

All systems included in this environment are owned or controlled specifically for educational and portfolio purposes.

---

## 2. Environment Objectives

The environment is designed to support:

- Network reconnaissance
- Host discovery
- Port and service enumeration
- Vulnerability assessment
- Web application testing
- Manual vulnerability validation
- Controlled exploitation
- Attack-path analysis
- Security reporting

The lab is intentionally kept small to allow findings to be investigated and documented thoroughly.

---

## 3. Assessment Workstation

| Attribute | Details |
|---|---|
| System | Security Assessment Workstation |
| Operating System | Kali Linux |
| IP Address | 192.168.247.128 |
| Role | Conduct security testing and analysis |
| Assessment Target | No |

The assessment workstation will be used to perform authorized testing against systems within the controlled environment.

Tools include:

- Nmap
- OWASP ZAP
- Metasploit Framework
- Wireshark
- Linux security utilities

---

## 4. Target Systems

### Target 01 — Vulnerable Linux Server

| Attribute | Details |
|---|---|
| Hostname | metasploitable |
| IP Address | 192.168.247.129 |
| Operating System | Ubuntu Linux (intentionally vulnerable) |
| Role | Simulated enterprise server |
| Scope | In Scope |

The server will provide network services that can be identified, enumerated and assessed for security weaknesses.

---

### Target 02 — Vulnerable Web Application

| Attribute | Details |
|---|---|
| Application | To Be Confirmed |
| IP Address / URL | To Be Confirmed |
| Platform | To Be Confirmed |
| Role | Simulated enterprise web application |
| Scope | In Scope |

The application will be used to assess web application security weaknesses within a controlled environment.

---

## 5. Network Configuration

The final network configuration will be documented after the virtual lab has been created.

The environment will use an isolated or controlled virtual network to prevent assessment traffic from affecting external systems.


## 6. Asset Inventory

| Asset ID | Asset | IP Address | Operating System | Function | Status |
|---|---|---|---|---|---|
| AS-01 | Kali Linux | 192.168.247.128 | Security Assessment Workstation | Active |
| TG-01 | Metasploitable 2 | 192.168.247.129 | Simulated Enterprise Server | Active |
| TG-02 | Vulnerable Web Application | Pending | Pending | Web Application | Pending |

This table will be updated after deployment and initial asset discovery.

---

## 7. Environment Security

The lab will be configured so that assessment activity remains within the controlled environment.

Testing will not intentionally target:

- The host operating system
- The physical network
- Public internet infrastructure
- Third-party services
- Production systems

The environment may be reset or restored if testing modifies a vulnerable target.

---

## 8. Environment Changes

Significant configuration changes made during the assessment will be documented where they affect testing results.

This helps maintain reproducibility and ensures findings can be understood within the correct technical context.

Environment changelog if applicable will be shown below:
