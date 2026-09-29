# F-001 — Unrestricted NFS Root Filesystem Export with Root Privilege Preservation

## Severity

**Critical**

---

## Status

**Confirmed**

Manual testing confirmed that the target server exported its entire root filesystem through NFS and allowed network clients to access the export without application-level authentication.

Controlled validation further demonstrated that a privileged remote client could create a **root-owned file on the target filesystem**, confirming that remote root identity was preserved through the export.

---

## Affected Asset

| Attribute | Details |
|---|---|
| Asset | Simulated Enterprise Server |
| Asset ID | TG-01 |
| IP Address | `192.168.247.129` |
| Service | Network File System (NFS) |
| Port | TCP/2049 |
| Status | Confirmed |

---

## Description

The target server exposed its entire root filesystem (`/`) through the Network File System service.

Enumeration showed that the root filesystem was exported broadly, while inspection of the server's export configuration confirmed:

```text
/ *(rw,sync,no_root_squash,no_subtree_check)
```

This configuration creates several significant security weaknesses:

- `/` exposes the server's entire root filesystem.
- `*` allows any client permitted to reach the NFS service to request the export.
- `rw` provides read and write capability.
- `no_root_squash` disables the normal restriction that maps remote root users to a less-privileged identity.

As a result, a network client with local root privileges can interact with the exported filesystem while retaining root-level identity on the target.

This provides a direct path to privileged filesystem modification.

---

## Evidence

### NFS Export Enumeration

NFS export enumeration identified:

```text
Export list for 192.168.247.129:

/ *
```

This demonstrated that the server's root filesystem was available as an NFS export.

Supporting evidence:

```text
evidence/enumeration/nfs-exports.txt
```

---

### Root Filesystem Access

The export was initially mounted read-only to inspect its contents without modifying the target:

```bash
sudo mount -t nfs -o ro 192.168.247.129:/ /mnt/asteria-nfs
```

The mounted filesystem exposed operating-system directories including:

```text
/bin
/boot
/etc
/home
/root
/usr
/var
```

This confirmed access to the target's root filesystem structure.

The exposed `/home` directory also revealed multiple local user directories.

The `/etc` directory contained operating-system and service configuration files.

Supporting evidence:

```text
evidence/validation/nfs-root-listing.txt
evidence/validation/nfs-home-listing.txt
evidence/validation/nfs-etc-listing.txt
```

---

### NFS Export Configuration

Inspection of the accessible NFS export configuration identified:

```text
/ *(rw,sync,no_root_squash,no_subtree_check)
```

The security-relevant options were:

| Option | Security Relevance |
|---|---|
| `/` | Exports the entire root filesystem |
| `*` | Broadly permits clients that can reach the service |
| `rw` | Allows filesystem modification |
| `no_root_squash` | Preserves remote root identity |
| `sync` | Controls write synchronization; not itself the security weakness |

Supporting evidence:

```text
evidence/validation/nfs-export-config.txt
```

---

### Controlled Write Validation

After confirming filesystem exposure, the export was remounted with write access for controlled validation:

```bash
sudo umount /mnt/asteria-nfs
sudo mount -t nfs -o rw 192.168.247.129:/ /mnt/asteria-nfs
```

A harmless temporary file was then created inside the target's `/tmp` directory:

```bash
sudo touch /mnt/asteria-nfs/tmp/nfs-security-validation.txt
```

The resulting file was:

```text
-rw-r--r-- 1 root root 0 Sep 27 2026 /mnt/asteria-nfs/tmp/nfs-security-validation.txt
```

The ownership:

```text
root root
```

confirmed that root identity from the assessment workstation was preserved when writing to the remote filesystem.

This demonstrated practical exploitation of the `rw` and `no_root_squash` configuration without altering security-sensitive files.

Supporting evidence:

```text
evidence/validation/nfs-write-validation.txt
```

---

## Validation Boundary

The objective of validation was to determine whether the NFS configuration permitted privileged remote filesystem modification.

That objective was satisfied once the assessment confirmed:

1. The root filesystem was exported.
2. The export was accessible from the assessment workstation.
3. The export permitted write operations.
4. `no_root_squash` was enabled.
5. A remote root user could create a root-owned file on the target.

Further exploitation was unnecessary.

The assessment therefore did **not**:

