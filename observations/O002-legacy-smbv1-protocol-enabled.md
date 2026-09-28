# O-002 — Legacy SMBv1 Protocol Enabled

## Classification

**Security Observation**

## Priority

**Medium**

---

## Status

**Confirmed**

SMB protocol enumeration confirmed that the target server supports the legacy SMBv1 protocol using the `NT LM 0.12` dialect.

No SMBv1-specific exploit was attempted or validated during this assessment. This observation is therefore limited to the confirmed use of a deprecated protocol and should not be interpreted as evidence that a specific SMB vulnerability or CVE is exploitable.

---

## Affected Asset

| Attribute | Details |
|---|---|
| Asset | Simulated Enterprise Server |
| IP Address | 192.168.247.129 |
| Service | SMB / Samba |
| Ports | TCP/139, TCP/445 |
| Observed Protocol | SMBv1 / NT LM 0.12 |

---

## Description

The target SMB service supports SMBv1, identified during protocol enumeration as the `NT LM 0.12` dialect.

SMBv1 is a legacy version of the Server Message Block protocol. Modern implementations have deprecated or disabled SMBv1 by default because newer SMB versions provide stronger security capabilities and reduce exposure associated with legacy protocol behavior.

The presence of SMBv1 increases the attack surface of the SMB service and may require older, weaker protocol behavior for compatibility.

This observation is separate from the anonymously writable SMB share documented in F-002. The writable share represents an access-control weakness, whereas this observation concerns the use of a legacy network protocol.

---

## Evidence

### SMB Service Exposure

The target exposed SMB-related services on:

```text
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
```

---

### Protocol Enumeration

SMB protocol enumeration returned:

```text
smb-protocols:
  dialects:
    NT LM 0.12 (SMBv1) [dangerous, but default]
```

This confirms that the target accepts the SMBv1 `NT LM 0.12` dialect.

---

## Security Impact

The use of SMBv1 increases security risk because it relies on a legacy protocol that has been deprecated by modern Samba and Microsoft implementations.

Potential security concerns include:

- Increased exposure to vulnerabilities that affect legacy SMB implementations
- Reduced availability of security protections provided by newer SMB protocol versions
- Continued dependence on outdated client or server compatibility
- Increased attack surface on systems where SMBv1 is not operationally required

The assessment did **not** demonstrate exploitation of a specific SMBv1 vulnerability.

Accordingly, this issue is documented as a security observation rather than being presented as proof of system compromise.

---

## Risk Analysis

### Likelihood

**Medium**

The SMB service is reachable from the assessment network and SMBv1 support was directly confirmed. However, no SMBv1-specific exploitability was tested as part of this observation.

### Impact

**Medium**

The potential impact depends on the specific Samba implementation, patch state, configuration, and whether a relevant SMBv1 weakness is present.

No direct compromise was demonstrated through SMBv1 during this assessment.

### Overall Priority

**Medium**

The protocol should be retired where it is not explicitly required, but the evidence collected does not support assigning the same severity as the confirmed anonymous read/write SMB share.

---

## Recommendation

Recommended actions include:

1. Disable SMBv1 support where it is not explicitly required.
2. Configure the SMB service to require SMBv2 or SMBv3 where supported.
3. Identify any legacy systems or applications that still depend on SMBv1 before disabling the protocol.
4. Upgrade or replace systems that cannot operate using modern SMB versions.
5. Keep Samba and the underlying operating system fully patched.
6. Restrict SMB access to trusted network segments using firewall rules or network segmentation.
7. Periodically review enabled SMB protocol versions as part of system-hardening activities.

For Samba deployments, modern versions can be configured to require SMB2 or later through the server minimum protocol configuration where appropriate.

---

## Remediation Validation

Following remediation, repeat SMB protocol enumeration:

```bash
nmap -p139,445 --script smb-protocols 192.168.247.129
```

The resulting output should no longer list:

```text
NT LM 0.12 (SMBv1)
```

Testing should also confirm that legitimate clients can continue to access required SMB resources using a supported modern protocol.

---

## Assessment Note

This observation should not be described as:

```text
"SMBv1 exploit confirmed"
```

or:

```text
"Known SMBv1 CVE successfully exploited"
```

The assessment only confirmed that SMBv1 is enabled.

A specific vulnerability would require separate version analysis, applicability validation, and controlled testing before being reported as a confirmed vulnerability.

---

## Evidence References

- `evidence/enumeration/smb-enumeration.txt`

---

## External Guidance

- Samba 4.11 release notes document SMB1 as deprecated and disabled by default in newer Samba configurations.
- Microsoft security guidance recommends removing or disabling SMBv1 where it is not required and migrating to SMBv2 or later.
