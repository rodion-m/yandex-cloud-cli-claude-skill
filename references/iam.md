# Yandex Cloud IAM CLI Reference

Complete guide to managing IAM resources in Yandex Cloud using the `yc` CLI.

**Последнее обновление:** 2026-01-16

---

## Table of Contents

1. [Service Account Management](#service-account-management)
2. [Key Types and Differences](#key-types-and-differences)
3. [Creating Keys](#creating-keys)
4. [Role Assignment](#role-assignment)
5. [Common Roles Reference](#common-roles-reference)
6. [CI/CD Use Cases](#cicd-use-cases)
7. [Command Reference](#command-reference)

---

## Service Account Management

### Creating a Service Account

**Basic syntax:**
```bash
yc iam service-account create --name <SERVICE_ACCOUNT_NAME>
```

**With description:**
```bash
yc iam service-account create \
  --name my-service-account \
  --description "Service account for production deployments"
```

**With async flag (for faster execution):**
```bash
yc iam service-account create \
  --name cameda-service \
  --description "Main service account" \
  --async
```

**With folder specification:**
```bash
yc iam service-account create \
  --name my-service-account \
  --folder-id <FOLDER_ID>
```

### Listing Service Accounts

```bash
yc iam service-account list
```

### Getting Service Account Details

```bash
yc iam service-account get <SERVICE_ACCOUNT_NAME>
yc iam service-account get --id <SERVICE_ACCOUNT_ID>
```

### Deleting a Service Account

```bash
yc iam service-account delete <SERVICE_ACCOUNT_NAME>
yc iam service-account delete --id <SERVICE_ACCOUNT_ID>
```

---

## Key Types and Differences

Yandex Cloud provides three types of keys for service accounts, each designed for specific use cases:

### 1. API Keys

- **Purpose:** Simplified authorization in Yandex Cloud API
- **Use case:** When automatic IAM token requests are not an option
- **Validity:** Can be limited during creation
- **Security:** Supports scope limits to reduce unauthorized use risk
- **Command:** `yc iam api-key create`

### 2. Authorized Keys (IAM Keys)

- **Purpose:** Public/private key pair for JWT creation and IAM token requests
- **Use case:** When you need full control over IAM token generation stages
- **Validity:** Unlimited
- **Format:** JSON file containing public and private keys
- **Command:** `yc iam key create`

### 3. Static Access Keys (AWS-Compatible)

- **Purpose:** Authentication for AWS-compatible services
- **Use case:** Yandex Object Storage, Yandex Managed Service for YDB
- **Validity:** Unlimited
- **Format:** Access Key ID and Secret Access Key (like AWS)
- **Command:** `yc iam access-key create`

**Summary:**
- **API keys** → Simplified authentication for certain services
- **Authorized keys** → Full control over IAM token generation
- **Static access keys** → AWS-compatible services (S3, etc.)

---

## Creating Keys

### API Key

**Basic creation:**
```bash
yc iam api-key create --service-account-name my-robot
```

**With description:**
```bash
yc iam api-key create \
  --service-account-name my-robot \
  --description "API key for automated tasks"
```

**Using service account ID:**
```bash
yc iam api-key create --service-account-id <SA_ID>
```

**Save output to file:**
```bash
yc iam api-key create --service-account-id $SA > api-key.yaml
```

### Authorized Key (IAM Key)

**Create and save to JSON file:**
```bash
yc iam key create \
  --service-account-name my-robot \
  --output key.json
```

**With description:**
```bash
yc iam key create \
  --service-account-name my-robot \
  --output key.json \
  --description "Key for CI/CD pipeline"
```

**Using service account ID:**
```bash
yc iam key create \
  --service-account-id <SA_ID> \
  --output key.json
```

### Static Access Key (for Object Storage)

**Basic creation:**
```bash
yc iam access-key create --service-account-name my-robot
```

**With description:**
```bash
yc iam access-key create \
  --service-account-name my-robot \
  --description "Access key for S3 bucket operations"
```

**Save to file:**
```bash
yc iam access-key create \
  --service-account-id $SA > static-access-key.txt
```

### Complete Workflow Example

```bash
# 1. Create service account
yc iam service-account create \
  --name deploy-service \
  --description "Deployment automation account" \
  --async

# 2. Get service account ID
SA_ID=$(yc iam service-account get deploy-service --format json | jq -r '.id')

# 3. Create all key types
yc iam api-key create --service-account-id $SA_ID > api-key.yaml
yc iam key create --service-account-id $SA_ID --output authorized-key.json
yc iam access-key create --service-account-id $SA_ID > s3-access-key.txt
```

---

## Role Assignment

### Understanding Role Assignment

There are two distinct commands for role assignment:

1. **`yc iam service-account add-access-binding`** - Controls who can ACCESS the service account as a resource
2. **`yc resource-manager folder add-access-binding`** - Controls what resources the service account can ACCESS

### Assigning Roles TO a Service Account (for folder resources)

**Basic syntax:**
```bash
yc resource-manager folder add-access-binding <FOLDER_NAME> \
  --role <ROLE_NAME> \
  --subject serviceAccount:<SERVICE_ACCOUNT_ID>
```

**Example - Editor role:**
```bash
yc resource-manager folder add-access-binding my-folder \
  --role editor \
  --subject serviceAccount:aje6o61dvog2h6g9a33s
```

**Example - Viewer role:**
```bash
yc resource-manager folder add-access-binding my-folder \
  --role viewer \
  --subject serviceAccount:aje6o61dvog2h6g9a33s
```

**Using folder ID:**
```bash
yc resource-manager folder add-access-binding \
  --id <FOLDER_ID> \
  --role compute.editor \
  --subject serviceAccount:<SA_ID>
```

### Multiple Role Assignment

```bash
# Assign multiple roles to the same service account
yc resource-manager folder add-access-binding --id <FOLDER_ID> \
  --role compute.editor \
  --subject serviceAccount:<SA_ID>

yc resource-manager folder add-access-binding --id <FOLDER_ID> \
  --role vpc.admin \
  --subject serviceAccount:<SA_ID>

yc resource-manager folder add-access-binding --id <FOLDER_ID> \
  --role load-balancer.editor \
  --subject serviceAccount:<SA_ID>
```

### Assigning Access TO a Service Account Resource

**Allow a user to manage a service account:**
```bash
yc iam service-account add-access-binding my-robot \
  --role editor \
  --subject userAccount:gfei8n54hmfhuk5nogse
```

**Allow another service account to manage it:**
```bash
yc iam service-account add-access-binding my-robot \
  --role editor \
  --subject serviceAccount:ajebqtreob2dpblin8pe
```

**Allow system groups:**
```bash
yc iam service-account add-access-binding my-robot \
  --role viewer \
  --subject system:allAuthenticatedUsers
```

### Cloud-Level Role Assignment

```bash
yc resource-manager cloud add-access-binding <CLOUD_NAME> \
  --role <ROLE_NAME> \
  --subject serviceAccount:<SERVICE_ACCOUNT_ID>
```

---

## Common Roles Reference

### General Roles

| Role | Description |
|------|-------------|
| `viewer` | Read-only access to resources |
| `editor` | View and modify resources, but not manage access |
| `admin` | Full access including access management |

### Compute Roles

| Role | Description |
|------|-------------|
| `compute.admin` | Full access to Compute Cloud resources |
| `compute.editor` | Create, modify, and delete VMs and related resources |
| `compute.viewer` | View Compute Cloud resources |

### Storage Roles

| Role | Description |
|------|-------------|
| `storage.admin` | Full access to Object Storage (create/delete buckets, manage ACL) |
| `storage.editor` | Create, modify, and delete buckets and objects |
| `storage.viewer` | Read access to buckets and objects |
| `storage.uploader` | Upload objects to buckets |

### Network Roles

| Role | Description |
|------|-------------|
| `vpc.admin` | Full access to VPC resources |
| `vpc.editor` | Create and modify networks, subnets, security groups |
| `vpc.viewer` | View VPC resources |

### Container Registry Roles

| Role | Description |
|------|-------------|
| `container-registry.admin` | Full access to Container Registry |
| `container-registry.images.pusher` | Push images to registry |
| `container-registry.images.puller` | Pull images from registry |

### Functions Roles

| Role | Description |
|------|-------------|
| `functions.admin` | Full access to Cloud Functions |
| `functions.editor` | Create, modify, and delete functions |
| `functions.viewer` | View functions |

### Load Balancer Roles

| Role | Description |
|------|-------------|
| `load-balancer.admin` | Full access to Load Balancer |
| `load-balancer.editor` | Create and modify load balancers |
| `load-balancer.viewer` | View load balancers |

---

## CI/CD Use Cases

### GitHub Actions Authentication

Yandex Cloud provides three authentication methods for GitHub Actions:

1. **`yc-sa-json-credentials`** - JSON with authorized key for service account (most common)
2. **`yc-iam-token`** - IAM token
3. **`yc-sa-id`** - Service Account ID (uses Workload Identity Federation - most secure)

### Container Registry Deployment

**Required roles:**
- `container-registry.images.pusher` - Push images
- `container-registry.images.puller` - Pull private images

**Example setup:**
```bash
# Create service account
yc iam service-account create --name github-ci

# Get SA ID
SA_ID=$(yc iam service-account get github-ci --format json | jq -r '.id')

# Assign role
yc resource-manager folder add-access-binding my-folder \
  --role container-registry.images.pusher \
  --subject serviceAccount:$SA_ID

# Create authorized key for GitHub Actions
yc iam key create \
  --service-account-id $SA_ID \
  --output github-ci-key.json
```

### Serverless Functions Deployment

**Required role:** `functions.editor`

```bash
yc resource-manager folder add-access-binding my-folder \
  --role functions.editor \
  --subject serviceAccount:$SA_ID
```

### Virtual Machine Management

**Required role:** `compute.admin`

```bash
yc resource-manager folder add-access-binding my-folder \
  --role compute.admin \
  --subject serviceAccount:$SA_ID
```

### Serverless Container Deployment

**Typical roles needed:**
- `serverless.containers.admin` - Deploy containers
- `storage.viewer` - Mount buckets
- `lockbox.payloadViewer` - Access secrets
- `kms.keys.decrypter` - Decrypt KMS-encrypted secrets

```bash
# Assign multiple roles for serverless container
yc resource-manager folder add-access-binding my-folder \
  --role serverless.containers.admin \
  --subject serviceAccount:$SA_ID

yc resource-manager folder add-access-binding my-folder \
  --role lockbox.payloadViewer \
  --subject serviceAccount:$SA_ID
```

### Complete CI/CD Example

```bash
#!/bin/bash

# Create service account for CI/CD
SA_NAME="ci-cd-deploy"
FOLDER_NAME="production"

# 1. Create service account
yc iam service-account create \
  --name $SA_NAME \
  --description "CI/CD deployment automation"

# 2. Get service account ID
SA_ID=$(yc iam service-account get $SA_NAME --format json | jq -r '.id')

# 3. Assign necessary roles
yc resource-manager folder add-access-binding $FOLDER_NAME \
  --role container-registry.images.pusher \
  --subject serviceAccount:$SA_ID

yc resource-manager folder add-access-binding $FOLDER_NAME \
  --role functions.editor \
  --subject serviceAccount:$SA_ID

yc resource-manager folder add-access-binding $FOLDER_NAME \
  --role compute.editor \
  --subject serviceAccount:$SA_ID

# 4. Create authorized key for GitHub Actions
yc iam key create \
  --service-account-id $SA_ID \
  --output ci-cd-key.json \
  --description "Key for GitHub Actions"

echo "Service account $SA_NAME created with ID: $SA_ID"
echo "Authorized key saved to ci-cd-key.json"
echo "Add this JSON content to GitHub Secrets as YC_SA_JSON_CREDENTIALS"
```

---

## Command Reference

### Service Account Commands

```bash
# Create
yc iam service-account create --name <NAME> [--description <DESC>]

# List
yc iam service-account list

# Get details
yc iam service-account get <NAME_OR_ID>

# Delete
yc iam service-account delete <NAME_OR_ID>

# Update
yc iam service-account update <NAME_OR_ID> --new-name <NEW_NAME>
```

### Key Management Commands

```bash
# API Key
yc iam api-key create --service-account-name <NAME>
yc iam api-key list --service-account-name <NAME>
yc iam api-key delete <KEY_ID>

# Authorized Key (IAM Key)
yc iam key create --service-account-name <NAME> --output <FILE>
yc iam key list --service-account-name <NAME>
yc iam key delete <KEY_ID>

# Access Key (Static)
yc iam access-key create --service-account-name <NAME>
yc iam access-key list --service-account-name <NAME>
yc iam access-key delete <KEY_ID>
```

### Role Assignment Commands

```bash
# Assign role to service account (for folder resources)
yc resource-manager folder add-access-binding <FOLDER> \
  --role <ROLE> \
  --subject serviceAccount:<SA_ID>

# Assign role to service account (for cloud resources)
yc resource-manager cloud add-access-binding <CLOUD> \
  --role <ROLE> \
  --subject serviceAccount:<SA_ID>

# Control who can access the service account
yc iam service-account add-access-binding <SA_NAME> \
  --role <ROLE> \
  --subject userAccount:<USER_ID>
```

### Get Help

```bash
# General help
yc iam --help

# Service account help
yc iam service-account --help
yc iam service-account create --help

# Key creation help
yc iam api-key create --help
yc iam key create --help
yc iam access-key create --help
```

---

## Quick Reference Cheat Sheet

### Common Operations

```bash
# Create service account with all key types
SA_NAME="my-service"
yc iam service-account create --name $SA_NAME
SA_ID=$(yc iam service-account get $SA_NAME --format json | jq -r '.id')
yc iam api-key create --service-account-id $SA_ID > api-key.yaml
yc iam key create --service-account-id $SA_ID --output key.json
yc iam access-key create --service-account-id $SA_ID > access-key.txt

# Assign editor role to folder
yc resource-manager folder add-access-binding my-folder \
  --role editor \
  --subject serviceAccount:$SA_ID

# List all service accounts
yc iam service-account list

# List all keys for a service account
yc iam api-key list --service-account-name $SA_NAME
yc iam key list --service-account-name $SA_NAME
yc iam access-key list --service-account-name $SA_NAME
```

### Subject Types

```bash
# User account
--subject userAccount:<USER_ID>

# Service account
--subject serviceAccount:<SA_ID>

# System groups
--subject system:allUsers                    # All users (including unauthorized)
--subject system:allAuthenticatedUsers       # All authenticated users
```

---

## Sources

- [Yandex Cloud IAM Service Account CLI Reference](https://cloud.yandex.com/en/docs/cli/cli-ref/managed-services/iam/service-account/)
- [Creating API Keys](https://cloud.yandex.com/en/docs/iam/operations/api-key/create)
- [Creating Static Access Keys](https://cloud.yandex.com/en/docs/iam/operations/sa/create-access-key)
- [Assigning Roles to Service Accounts](https://cloud.yandex.com/en/docs/iam/operations/sa/assign-role-for-sa)
- [Setting up Folder Access Rights](https://cloud.yandex.com/en/docs/resource-manager/operations/folder/set-access-bindings)
- [Service Account Add Access Binding](https://cloud.yandex.com/en/docs/cli/cli-ref/managed-services/iam/service-account/add-access-binding)
- [API Key Documentation](https://cloud.yandex.com/en/docs/iam/concepts/authorization/api-key)
- [Authorized Keys Documentation](https://cloud.yandex.com/en/docs/iam/concepts/authorization/key)
- [Static Access Keys Documentation](https://cloud.yandex.com/en/docs/iam/concepts/authorization/access-key)
- [GitHub Actions for Yandex Cloud](https://github.com/yc-actions)
- [Yandex Cloud Roles](https://cloud.yandex.com/en/docs/iam/concepts/access-control/roles)
