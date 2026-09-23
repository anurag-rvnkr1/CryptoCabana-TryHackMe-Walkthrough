# ☁️ CryptoCabana — Azure Cloud Security Walkthrough (TryHackMe)

<div align="center">

# 🌴 CryptoCabana

### *Breaking Trust Chains in Azure Cloud Infrastructure*

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Hacker%20Holidays-red?style=for-the-badge\&logo=tryhackme)](https://tryhackme.com/)
[![Azure](https://img.shields.io/badge/Microsoft-Azure-0078D4?style=for-the-badge\&logo=microsoftazure\&logoColor=white)](https://azure.microsoft.com/)
[![Cloud Security](https://img.shields.io/badge/Cloud-Security-0ea5e9?style=for-the-badge\&logo=icloud)](#)
[![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange?style=for-the-badge)](#)

*A professional cloud penetration testing walkthrough demonstrating Azure Storage exploitation, identity abuse, Key Vault reconnaissance, RBAC analysis, and cloud trust-chain compromise.*

**Author:** Anurag Revankar

</div>

---

## 📖 About This Repository

**CryptoCabana** is a cloud-focused **TryHackMe Hacker Holidays** room built around Microsoft Azure security.

Instead of exploiting a vulnerable web application, this room demonstrates how **misconfigured cloud services** can expose sensitive identities and secrets through a chain of seemingly minor security weaknesses.

This repository documents the **complete investigation process** performed during the lab while intentionally hiding all flags, secrets, access tokens, SAS signatures, and sensitive Azure identifiers.

> **Portfolio Edition**
>
> This walkthrough is rewritten from scratch for GitHub portfolio purposes and focuses on methodology, cloud architecture, detection opportunities, and defensive lessons rather than publishing challenge answers.

---

# 🎯 Objectives

* Investigate an Azure-hosted cryptocurrency backup application.
* Perform client-side cloud reconnaissance.
* Analyze Azure Storage SAS permissions.
* Enumerate Azure Blob Storage containers.
* Discover hidden cloud resources.
* Recover exposed Azure Service Principal credentials.
* Authenticate into Azure using Azure CLI.
* Enumerate Azure Key Vault resources.
* Analyze RBAC permission boundaries.
* Recover historical secret versions through Azure REST APIs.
* Understand the complete Azure attack chain.

---

# 🧩 Room Information

| Property           | Details                                                          |
| ------------------ | ---------------------------------------------------------------- |
| **Platform**       | TryHackMe                                                        |
| **Room**           | CryptoCabana                                                     |
| **Category**       | Cloud Security                                                   |
| **Difficulty**     | Medium                                                           |
| **Cloud Provider** | Microsoft Azure                                                  |
| **Focus Areas**    | Azure Storage, Blob Storage, Key Vault, Service Principals, RBAC |

---

# 🚀 Skills Demonstrated

<table>
<tr>
<td width="50%">

### Azure Cloud Security

* Azure Storage Enumeration
* Azure Blob Storage
* SAS Token Analysis
* Azure Static Websites
* Azure Service Principals
* Azure CLI Authentication
* Azure Key Vault
* RBAC Investigation
* Azure REST API

</td>
<td width="50%">

### Offensive Cloud Security

* Client-side Reconnaissance
* Cloud Resource Enumeration
* Identity Enumeration
* Secret Discovery
* Metadata Analysis
* Secret Version Enumeration
* Trust Relationship Analysis
* Cloud Misconfiguration Assessment

</td>
</tr>
</table>

---

# 🛠️ Tools & Technologies

| Tool             | Purpose                                 |
| ---------------- | --------------------------------------- |
| Azure CLI        | Cloud authentication and enumeration    |
| Curl             | Azure REST API requests                 |
| Browser DevTools | JavaScript inspection                   |
| PowerShell       | Blob enumeration                        |
| Azure REST API   | Secret metadata and version enumeration |
| Microsoft Azure  | Target cloud infrastructure             |

---

# 📂 Repository Structure

```text
CryptoCabana-TryHackMe-Walkthrough
│
├── README.md
├── LICENSE
├── SECURITY.md
│
├── Documentation
│   └── Documentation.md
│
├── Resources
│   └── notes.md
│
├── Screenshots
│   ├── 01_cryptocabana_landing.png
│   ├── 02_frontend_appjs.png
│   ├── 03_sas_permissions_403.png
│   ├── 04_blob_container_enum.png
│   ├── 05_service_principal_json.png
│   ├── 06_azure_login.png
│   ├── 07_keyvault_rbac.png
│   ├── 08_rest_secret_versions.png
│   └── 09_historical_version.png
│
└── docs
    ├── index.md
    └── assets
```

---

# 🌐 Challenge Scenario

A fictional Azure-hosted hotel offers guests a service to **securely back up cryptocurrency recovery phrases**.

The web application appears harmless.

Behind the scenes, however, multiple Azure resources trust one another in unsafe ways.

The challenge demonstrates how an attacker can move from a **public static website** to sensitive cloud secrets by abusing exposed permissions and cloud identities.

---

# ☁️ Azure Attack Chain

```text
                   Internet User
                         │
                         ▼
            Azure Static Website ($web)
                         │
                  Client-side JavaScript
                         │
                Hardcoded Storage SAS
                         │
                         ▼
          Azure Blob Storage Account
        ┌──────────────┬───────────────┐
        │              │               │
      $web         backups         vault
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
                                      │
                                      ▼
                  Room Flag Reconstruction (Redacted)
```

---

# 🔍 Walkthrough Overview

## Phase 1 — Azure Static Website Reconnaissance

The investigation begins with a publicly accessible Azure Static Website.

Minimal functionality suggests that the interesting logic exists client-side.

📸 **Screenshot**

`Screenshots/01_cryptocabana_landing.png`

---

## Phase 2 — Client-Side Cloud Configuration Discovery

Inspecting the JavaScript bundle reveals Azure Storage configuration embedded directly inside the frontend.

Key observations include:

* Storage Account
* Blob Container
* Storage Endpoint
* SAS Token

📸 **Screenshot**

`Screenshots/02_frontend_appjs.png`

---

## Phase 3 — SAS Permission Analysis

Rather than uploading data through the UI, the SAS token is analyzed independently.

The permission scope allows resource enumeration while preventing uploads.

This exposes an important cloud authorization boundary.

📸 **Screenshot**

`Screenshots/03_sas_permissions_403.png`

---

## Phase 4 — Azure Blob Storage Enumeration

Using the SAS token directly against Azure Blob Storage reveals containers that are not referenced anywhere in the application.

This demonstrates why **security through obscurity fails**.

📸 **Screenshot**

`Screenshots/04_blob_container_enum.png`

---

## Phase 5 — Hidden Administrative Container Discovery

A hidden container stores backup automation artifacts.

One configuration file exposes Azure Service Principal authentication details.

📸 **Screenshot**

`Screenshots/05_service_principal_json.png`

---

## Phase 6 — Azure Identity Authentication

The recovered identity successfully authenticates through Azure CLI.

The walkthrough analyzes what permissions become available after authentication.

📸 **Screenshot**

`Screenshots/06_azure_login.png`

---

## Phase 7 — Azure Key Vault Enumeration

The authenticated identity can enumerate Key Vault secrets but cannot retrieve current values.

This demonstrates an RBAC permission boundary.

📸 **Screenshot**

`Screenshots/07_keyvault_rbac.png`

---

## Phase 8 — Secret Version Enumeration

The room hint points toward secret rotation history.

Azure REST APIs expose metadata that identifies secrets with multiple historical versions.

📸 **Screenshot**

`Screenshots/08_rest_secret_versions.png`

---

## Phase 9 — Historical Secret Recovery

A previous secret version remains accessible.

The walkthrough explains the methodology without exposing any sensitive shard or room flag.

📸 **Screenshot**

`Screenshots/09_historical_version.png`

---

# 🔐 Security Findings

| Finding                               | Impact                                                   |
| ------------------------------------- | -------------------------------------------------------- |
| Client-side SAS Token Exposure        | Storage authorization leaked to every visitor.           |
| Overly Broad SAS Scope                | Read/List permissions allow unintended enumeration.      |
| Hidden Blob Container                 | Sensitive storage reachable through the same SAS.        |
| Service Principal Credential Exposure | Azure workload identity compromised.                     |
| Key Vault Metadata Enumeration        | Secret discovery without current-value access.           |
| Historical Secret Version Exposure    | Older secret material remains accessible after rotation. |

---

# 🧠 Root Cause Analysis

The compromise succeeds because several Azure security weaknesses combine together:

### Storage Trust

Client-side code exposes reusable cloud authorization.

### Resource Segmentation Failure

Hidden containers remain reachable with existing permissions.

### Identity Exposure

Blob Storage contains authentication material for an Azure workload identity.

### Authorization Boundary

RBAC blocks current secrets but still allows useful metadata enumeration.

### Secret Lifecycle Weakness

Historical Key Vault versions remain accessible after rotation.

---

# 🛡️ MITRE ATT&CK Mapping

| Technique | Description                    |
| --------- | ------------------------------ |
| **T1552** | Unsecured Credentials          |
| **T1078** | Valid Accounts                 |
| **T1087** | Account Discovery              |
| **T1526** | Cloud Service Discovery        |
| **T1528** | Steal Application Access Token |
| **T1550** | Use of Stolen Credentials      |

---

# 🛡️ Defensive Recommendations

* Avoid embedding SAS tokens inside frontend applications.
* Use Microsoft Entra ID instead of reusable client-side credentials.
* Generate short-lived, least-privilege SAS tokens.
* Isolate administrative storage containers from public storage accounts.
* Store Service Principal secrets inside Azure Key Vault only.
* Replace Service Principals with Managed Identities where possible.
* Audit Azure RBAC assignments regularly.
* Review Key Vault secret version access policies.
* Remove obsolete secret versions after credential rotation.
* Monitor Azure Activity Logs for storage and identity enumeration.

---

# 📚 What I Learned

This room demonstrates a realistic **cloud attack path** where an attacker chains together:

* Frontend reconnaissance
* Cloud storage enumeration
* Credential exposure
* Identity abuse
* Secret metadata discovery
* Historical secret recovery

It reinforces an important Azure security principle:

> **Cloud security depends on protecting trust relationships, not just individual resources.**

---

# 📈 Key Takeaways

<table>
<tr>
<td>

### Offensive Security

* Azure Storage Enumeration
* SAS Token Abuse
* Cloud Identity Discovery
* Key Vault Enumeration
* REST API Analysis
* Secret Version Investigation

</td>
<td>

### Defensive Security

* Least Privilege
* Secure SAS Design
* Identity Protection
* Key Vault Hardening
* RBAC Auditing
* Secret Lifecycle Management

</td>
</tr>
</table>

---

# ⚠️ Ethical Notice

This repository contains documentation for an **authorized TryHackMe lab** completed for cybersecurity learning and portfolio development.

### Sensitive Information Removed
* ❌ Room Flag
* ❌ SAS Token Signature
* ❌ Client Secret
* ❌ Tenant ID
* ❌ Access Tokens
* ❌ Secret Values
* ❌ Secret Shards
* ❌ Subscription Identifiers

The repository demonstrates **methodology and cloud security analysis only**.

---

# 🤝 Connect With Me

**Cybersecurity Portfolio • Cloud Security • Azure Security • TryHackMe Walkthroughs**

If this repository helped you understand Azure cloud attack paths, consider ⭐ starring the project.

---

<div align="center">

**Built for a Professional Cybersecurity Portfolio**

*Azure Cloud Security • TryHackMe • Ethical Hacking • Microsoft Azure*

</div>
