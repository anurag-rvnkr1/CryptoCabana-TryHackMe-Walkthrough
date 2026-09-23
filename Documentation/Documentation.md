# TryHackMe - CryptoCabana
## Professional Cloud Security CTF Documentation

> **Scope:** Authorized TryHackMe lab only  
> **Difficulty:** Medium  
> **Category:** Cloud / Azure  
> **Portfolio edition:** Sensitive values and flags redacted

---

## 1. Executive Summary

CryptoCabana models an Azure-backed cryptocurrency backup service. The challenge is solved by following a chain of cloud trust relationships rather than by attacking a conventional host directly.

The investigation begins at an Azure Static Website. Client-side JavaScript reveals storage configuration and a SAS token. Although the token does not provide write access, its read/list capability permits Blob Storage enumeration. An additional `vault` container contains a service-principal configuration file.

The recovered service identity can authenticate to Azure. Its Key Vault permissions allow discovery of secret objects and metadata, while the obvious current-value retrieval path is denied by RBAC. The final pivot is to inspect Key Vault secret-version history through the REST API and request a prior revision. One shard contains the historical material required to reconstruct the protected room answer.

The public repository intentionally omits the real TryHackMe flag, live credentials, SAS signatures, access tokens, and exact shard contents.

---

## 2. Attack Chain

```text
Azure Static Website
        |
        v
Client-side JavaScript
        |
        v
Hardcoded Blob SAS
        |
        v
Blob container enumeration
        |
        v
Hidden "vault" container
        |
        v
Service-principal configuration
        |
        v
Azure CLI authentication
        |
        v
Key Vault secret enumeration
        |
        v
RBAC blocks current value
        |
        v
Key Vault REST API
        |
        v
Secret-version metadata
        |
        v
Historical shard retrieval
        |
        v
Shard reconstruction
        |
        v
FLAG - REDACTED
```

---

## 3. Scope and Safety

This write-up documents work performed against the TryHackMe CryptoCabana lab. The techniques are presented for authorized security testing and education.

The public version excludes:
- the live room flag
- SAS signatures
- service-principal secrets
- access tokens
- exact shard values
- other challenge-specific authentication material

---

## 4. Initial Reconnaissance

The target is an Azure Static Website. Its landing page is deliberately minimal, so the first useful question is what the browser is being given behind the interface.

![Landing Page](../Screenshots/01_cryptocabana_landing.png)

**Figure 1 - CryptoCabana landing page viewed from a local browser session.**

The application itself does not expose the storage architecture directly. That makes frontend inspection the natural next step.

---

## 5. Client-Side Configuration Review

Reviewing page source and JavaScript resources reveals storage configuration and a SAS token delivered to the browser.

![Frontend JavaScript](../Screenshots/02_frontend_appjs.png)

**Figure 2 - Sanitized `app.js` configuration showing the client-exposed storage access artifact.**

The screenshot removes the real token signature and environment-specific identifiers.

### Security observation

A value required by the browser is not a secret. Once a SAS token is present in client-side code, an observer can extract and replay it against the storage endpoint for the lifetime of that token.

---

## 6. SAS Permission Analysis

The important field is:

`sp=rl`

Relevant permissions:
- `r` - read
- `l` - list

A write request is expected to fail because the token lacks write/create capability.

![SAS Permission Check](../Screenshots/03_sas_permissions_403.png)

**Figure 3 - Local PowerShell validation of the permission boundary and resulting 403 response.**

Azure documents that the SAS `sp` field determines which operations the token can perform.

---

## 7. Blob Storage Enumeration

The token is then used directly against the Blob service.

![Blob Enumeration](../Screenshots/04_blob_container_enum.png)

**Figure 4 - Sanitized Blob Storage enumeration from a local PowerShell session.**

The lab reveals:
- `$web`
- `backups`
- `vault`

The `vault` container is not referenced by the web interface. Enumerating its objects reveals `backup-service-account.json`.

This is the key transition from application-level observation to cloud-resource discovery.

---

## 8. Recovering the Service Identity

The discovered JSON file contains Azure authentication material and Key Vault information. In the portfolio copy, these values are placeholders.

![Service Principal JSON](../Screenshots/05_service_principal_json.png)

**Figure 5 - Sanitized service-principal configuration recovered from Blob Storage.**

The meaningful trust relationship is:

`Blob read/list access -> credential material -> Azure service identity`

---

## 9. Azure Authentication

The recovered lab identity can be supplied to Azure CLI:

```powershell
az login --service-principal `
  --username "<REDACTED-CLIENT-ID>" `
  --password "<REDACTED-CLIENT-SECRET>" `
  --tenant "<REDACTED-TENANT-ID>"
```

![Azure CLI Login](../Screenshots/06_azure_login.png)

**Figure 6 - Successful Azure CLI authentication with sensitive identifiers removed.**

Identity context can then be confirmed with `az account show`.

---

## 10. Key Vault Enumeration and RBAC Boundary

The service identity can enumerate secret objects:

```powershell
az keyvault secret list --vault-name <REDACTED-VAULT>
```

The lab exposes several shard-like secret names and a master-key object. A direct value retrieval is denied.

![Key Vault RBAC](../Screenshots/07_keyvault_rbac.png)

**Figure 7 - Key Vault enumeration followed by the RBAC denial on direct current-value retrieval.**

The observed boundary is:

```text
List metadata      = allowed
Current value      = denied
```

At this point the room hint about recent rotation becomes important.

---

## 11. Inspecting Key Vault Secret Versions

An Azure access token can be obtained for the Key Vault resource:

```powershell
az account get-access-token `
  --resource https://vault.azure.net `
  --query accessToken -o tsv
