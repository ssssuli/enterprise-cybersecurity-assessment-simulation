# F-003 — Weak Credentials Permit Unauthorized Access to Apache Tomcat Manager

## Severity

**High**

---

## Status

**Confirmed**

Testing confirmed that the Apache Tomcat Manager interface was protected from unauthenticated access but could be successfully accessed using the weak credential pair `tomcat:tomcat`.

An unauthenticated request to the Manager interface returned HTTP `401 Unauthorized`.

The same request supplied with the tested credentials returned HTTP `200 OK`, and the Tomcat Web Application Manager interface was successfully accessed through a web browser.

No application deployment, command execution, reverse shell, or other post-authentication exploitation was performed.

---

## Affected Asset

| Attribute | Details |
|---|---|
| Asset | Simulated Enterprise Server |
| IP Address | 192.168.247.129 |
| Service | Apache Tomcat |
| Port | TCP/8180 |
| Identified Version | Apache Tomcat/5.5 |
| Administrative Interface | `/manager/html` |
| Tested Credentials | `tomcat:tomcat` |

---

## Description

The target server exposes the Apache Tomcat Web Application Manager interface on TCP/8180.

The Manager interface correctly rejected an unauthenticated request with HTTP `401 Unauthorized`, confirming that authentication was required.

However, authentication using the weak credential pair `tomcat:tomcat` succeeded. The authenticated request returned HTTP `200 OK`, and manual browser validation confirmed access to the Tomcat Web Application Manager interface.

This represents a significant authentication weakness because a remote user who obtains or guesses the credential pair can gain access to a privileged application-management interface.

The assessment stopped after confirming authenticated administrative access. No application was uploaded or deployed, and no attempt was made to obtain operating-system command execution.

---

## Evidence

### Tomcat Service Identification

Web enumeration identified the service exposed on TCP/8180 as Apache Tomcat:

```text
8180/tcp open
HTTP title: Apache Tomcat/5.5
```

The identified administrative interface was:

```text
http://192.168.247.129:8180/manager/html
```

---

### Unauthenticated Access Test

The Tomcat Manager interface was first requested without credentials:

```bash
curl -i http://192.168.247.129:8180/manager/html
```

The server returned:

```text
HTTP 401 Unauthorized
```

This demonstrated that direct unauthenticated access to the Manager interface was denied and that authentication was required.

---

### Authenticated Access Validation

The same Manager endpoint was then tested using the credential pair:

```text
tomcat:tomcat
```

The request was performed using:

```bash
curl -i -u tomcat:tomcat http://192.168.247.129:8180/manager/html
```

The server returned:

```text
HTTP 200 OK
```

The difference between the unauthenticated `401` response and authenticated `200` response confirms that the supplied credentials were accepted by the Tomcat Manager interface.

---

### Browser Validation

The authenticated Tomcat Manager interface was also accessed through a web browser.

Successful access to the **Tomcat Web Application Manager** interface visually confirmed that the tested credentials provided access to the administrative management console.

A screenshot was retained as assessment evidence.

No credentials were intentionally exposed within the retained screenshot.

---

## Validation Boundary

The objective of validation was to determine whether unauthorized administrative access was possible using weak credentials.

That objective was satisfied once the assessment established all of the following:

1. The Tomcat Manager interface was externally reachable from the assessment workstation.
2. Unauthenticated access was rejected.
3. The tested credentials were accepted.
4. An authenticated request returned HTTP `200 OK`.
5. The Tomcat Web Application Manager interface was accessible through a browser.

Further exploitation was not necessary to establish the security weakness.

The assessment therefore did **not**:

- Upload or deploy a WAR application
- Execute operating-system commands
- Establish a reverse shell
- Modify existing applications
- Create persistence
- Alter server configuration
- Access unrelated application data

This maintained a controlled and minimally invasive validation approach.

---

## Security Impact

Access to the Tomcat Manager interface provides an unauthorized user with administrative application-management capabilities.

Depending on the roles and permissions assigned to the compromised Manager account, potential consequences may include:

- Viewing deployed web applications
- Managing application state
- Deploying or modifying web applications
- Removing applications
- Disrupting hosted services
- Introducing unauthorized server-side application content
- Potential escalation to server-side code execution where deployment functionality and account permissions permit it

The assessment confirmed access to the administrative Manager interface but deliberately did not test application deployment or operating-system command execution.

Therefore, this finding demonstrates a serious authentication and administrative-access failure without claiming full host compromise.

---

## Risk Analysis

### Likelihood

**High**

The Tomcat Manager interface was directly reachable from the assessment network, and the credential pair `tomcat:tomcat` successfully authenticated without requiring prior compromise of another account or service.

The credential pair is weak because the password is identical to the username and is readily guessable.

### Impact

**High**

Successful authentication provided access to an administrative application-management interface.

Compromise of such an interface can permit unauthorized modification or disruption of hosted applications and may create a path toward more serious server compromise depending on the permissions available to the authenticated account.

### Overall Risk

**High**

The combination of a remotely accessible administrative interface and easily guessable credentials creates a significant risk of unauthorized administrative access.

The issue is rated **High** rather than Critical because operating-system command execution or full server compromise was not validated during the assessment.

---

## Remediation

Recommended remediation actions include:

1. Immediately replace weak or predictable Tomcat Manager credentials with strong, unique credentials.
2. Remove unused or unnecessary Manager accounts.
3. Ensure administrative account passwords are not identical to usernames or based on common credential patterns.
4. Restrict access to the Tomcat Manager interface to authorized administrative hosts or trusted management networks.
5. Apply firewall rules or network segmentation to prevent general user networks from reaching the management interface.
6. Disable the Tomcat Manager application entirely where it is not operationally required.
7. Review Tomcat role assignments and grant only the minimum management permissions required.
8. Periodically audit Tomcat users, roles, and credential configuration.
9. Monitor authentication attempts to the Manager interface for repeated failures or suspicious access.
10. Upgrade unsupported or obsolete Tomcat deployments and maintain current security patches.

---

## Remediation Validation

Following remediation, validation should confirm that the previously accepted credential pair no longer authenticates.

An unauthenticated request should continue to be denied:

```bash
curl -i http://192.168.247.129:8180/manager/html
```

The former weak credentials should also fail:

```bash
curl -i -u tomcat:tomcat http://192.168.247.129:8180/manager/html
```

The expected result is that unauthorized authentication is rejected.

Where network restrictions are implemented, the Manager interface should additionally be unreachable from non-administrative network segments.

Authorized administrators should confirm that legitimate management access remains functional using approved credentials and permitted management systems.

---

## Evidence References

- `evidence/enumeration/web-enumeration.txt`
- `evidence/enumeration/tomcat-manager-unauthenticated.txt`
- `evidence/validation/tomcat-manager-authenticated.txt`
- `evidence/validation/tomcat-manager-authenticated-access.png`

---

## Assessment Conclusion

The assessment confirmed that the Apache Tomcat Manager interface required authentication but accepted the weak credential pair `tomcat:tomcat`.

The combination of HTTP `401 Unauthorized` without credentials, HTTP `200 OK` after authentication, and successful browser access to the Tomcat Web Application Manager provides sufficient evidence that unauthorized administrative access was possible.

Additional exploitation was intentionally avoided because it was not necessary to validate the underlying authentication weakness.

This finding is therefore recorded as a **confirmed High-severity vulnerability**.