- Modify `/etc/passwd`
- Modify `/etc/shadow`
- Add SSH authorized keys
- Create new privileged users
- Modify scheduled tasks
- Modify startup configuration
- Replace system binaries
- Establish persistence
- Execute malicious code through the exported filesystem

The temporary validation file was removed after testing.

---

## Security Impact

The confirmed NFS configuration provides a network-accessible client with the ability to modify target filesystem content while retaining root-level identity.

Potential consequences include:

- Modification of operating-system configuration
- Modification of authentication-related files
- Unauthorized creation or alteration of user accounts
- Modification of SSH configuration or authorized keys
- Modification of scheduled tasks or startup configuration
- Application or web-content tampering
- Service configuration modification
- Exposure of sensitive configuration or credential material
- Establishment of persistence
- Potential full compromise of the affected server

These downstream actions were **not performed during the assessment**.

They represent potential consequences reasonably supported by the privileged filesystem access that was directly demonstrated.

---

## Risk Analysis

### Likelihood — High

The likelihood of abuse is assessed as **High** because:

- NFS was directly reachable from the assessment network.
- The root filesystem could be mounted without application-level authentication.
- The export was broadly available using `*`.
- Write permissions were enabled.
- Exploitation required minimal complexity once the service was identified.
- Manual testing successfully reproduced the insecure behavior.

---

### Impact — Critical

The impact is assessed as **Critical** because:

- The target's entire root filesystem was exposed.
- Remote write capability was confirmed.
- Remote root identity was preserved.
- A root-owned file was successfully created through the NFS export.
- The confirmed privilege level could permit modification of security-critical operating-system files.

---

### Overall Severity — Critical

**Critical**

The combination of **High likelihood** and **Critical impact** results in a **Critical overall severity** under the assessment's qualitative risk methodology.

The rating is based on demonstrated privileged filesystem modification rather than merely on the presence of NFS or a potentially insecure configuration.

See:

[`../documentation/risk-methodology.md`](../documentation/risk-methodology.md)

---

## Remediation

The server's root filesystem should not be exported broadly through NFS.

Recommended remediation actions are:

1. Remove `/` from the NFS export configuration.
2. Export only directories required for legitimate business functionality.
3. Restrict exports to explicitly authorized hosts or trusted network segments instead of using `*`.
4. Remove `no_root_squash` unless there is a specific and documented administrative requirement.
5. Use the default `root_squash` behavior to prevent remote root users from retaining equivalent server-side privileges.
6. Configure exports as read-only where write access is unnecessary.
7. Apply firewall controls to restrict access to NFS services.
8. Segment NFS services from general user or untrusted network segments.
9. Review all configured NFS exports for excessive permissions.
10. Monitor NFS configuration and sensitive exported directories for unauthorized changes.
11. Apply the principle of least privilege to all network file-sharing services.

---

## Remediation Validation

After remediation, validation should confirm that the insecure configuration is no longer present.

### Export Review

Run:

```bash
showmount -e 192.168.247.129
```

The root filesystem:

```text
/
```

should no longer be broadly exported.

---

### Unauthorized Mount Test

A client outside the approved access list should be unable to mount restricted exports.

Where legitimate NFS access remains required, only explicitly authorized systems should be permitted.

---

### Privilege Validation

A remote privileged user should no longer retain unrestricted root identity on the target.

Controlled attempts to create files should either:

- Be denied, or
- Be mapped to an appropriately restricted server-side identity through root squashing.

---

### Permission Validation

Remaining exports should provide only the minimum permissions required for legitimate operation.

Read-only access should be used wherever write access is unnecessary.

---

## Evidence References

- `evidence/enumeration/nfs-exports.txt`
- `evidence/validation/nfs-root-listing.txt`
- `evidence/validation/nfs-etc-listing.txt`
- `evidence/validation/nfs-home-listing.txt`
- `evidence/validation/nfs-export-config.txt`
- `evidence/validation/nfs-write-validation.txt`

---

## Assessment Conclusion

The assessment confirmed that the target exported its entire root filesystem through NFS using read/write permissions with `no_root_squash` enabled.

Manual validation demonstrated that a remote privileged client could create a **root-owned file on the target filesystem**.

This represents a direct privileged filesystem modification capability and provides sufficient evidence to classify the vulnerability as **Critical**.

Further modification of sensitive operating-system files was deliberately avoided because the controlled write test provided sufficient proof of impact.