```

The secret-version endpoint can then be queried for a candidate shard:

```http
GET https://<vault>.vault.azure.net/secrets/<secret>/versions?api-version=7.4
Authorization: Bearer <REDACTED>
```

![Secret Versions](../Screenshots/08_rest_secret_versions.png)

**Figure 8 - Sanitized REST API request showing two versions of one shard.**

Microsoft documents that listing secret versions returns version identifiers and attributes such as creation and update information, and requires the relevant list permission.

---

## 12. Historical Version Retrieval

A specific secret revision can be requested by including its version identifier in the URL:

```http
GET https://<vault>.vault.azure.net/secrets/<secret>/<version>?api-version=7.4
Authorization: Bearer <REDACTED>
```

![Historical Version](../Screenshots/09_historical_version.png)

**Figure 9 - Historical secret-version retrieval with shard content and final flag redacted.**

Microsoft documents that specifying the version targets that exact secret revision, while omitting the version requests the latest version.

In the lab, the older revision provides the missing shard material even though the obvious current-value retrieval route is blocked.

---

## 13. Shard Reconstruction

The challenge stores the protected answer across multiple secret fragments. The historical revision supplies the missing fragment after the rotation clue is followed.

The public write-up therefore records only the methodology:

```text
Enumerate shards
  -> identify rotated shard
  -> enumerate versions
  -> select historical revision
  -> retrieve historical value
  -> combine shard material
  -> submit room answer
```

**Final flag: `THM{REDACTED}`**

---

## 14. Technical Findings

### Finding 1 - Client-exposed SAS
Storage authorization is shipped to the browser, making it discoverable and replayable.

### Finding 2 - Excessive storage reach
The authorization can enumerate a container outside the application workflow.

### Finding 3 - Credential exposure
A service-principal configuration file is stored within accessible Blob Storage.

### Finding 4 - Discovery privileges
The recovered identity can enumerate Key Vault objects and metadata even though direct current-value retrieval is blocked.

### Finding 5 - Historical version exposure
A previous secret revision remains useful after rotation.

---

## 15. Root Cause Analysis

The compromise results from multiple trust-boundary failures:

1. Browser-visible authorization was treated as if it were secret.
2. The storage permission scope exceeded the minimum required application path.
3. Administrative credentials were stored with accessible cloud data.
4. The service identity exposed meaningful Key Vault discovery capabilities.
5. Secret rotation did not remove the utility of the historical revision.

This is the central lesson of the room: cloud controls must be assessed as a connected system, not as isolated services.

---

## 16. Defensive Recommendations

- Keep durable credentials and privileged configuration server-side.
- Prefer Entra ID-backed application authorization where practical.
- When SAS is required, scope it to the smallest possible resource, permission set, and lifetime.
- Do not store service-principal secrets in Blob Storage or frontend assets.
- Prefer managed identities/workload identities over static client secrets where supported.
- Apply least privilege to Key Vault roles.
- Review historical secret versions as part of the secret lifecycle.
- Monitor Blob and Key Vault enumeration and unusual identity activity.

Microsoft recommends using Microsoft Entra credentials with user-delegation SAS where a SAS design is required, when possible.

---

## 17. MITRE ATT&CK Mapping

| Technique | Name | Relevance |
|---|---|---|
| T1552 | Unsecured Credentials | Credential material is recovered from cloud storage. |
| T1078 | Valid Accounts | A recovered service identity is used for authenticated cloud access. |
| T1087 | Account Discovery | Azure identity/account context is inspected after authentication. |
| T1526 | Cloud Service Discovery | Storage and Key Vault services are identified and queried. |
| T1528 | Steal Application Access Token | Client-exposed authorization is treated as a reusable access artifact. |

---

## 18. Tools Used

- Browser developer tools
- PowerShell
- Azure CLI
- cURL
- Azure Blob Storage REST endpoints
- Azure Key Vault REST API

---

## 19. Lessons Learned

The room demonstrates a realistic cloud trust chain:

```text
Client exposure
   ↓
Limited storage authorization
   ↓
Hidden resource discovery
   ↓
Credential disclosure
   ↓
Authenticated cloud identity
   ↓
Metadata discovery
   ↓
Secret-version analysis
   ↓
Historical-value recovery
```

A permission that appears restrictive in isolation can still contribute to a full compromise when combined with another exposed relationship.

---

## 20. References

- TryHackMe - CryptoCabana
- Microsoft Learn - Create a Service SAS
- Microsoft Learn - Azure Key Vault REST API
- Microsoft Learn - Get Secret
- Microsoft Learn - Get Secret Versions

---

## Portfolio Note

This edition is intentionally sanitized. The screenshots are reconstructed local lab-style visuals designed to resemble a realistic workstation session while ensuring challenge flags and credentials are not published.
