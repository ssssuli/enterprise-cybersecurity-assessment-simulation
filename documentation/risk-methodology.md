# Risk Assessment Methodology

## 1. Purpose

This document defines the qualitative risk-rating methodology used during the simulated cybersecurity assessment for **Asteria Solutions Sdn. Bhd.**

The purpose of the methodology is to ensure that security findings are rated consistently based on:

- The likelihood that a weakness could be abused
- The demonstrated or reasonably supported security impact
- The level of access required
- The complexity of exploitation
- The exposure of the affected service
- The confidence of the available evidence

The assessment intentionally uses a **qualitative risk model** rather than assigning CVSS scores.

This approach was selected because the project focuses on demonstrating defensible analyst judgment based on observed evidence rather than creating numerical scores that may imply greater precision than the available lab context supports.

---

## 2. Risk Rating Approach

Confirmed findings are assessed using two primary factors:

```text
Overall Risk = Likelihood × Impact
```

Both **Likelihood** and **Impact** are rated using the following levels:

- Low
- Medium
- High
- Critical, where appropriate for impact

The final severity is determined through analyst judgment using the defined criteria and the risk matrix in this document.

A finding is not assigned a high severity solely because:

- A service is old
- A port is open
- A tool labels something as dangerous
- A software version has known vulnerabilities
- A hypothetical attack could produce serious consequences

Severity must be supported by the behavior actually observed during testing.

---

# 3. Likelihood Assessment

Likelihood represents how practical it would be for an attacker with the assumed level of access to abuse the identified weakness.

The assessment assumes an attacker already has **network connectivity to the simulated enterprise environment** but does not initially possess authenticated access to the target.

## Low Likelihood

A weakness is considered Low likelihood when exploitation would require significant additional conditions.

Examples include:

- Specialized access not available from the assessment network
- Multiple prerequisite vulnerabilities
- Significant user interaction
- Highly complex exploitation
- Conditions that were not demonstrated during testing
- A theoretical weakness with limited practical accessibility

---

## Medium Likelihood

A weakness is considered Medium likelihood when exploitation is feasible but depends on additional conditions.

Examples include:

- A reachable vulnerable service where exploitability has not been fully established
- An insecure protocol that increases exposure but does not directly provide unauthorized access
- Access requiring some prior knowledge or environmental condition
- A weakness whose practical impact depends heavily on how another service uses the affected resource

---

## High Likelihood

A weakness is considered High likelihood when exploitation is straightforward from the assessment network.

Indicators include:

- No authentication is required
- Weak or predictable credentials successfully authenticate
- The service is directly reachable
- Exploitation requires minimal technical complexity
- The insecure condition was manually reproduced
- No additional compromise is necessary before abusing the weakness

Examples from this assessment include:

- Unauthenticated NFS filesystem access
- Anonymous writable SMB access
- Successful authentication to Tomcat Manager using weak credentials

---

# 4. Impact Assessment

Impact represents the potential effect on the confidentiality, integrity, or availability of the affected system.

The assessment distinguishes between:

1. **Impact directly demonstrated during testing**
2. **Potential downstream impact reasonably supported by the confirmed access**

Potential consequences are not treated as if they were actually performed.

---

## Low Impact

Low impact applies where the identified condition provides limited access or exposure and does not meaningfully affect sensitive information or system integrity.

Examples include:

- Exposure of non-sensitive information
- Anonymous access to an empty service directory
- Minor configuration weaknesses with limited practical effect
- Security hardening opportunities without demonstrated compromise

---

## Medium Impact

Medium impact applies where an attacker could affect a limited system resource or create a meaningful security condition without demonstrating privileged control.

Examples include:

- Unauthorized file placement in a writable shared directory
- Modification of non-critical shared content
- Exposure of legacy or insecure protocols
- Access whose final impact depends on interaction with another application or process

---

## High Impact

High impact applies where unauthorized access could significantly affect application or system security.

Examples include:

- Unauthorized administrative interface access
- Modification of important application resources
- Compromise of privileged application-management functionality
- Significant confidentiality, integrity, or availability consequences

---

## Critical Impact

