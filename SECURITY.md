# Security Policy

## CryptoCabana — Azure Cloud Security Walkthrough

Thank you for taking the time to help improve the security and quality of this repository.

This repository documents an **authorized TryHackMe cloud security lab** focused on Microsoft Azure security concepts, cloud penetration testing methodology, and defensive analysis. It is maintained as part of a cybersecurity portfolio and educational knowledge base.

> **Repository Scope**
>
> This project contains documentation, screenshots, notes, and walkthrough material for the **CryptoCabana** TryHackMe room. It does **not** contain production infrastructure, deployable Azure resources, or live credentials.

---

# Supported Versions

The latest version of this repository is actively maintained.

| Version                          | Supported |
| -------------------------------- | --------- |
| Latest `main` branch             | ✅ Yes     |
| Previous documentation revisions | ✅ Yes     |
| Archived or forked copies        | ❌ No      |

Please report issues against the latest version available on the `main` branch.

---

# Reporting a Security Issue

If you discover a security issue within this repository (for example, accidentally exposed credentials or sensitive information), please report it responsibly.

### Please report issues such as:

* Accidentally committed secrets or API keys.
* Azure credentials or SAS tokens that should not be public.
* Sensitive screenshots exposing confidential information.
* Personally identifiable information (PII).
* Broken sanitization of challenge artifacts.
* Repository configuration issues.

### Please do NOT report:

* TryHackMe room flags.
* Intended challenge solutions.
* Public Microsoft Azure documentation issues.
* Vulnerabilities inside TryHackMe infrastructure.
* Vulnerabilities in Microsoft Azure services themselves.

---

# Responsible Disclosure

Please follow responsible disclosure practices.

1. **Do not publish** exposed secrets or credentials publicly.
2. Create a private GitHub issue or contact the repository owner.
3. Provide enough information to reproduce the issue.
4. Allow time for remediation before public discussion.

Reports should include:

* Description of the issue.
* File path or affected document.
* Screenshot (if applicable).
* Suggested remediation.

---

# Security Scope

This repository intentionally excludes all sensitive challenge artifacts.

## Included

* Technical documentation.
* Security methodology.
* Azure architecture explanations.
* Screenshots from an authorized lab.
* MITRE ATT&CK mapping.
* Defensive recommendations.

## Explicitly Removed

* TryHackMe room flags.
* Azure SAS token signatures.
* Azure Tenant IDs.
* Client Secrets.
* OAuth Access Tokens.
* Subscription IDs.
* Secret shard values.
* Historical Key Vault secret contents.
* Live Azure resource identifiers.

Any appearance of these values should be considered accidental and reported immediately.

---

# Secrets Management Policy

This repository follows a strict **no-secrets policy**.

Sensitive values are replaced with placeholders such as:

```text
THM{REDACTED}
<CLIENT_SECRET_REDACTED>
<SAS_SIGNATURE_REDACTED>
<ACCESS_TOKEN_REDACTED>
<SECRET_VALUE_REDACTED>
```

Never commit:

* Azure Storage Account Keys.
* SAS Tokens.
* Azure Client Secrets.
* Microsoft Entra Tokens.
* Personal Access Tokens.
* SSH Private Keys.
* Cloud credentials of any kind.

---

# Supported Security Practices

This repository follows Microsoft Azure security best practices where applicable.

### Azure Storage

* Least-privilege SAS permissions.
* Short-lived delegated authorization.
* No storage account keys in documentation.

### Azure Identity

* Managed Identities preferred over long-lived Service Principals.
* Credential rotation after exposure.
* Principle of Least Privilege.

### Azure Key Vault

* RBAC-based authorization.
* Secret lifecycle management.
* Version auditing.
* Secure secret storage.

---

# Repository Hardening

The repository is designed for documentation only.

Security controls include:

* No executable exploitation payloads.
* No live infrastructure deployment templates.
* Sanitized screenshots.
* Redacted configuration examples.
* GitHub Pages static documentation only.

---

# Educational Use Policy

This project is intended for:

* Cybersecurity education.
* Azure cloud security learning.
* Penetration testing methodology.
* Defensive cloud security awareness.
* Professional portfolio demonstration.

It is **not** intended for unauthorized testing of real Azure environments.

---

# Safe Testing Guidelines

Only perform the documented techniques against:

* TryHackMe labs.
* Personal lab environments.
* Azure subscriptions you own.
* Systems where you have explicit authorization.

Never reuse these techniques against systems without permission.

---

# Third-Party Services

This repository references publicly documented Azure services and educational resources.

Referenced technologies include:

* Microsoft Azure Storage.
* Azure Blob Storage.
* Azure Key Vault.
* Microsoft Entra ID.
* Azure CLI.
* Azure REST APIs.
* TryHackMe.

No proprietary Microsoft resources or private APIs are included.

---

# Security Review Checklist

Before publishing updates, verify:

* [x] No flags included.
* [x] No Azure credentials included.
* [x] No SAS signatures included.
* [x] No OAuth tokens included.
* [x] Screenshots reviewed for sensitive information.
* [x] Configuration examples sanitized.
* [x] Documentation references authorized lab content only.

---

# Compliance Statement

This repository documents activities performed inside an **authorized TryHackMe environment** for educational purposes.

All screenshots, commands, and methodologies have been reviewed to remove sensitive information while preserving educational value.

---

# Contact

If you identify a security concern related to this repository, please open a **GitHub Security Advisory** or create a **private issue** describing the problem.

Maintainer: **Anurag Revankar**

---

<div align="center">

### 🔒 Security First • Responsible Disclosure • Educational Use Only

**CryptoCabana — Azure Cloud Security Walkthrough**

*Cybersecurity Portfolio Project*

</div>
