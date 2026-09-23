---

layout: default
title: CryptoCabana — Azure Cloud Security Walkthrough
description: Professional TryHackMe CryptoCabana documentation for GitHub Pages.
--------------------------------------------------------------------------------

<div align="center">

# ☁️ CryptoCabana

## Azure Cloud Security Walkthrough

### *A Professional Microsoft Azure Penetration Testing Case Study*

<img src="assets/01_cryptocabana_landing.png" width="100%" alt="CryptoCabana Banner"/>

<br>

![TryHackMe](https://img.shields.io/badge/TryHackMe-Hacker%20Holidays-red?style=for-the-badge\&logo=tryhackme)
![Azure](https://img.shields.io/badge/Microsoft-Azure-0078D4?style=for-the-badge\&logo=microsoftazure\&logoColor=white)
![Cloud Security](https://img.shields.io/badge/Cloud-Security-blue?style=for-the-badge)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange?style=for-the-badge)

**Author:** **Anurag Revankar**

*Cybersecurity Portfolio • Azure Security • Cloud Pentesting • TryHackMe*

</div>

---

# 📖 Executive Summary

**CryptoCabana** is a cloud-focused Microsoft Azure security challenge that demonstrates how **multiple Azure misconfigurations** can be chained together into a complete compromise.

The room begins with a publicly accessible **Azure Static Website** and progresses through Azure Storage enumeration, Service Principal authentication, Azure Key Vault investigation, RBAC analysis, and historical secret version recovery.

This GitHub Pages documentation presents the entire assessment from a penetration tester's perspective while intentionally removing all challenge flags and sensitive cloud credentials.

> **Portfolio Edition**
>
> All secrets, SAS signatures, Azure Tenant IDs, Service Principal credentials, OAuth tokens, secret shards, and room flags have been removed.

---

# 🎯 Assessment Objectives

<table>
<tr>
<th>Cloud Security Goals</th>
<th>Investigation Goals</th>
</tr>

<tr>
<td>

* Azure Storage Enumeration
* SAS Token Analysis
* Blob Container Discovery
* Azure Identity Investigation
* Azure Key Vault Enumeration

</td>

<td>

* Client-side Reconnaissance
* RBAC Analysis
* REST API Enumeration
* Secret Version Discovery
* Cloud Trust Chain Analysis

</td>
</tr>
</table>

---

# ☁️ Azure Attack Chain

## Complete Cloud Compromise Path

```text
Internet
    │
    ▼
Azure Static Website
    │
    ▼
JavaScript Configuration Leak
    │
    ▼
Shared Access Signature (SAS)
    │
    ▼
Azure Blob Storage Enumeration
    │
    ▼
Hidden Administrative Container
    │
    ▼
Service Principal Credentials
    │
    ▼
Microsoft Entra Authentication
    │
    ▼
Azure Key Vault
    │
    ▼
Secret Version Enumeration
    │
    ▼
Historical Secret Recovery
```

The challenge demonstrates how **authorization artifacts**, **cloud identities**, and **secret lifecycle management** combine into a realistic Azure attack path.

---

# 🏗️ Azure Architecture Overview

```text
                        Internet
                           │
                           ▼
               Azure Static Website ($web)
                           │
                    app.js exposes SAS
                           │
                           ▼
             Azure Blob Storage Account
             ├──────── backups
             ├──────── $web
             └──────── vault
                           │
                           ▼
            backup-service-account.json
                           │
                           ▼
             Azure Service Principal
                           │
                           ▼
                Azure Key Vault
         ├── key-shard-1
         ├── key-shard-2
         ├── key-shard-3
         └── master-key
                           │
                           ▼
           Historical Secret Version
```

---

# 🧩 Assessment Timeline

| Phase  | Description                                |
| ------ | ------------------------------------------ |
| **01** | Azure Static Website reconnaissance.       |
| **02** | Client-side JavaScript analysis.           |
| **03** | Shared Access Signature permission review. |
| **04** | Azure Blob Storage container enumeration.  |
| **05** | Hidden vault container discovery.          |
| **06** | Azure Service Principal authentication.    |
| **07** | Azure Key Vault RBAC investigation.        |
| **08** | Azure REST API metadata enumeration.       |
| **09** | Historical secret version recovery.        |

---

# 🔍 Phase 1 — Azure Static Website Reconnaissance

The assessment begins by inspecting a publicly accessible Azure Static Website hosting a cryptocurrency backup application.

Important observations include:

* Static HTML interface.
* JavaScript-based storage operations.
* No authentication barrier.
* Azure-hosted frontend.

<p align="center">
<img src="assets/01_cryptocabana_landing.png" width="95%">
</p>

<p align="center"><b>Figure 1.</b> Initial reconnaissance against the CryptoCabana application.</p>

---

# 🔍 Phase 2 — Client-side JavaScript Analysis

Inspecting the frontend JavaScript reveals Azure Storage configuration embedded directly into client-side code.

**Key Findings**

* Storage Account Name.
* Blob Container.
* Blob Endpoint.
* Shared Access Signature.

<p align="center">
<img src="assets/02_frontend_appjs.png" width="95%">
</p>

<p align="center"><b>Figure 2.</b> JavaScript exposing Azure Storage configuration (sanitized).</p>

### Security Observation

Authorization artifacts embedded into frontend applications increase cloud attack surface.

---

# 🔐 Phase 3 — SAS Token Permission Analysis

The exposed SAS token contains delegated permissions for Azure Storage operations.

<table>
<tr>
<th>Permission</th>
<th>Meaning</th>
</tr>

<tr>
<td>Read</td>
<td>Blob contents accessible.</td>
</tr>

<tr>
<td>List</td>
<td>Container enumeration allowed.</td>
</tr>

</table>

Upload operations are denied by Azure authorization.

<p align="center">
<img src="assets/03_sas_permissions_403.png" width="95%">
</p>

<p align="center"><b>Figure 3.</b> Authorization boundary verification using Azure Storage.</p>

---

# 📦 Phase 4 — Azure Blob Storage Enumeration

Using the delegated SAS authorization, Azure Blob Storage becomes enumerable.

Containers discovered include:

* `$web`
* `backups`
* `vault`

The **vault** container is never referenced by the application.

<p align="center">
<img src="assets/04_blob_container_enum.png" width="95%">
</p>

<p align="center"><b>Figure 4.</b> Azure Blob Storage enumeration revealing hidden resources.</p>

---

# 👤 Phase 5 — Service Principal Exposure

The hidden administrative container stores an Azure Service Principal configuration used by backend automation.

<p align="center">
<img src="assets/05_service_principal_json.png" width="95%">
</p>

<p align="center"><b>Figure 5.</b> Service Principal configuration recovered from Blob Storage (redacted).</p>

### Why This Matters

Service Principals authenticate applications directly into Microsoft Azure.

Exposing these credentials creates a legitimate cloud authentication path.

---

# 🛠️ Phase 6 — Azure CLI Authentication

The recovered Service Principal authenticates successfully into Azure.

<p align="center">
<img src="assets/06_azure_login.png" width="95%">
</p>

<p align="center"><b>Figure 6.</b> Successful Azure CLI authentication using the recovered workload identity.</p>

### Validation

* Tenant context confirmed.
* OAuth token issued.
* Subscription context available.
* Azure resources accessible.

---

# 🔐 Phase 7 — Azure Key Vault Investigation

The authenticated identity can enumerate Azure Key Vault secrets but cannot retrieve current secret values.

<p align="center">
<img src="assets/07_keyvault_rbac.png" width="95%">
</p>

<p align="center"><b>Figure 7.</b> Azure Key Vault RBAC authorization boundary.</p>

### Metadata Accessible

* Secret Names.
* Secret Tags.
* Creation Timestamps.
* Update Timestamps.
* Version Metadata.

Current values remain protected.

---

# 🌐 Phase 8 — Azure REST API Enumeration

Azure REST APIs expose additional metadata describing Key Vault secret lifecycle events.

<p align="center">
<img src="assets/08_rest_secret_versions.png" width="95%">
</p>

<p align="center"><b>Figure 8.</b> Secret version metadata enumeration through Azure REST API.</p>

### Metadata Investigation

* Version IDs.
* Secret Rotation Timeline.
* Secret Attributes.
* Version Count.

---

# 🕒 Phase 9 — Historical Secret Recovery

A previous version of a rotated secret remains accessible.

<p align="center">
<img src="assets/09_historical_version.png" width="95%">
</p>

<p align="center"><b>Figure 9.</b> Historical secret version recovery (sanitized).</p>

### Final Result

The historical shard replaces the rotated value and allows local reconstruction of the room secret.

> **Room flag intentionally removed.**

---

# ☁️ Azure Trust Relationship Analysis

```text
Browser
   │
   ▼
Shared Access Signature
   │
   ▼
Azure Blob Storage
   │
   ▼
Administrative Container
   │
   ▼
Service Principal
   │
   ▼
Microsoft Entra ID
   │
   ▼
Azure Key Vault
```

This room demonstrates why cloud trust boundaries must be secured end-to-end.

---

# 🚨 Security Findings

| Finding                               | Severity    |
| ------------------------------------- | ----------- |
| Client-side SAS Token Exposure        | 🔴 High     |
| Hidden Blob Container Enumeration     | 🔴 High     |
| Service Principal Credential Exposure | 🔴 Critical |
| Key Vault Metadata Enumeration        | 🟠 Medium   |
| Historical Secret Version Exposure    | 🔴 High     |

---

# 🎯 MITRE ATT&CK Techniques

| Technique | Description                    |
| --------- | ------------------------------ |
| **T1552** | Unsecured Credentials          |
| **T1528** | Steal Application Access Token |
| **T1526** | Cloud Service Discovery        |
| **T1078** | Valid Accounts                 |
| **T1087** | Account Discovery              |
| **T1550** | Use of Stolen Credentials      |

---

# 🛡️ Defensive Recommendations

## Azure Storage

* Avoid embedding SAS tokens inside frontend code.
* Use least-privilege SAS permissions.
* Restrict SAS expiration windows.
* Separate administrative storage from public assets.

## Azure Identity

* Replace Service Principals with Managed Identities.
* Rotate exposed credentials immediately.
* Store secrets only inside Azure Key Vault.

## Azure Key Vault

* Review RBAC assignments regularly.
* Audit secret metadata permissions.
* Remove obsolete secret versions after rotation.

## Monitoring

Enable alerts for:

* Blob container enumeration.
* Service Principal sign-ins.
* Secret version enumeration.
* Azure Key Vault metadata access.

---

# 📚 Skills Demonstrated

<table>
<tr>
<th>Cloud Security</th>
<th>Offensive Security</th>
</tr>

<tr>
<td>

* Azure Storage
* Blob Storage
* Key Vault
* Microsoft Entra ID
* RBAC
* Azure REST APIs

</td>

<td>

* Client-side Reconnaissance
* Storage Enumeration
* Cloud Identity Abuse
* Secret Discovery
* Metadata Enumeration
* Trust Chain Analysis

</td>
</tr>
</table>

---

# 📂 Repository Resources

| File                               | Description                                     |
| ---------------------------------- | ----------------------------------------------- |
| **README.md**                      | Repository overview and walkthrough summary.    |
| **Documentation/Documentation.md** | Complete technical assessment report.           |
| **Resources/notes.md**             | Azure security notes and pentesting cheatsheet. |
| **Screenshots/**                   | Evidence collected during the assessment.       |

---

# 📖 References

* TryHackMe — CryptoCabana.
* Microsoft Learn — Azure Storage SAS Tokens.
* Microsoft Learn — Azure Blob Storage REST API.
* Microsoft Learn — Azure Key Vault.
* Microsoft Learn — Azure RBAC.

---

# ⚠️ Portfolio Disclaimer

This repository documents an **authorized TryHackMe cloud security lab** completed for cybersecurity education and portfolio development.

The following items have been intentionally removed:

* ❌ TryHackMe Flag
* ❌ SAS Token Signature
* ❌ Azure Client Secret
* ❌ OAuth Access Tokens
* ❌ Tenant ID
* ❌ Subscription ID
* ❌ Secret Values
* ❌ Historical Secret Shards

The repository demonstrates **cloud penetration testing methodology and Azure security analysis** without publishing challenge solutions.

---

<div align="center">

## ⭐ Cybersecurity Portfolio Project

### CryptoCabana — Azure Cloud Security Walkthrough

*Built with Microsoft Azure • TryHackMe • Cloud Security • Ethical Hacking*

**Author — Anurag Revankar**

</div>
