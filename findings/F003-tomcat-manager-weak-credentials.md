# F-003 — Weak Credentials Permit Unauthorized Access to Apache Tomcat Manager

## Severity

**High**

---

## Status

**Confirmed**

Manual testing confirmed that the Apache Tomcat Manager interface required authentication but accepted the weak credential pair `tomcat:tomcat`.

An unauthenticated request returned:

```text
HTTP 401 Unauthorized
```

The same endpoint returned:

```text
HTTP 200 OK
```

after the tested credentials were supplied.

Manual browser validation also confirmed access to the **Tomcat Web Application Manager** administrative interface.

No application deployment, command execution, reverse shell, persistence, or other post-authentication exploitation was performed.

---

## Affected Asset

| Attribute | Details |
|---|---|
| Asset | Simulated Enterprise Server |
| Asset ID | TG-01 |
| IP Address | `192.168.247.129` |
| Service | Apache Tomcat |
| Port | TCP/8180 |
| Identified Version | Apache Tomcat/5.5 |
| Administrative Interface | `/manager/html` |
| Tested Credentials | `tomcat:tomcat` |
| Status | Confirmed |

---

## Description

The target server exposed the Apache Tomcat Web Application Manager interface on TCP/8180.

The Manager interface correctly rejected an unauthenticated request with HTTP `401 Unauthorized`, demonstrating that authentication was required.

However, the weak credential pair:

```text
tomcat:tomcat
```

was successfully accepted.

The authenticated request returned HTTP `200 OK`, and browser validation confirmed access to the Tomcat Web Application Manager interface.

This represents a significant authentication weakness because an attacker with network access to the management interface could potentially guess or obtain the predictable credentials and gain administrative application-management access.

Testing stopped once authenticated administrative access had been sufficiently demonstrated.

---

## Evidence

### Tomcat Service Identification

Web enumeration identified Apache Tomcat on TCP/8180.

The service returned:

```text
8180/tcp open
HTTP title: Apache Tomcat/5.5
```

The administrative interface was identified at:

```text
http://192.168.247.129:8180/manager/html
```

Supporting evidence:

```text
evidence/enumeration/web-enumeration.txt
```

---

### Unauthenticated Access Test

The Tomcat Manager interface was requested without authentication:

```bash
curl -i http://192.168.247.129:8180/manager/html
```

The server returned:

```text
HTTP/1.1 401 Unauthorized
```

The response also identified the authentication realm as:

```text
Tomcat Manager Application
```

This confirmed that the Manager endpoint required authentication.

Supporting evidence:

```text
evidence/enumeration/tomcat-manager-unauthenticated.txt
```

---

### Authenticated Access Validation

The same endpoint was then tested using:

```text
tomcat:tomcat
```

The authenticated request was performed using:

```bash
curl -i -u tomcat:tomcat http://192.168.247.129:8180/manager/html
```

The server returned:

```text
HTTP/1.1 200 OK
```

The transition from:

```text
401 Unauthorized
```

to:

```text
200 OK
```

demonstrated that the supplied credentials were successfully accepted by the Tomcat Manager application.

Supporting evidence:

```text
evidence/validation/tomcat-manager-authenticated.txt
```

---

### Browser Validation

The same credentials were used to access the Tomcat Manager interface through a web browser.

Successful access to the **Tomcat Web Application Manager** confirmed that the authenticated session provided access to the administrative management console.

A screenshot of the authenticated Manager interface was retained as supporting evidence:

```text
evidence/validation/tomcat-manager-authenticated-access.png
```

No password was intentionally exposed within the retained screenshot.

---

## Validation Boundary

The purpose of validation was to determine whether weak credentials permitted unauthorized administrative access to the Tomcat Manager interface.

That objective was satisfied once the assessment confirmed:

1. TCP/8180 was reachable from the assessment network.
2. The Tomcat Manager interface was exposed.
3. Unauthenticated access was denied.
4. The weak credential pair was accepted.
5. An authenticated request returned HTTP `200 OK`.
6. The administrative Manager interface was accessible through a web browser.

Further exploitation was not required to validate the authentication weakness.

The assessment therefore did **not**:

- Upload a WAR application
- Deploy attacker-controlled server-side content
- Execute operating-system commands
- Establish a reverse shell
- Modify existing applications
- Stop or remove applications
- Create persistence
- Alter server configuration
- Access unrelated application data
- Attempt full host compromise

Stopping at confirmed administrative access maintained a controlled and minimally invasive validation approach.

