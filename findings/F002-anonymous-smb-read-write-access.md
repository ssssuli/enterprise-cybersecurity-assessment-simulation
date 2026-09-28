# F-002 — Anonymous Read/Write Access to SMB Temporary Share

## Severity

**High**

---

## Status

**Confirmed**

Anonymous access to the SMB `tmp` share was successfully validated.

Testing confirmed that an unauthenticated user could connect to the share, enumerate its contents, upload a file, verify that the file was present on the remote share, and remove the test artifact.

---

## Affected Asset

| Attribute | Details |
|---|---|
| Asset | Simulated Enterprise Server |
| IP Address | 192.168.247.129 |
| Service | SMB / Samba |
| Ports | TCP/139, TCP/445 |
| Share | `tmp` |

---

## Description

The target server exposes an SMB share named `tmp` that permits anonymous access.

Automated enumeration identified the share as accessible with anonymous read/write permissions.

Manual validation confirmed that an unauthenticated client could connect to the share without supplying credentials, enumerate its contents, and upload a file.

The share exposes the target server's temporary-file area, increasing the risk that unauthorized users could place or modify content on the system.

---

## Evidence

### Anonymous Share Enumeration

Anonymous SMB enumeration identified multiple shares, including:

```text
print$
tmp
opt
IPC$
ADMIN$
```

The `tmp` share was identified as anonymously accessible.

---

### Anonymous Access Validation

The share was accessed without providing credentials:

```bash
smbclient //192.168.247.129/tmp -N
```

The connection succeeded and returned:

```text
Anonymous login successful
Current directory is \\192.168.247.129\tmp\
```

Directory enumeration exposed files and runtime artifacts within the remote temporary directory.

---

### Controlled Write Validation

A harmless local test file was created:

```bash
printf 'Controlled SMB write validation - authorized lab only\n' > smb-write-validation.txt
```

The file was uploaded to the share using:

```text
put smb-write-validation.txt
```

The SMB client confirmed successful upload:

```text
putting file smb-write-validation.txt as \smb-write-validation.txt
```

Subsequent directory enumeration confirmed that the file existed on the remote share:

```text
smb-write-validation.txt
```

This demonstrated that an anonymous network user could modify content within the SMB share.

The test file was then deleted from the remote share and a follow-up directory listing confirmed successful cleanup.

No production or sensitive system files were modified during validation.

---

## Security Impact

Anonymous read/write access to a network file share allows unauthenticated users to interact with server-side content.

Potential consequences include:

- Unauthorized file placement
- Modification or deletion of shared content
- Use of the share as a staging location for malicious files
- Tampering with temporary or application-related data
- Increased opportunity for lateral movement where other systems or services consume content from the share
- Potential service disruption where applications rely on files stored within the exposed directory

The assessment did not demonstrate code execution or compromise through the SMB share. For this reason, the issue is rated **High** rather than Critical.

---

## Risk Analysis

### Likelihood

**High**

The SMB service is directly reachable from the assessment network, the share can be accessed without credentials, and write capability was successfully validated.

### Impact

**High**

An attacker could place or modify files within the exposed server-side directory. The ultimate impact depends on how other services or applications interact with the contents of the share.

### Overall Risk

**High**

Unauthenticated read/write access represents a significant access-control failure and provides an attacker with the ability to modify remote server-side content.

---

## Remediation

Recommended remediation actions include:

1. Disable anonymous or guest access to SMB shares unless there is a documented business requirement.
2. Require authenticated user access for all writable shares.
3. Apply least-privilege permissions to SMB resources.
4. Remove write access from users who do not explicitly require it.
5. Restrict SMB access to trusted network segments using firewall rules or network segmentation.
6. Review existing Samba share definitions for overly permissive guest-access settings.
7. Monitor writable network shares for unauthorized file creation or modification.
8. Ensure application or service directories are not exposed through unauthenticated writable shares.

---

## Remediation Validation

Following remediation, validation should confirm that:

- Anonymous users cannot connect to the `tmp` share.
- Unauthenticated share enumeration is restricted where appropriate.
- Unauthorized users cannot upload, modify, or delete files.
- Only approved users retain required access.
- Share permissions follow the principle of least privilege.

Validation may include:

```bash
smbclient -L //192.168.247.129 -N
```

and:

```bash
smbclient //192.168.247.129/tmp -N
```

Unauthenticated access should be denied following remediation.

---

## Evidence References

- `evidence/enumeration/smbclient-share-list.txt`
- `evidence/enumeration/smb-enumeration.txt`
- `evidence/validation/smb-anonymous-access.txt`
- `evidence/validation/smb-write-validation-session.txt`