Critical impact applies where the confirmed weakness provides a direct path toward privileged operating-system compromise or unrestricted modification of highly sensitive system resources.

Examples include:

- Root-level filesystem modification
- Ability to directly alter operating-system authentication or security configuration
- Privileged control capable of affecting the entire host
- A condition that could reasonably enable persistent full-system compromise

Critical impact does not require that destructive actions actually be performed when a safer proof sufficiently demonstrates the privilege level available.

---

# 5. Qualitative Risk Matrix

The following matrix is used as a guide when assigning overall severity.

| Likelihood | Low Impact | Medium Impact | High Impact | Critical Impact |
|---|---:|---:|---:|---:|
| **Low** | Low | Low | Medium | Medium |
| **Medium** | Low | Medium | Medium | High |
| **High** | Medium | Medium | High | Critical |

The matrix supports consistent classification but does not replace analyst judgment.

Where uncertainty exists, the assessment favors the rating best supported by demonstrated evidence rather than automatically selecting the highest conceivable impact.

---

# 6. Evidence Confidence

Risk classification also considers the strength of the available evidence.

## Confirmed

A finding is considered confirmed when manual testing directly demonstrates the insecure behavior.

Examples include:

- Successfully mounting an exposed filesystem
- Successfully writing a remote file
- Successfully connecting anonymously to a share
- Successfully authenticating to an administrative interface

Confirmed findings may receive severity ratings based on demonstrated and reasonably supported impact.

---

## Partially Validated

A potential weakness may be partially validated where:

- The service or configuration is confirmed
- Some relevant insecure behavior is observed
- Full exploitability or practical impact remains uncertain

Such issues may be documented as lower-severity findings or security observations.

---

## Unconfirmed

A condition is considered unconfirmed where evidence consists only of:

- Product/version identification
- Automated scanner output
- General vulnerability databases
- Theoretical applicability
- Open service exposure without demonstrated weakness

Unconfirmed conditions are not reported as confirmed vulnerabilities.

---

# 7. Finding vs. Security Observation

The assessment distinguishes between a **Confirmed Finding** and a **Security Observation**.

## Confirmed Finding

A primary finding requires evidence demonstrating a meaningful security weakness with practical impact.

Examples from this engagement include:

- Unauthorized privileged filesystem modification
- Anonymous remote file modification
- Unauthorized administrative interface access

Confirmed findings receive a formal severity rating.

---

## Security Observation

A security observation documents a condition that increases risk or represents weak security practice but does not have enough demonstrated impact to justify treatment as a primary vulnerability.

Examples include:

- Anonymous access to an FTP service where no meaningful data is exposed and writes are denied
- Support for a legacy protocol where no protocol-specific exploitability was demonstrated

Observations receive a **priority** rather than being presented as equivalent to validated vulnerabilities.

---

# 8. Assessment-Specific Severity Decisions

## F-001 — Unrestricted NFS Root Filesystem Export with Root Privilege Preservation

**Likelihood: High**

Reasons:

- NFS was directly reachable from the assessment network.
- No application-level authentication was required to access the export.
- The entire root filesystem was exposed.
- The export permitted read/write operations.
- `no_root_squash` preserved privileged client identity.
- Manual testing successfully demonstrated remote write capability.

**Impact: Critical**

Reasons:

- Testing confirmed creation of a remotely written file owned by `root`.
- The export exposed operating-system directories and configuration.
- The confirmed permissions could permit modification of security-critical system files.

**Overall Severity: Critical**

The assessment did not modify sensitive operating-system files because controlled file creation was sufficient to demonstrate privileged filesystem access.

---

## F-002 — Anonymous Read/Write Access to SMB Temporary Share

**Likelihood: High**

Reasons:

- SMB was directly accessible from the assessment network.
- No credentials were required to access the `tmp` share.
- Manual testing successfully demonstrated file upload and deletion.

**Impact: Medium**

Reasons:

- Anonymous users could modify content within the exposed temporary share.
- Unauthorized file placement and tampering were demonstrated.
- No evidence established that uploaded content would be executed or consumed by a privileged application.
- No code execution, privilege escalation, or host compromise was demonstrated.