---

## Security Impact

The directly demonstrated impact was unauthorized access to a privileged application-management interface using weak credentials.

Administrative access to Tomcat Manager can expose functionality capable of managing hosted web applications.

Depending on the privileges assigned to the compromised account, potential consequences may include:

- Viewing deployed applications
- Managing application state
- Deploying applications
- Modifying hosted application content
- Removing applications
- Disrupting hosted services
- Introducing unauthorized server-side application content

Where an account has sufficient deployment privileges, administrative application access may also provide a path toward server-side code execution.

However, application deployment and operating-system command execution were **not tested during this assessment**.

The finding therefore confirms unauthorized administrative access without claiming full server compromise.

---

## Risk Analysis

### Likelihood — High

The likelihood of abuse is assessed as **High** because:

- The Tomcat Manager interface was directly reachable from the assessment network.
- The authentication interface was exposed.
- The tested password was identical to the username.
- The credential pair was predictable and easily guessable.
- Successful authentication required minimal technical complexity.
- Manual testing reproduced the weakness successfully.

---

### Impact — High

The impact is assessed as **High** because:

- The compromised credentials provided access to an administrative application-management interface.
- Administrative functionality could permit significant alteration or disruption of hosted applications.
- The access obtained was substantially more privileged than normal application-user access.

The impact is not classified as Critical because:

- Server-side code execution was not validated.
- Operating-system command execution was not validated.
- Full host compromise was not demonstrated.
- Persistence was not established.

---

### Overall Severity — High

**High**

Under the assessment's qualitative risk methodology, the combination of **High likelihood** and **High impact** results in an overall **High severity**.

The rating is based on confirmed unauthorized administrative access rather than hypothetical full-system compromise.

See:

[`../documentation/risk-methodology.md`](../documentation/risk-methodology.md)

---

## Remediation

Recommended remediation actions include:

1. Immediately replace weak or predictable Tomcat Manager credentials with strong, unique credentials.
2. Remove unused or unnecessary Manager accounts.
3. Ensure passwords are not identical to usernames or based on common credential patterns.
4. Restrict access to the Tomcat Manager interface to explicitly authorized administrative hosts.
5. Place administrative interfaces on dedicated management networks where possible.
6. Apply firewall rules or network segmentation to prevent general user networks from reaching TCP/8180.
7. Disable the Tomcat Manager application entirely where it is not operationally required.
8. Review Tomcat role assignments and grant only the minimum administrative permissions required.
9. Periodically review configured Tomcat users and roles.
10. Monitor authentication attempts for repeated failures or suspicious administrative access.
11. Upgrade unsupported or obsolete Tomcat deployments and maintain current security patches.

---

## Remediation Validation

Following remediation, the previously accepted credential pair should no longer authenticate.

### Unauthenticated Test

Repeat:

```bash
curl -i http://192.168.247.129:8180/manager/html
```

Unauthenticated access should continue to be denied.

---

### Former Credential Test

Repeat:

```bash
curl -i -u tomcat:tomcat http://192.168.247.129:8180/manager/html
```

The former credential pair should no longer return:

```text
HTTP 200 OK
```

Authentication should fail.

---

### Network Restriction Test

Where management-network restrictions have been implemented, the Manager interface should not be accessible from unauthorized network segments.

---

### Authorized Administrative Test

Legitimate administrators should confirm that required management functionality remains available using:

- Approved management systems
- Authorized accounts
- Strong unique credentials
- Appropriate Tomcat roles

---

## Evidence References

- `evidence/enumeration/web-enumeration.txt`
- `evidence/enumeration/tomcat-manager-unauthenticated.txt`
- `evidence/validation/tomcat-manager-authenticated.txt`
- `evidence/validation/tomcat-manager-authenticated-access.png`

---

## Assessment Conclusion

The assessment confirmed that the Apache Tomcat Manager interface required authentication but accepted the weak credential pair `tomcat:tomcat`.

The evidence chain consisted of:

```text
Manager interface reachable
        ↓
Unauthenticated request → 401 Unauthorized
        ↓
Weak credentials supplied
        ↓
Authenticated request → 200 OK
        ↓
Administrative Manager interface accessible
```

This provides sufficient evidence of a serious authentication weakness resulting in unauthorized administrative application access.

Additional exploitation was deliberately avoided because it was unnecessary to establish the vulnerability.

The issue is therefore classified as a **confirmed High-severity finding**.
