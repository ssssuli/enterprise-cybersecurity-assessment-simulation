# Security Assessment Methodology

## 1. Overview

The assessment followed a structured cybersecurity assessment methodology designed to move from broad attack-surface discovery to evidence-based vulnerability validation and remediation.

The process used throughout the engagement was:

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

The methodology emphasized **manual validation, evidence quality, controlled testing, and accurate risk classification**.

An open port, software version, or automated tool result was not automatically treated as a confirmed vulnerability.

---

## 2. Scoping and Rules of Engagement

The assessment began by defining:

- Authorized systems
- Assessment objectives
- Network boundaries
- Permitted testing activities
- Prohibited activities
- Validation limits
- Evidence-handling requirements
- Cleanup requirements

The authorized assessment target was:

```text
192.168.247.129
```

The target represented the simulated enterprise server for **Asteria Solutions Sdn. Bhd.**

The Kali Linux assessment workstation at:

```text
192.168.247.128
```

was not considered an assessment target.

Testing was performed within an isolated VMware host-only environment.

Detailed scope and authorization boundaries are documented in:

[`scope-and-rules-of-engagement.md`](scope-and-rules-of-engagement.md)

---

## 3. Reconnaissance and Host Discovery

The first technical phase established whether the authorized target was reachable and identified its exposed network attack surface.

Activities included:

- Host discovery
- Network connectivity validation
- Initial TCP port scanning
- Identification of exposed services

The objective was not to immediately identify vulnerabilities.

Instead, reconnaissance established:

```text
What is reachable?
       ↓
What services are exposed?
       ↓
Which services require deeper investigation?
```

Initial reconnaissance identified **23 open TCP ports** on the target.

Supporting analysis is documented in:

[`reconnaissance.md`](reconnaissance.md)

---

## 4. Service Enumeration

Following initial discovery, exposed services were analyzed in greater detail.

Enumeration included:

- Service identification
- Product/version detection
- Protocol analysis
- File-share enumeration
- Authentication behavior
- Web application discovery
- Administrative-interface identification
- Service-specific enumeration

Examples included:

- NFS export enumeration
- SMB share and protocol enumeration
- FTP anonymous-access enumeration
- Apache/Tomcat web enumeration

The purpose of enumeration was to generate hypotheses for further investigation.

For example:

```text
NFS service discovered
        ↓
Export enumeration
        ↓
Root filesystem export identified
        ↓
Permission validation required
```

Enumeration results alone were not treated as sufficient evidence of vulnerability.

---

## 5. Vulnerability Analysis

Enumeration results were reviewed to identify security conditions that warranted deeper investigation.

Potential weaknesses considered during this phase included:

- Excessively exposed services
- Weak authentication controls
- Anonymous access
- Insecure file-sharing permissions
- Administrative-interface exposure
- Legacy protocols
- Insecure service configuration
- Excessive privileges
- Potentially vulnerable software versions

The assessment deliberately distinguished between:

```text
Potential weakness
```

and:

```text
Confirmed vulnerability
```

A vulnerability was not considered confirmed merely because:

- A service was running
- A version appeared outdated
- A scanner returned a warning
- A product was historically associated with known vulnerabilities
- A theoretical attack path could exist

Further evidence was required.

---

## 6. Manual Vulnerability Validation

Selected security weaknesses were manually tested to determine whether the suspected condition could be reproduced.

Manual validation activities included:

- Mounting an exposed NFS filesystem
- Inspecting exported filesystem content
- Reviewing NFS export configuration
- Creating a controlled temporary file to validate NFS write privileges
- Connecting anonymously to SMB shares
- Uploading and removing a harmless SMB validation file
- Authenticating anonymously to FTP
- Testing FTP write permissions
- Comparing unauthenticated and authenticated Tomcat Manager responses
- Confirming Tomcat Manager access through a browser

Testing followed a **minimum-impact validation approach**.

The guiding principle was:

> Demonstrate enough impact to prove the weakness without performing unnecessary destructive or post-compromise activity.

---

## 7. Validation Boundaries

Successful validation did not automatically justify deeper exploitation.

For example:

### NFS

Once remote creation of a root-owned file demonstrated privileged filesystem modification, testing stopped.

The assessment did not modify:

- `/etc/passwd`
- `/etc/shadow`
- SSH authorized keys
- Startup configuration
- Scheduled tasks
- System binaries

---

### SMB

Once anonymous remote file creation and deletion were confirmed, testing stopped.

The assessment did not attempt to:

- Execute uploaded content
- Modify privileged files
- Compromise another service
- Demonstrate lateral movement

---

### Apache Tomcat

Once weak credentials provided access to the Tomcat Manager administrative interface, testing stopped.

The assessment did not:

- Deploy a WAR application
- Establish a reverse shell
- Execute operating-system commands
- Modify hosted applications
- Create persistence

---

### FTP

Anonymous authentication was confirmed, followed by directory and write-permission testing.

Because no meaningful content was exposed and uploads were denied, no further FTP exploitation was required.

---

## 8. Finding Classification

Assessment results were divided into two primary categories.

### Confirmed Finding

A condition was classified as a confirmed finding when manual testing demonstrated a meaningful security weakness with sufficient supporting evidence.

The assessment produced three confirmed findings:

| ID | Finding | Severity |
|---|---|---:|
| F-001 | Unrestricted NFS Root Filesystem Export with Root Privilege Preservation | Critical |
| F-002 | Anonymous Read/Write Access to SMB Temporary Share | Medium |
| F-003 | Weak Credentials Permit Unauthorized Access to Apache Tomcat Manager | High |