**Overall Severity: Medium**

The severity is intentionally limited to Medium because the assessment confirmed unauthorized file modification but did not demonstrate a direct path to privileged execution or sensitive system compromise.

---

## F-003 — Weak Credentials Permit Unauthorized Access to Apache Tomcat Manager

**Likelihood: High**

Reasons:

- The Tomcat Manager interface was directly reachable.
- Unauthenticated access correctly returned HTTP `401 Unauthorized`.
- The predictable credential pair `tomcat:tomcat` successfully authenticated.
- The authenticated request returned HTTP `200 OK`.
- Browser testing confirmed access to the administrative Manager interface.

**Impact: High**

Reasons:

- The compromised account provided access to a privileged application-management interface.
- Administrative access could permit significant modification or disruption of hosted applications depending on assigned permissions.
- Application deployment or operating-system code execution was not tested.

**Overall Severity: High**

The finding is not rated Critical because full server compromise or operating-system command execution was intentionally not validated.

---

# 9. Security Observation Priorities

## O-001 — Anonymous FTP Access with Plaintext Transport

**Priority: Low**

Reasons:

- Anonymous authentication was confirmed.
- FTP traffic was transmitted without encryption.
- The anonymous directory contained no meaningful exposed files.
- File upload was denied.
- No sensitive information exposure or remote modification was demonstrated.

The condition represents unnecessary exposure and insecure service configuration but had limited demonstrated impact.

---

## O-002 — Legacy SMBv1 Protocol Enabled

**Priority: Medium**

Reasons:

- SMBv1 support was directly confirmed.
- SMBv1 is a legacy protocol that increases attack surface and should be removed where unnecessary.
- No SMBv1-specific vulnerability or CVE was validated.
- No compromise was demonstrated through SMBv1 itself.

The observation is therefore prioritized for remediation without being misrepresented as a confirmed SMBv1 exploit.

---

# 10. Risk Rating Principles

The following principles were applied throughout the engagement:

1. **Open ports are attack-surface indicators, not vulnerabilities by themselves.**
2. **Service versions are enumeration evidence, not proof of exploitability.**
3. **Automated scanner output requires analyst review.**
4. **Manual validation provides stronger evidence than tool labels.**
5. **Demonstrated impact and hypothetical impact must be clearly separated.**
6. **Severity should not be inflated to make a finding appear more impressive.**
7. **The minimum safe validation necessary to establish impact is preferred.**
8. **Uncertainty should be clearly documented rather than hidden behind a numerical score.**
9. **Risk ratings should support remediation prioritization, not merely technical description.**
10. **Findings must remain defensible when reviewed independently.**

---

# 11. Remediation Prioritization

Based on the assessment results, remediation should generally be prioritized in the following order:

### Immediate

**F-001 — Critical**

Remove the unrestricted NFS root export, restrict authorized clients, remove `no_root_squash`, and apply least-privilege filesystem permissions.

---

### High Priority

**F-003 — High**

Replace weak Tomcat Manager credentials, restrict administrative-interface exposure, review administrative roles, and disable the Manager interface if it is not operationally required.

---

### Medium Priority

**F-002 — Medium**

Disable anonymous SMB write access, require authentication, restrict share permissions, and limit SMB exposure to trusted systems.

**O-002 — Medium**

Disable SMBv1 where it is not required and migrate supported systems to modern SMB versions.

---

### Lower Priority

**O-001 — Low**

Disable unnecessary anonymous FTP access and replace plaintext FTP with an encrypted file-transfer mechanism where appropriate.

---

# 12. Methodology Limitation

Risk ratings in this project apply specifically to the **simulated environment and evidence collected during this assessment**.

They should not be interpreted as universal severity ratings for every deployment of the same technologies.

Risk in a production environment would also depend on factors such as:

- Network exposure
- Business criticality
- Data sensitivity
- Existing compensating controls
- User population
- Authentication architecture
- Monitoring and detection capabilities
- Patch state
- Service dependencies
- Regulatory requirements

The ratings documented here represent the assessed conditions within the defined lab scope.
