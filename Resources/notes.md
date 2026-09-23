# CryptoCabana - Quick Notes

## Room Profile

- Platform: TryHackMe
- Room: CryptoCabana
- Category: Cloud / Azure
- Difficulty: Medium

## Attack Path

```text
Static Website
 -> app.js
 -> SAS token
 -> Blob enumeration
 -> hidden vault
 -> service-principal JSON
 -> Azure CLI login
 -> Key Vault enumeration
 -> RBAC denial
 -> REST API
 -> secret versions
 -> historical shard
 -> shard reconstruction
 -> flag (redacted)
```

## Key Observations

- Client-side storage configuration is discoverable in JavaScript.
- SAS permission field observed: `sp=rl`.
- Write attempt returns HTTP 403.
- Containers observed: `$web`, `backups`, `vault`.
- Sensitive blob: `backup-service-account.json`.
- Service principal can authenticate to the lab.
- Direct current-value retrieval from Key Vault is denied with `ForbiddenByRbac`.
- One shard has two secret versions.
- Historical version is the important pivot.

## Core Commands

```powershell
curl "https://<storage>.blob.core.windows.net/?comp=list&<SAS>"
curl "https://<storage>.blob.core.windows.net/<container>?restype=container&comp=list&<SAS>"
az login --service-principal --username "<CLIENT-ID>" --password "<CLIENT-SECRET>" --tenant "<TENANT-ID>"
az account show
az keyvault secret list --vault-name <VAULT>
az account get-access-token --resource https://vault.azure.net --query accessToken -o tsv
```

```http
GET https://<vault>.vault.azure.net/secrets/<secret>/versions?api-version=7.4
Authorization: Bearer <REDACTED>
```

```http
GET https://<vault>.vault.azure.net/secrets/<secret>/<version>?api-version=7.4
Authorization: Bearer <REDACTED>
```

## Redaction Checklist

Never publish:
- SAS `sig` values
- Client secrets
- Access tokens
- Exact shard contents
- TryHackMe flags

## Learning Focus

- Treat browser-side configuration as public.
- Validate authorization boundaries with actual service responses.
- Enumerate underlying cloud services, not just application paths.
- Separate metadata permissions from data permissions.
- Treat old secret versions as part of the sensitive-data lifecycle.
