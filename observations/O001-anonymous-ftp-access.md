# O-001 — Anonymous FTP Access with Plaintext Transport

## Classification

**Security Observation**

## Priority

**Low**

---

## Status

**Confirmed**

Anonymous authentication to the FTP service was successfully validated.

Manual testing confirmed that an anonymous user could authenticate and enumerate the assigned FTP root directory. However, no files or subdirectories of interest were exposed, and an attempted file upload was denied.

---

## Affected Asset

| Attribute | Details |
|---|---|
| Asset | Simulated Enterprise Server |
| IP Address | 192.168.247.129 |
| Service | FTP |
| Port | TCP/21 |
| Product | vsFTPd 2.3.4 |

---

## Description

The FTP service permits anonymous authentication without requiring a valid local user account.

Automated enumeration confirmed that anonymous login was enabled and that FTP control and data connections were transmitted in plaintext.

Manual validation successfully authenticated using the anonymous account and confirmed access to the FTP root directory.

The exposed directory did not contain accessible files or subdirectories beyond the standard `.` and `..` entries.

A controlled upload test was also performed to determine whether the anonymous account could modify server-side content. The upload was denied by the FTP server.

---

## Evidence

### Anonymous Authentication

The FTP service accepted an anonymous login:

```text
220 (vsFTPd 2.3.4)
331 Please specify the password.
230 Login successful.
```

The authenticated session identified the remote directory as:

```text
/
```

Directory enumeration returned only:

```text
.
..
```

No files or additional directories were exposed through the anonymous FTP root.

---

### Plaintext FTP Transport

Service enumeration identified that both FTP control and data connections were transmitted without encryption.

Because standard FTP does not protect session data in transit, credentials and transferred content may be exposed to interception where an attacker is positioned to observe the network path.

---

### Controlled Write Validation

A harmless local test file was created for write-access validation:

```text
Controlled FTP write validation - authorized lab only
```

The file was then submitted using the FTP `put` command.

The server returned:

```text
553 Could not create file.
```

This confirmed that the anonymous account was not permitted to upload files into the exposed FTP root directory.

No server-side files were modified during the assessment.

---

## Security Impact

Anonymous FTP access increases the exposed attack surface by allowing unauthenticated users to establish sessions with the FTP service.

In the assessed configuration, the impact was limited because:

- No files or subdirectories of interest were exposed.
- No sensitive information was identified through anonymous access.
- Anonymous file uploads were denied.
- No write capability was demonstrated.

However, the FTP service uses plaintext transport, which provides no confidentiality for authentication or file-transfer traffic.

For these reasons, the issue was documented as a lower-priority security observation rather than a primary vulnerability finding.

---

## Recommendation

Recommended actions include:

1. Disable anonymous FTP access unless it is explicitly required for a documented business purpose.
2. Restrict FTP access to authorized users and trusted network segments.
3. Replace plaintext FTP with an encrypted alternative such as SFTP or FTPS where file-transfer functionality is required.
4. Review FTP permissions regularly to ensure anonymous users cannot access or modify sensitive data.
5. Apply firewall or network-access controls to limit exposure of TCP/21.

---

## Validation

Following remediation, validation should confirm that:

- Anonymous authentication is disabled where it is not required.
- Unauthorized users cannot enumerate FTP content.
- Anonymous write permissions remain disabled.
- File-transfer traffic uses an encrypted protocol where appropriate.

---

## Evidence References

- `evidence/validation/ftp-anonymous-access.txt`
- `evidence/validation/ftp-write-validation.txt`
- `evidence/reconnaissance/service-version-scan.txt`
