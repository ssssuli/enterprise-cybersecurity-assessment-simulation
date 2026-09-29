# Client Scenario

## Organization

**Asteria Solutions Sdn. Bhd.**

Asteria Solutions Sdn. Bhd. is a **fictional Malaysian small-to-medium enterprise (SME)** created solely for this simulated cybersecurity assessment.

The organization is assumed to provide digital and technology-enabled services supported by a small internal IT environment.

No real organization, customer, employee, or production infrastructure is represented by this scenario.

---

## Business Context

Asteria Solutions relies on a Linux-based server to provide a range of internal network and application services.

For the purpose of this assessment, the simulated enterprise server hosts services associated with:

- File sharing
- Remote administration
- Web applications
- Application management
- File transfer
- Database services
- Supporting network services

Because these services may process or provide access to business systems and information, weaknesses in their configuration could expose the organization to unauthorized access, data modification, service disruption, or application compromise.

---

## Assessment Requirement

Management requested a security assessment of the simulated server environment to identify security weaknesses that could affect the confidentiality, integrity, or availability of organizational systems and information.

The assessment was required to:

- Identify exposed network services and applications.
- Evaluate the server's externally reachable attack surface from the assessment network.
- Identify insecure service configurations and access-control weaknesses.
- Manually validate significant findings where safe and appropriate.
- Determine the potential technical and business impact of confirmed weaknesses.
- Assign appropriate risk levels to validated findings.
- Recommend practical remediation actions.
- Produce clear technical and management-level security documentation.

---

## Primary Security Concerns

The engagement focused on identifying weaknesses that could contribute to:

- Unauthorized access to systems or administrative interfaces
- Unauthorized access to shared files or server resources
- Unauthorized modification of server-side data
- Exposure of sensitive system or configuration information
- Compromise of hosted applications or services
- Service disruption resulting from insecure configurations
- Potential compromise of the affected server

Potential impact was documented separately from actions directly demonstrated during testing.

---

## Engagement Approach

The assessment was conducted from the perspective of an attacker who had network access to the simulated enterprise environment but did not initially possess authenticated access to the target server.

Testing followed a structured process consisting of:

`Scoping → Reconnaissance → Service Enumeration → Vulnerability Analysis → Manual Validation → Risk Assessment → Remediation → Reporting`

Potential vulnerabilities were not treated as confirmed findings solely because a port was open, a product version was identified, or an automated tool produced an alert.

Significant weaknesses were manually investigated to determine whether the observed condition could be validated within the defined Rules of Engagement.

---

## Simulated Environment

The engagement used the following systems:

| Asset | Role |
|---|---|
| Kali Linux | Security assessment workstation |
| Metasploitable 2 | Simulated Asteria Solutions enterprise server |

The systems operated within an isolated VMware host-only network specifically configured for authorized cybersecurity testing.

Detailed technical information is documented in [`environment.md`](environment.md).

---

## Engagement Outcome

The assessment identified multiple confirmed security weaknesses affecting file-sharing services and an exposed administrative application interface.

Additional lower-priority security observations were also documented where insecure conditions were confirmed but the available evidence did not justify classification as a primary vulnerability finding.

All testing was performed using a minimum-impact validation approach, and unnecessary destructive or post-compromise activity was intentionally avoided.

Detailed results are documented within the `findings/`, `observations/`, and `report/` sections of this repository.

---

## Disclaimer

**Asteria Solutions Sdn. Bhd. is entirely fictional.**

This scenario exists only to provide realistic business context for an educational cybersecurity portfolio project conducted in a controlled and intentionally vulnerable lab environment.
