# F-001 — Unrestricted NFS Root Filesystem Export with Root Privilege Preservation

## Severity

**Critical**

---

## Status

**Confirmed**

Unauthenticated network access to the target's exported root filesystem was successfully demonstrated.

Controlled validation also confirmed that a privileged remote client could create a root-owned file on the target filesystem through NFS.

---

## Affected Asset

| Attribute | Details |
|---|---|
| Asset | Simulated Enterprise Server |
| IP Address | 192.168.247.129 |
| Service | Network File System (NFS) |
| Port | TCP/2049 |
| Protocol | NFS |

---

## Description

The target server exposes its entire root filesystem (`/`) through the Network File System service.

The export is accessible to any network client permitted to reach the NFS service and is configured with read/write permissions.

Server-side configuration further disables NFS root squashing through the `no_root_squash` option.

This configuration allows privileged users on a remote NFS client to retain root-level identity when interacting with the exported filesystem.

As a result, an unauthenticated network client with sufficient local privileges may access and potentially modify files throughout the target operating system.

---

## Evidence

### NFS Export Enumeration

NFS enumeration identified the root filesystem as an available export:

```text
Export list for 192.168.247.129:

/ *
```

This indicates that the root filesystem is exported to all permitted network clients.

---

### Root Filesystem Access

The exported filesystem was initially mounted read-only from the assessment workstation:

```bash
sudo mount -t nfs -o ro 192.168.247.129:/ /mnt/asteria-nfs
```

Enumeration confirmed access to operating system directories including:

```text
/bin
/boot
/etc
/home
/root
/usr
/var
```

The exposed `/home` directory also revealed multiple local user directories.

The `/etc` directory exposed operating system and service configuration files.

---

### NFS Export Configuration

Inspection of the NFS export configuration identified:

```text
/ *(rw,sync,no_root_squash,no_subtree_check)
```

The configuration introduces several significant security concerns:

- `/` exports the entire root filesystem.
- `*` permits access from any client allowed to reach the NFS service.
- `rw` permits both read and write operations.
- `no_root_squash` preserves remote root privileges instead of mapping them to an unprivileged account.

---

### Controlled Write Validation

To validate the practical impact without modifying sensitive system files, a temporary test file was created within the exported `/tmp` directory.

```bash
sudo touch /mnt/asteria-nfs/tmp/nfs-security-validation.txt
```

The resulting file was:

```text
-rw-r--r-- 1 root root 0 Sep 27 2026 /mnt/asteria-nfs/tmp/nfs-security-validation.txt
```

The `root root` ownership confirms that root privileges from the assessment workstation were preserved through the NFS export.

The temporary test file was removed immediately following validation.

No production or sensitive system files were modified during testing.

---

## Security Impact

The combination of an unrestricted root filesystem export, read/write access and disabled root squashing could allow a network-based attacker with privileged access on their own system to modify files on the target as root.

Potential consequences include:

- Modification of system configuration
- Modification of authentication-related files
- Unauthorized creation or modification of user accounts
- Modification of SSH configuration or authorized keys
- Modification of scheduled tasks or startup configuration
- Modification of application and web content
- Service configuration tampering
- Credential or sensitive information exposure
- Establishment of persistence
- Potential full compromise of the affected system

The assessment did not perform these actions because the controlled write test was sufficient to validate the security impact.

---

## Risk Analysis

### Likelihood

**High**

The NFS service is directly accessible from the assessment network and does not require application-level authentication before the exported root filesystem can be mounted.

The export is available broadly through the `*` client definition.

### Impact

**Critical**

Successful abuse could permit unauthorized modification of root-owned files throughout the target operating system.

This may lead to full system compromise, loss of confidentiality and integrity, or persistence on the affected server.

### Overall Risk

**Critical**

The combination of broad network accessibility, root filesystem exposure, read/write permissions and `no_root_squash` creates a direct path to privileged filesystem modification.

---

## Remediation

The root filesystem should never be broadly exported through NFS.

Recommended remediation actions include:

1. Remove the root filesystem (`/`) from the NFS export configuration.
2. Export only directories explicitly required for legitimate business operations.
3. Restrict NFS access to specific authorized hosts or trusted network segments rather than using `*`.
4. Remove the `no_root_squash` option unless there is a strictly justified administrative requirement.
5. Enable root squashing to prevent remote root identities from retaining equivalent privileges on the server.
6. Configure exports as read-only where write access is unnecessary.
7. Apply firewall rules or network segmentation to limit access to NFS services.
8. Review existing NFS exports for excessive permissions.
9. Monitor changes to NFS configuration and exported directories.
10. Apply the principle of least privilege to all file-sharing services.

---

## Remediation Validation

Following remediation, validation should confirm that:

- The root filesystem is no longer exported.
- Unauthorized clients cannot mount restricted directories.
- Only approved systems can access required NFS exports.
- Root squashing is enabled where appropriate.
- Exported directories use the minimum required permissions.
- Unnecessary write access has been removed.

Validation may include:

```bash
showmount -e 192.168.247.129
```

followed by controlled mount attempts from an unauthorized client.

Attempts to write files as a remote privileged user should fail or be mapped to an appropriately restricted identity.

---

## Evidence References

- `evidence/enumeration/nfs-exports.txt`
- `evidence/validation/nfs-root-listing.txt`
- `evidence/validation/nfs-etc-listing.txt`
- `evidence/validation/nfs-home-listing.txt`
- `evidence/validation/nfs-export-config.txt`
- `evidence/validation/nfs-write-validation.txt`
