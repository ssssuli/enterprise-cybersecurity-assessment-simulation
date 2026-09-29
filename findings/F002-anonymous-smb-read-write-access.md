# F-002 — Anonymous Read/Write Access to SMB Temporary Share

## Severity

**Medium**

---

## Status

**Confirmed**

Manual testing confirmed that an unauthenticated network user could connect to the SMB `tmp` share, enumerate its contents, upload a controlled file, verify that the file existed on the remote share, and remove the assessment-created artifact.

No valid user credentials were required.

---

## Affected Asset

| Attribute | Details |
|---|---|
| Asset | Simulated Enterprise Server |
| Asset ID | TG-01 |
| IP Address | `192.168.247.129` |
| Service | SMB / Samba |
| Ports | TCP/139, TCP/445 |
| Share | `tmp` |
| Authentication | Anonymous / Guest Access |
| Status | Confirmed |

---

## Description

The target server exposed an SMB share named `tmp` that permitted access without valid user credentials.

Initial SMB enumeration identified the share and reported anonymous read/write permissions.

Manual validation subsequently confirmed that an unauthenticated client could:

- Connect to the `tmp` share
- Enumerate directory contents
- Upload a controlled test file
- Verify that the uploaded file existed remotely
- Delete the assessment-created file during cleanup

This represents an access-control weakness because an unauthenticated network user can place content onto a server-side network share.

The assessment did not establish that files placed within this share would be automatically executed, processed by a privileged application, or used to obtain additional system privileges.

---

## Evidence

### SMB Service Exposure

Initial enumeration identified SMB services on:

```text
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
```

Supporting evidence:

```text
evidence/enumeration/smb-enumeration.txt
```

---

### Anonymous Share Enumeration

Anonymous share enumeration was performed using:

```bash
smbclient -L //192.168.247.129 -N
```

The request completed without valid user credentials and returned:

```text
Anonymous login successful
```

Exposed shares included:

```text
print$
tmp
opt
IPC$
ADMIN$
```

The `tmp` share was therefore selected for deeper validation.

Supporting evidence:

```text
evidence/enumeration/smbclient-share-list.txt
```

---

### Anonymous Permission Enumeration

Additional SMB enumeration identified the `tmp` share as permitting anonymous read/write access.

The assessment output reported:

```text
\\192.168.247.129\tmp

Anonymous access: READ/WRITE
```

This result was treated as an enumeration finding and was subsequently validated manually rather than accepted as sufficient proof by itself.

Supporting evidence:

```text
evidence/enumeration/smb-enumeration.txt
```

---

### Manual Anonymous Access Validation

The share was accessed without supplying credentials:

```bash
smbclient //192.168.247.129/tmp -N
```

The connection returned:

```text
Anonymous login successful
```

The SMB client reported the current location as:

```text
\\192.168.247.129\tmp\
```

Directory enumeration was then performed successfully.

This confirmed that an unauthenticated network user could access and inspect the contents of the share.

Supporting evidence:

```text
evidence/validation/smb-anonymous-access.txt
```

---

### Controlled Write Validation

A harmless local validation file was created:

```bash
printf 'Controlled SMB write validation - authorized lab only\n' > smb-write-validation.txt
```

The assessment workstation then connected anonymously to the share:

```bash
smbclient //192.168.247.129/tmp -N
```

The file was uploaded using:

```text
put smb-write-validation.txt
```

The SMB client returned:

```text
putting file smb-write-validation.txt as \smb-write-validation.txt
```

A subsequent directory listing showed:

```text
smb-write-validation.txt
```

This confirmed that an unauthenticated network user could create content on the remote SMB share.

---

### Cleanup Validation

The assessment-created file was removed following validation.

A subsequent directory listing confirmed that:

```text
smb-write-validation.txt
```

was no longer present.

The deletion test was limited to the file created specifically for the assessment.

Existing server-side files were not intentionally altered or deleted.

Supporting evidence:

```text
evidence/validation/smb-write-validation-session.txt
```

---

## Validation Boundary

The purpose of manual validation was to determine whether anonymous SMB access provided practical write capability.

That objective was satisfied once the assessment established:

