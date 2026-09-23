# ☁️ CryptoCabana — Technical Notes

> Personal penetration testing notes from the **TryHackMe CryptoCabana** cloud security lab.
>
> These notes summarize reconnaissance methodology, Azure concepts, commands, attack observations, and defensive lessons. Sensitive values such as flags, SAS signatures, access tokens, and secrets have been intentionally removed.

---

# 📌 Lab Summary

| Item           | Details         |
| -------------- | --------------- |
| Platform       | TryHackMe       |
| Room           | CryptoCabana    |
| Category       | Cloud Security  |
| Difficulty     | Medium          |
| Cloud Provider | Microsoft Azure |

---

# 🎯 Learning Objectives

* Azure Static Website reconnaissance.
* Client-side cloud configuration discovery.
* Azure Blob Storage enumeration.
* Shared Access Signature (SAS) analysis.
* Azure Service Principal authentication.
* Azure Key Vault enumeration.
* RBAC permission analysis.
* Azure REST API usage.
* Secret version enumeration.

---

# ☁️ Azure Services Observed

| Azure Service           | Purpose                               |
| ----------------------- | ------------------------------------- |
| Static Website          | Public web frontend.                  |
| Storage Account         | Blob storage hosting.                 |
| Blob Containers         | Backup storage.                       |
| Shared Access Signature | Delegated storage authorization.      |
| Service Principal       | Azure workload identity.              |
| Azure CLI               | Cloud authentication and enumeration. |
| Azure Key Vault         | Secret management service.            |
| REST API                | Secret metadata enumeration.          |

---

# 🔍 Phase 1 — Reconnaissance

## Initial Observations

* Azure-hosted static website.
* Minimal frontend functionality.
* JavaScript handled storage interactions.
* No obvious backend endpoints exposed.

## Enumeration Checklist

* Inspect HTML source.
* Inspect JavaScript bundles.
* Review browser developer tools.
* Identify Azure endpoints.
* Look for hardcoded cloud configuration.

### Notes

* Client-side JavaScript often exposes cloud endpoints.
* Storage endpoints can reveal Azure architecture.

---

# 🧠 Phase 2 — Client-Side Analysis

## Interesting JavaScript Indicators

* Storage account name.
* Blob container.
* SAS query string.
* Storage endpoint URL.

### Why This Matters

Frontend JavaScript should not contain reusable authorization artifacts.

### Indicators to Look For

```javascript
const STORAGE_ACCOUNT = "...";
const CONTAINER = "...";
const SAS = "...";
```

### Observation

A reusable storage authorization was delivered to every visitor.

---

# 🔑 Phase 3 — Shared Access Signature (SAS)

## SAS Basics

A SAS token grants delegated access to Azure Storage resources.

### Components

| Component | Meaning                 |
| --------- | ----------------------- |
| sp        | Permissions             |
| st        | Start time              |
| se        | Expiration time         |
| sv        | Storage API version     |
| sr        | Resource type           |
| sig       | Cryptographic signature |

---

## Observed Permissions

```text
sp=rl
```

Meaning:

* Read
* List

### Expected Result

| Operation   | Outcome |
| ----------- | ------- |
| Read blobs  | Allowed |
| List blobs  | Allowed |
| Upload blob | Denied  |
| Delete blob | Denied  |

### Security Lesson

Least privilege reduces impact, but read/list permissions can still leak sensitive resources.

---

# 📦 Phase 4 — Azure Blob Storage Enumeration

## Storage Enumeration Goals

* Discover containers.
* Enumerate blob names.
* Identify hidden administrative storage.

### Useful Azure REST Operations

| Operation       | Purpose                      |
| --------------- | ---------------------------- |
| List Containers | Storage discovery.           |
| List Blobs      | Blob discovery.              |
| Download Blob   | Retrieve accessible objects. |

### Enumeration Workflow

1. List storage containers.
2. Inspect unexpected containers.
3. Enumerate blob contents.
4. Identify configuration files.

### Security Observation

Hidden containers are **not** security controls.

---

# 📂 Blob Containers Identified

| Container | Purpose                 |
| --------- | ----------------------- |
| `$web`    | Static website content. |
| `backups` | User backup data.       |
| `vault`   | Administrative storage. |

### Important Note

The administrative container was not referenced anywhere in the application UI.

---

# 👤 Phase 5 — Azure Service Principal

## Service Principal Overview

A Service Principal is an Azure identity used by applications and automation.

### Common Fields

| Field         | Purpose                |
| ------------- | ---------------------- |
| Tenant ID     | Azure tenant.          |
| Client ID     | Application identity.  |
| Client Secret | Authentication secret. |
| Vault URI     | Key Vault endpoint.    |

### Security Observation

Service Principal credentials should never be stored inside Blob Storage.

---

# 🛠️ Azure CLI Notes

## Authentication

Azure CLI supports Service Principal authentication for automation workloads.

### Verification Steps

