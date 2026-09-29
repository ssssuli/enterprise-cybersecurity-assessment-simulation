# O-001 — Anonymous FTP Access with Plaintext Transport

## Classification

**Security Observation**

## Priority

**Low**

---

## Status

**Confirmed**

Manual testing confirmed that the FTP service permitted anonymous authentication without requiring a valid local user account.

The anonymous user could access and enumerate the assigned FTP root directory.

However:

- No meaningful files or subdirectories were exposed.
- No sensitive information was identified.
- A controlled upload attempt was rejected.
- No server-side write capability was demonstrated.

The issue is therefore documented as a **Low-priority security observation** rather than a primary vulnerability finding.

---

## Affected Asset

| Attribute | Details |
|---|---|
| Asset | Simulated Enterprise Server |
| Asset ID | TG-01 |
| IP Address | `192.168.247.129` |
| Service | FTP |
| Port | TCP/21 |
| Product | vsFTPd 2.3.4 |
| Authentication | Anonymous Access Permitted |
| Status | Confirmed |

---

## Description

The target server exposed an FTP service on TCP/21 that permitted anonymous authentication.

Initial enumeration identified:

```text
Anonymous FTP login allowed (FTP code 230)
```

The FTP server status also reported:

```text
Control connection is plain text
Data connections will be plain text
```

Manual validation subsequently confirmed that the anonymous account could successfully authenticate and access the FTP root directory.

The exposed directory did not contain meaningful files or additional subdirectories.

A controlled upload test was then performed to determine whether anonymous users could create content on the target.

The server denied the upload.

---

## Evidence

### FTP Service Enumeration

Nmap FTP enumeration was performed against TCP/21.

The scan identified:

```text
21/tcp open ftp
```

and reported:

```text
Anonymous FTP login allowed (FTP code 230)
```

The server identified itself as:

```text
vsFTPd 2.3.4
```

Supporting evidence:

```text
evidence/enumeration/ftp-enumeration.txt
```

---

### Plaintext Transport

FTP service enumeration reported:

```text
Control connection is plain text
Data connections will be plain text
```

This indicates that standard FTP was being used without transport encryption.

As a result, FTP authentication and transferred content would not receive confidentiality protection from the protocol itself.

The practical risk of interception depends on an attacker's ability to observe the network path.

Supporting evidence:

```text
evidence/enumeration/ftp-enumeration.txt
```

---

### Anonymous Authentication Validation

Manual testing connected to the FTP service:

```bash
ftp 192.168.247.129
```

The username:

```text
anonymous
```

was supplied.

The server returned:

```text
230 Login successful.
```

This confirmed that no valid local user account was required to establish an FTP session.

Supporting evidence:

```text
evidence/validation/ftp-anonymous-access.txt
```

---

### Directory Enumeration

After successful anonymous authentication, the current remote directory was:

```text
/
```

Directory enumeration returned only:

```text
.
..
```

No accessible files or additional directories of interest were identified within the anonymous FTP root.

This significantly limited the demonstrated security impact of the anonymous access.

---

### Controlled Write Validation

A harmless local file was created:

```bash
printf 'Controlled FTP write validation - authorized lab only\n' > ftp-write-validation.txt
```

The file was then submitted using:

```text
put ftp-write-validation.txt
```

The FTP server returned:

```text
553 Could not create file.
```

This demonstrated that the anonymous account was **not permitted to upload files** into the exposed FTP root directory.

No remote validation artifact was created.

Supporting evidence:

```text
evidence/validation/ftp-write-validation.txt
```

---

## Validation Boundary

The objective of testing was to determine:

1. Whether anonymous FTP authentication was permitted.
2. What content was exposed to the anonymous account.
3. Whether anonymous users could modify the FTP directory.

Testing established that:

```text
Anonymous authentication     → Confirmed
Directory access              → Confirmed
Meaningful data exposure      → Not identified
Anonymous file upload         → Denied
Remote write capability       → Not demonstrated
```

No further FTP testing was required to classify the condition accurately.

---

## Security Impact

Anonymous FTP access allows unauthenticated network users to establish sessions with the FTP service.

In the assessed configuration, the demonstrated impact was limited because:

- The anonymous directory contained no meaningful files.
- No sensitive information was identified.
- No additional useful directories were exposed.
- Anonymous uploads were denied.
- No remote file modification was demonstrated.

The use of plaintext FTP also means the protocol does not provide encryption for authentication or file-transfer traffic.

However, because the assessment did not identify sensitive anonymous content or writable FTP resources, the issue does not justify the same treatment as the confirmed vulnerability findings.

---

## Priority Analysis

### Likelihood — High

Anonymous access itself is straightforward because:

- TCP/21 was reachable from the assessment network.
- No valid account credentials were required.
- Manual authentication using the anonymous account succeeded.

---

### Impact — Low

The demonstrated impact is **Low** because:

- No sensitive content was exposed.
- No meaningful files were available.
- File uploads were denied.
- No modification capability was demonstrated.
- No application or system compromise resulted from the anonymous access.

---

### Overall Priority — Low

**Low**

The condition is retained as a security observation because it unnecessarily increases the exposed attack surface and uses plaintext transport, but the evidence demonstrates limited practical impact in the assessed configuration.

See:

[`../documentation/risk-methodology.md`](../documentation/risk-methodology.md)

---

## Recommendation

Recommended remediation actions include:

1. Disable anonymous FTP access unless there is an explicitly documented business requirement.
2. Require authenticated access for file-transfer services where appropriate.
3. Replace plaintext FTP with an encrypted alternative such as SFTP or FTPS.
4. Restrict file-transfer services to trusted systems or network segments.
5. Review FTP directory permissions to ensure anonymous users cannot access sensitive content.
6. Continue to prevent anonymous write access.
7. Remove the FTP service entirely where it is not operationally required.
8. Monitor file-transfer services for unexpected authentication or access activity.

---

## Remediation Validation

Following remediation, anonymous authentication should be tested again.

Attempt:

```bash
ftp 192.168.247.129
```

and provide:

```text
anonymous
```

Where anonymous access is no longer required, authentication should be rejected.

If FTP functionality remains operationally necessary, validation should also confirm that:

- Only approved users can authenticate.
- Sensitive directories cannot be accessed anonymously.
- Anonymous write permissions remain disabled.
- File-transfer traffic uses an encrypted protocol where possible.
- Network access is restricted to approved systems.

---

## Evidence References

- `evidence/enumeration/ftp-enumeration.txt`
- `evidence/validation/ftp-anonymous-access.txt`
- `evidence/validation/ftp-write-validation.txt`
- `evidence/reconnaissance/service-version-scan.txt`

---

## Assessment Conclusion

The assessment confirmed that the FTP service permitted anonymous authentication and used plaintext control and data connections.

Manual testing demonstrated access to the anonymous FTP root directory, but no meaningful content was exposed and a controlled file-upload attempt was denied.

Because the condition increased attack surface without demonstrating significant confidentiality, integrity, or availability impact, it is classified as a **Low-priority security observation**.
