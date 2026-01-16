# Profile Management Reference

## Multi-Region Setup

Yandex Cloud operates in multiple regions with different endpoints:

| Region | Endpoint | OAuth | Console |
|--------|----------|-------|---------|
| Russia | api.cloud.yandex.net:443 | oauth.yandex.ru | console.yandex.cloud |
| Kazakhstan | api.yandexcloud.kz:443 | oauth.yandex.kz | kz.console.yandex.cloud |

## Profile Commands

```bash
# List all profiles
yc config profile list

# Show current config
yc config list

# Get specific profile config
yc config profile get <name>

# Create profile
yc config profile create <name>

# Activate profile
yc config profile activate <name>

# Delete profile
yc config profile delete <name>
```

## Setting Up Russia Profile

```bash
# Create and activate
yc config profile create russia
yc config profile activate russia

# Set endpoint (default for Russia, but explicit is better)
yc config set endpoint api.cloud.yandex.net:443

# Get OAuth token from:
# https://oauth.yandex.ru/authorize?response_type=token&client_id=1a6990aa636648e9b2ef855fa7bec2fb

# Set token
yc config set token <your-token>

# Select cloud
yc resource-manager cloud list
yc config set cloud-id <cloud-id>

# Select folder
yc resource-manager folder list
yc config set folder-id <folder-id>
```

## Setting Up Kazakhstan Profile

```bash
# Create and activate
yc config profile create turbocode-kz
yc config profile activate turbocode-kz

# Set Kazakhstan endpoint
yc config set endpoint api.yandexcloud.kz:443

# Get OAuth token from:
# https://oauth.yandex.kz/authorize?response_type=token&client_id=1a6990aa636648e9b2ef855fa7bec2fb

# Set token
yc config set token <your-token>

# Select cloud and folder
yc resource-manager cloud list
yc config set cloud-id <cloud-id>
yc resource-manager folder list
yc config set folder-id <folder-id>
```

## Using Profiles

```bash
# Quick switch
yc config profile activate russia
yc config profile activate turbocode-kz

# Run single command with different profile (no permanent switch)
yc compute instance list --profile russia
yc compute instance list --profile turbocode-kz
```

## Config Properties

```bash
# Set individual properties
yc config set token <value>
yc config set cloud-id <value>
yc config set folder-id <value>
yc config set compute-default-zone kz1-a
yc config set endpoint api.yandexcloud.kz:443

# Unset property
yc config unset compute-default-zone
```

## Service Account Authentication

For automation and CI/CD:

```bash
# Create service account key
yc iam key create --service-account-name my-sa --output key.json

# Authenticate with key file
yc config set service-account-key key.json

# Or use environment variable
export YC_SERVICE_ACCOUNT_KEY_FILE=/path/to/key.json
```

## Kazakhstan Region Limitations

Services NOT available in Kazakhstan:
- Serverless (API Gateway, Cloud Functions, Serverless Containers)
- Data Streams, Cloud Postbox, Yandex Query
- IoT Core, Managed YDB, CDN
- SpeechKit (partial)

Services available:
- Compute, Object Storage, VPC
- Lockbox, Cloud DNS, Monitoring
- Message Queue, Certificate Manager
- Container Registry, Key Management Service
- Data Transfer, Cloud Logging