* Authenticate.
* Verify current account.
* Verify tenant.
* Verify subscription context.

### Useful Commands

```bash
az login --service-principal
az account show
az account list
```

### Notes

Authentication succeeded using the exposed workload identity.

---

# 🔐 Phase 6 — Azure Key Vault Enumeration

## Key Vault Purpose

Centralized Azure secret management.

### Enumeration Goals

* Identify secret names.
* Inspect metadata.
* Understand RBAC permissions.
* Investigate secret versions.

### Observed Secrets

* Key shards.
* Master key.
* Multiple versions.

---

## RBAC Observation

The authenticated identity could:

* Enumerate secrets.
* View metadata.

The identity could **not** retrieve current secret values.

### Security Lesson

Metadata permissions can still provide useful intelligence.

---

# 🌐 Phase 7 — Azure REST API

## Why REST Instead of CLI?

The REST API exposed metadata not immediately visible during the initial CLI workflow.

### Metadata Examples

* Secret ID.
* Version IDs.
* Creation timestamps.
* Update timestamps.
* Tags.

### Investigation Goal

Identify recently rotated secrets.

---

# 🧩 Phase 8 — Secret Version Enumeration

## Version History

Azure Key Vault stores multiple versions of a secret after rotation.

### Investigation Steps

1. Enumerate versions.
2. Compare timestamps.
3. Identify historical versions.
4. Request previous version.

### Observation

Only one secret contained multiple versions.

---

# 🕒 Secret Rotation Notes

| Concept          | Description                |
| ---------------- | -------------------------- |
| Rotation         | Secret value updated.      |
| Version          | Immutable historical copy. |
| Latest Version   | Current active secret.     |
| Previous Version | Historical secret value.   |

### Security Lesson

Secret rotation alone does not eliminate historical exposure.

---

# 🔍 Historical Secret Retrieval

### Objective

Retrieve an older version of the rotated secret.

### Result

* Historical version accessible.
* Previous shard recovered.
* Final flag reconstructed locally.

> **Flag intentionally removed.**

---

# 🧠 Attack Chain Summary

```text
Azure Static Website
      │
      ▼
Client-side JavaScript
      │
      ▼
SAS Token Discovery
      │
      ▼
Blob Enumeration
      │
      ▼
Hidden Container
      │
      ▼
Service Principal Exposure
      │
      ▼
Azure Authentication
      │
      ▼
Key Vault Enumeration
      │
      ▼
REST Metadata Analysis
      │
      ▼
Historical Secret Version
      │
      ▼
Flag Reconstruction (Redacted)
```

---

# ⚠️ Security Findings

| Finding                               | Severity |
| ------------------------------------- | -------- |
| Client-side SAS exposure              | High     |
| Hidden Blob container accessible      | High     |
| Service Principal credential exposure | Critical |
| Metadata enumeration through RBAC     | Medium   |
| Historical secret version exposure    | High     |

---

# 🛡️ MITRE ATT&CK Notes

| Technique | Description                       |
| --------- | --------------------------------- |
| T1552     | Unsecured Credentials             |
| T1528     | Application Access Token Exposure |
| T1526     | Cloud Service Discovery           |
| T1087     | Account Discovery                 |
| T1078     | Valid Accounts                    |
| T1550     | Use of Stolen Credentials         |

---

# 🛡️ Defensive Recommendations

## Azure Storage

* Avoid long-lived SAS tokens.
* Use least privilege.
* Scope SAS to individual resources.
* Prefer user delegation SAS.

## Azure Identity

* Replace Service Principals with Managed Identities.
* Rotate exposed credentials immediately.
* Restrict workload permissions.

## Azure Key Vault

* Audit RBAC assignments.
* Restrict metadata visibility.
* Review historical secret versions.
* Remove obsolete versions when appropriate.

## Monitoring

Enable monitoring for:

* Blob enumeration.
* SAS usage.
* Service Principal authentication.
* Key Vault enumeration.
* Secret version access.

---

# 💡 Key Concepts Learned

## Azure Storage

* Storage Accounts
* Blob Containers
* SAS Tokens
* Static Websites

## Azure Identity

* Service Principals
* Azure CLI Authentication
* RBAC Permissions

## Azure Secrets

* Key Vault
* Secret Metadata
* Secret Versions
* REST API Enumeration

---

# 📚 References

* TryHackMe — CryptoCabana.
* Microsoft Learn — Azure Storage Shared Access Signatures.
* Microsoft Learn — Azure Blob Storage REST API.
* Microsoft Learn — Azure Key Vault REST API.
* Microsoft Learn — Azure RBAC for Key Vault.

---

# ✅ Portfolio Notes

This document contains **technical notes only**.

Intentionally removed:

* Room Flag
* SAS Token Signature
* Azure Client Secret
* Access Tokens
* Secret Shards
* Tenant IDs
* Subscription IDs

These notes are intended for **cybersecurity interview preparation, cloud security revision, and GitHub portfolio documentation**.