---

### Security Observation

A condition was classified as a security observation when the insecure behavior was confirmed but the demonstrated impact did not justify treatment as a primary vulnerability.

The assessment documented:

| ID | Observation | Priority |
|---|---|---:|
| O-001 | Anonymous FTP Access with Plaintext Transport | Low |
| O-002 | Legacy SMBv1 Protocol Enabled | Medium |

This distinction helped avoid exaggerating risk where evidence was limited.

---

## 9. Risk Assessment

Confirmed findings were assessed using a qualitative risk methodology based primarily on:

```text
Likelihood × Impact
```

Factors considered included:

- Required attacker access
- Authentication requirements
- Attack complexity
- Network accessibility
- Permissions obtained
- Confidentiality impact
- Integrity impact
- Availability impact
- Demonstrated privilege level
- Potential downstream consequences
- Confidence in the available evidence

The assessment prioritized **demonstrated impact** over hypothetical worst-case outcomes.

For example, the anonymous SMB writable share was classified as **Medium**, despite high likelihood, because testing demonstrated unauthorized file placement but did not demonstrate code execution or privileged system compromise.

The complete risk-rating approach is documented in:

[`risk-methodology.md`](risk-methodology.md)

---

## 10. Attack-Path Analysis

Confirmed findings were reviewed from an attacker-perspective to determine how each weakness could contribute to broader compromise.

The assessment did not artificially combine unrelated weaknesses into a single attack chain.

Instead, the findings were treated as **independent attack avenues** from a network-accessible attacker:

```text
Network-accessible attacker
        |
        +-- NFS :2049
        |      ↓
        |   Root filesystem export
        |      ↓
        |   Privileged remote write confirmed
        |
        +-- Tomcat :8180
        |      ↓
        |   Weak credentials accepted
        |      ↓
        |   Administrative access confirmed
        |
        +-- SMB :445
               ↓
           Anonymous tmp access
               ↓
           Remote file placement confirmed
```

Potential downstream compromise is documented separately from actions directly performed during testing.

This prevents the report from implying lateral movement or multi-stage compromise that was not demonstrated.

---

## 11. Remediation Analysis

Each confirmed finding and security observation includes remediation guidance.

Recommendations were designed to address the **underlying security weakness**, not simply block the exact test performed during the assessment.

For example:

### NFS

Remediation focuses on:

- Removing the root filesystem export
- Restricting authorized clients
- Removing `no_root_squash`
- Applying least privilege

rather than merely blocking the assessment workstation.

---

### SMB

Remediation focuses on:

- Removing anonymous write permissions
- Requiring authenticated access
- Restricting SMB exposure
- Applying least-privilege share permissions

---

### Tomcat

Remediation focuses on:

- Replacing weak credentials
- Restricting management-interface exposure
- Reviewing administrative roles
- Disabling unnecessary management functionality

---

## 12. Remediation Prioritization

Remediation was prioritized according to the validated risk level.

### Immediate

**F-001 — Critical**

Address the unrestricted NFS root export and privileged remote write capability.

### High Priority

**F-003 — High**

Replace weak Tomcat Manager credentials and restrict access to the administrative interface.

### Medium Priority

**F-002 — Medium**

Remove anonymous SMB write access and apply authenticated least-privilege access.

**O-002 — Medium**

Remove SMBv1 where operationally unnecessary.

### Lower Priority

**O-001 — Low**

Disable unnecessary anonymous FTP access and migrate away from plaintext FTP where possible.

---

## 13. Evidence Handling

Assessment conclusions were supported using retained evidence including:

- Nmap scan output
- Service enumeration output
- Command-line validation sessions
- Directory listings
- Configuration evidence
- HTTP responses
- Controlled write-validation results
- Screenshots

Evidence was organized into:

```text
evidence/reconnaissance/
evidence/enumeration/
evidence/validation/
```

Technical evidence was retained to allow findings to be independently reviewed and traced back to the observed behavior.

---

## 14. Reporting

The reporting phase consolidates the assessment into both technical and management-oriented documentation.

The final assessment report includes:

- Executive summary
- Engagement scope
- Environment overview
- Assessment methodology
- Risk methodology
- Findings summary
- Detailed vulnerability findings
- Security observations
- Attack-path analysis
- Prioritized remediation plan
- Assessment limitations
- Conclusion
- Evidence references

The repository also retains the detailed individual finding documents and supporting evidence so that technical conclusions remain auditable.

---

## 15. Assessment Principles

The following principles guided the engagement:

1. **Open ports are not vulnerabilities by themselves.**
2. **Version strings do not prove exploitability.**
3. **Automated output requires analyst review.**
4. **Manual validation should confirm meaningful security conditions.**
5. **Testing should stop once sufficient evidence has been obtained.**
6. **Destructive or unnecessary exploitation should be avoided.**
7. **Confirmed impact must be separated from potential impact.**
8. **Severity should reflect demonstrated evidence rather than the most dramatic theoretical outcome.**
9. **Findings should be reproducible and defensible.**
10. **Remediation should address root causes rather than individual test commands.**

---

## 16. Methodology Outcome

Applying this methodology reduced an initial attack surface of **23 open TCP ports** into a focused set of validated security issues.

The final assessment produced:

- **3 confirmed vulnerability findings**
- **2 security observations**
- Controlled supporting evidence
- Risk-based severity classification
- Remediation recommendations
- Attack-path analysis
- A final consulting-style assessment report

This approach demonstrates a complete security-assessment lifecycle rather than a collection of isolated scanning or exploitation exercises.
