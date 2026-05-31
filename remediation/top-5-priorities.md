# Top 5 Remediation Priorities

These five actions cover the highest-impact findings and can all be done in under an hour across both environments.

---

## 1. Remove the Editor Role from GCP Default Service Accounts

**Finding:** GCP-001 | **CIS:** GCP v1.3 — 1.4

```bash
# Find the default Compute Engine service account
gcloud iam service-accounts list \
  --filter="displayName:Compute Engine default service account"

# Remove the Editor role
gcloud projects remove-iam-policy-binding <PROJECT_ID> \
  --member="serviceAccount:<PROJECT_NUMBER>-compute@developer.gserviceaccount.com" \
  --role="roles/editor"

# If the VM doesn't need a service account, remove it entirely
# Edit instance → Service account → No service account
```

---

## 2. Enable Network Flow Logs on Both Platforms

**Findings:** AZ-001, GCP-002 | **CIS:** Azure v2.0 — 6.4 · GCP v1.3 — 3.9

**Azure:**
```bash
az network watcher flow-log create \
  --resource-group <RG> \
  --nsg <NSG_NAME> \
  --storage-account <STORAGE_ACCOUNT_ID> \
  --enabled true \
  --retention 90 \
  --log-version 2
```

**GCP:**
```bash
gcloud compute networks subnets update <SUBNET_NAME> \
  --region=<REGION> \
  --enable-flow-logs \
  --logging-flow-sampling=1.0 \
  --logging-metadata=include-all
```

---

## 3. Disable Public Blob Access on Azure Storage

**Finding:** AZ-002 | **CIS:** Azure v2.0 — 3.5

```bash
az storage account update \
  --name <STORAGE_ACCOUNT_NAME> \
  --resource-group <RG> \
  --allow-blob-public-access false

# Verify
az storage account show \
  --name <STORAGE_ACCOUNT_NAME> \
  --query allowBlobPublicAccess
```

Enforce going forward via Azure Policy: `Deny-Storage-AllowBlobPublicAccess`

---

## 4. Enable Data Access Audit Logs

**Findings:** GCP-003, AZ-003 | **CIS:** Azure v2.0 — 5.1 · GCP v1.3 — 2.1

**GCP — via Console:**
```
IAM & Admin → Audit Logs
Select each service → enable DATA_READ and DATA_WRITE

Apply to: Cloud Storage, Compute Engine, Cloud KMS, BigQuery, Cloud SQL
```

**Azure — via CLI:**
```bash
az monitor diagnostic-settings create \
  --name "storage-diagnostics" \
  --resource <STORAGE_ACCOUNT_ID> \
  --workspace <LOG_ANALYTICS_WORKSPACE_ID> \
  --logs '[{"category":"StorageRead","enabled":true},
           {"category":"StorageWrite","enabled":true},
           {"category":"StorageDelete","enabled":true}]'
```

---

## 5. Alert on IAM Changes

**Findings:** AZ-005, GCP-006 | **CIS:** Azure v2.0 — 5.2 · GCP v1.3 — 2.4, 2.5

**GCP — log-based metrics:**
```bash
# IAM policy changes
gcloud logging metrics create iam-policy-changes \
  --log-filter='protoPayload.methodName="SetIamPolicy"'

# Service account key creation
gcloud logging metrics create sa-key-creation \
  --log-filter='protoPayload.methodName="google.iam.admin.v1.CreateServiceAccountKey"'

# Then create alert policies in Cloud Monitoring on these metrics
```

**Azure — Sentinel KQL query:**
```kql
AzureActivity
| where OperationNameValue contains "roleAssignments"
    or OperationNameValue contains "policyAssignments"
| where ActivityStatusValue == "Success"
```

Set alert threshold to trigger on any result.
