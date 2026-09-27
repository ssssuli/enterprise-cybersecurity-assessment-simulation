# F-001 — Overly Permissive NFS Root Filesystem Export

## Severity

**High — Provisional**

Final severity will be reviewed after validating the server-side NFS export permissions.

---

## Status

**Confirmed**

Unauthenticated network access to the exported root filesystem was successfully demonstrated from the assessment workstation.

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

The target server exposes its root filesystem (`/`) through the Network File System service to network clients within the assessment environment.

NFS export enumeration identified the root filesystem as an available export.

The exported filesystem was subsequently mounted from the assessment workstation without requiring application-level authentication.

The mounted filesystem exposed the structure of the target operating system, including system configuration and user directories.

---

## Evidence

NFS export enumeration identified the following exported filesystem:

```text
Export list for 192.168.247.129:

/ *
```

The exported root filesystem was mounted on the assessment workstation using a read-only client-side mount:

```bash
sudo mount -t nfs -o ro 192.168.247.129:/ /mnt/asteria-nfs
```

Filesystem enumeration confirmed access to system directories including:

```text
/bin
/boot
/etc
/home
/root
/usr
/var
```

Further enumeration of `/home` identified multiple local user directories.

The `/etc` directory also exposed operating system and service configuration files.

Supporting evidence is retained under the project's `evidence/` directory.

---

## Security Impact

Exposing the server's root filesystem through NFS may allow unauthorized network users to access system information that would normally only be available locally.

An attacker could potentially use the exposed filesystem to gather:

- User and account information
- Network configuration
- Installed service configuration
- Web and database configuration
- Application configuration
- Authentication-related files
- Service versions and operating system information

Information obtained through the exposed filesystem could support credential attacks, service exploitation, lateral movement or additional privilege escalation attempts.

The final impact will depend on the permissions configured for the NFS export.

---

## Risk Analysis

### Likelihood

**High**

The NFS service is directly accessible from the assessment network and the root filesystem can be mounted without application-level authentication.

### Impact

**High**

Exposure of the operating system filesystem may disclose sensitive configuration and system information useful for further compromise.

### Overall Risk

**High — Provisional**

The severity will be reassessed after confirming whether the NFS export permits modification of remote files.

---

## Remediation

The organization should avoid exporting the server's root filesystem through NFS.

Recommended actions include:

1. Remove the root filesystem (`/`) from the NFS export configuration.
2. Export only directories specifically required for legitimate business purposes.
3. Restrict NFS access to explicitly authorized hosts or trusted network segments.
4. Configure exports using the minimum permissions required.
5. Prefer read-only exports where write access is unnecessary.
6. Review NFS identity-mapping and privilege-handling settings.
7. Apply network-level controls to prevent unnecessary access to NFS services.
8. Regularly review active NFS exports for excessive permissions.

---

## Remediation Validation

Following remediation, the assessment should verify that:

- The root filesystem is no longer exported.
- Unauthorized systems cannot mount restricted directories.
- Only approved systems can access required NFS shares.
- Export permissions follow the principle of least privilege.

Validation can be performed using:

```bash
showmount -e 192.168.247.129
```

and controlled mount attempts from an unauthorized client.

---

## Evidence References

- `evidence/enumeration/nfs-exports.txt`
- `evidence/validation/nfs-root-listing.txt`
- `evidence/validation/nfs-etc-listing.txt`
- `evidence/validation/nfs-home-listing.txt`
- `evidence/validation/nfs-export-config.txt`
