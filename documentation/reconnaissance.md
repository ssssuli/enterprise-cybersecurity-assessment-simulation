# Reconnaissance & Initial Attack Surface Analysis

## Overview

Initial reconnaissance was performed against the authorized target `192.168.247.129` to identify exposed TCP services and establish the target's network attack surface.

The assessment workstation used for testing was Kali Linux at `192.168.247.128`.

---

## Initial TCP Port Discovery

An initial Nmap TCP scan identified 23 open TCP ports on the target.

Command used:

`nmap 192.168.247.129`

The scan identified services associated with remote administration, web applications, file sharing, databases, remote graphical access and infrastructure services.

| Category | Ports | Observed Services |
|---|---|---|
| Remote Access | 21, 22, 23, 512, 513, 514 | FTP, SSH, Telnet and Unix remote services |
| Web / Application | 80, 8009, 8180 | HTTP and application services |
| File Sharing | 139, 445, 2049 | SMB and NFS |
| Databases | 3306, 5432 | MySQL and PostgreSQL |
| Remote Display | 5900, 6000 | VNC and X11 |
| Infrastructure | 25, 53, 111 | SMTP, DNS and RPC |
| Additional Services | 1099, 1524, 2121, 6667 | RMI, unidentified/legacy services, FTP-related service and IRC |

---

## Initial Assessment

The number and variety of exposed services indicate a broad network attack surface.

Several exposed services use legacy remote-access or file-sharing protocols that warrant further investigation. Database, web application and remote management services are also directly reachable from the assessment network.

Further service enumeration and version identification will be performed before vulnerability conclusions are made.

This assessment will prioritise vulnerable and open ports that come with more risk if an attacker were to find a pathway into them, risk is weighed based on the documented risk and threat detection system.

---

## Next Phase

The next phase will perform service and version detection against identified open ports.

Results will be used to prioritize deeper enumeration and vulnerability validation activities.

Raw scan evidence is retained under:

`evidence/reconnaissance/initial-port-scan.txt`