1. The SMB service was reachable.
2. Shares could be enumerated without credentials.
3. The `tmp` share could be accessed anonymously.
4. Directory contents could be enumerated.
5. A controlled file could be uploaded successfully.
6. The uploaded file was present on the remote share.
7. The assessment-created artifact could be removed.

Further exploitation was unnecessary.

The assessment therefore did **not** attempt to:

- Execute uploaded files
- Replace existing application content
- Modify privileged operating-system files
- Determine whether another service automatically processed uploaded content
- Escalate privileges
- Establish persistence
- Use the share to compromise another host
- Demonstrate lateral movement

---

## Security Impact

Anonymous write access allows an unauthenticated network user to place files onto a server-side SMB share.

The directly demonstrated impact includes:

- Unauthorized access to the exposed share
- Unauthorized directory enumeration
- Unauthorized remote file creation
- Storage of attacker-controlled content within the writable share

Depending on how a production system used such a share, additional consequences could potentially include:

- Tampering with shared data
- Storage of unauthorized or malicious content
- Consumption of attacker-controlled files by users or applications
- Loss of integrity for data stored within the affected share

These downstream consequences were **not demonstrated during this assessment**.

No evidence established that uploaded content would be executed by the server or processed by a privileged service.

---

## Risk Analysis

### Likelihood — High

The likelihood of abuse is assessed as **High** because:

- SMB was directly reachable from the assessment network.
- No valid credentials were required.
- Anonymous access to the `tmp` share was manually confirmed.
- File upload required minimal technical complexity.
- Remote file creation was successfully reproduced.

---

### Impact — Medium

The impact is assessed as **Medium** because:

- Unauthorized remote file placement was directly demonstrated.
- The weakness affects the integrity of the exposed share.
- An attacker can introduce arbitrary content into the writable location.

However:

- No privileged file modification was demonstrated.
- No code execution was demonstrated.
- No privilege escalation was demonstrated.
- No sensitive data compromise was established.
- No evidence showed that another application automatically consumed or executed uploaded files.
- No host compromise was demonstrated.

---

### Overall Severity — Medium

**Medium**

Under the assessment's qualitative risk methodology, the combination of **High likelihood** and **Medium impact** results in an overall **Medium severity**.

The rating reflects the access and modification capability actually demonstrated rather than assuming more serious downstream exploitation.

See:

[`../documentation/risk-methodology.md`](../documentation/risk-methodology.md)

---

## Remediation

Recommended remediation actions include:

1. Disable anonymous or guest access to the `tmp` SMB share unless there is an explicitly documented business requirement.
2. Require authenticated access for writable network shares.
3. Apply least-privilege permissions to all SMB resources.
4. Grant write permissions only to users or service accounts that require them.
5. Restrict SMB access to approved hosts and trusted network segments.
6. Review Samba share definitions for guest-access or permissive write configurations.
7. Remove unnecessary publicly or anonymously accessible shares.
8. Separate writable temporary storage from locations used by privileged applications.
9. Monitor SMB shares for unexpected file creation, modification, or deletion.
10. Periodically review share permissions and authorized users.

---

## Remediation Validation

Following remediation, repeat anonymous share enumeration:

```bash
smbclient -L //192.168.247.129 -N
```

Anonymous enumeration should be restricted where it is not explicitly required.

Attempt anonymous access to the affected share:

```bash
smbclient //192.168.247.129/tmp -N
```

Expected behavior:

```text
Access denied
```

or equivalent authentication failure.

If legitimate anonymous read access must remain for a documented requirement, write operations should still be denied.

A controlled upload attempt should fail.

Authenticated testing should also confirm that only approved users retain the permissions required for legitimate operations.

---

## Evidence References

- `evidence/enumeration/smbclient-share-list.txt`
- `evidence/enumeration/smb-enumeration.txt`
- `evidence/validation/smb-anonymous-access.txt`
- `evidence/validation/smb-write-validation-session.txt`

---

## Assessment Conclusion

The assessment confirmed that the SMB `tmp` share could be accessed without valid user credentials and permitted anonymous file creation.

Manual testing successfully demonstrated anonymous connection, directory enumeration, file upload, remote file verification, and cleanup of the assessment-created artifact.

This represents a confirmed access-control weakness affecting the integrity of the exposed SMB share.

Because testing did not demonstrate code execution, privileged file modification, sensitive-data compromise, or host compromise, the finding is classified as **Medium severity**.
