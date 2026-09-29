# O-002 — Legacy SMBv1 Protocol Enabled

## Classification

**Security Observation**

## Priority

**Medium**

---

## Status

**Confirmed**

SMB protocol enumeration confirmed that the target server supported the legacy SMBv1 protocol using the `NT LM 0.12` dialect.

No SMBv1-specific vulnerability or CVE was validated during the assessment.

This observation therefore documents the confirmed use of a legacy protocol and does not claim that a specific SMB exploit was successful or applicable.

---

## Affected Asset

| Attribute | Details |
|---|---|
| Asset | Simulated Enterprise Server |
| Asset ID | TG-01 |
| IP Address | `192.168.247.129` |
| Service | SMB / Samba |
| Ports | TCP/139, TCP/445 |
| Observed Protocol | SMBv1 / `NT LM 0.12` |
| Status | Confirmed |

---

## Description

The target SMB service accepted the SMBv1 protocol.

Protocol enumeration identified the accepted dialect as:

```text
NT LM 0.12 (SMBv1)
```

SMBv1 is a legacy version of the Server Message Block protocol.

Retaining unnecessary legacy protocols increases the exposed attack surface and may require continued compatibility with older protocol behavior.

This observation is separate from:

**F-002 — Anonymous Read/Write Access to SMB Temporary Share**

F-002 concerns an independently validated access-control weakness.

O-002 concerns the continued availability of a legacy SMB protocol.

The two issues should therefore remain separately documented.

---

## Evidence

### SMB Service Exposure

The target exposed SMB-related services on:

```text
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
```

Supporting evidence:

```text
evidence/enumeration/smb-enumeration.txt
```

---

### SMB Protocol Enumeration

Protocol enumeration was performed using the Nmap `smb-protocols` script.

The assessment output returned:

```text
smb-protocols:
  dialects:
    NT LM 0.12 (SMBv1) [dangerous, but default]
```

This confirms that the target accepted the SMBv1 `NT LM 0.12` dialect.

Supporting evidence:

```text
evidence/enumeration/smb-enumeration.txt
```

---

## Validation Boundary

The purpose of this observation was to determine whether SMBv1 was enabled.

That objective was satisfied once protocol enumeration confirmed:

```text
NT LM 0.12 (SMBv1)
```

The assessment did **not** proceed to:

- Associate the service with a specific SMBv1 CVE without validation
- Attempt an SMBv1-specific exploit
- Execute remote code through SMBv1
- Establish a reverse shell
- Demonstrate privilege escalation
- Claim host compromise through the protocol

Those conclusions would require additional evidence beyond confirming that SMBv1 was enabled.

---

## Security Impact

The use of SMBv1 increases security exposure by retaining support for a legacy protocol.

Potential concerns include:

- Increased attack surface
- Continued reliance on legacy protocol behavior
- Reduced benefit from security improvements available in newer SMB versions
- Compatibility requirements that may prevent full service hardening

However, this assessment did not demonstrate:

- SMBv1-specific code execution
- SMBv1-specific privilege escalation
- Exploitation of a known CVE
- Host compromise attributable to SMBv1

The demonstrated condition is therefore limited to **legacy protocol exposure**.

---

## Priority Analysis

### Likelihood — Medium

The likelihood is assessed as **Medium** because:

- SMB was directly reachable from the assessment network.
- SMBv1 support was directly confirmed.
- A network client could negotiate the legacy protocol.

However:

- A specific SMBv1 vulnerability was not validated.
- Exploitability depends on additional factors not established by this observation.

---

### Impact — Medium

The impact is assessed as **Medium** because retaining SMBv1 increases attack surface and may expose the system to weaknesses associated with legacy SMB implementations.

However:

- No protocol-specific compromise was demonstrated.
- No code execution was demonstrated.
- No privilege escalation was demonstrated.
- No specific CVE was validated.

---

### Overall Priority — Medium

**Medium**

The issue should be remediated as part of system hardening, but the available evidence does not justify treating SMBv1 support itself as a confirmed high-severity vulnerability.

See:

[`../documentation/risk-methodology.md`](../documentation/risk-methodology.md)

---

## Relationship to F-002

This observation should not be confused with:

[`../findings/F002-anonymous-smb-read-write-access.md`](../findings/F002-anonymous-smb-read-write-access.md)

The two issues represent different security conditions.

### F-002

```text
Anonymous SMB access
        ↓
Writable tmp share
        ↓
Remote file creation confirmed
```

This represents a directly validated access-control weakness.

### O-002

```text
SMB service reachable
        ↓
Protocol enumeration
        ↓
SMBv1 accepted
```

This represents legacy protocol exposure.

Maintaining this distinction prevents the SMBv1 observation from being incorrectly presented as evidence supporting the anonymous-write vulnerability or vice versa.

---

## Recommendation

Recommended remediation actions include:

1. Disable SMBv1 where it is not explicitly required.
2. Configure the SMB service to permit only supported modern SMB protocol versions.
3. Identify legacy systems or applications that depend on SMBv1 before disabling compatibility.
4. Upgrade or replace systems that cannot operate using newer SMB versions.
5. Keep Samba and the underlying operating system appropriately maintained.
6. Restrict SMB access to trusted systems and network segments.
7. Review enabled SMB protocol versions during regular system-hardening assessments.
8. Remove SMB services entirely where network file sharing is not required.

---

## Remediation Validation

Following remediation, repeat protocol enumeration:

```bash
nmap -p139,445 --script smb-protocols 192.168.247.129
```

The resulting output should no longer contain:

```text
NT LM 0.12 (SMBv1)
```

Testing should also confirm that legitimate SMB functionality remains available through the approved protocol versions.

---

## Evidence References

- `evidence/enumeration/smb-enumeration.txt`

---

## Assessment Note

The evidence supports the following statement:

```text
SMBv1 support was confirmed on the target.
```

It does **not** support statements such as:

```text
SMBv1 remote code execution was confirmed.
```

or:

```text
A known SMBv1 CVE was successfully exploited.
```

A separate vulnerability assessment and controlled validation process would be required before making those claims.

---

## Assessment Conclusion

The assessment confirmed that the target SMB service accepted the legacy SMBv1 `NT LM 0.12` dialect.

Because no SMBv1-specific vulnerability, CVE, code execution, or host compromise was demonstrated, the condition is documented as a **Medium-priority security observation** rather than a confirmed vulnerability finding.

This classification accurately reflects the evidence collected during the assessment without overstating the security impact.
