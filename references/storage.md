# Yandex Cloud Object Storage - CLI Command Reference

## Overview

Yandex Cloud Object Storage is an S3-compatible service manageable through:
1. **Native Yandex Cloud CLI** (`yc`) - bucket management
2. **AWS CLI** - comprehensive object and bucket operations

## Yandex Cloud CLI Commands

### Bucket Operations

```bash
# List buckets
yc storage bucket list

# Get bucket info
yc storage bucket get <bucket_name> --full

# Update bucket (public access, CORS)
yc storage bucket update <bucket_name> \
  --public-read \
  --public-list \
  --cors allowed-methods=GET,POST \
  --cors allowed-origins=https://example.com
```

## AWS CLI Setup

### Create Static Access Key

```bash
yc iam access-key create --service-account-name my-robot
```

**Save immediately** - secret key cannot be retrieved later.

### Configure AWS CLI

```bash
aws configure
# AWS Access Key ID: <your_access_key_id>
# AWS Secret Access Key: <your_secret_key>
# Default region: ru-central1
```

**Critical:** Always use `--endpoint-url=https://storage.yandexcloud.net`

## Bucket Operations (AWS CLI)

```bash
# Create bucket
aws --endpoint-url=https://storage.yandexcloud.net \
  s3 mb s3://bucket-name

# List buckets
aws --endpoint-url=https://storage.yandexcloud.net s3 ls

# List objects
aws --endpoint-url=https://storage.yandexcloud.net \
  s3 ls --recursive s3://bucket-name

# Delete bucket
aws --endpoint-url=https://storage.yandexcloud.net \
  s3 rb s3://bucket-name
```

## Object Operations

```bash
# Upload file
aws --endpoint-url=https://storage.yandexcloud.net \
  s3 cp local-file.txt s3://bucket-name/path/

# Upload directory (recursive)
aws --endpoint-url=https://storage.yandexcloud.net \
  s3 cp --recursive local_files/ s3://bucket-name/prefix/

# Download file
aws s3 cp --endpoint-url=https://storage.yandexcloud.net \
  s3://bucket-name/object-key local-path

# Download directory
aws s3 cp --endpoint-url=https://storage.yandexcloud.net \
  --recursive s3://bucket-name/prefix/ local-path/

# Delete object
aws --endpoint-url=https://storage.yandexcloud.net \
  s3 rm s3://bucket-name/object-key

# Delete recursively
aws --endpoint-url=https://storage.yandexcloud.net \
  s3 rm s3://bucket-name/prefix/ --recursive
```

## S3 API Commands

```bash
# List with query filter
aws s3api list-objects \
  --endpoint-url https://storage.yandexcloud.net \
  --bucket sample-bucket \
  --query 'Contents[?starts_with(Key, `date-20231002`)].Key' \
  --output text

# Get object
aws s3api get-object \
  --endpoint-url https://storage.yandexcloud.net \
  --bucket bucket-name \
  --key object-key output-file.txt

# Put object
aws s3api put-object \
  --endpoint-url https://storage.yandexcloud.net \
  --bucket bucket-name \
  --key new-file.txt \
  --body ./local-file.txt
```

## Common Use Cases

### Backup Management with Lifecycle

Delete old backups after 180 days:

```json
{
  "Rules": [{
    "ID": "DeleteOldBackups",
    "Filter": {"Prefix": "backup/"},
    "Status": "Enabled",
    "Expiration": {"Days": 180}
  }]
}
```

### Static Website Hosting

```bash
# 1. Create bucket (name must match domain)
aws --endpoint-url=https://storage.yandexcloud.net \
  s3 mb s3://www.example.com

# 2. Upload website files
aws --endpoint-url=https://storage.yandexcloud.net \
  s3 cp --recursive ./website/ s3://www.example.com/

# 3. Configure public access
yc storage bucket update www.example.com \
  --public-read --public-list
```

**Access URL:** `https://{bucket-name}.website.yandexcloud.net`

## IAM Roles

Assign to:
- Service accounts
- Users/Federated users
- User groups

Common roles:
- `storage.admin`
- `storage.editor`
- `storage.viewer`
- `storage.uploader`

## Important Notes

- **Region**: Always use `ru-central1`
- **Endpoint**: Required for all AWS CLI commands
- **Lifecycle**: Max 1,000 rules per bucket
- **Static hosting**: Client-side only (HTML/CSS/JS)
