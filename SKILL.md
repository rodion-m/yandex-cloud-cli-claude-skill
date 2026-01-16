---
name: yandex-cloud-cli
description: Comprehensive Yandex Cloud CLI (yc) command reference for cloud infrastructure management. Use when working with VMs, disks, networks, storage, databases, Kubernetes, containers, IAM, secrets (Lockbox), certificates, DNS, load balancers, or any yc/AWS CLI commands for Yandex Cloud. Triggers on infrastructure provisioning, configuration, security, CI/CD automation, and multi-region (Russia/Kazakhstan) operations.
---

# Yandex Cloud CLI

## Quick Reference

```bash
# Installation (see references/installation.md for full guide)
curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash  # Linux/macOS
# Windows: iex (New-Object System.Net.WebClient).DownloadString('https://storage.yandexcloud.net/yandexcloud-yc/install.ps1')

# Initialize (interactive)
yc init              # Russia region
yc init --region=kz  # Kazakhstan region

# Authentication
# OAuth: https://oauth.yandex.ru/authorize?response_type=token&client_id=1a6990aa636648e9b2ef855fa7bec2fb
yc config set token <oauth-token>
# Service account: yc config set service-account-key key.json

# Profile management
yc config profile list
yc config profile activate <name>
yc config profile create <name>
yc config list  # Show current config

# Common operations
yc compute instance list
yc compute instance get <id>
yc compute disk list
yc vpc network list
yc vpc security-group list
```

## Region Endpoints

| Region | Endpoint | Console |
|--------|----------|---------|
| Russia | `api.cloud.yandex.net:443` | console.yandex.cloud |
| Kazakhstan | `api.yandexcloud.kz:443` | kz.console.yandex.cloud |

## Command Structure

```
yc <service> <resource> <action> [flags]
```

**Services**: compute, vpc, iam, storage, managed-postgresql, managed-mysql, managed-mongodb, container-registry, certificate-manager, lockbox, dns, logging

**Common flags**:
- `--profile <name>` - Use specific profile
- `--format yaml|json|table` - Output format
- `--async` - Don't wait for completion
- `--help` - Command help

## Core Operations

### Compute (VMs)

```bash
# List/get VMs
yc compute instance list
yc compute instance get <id>
yc compute instance get --name <name>

# Create VM (Ubuntu 24.04)
yc compute instance create \
  --name my-vm \
  --zone kz1-a \
  --network-interface subnet-name=default-kz1-a,nat-ip-version=ipv4 \
  --create-boot-disk image-folder-id=standard-images,image-family=ubuntu-2404-lts,size=20,type=network-ssd \
  --memory 4G \
  --cores 2 \
  --ssh-key ~/.ssh/id_ed25519.pub

# Control VM
yc compute instance start <id>
yc compute instance stop <id>
yc compute instance restart <id>
yc compute instance delete <id>

# Update resources (VM must be stopped)
yc compute instance update <id> --memory 8G --cores 4
```

### Disks

```bash
yc compute disk list
yc compute disk create --name data --zone kz1-a --size 50 --type network-ssd
yc compute disk resize <id> --size 100  # No downtime

# Disk types: network-hdd, network-ssd, network-ssd-io-m3, network-ssd-nonreplicated
```

### Networking (VPC)

```bash
# Networks
yc vpc network list
yc vpc network create --name my-network

# Subnets
yc vpc subnet list
yc vpc subnet create --name my-subnet --zone kz1-a --network-name my-network --range 10.1.0.0/24

# Security groups
yc vpc security-group list
yc vpc security-group get <id>
yc vpc security-group create \
  --name web-sg \
  --network-id <network-id> \
  --rule "direction=ingress,port=22,protocol=tcp,v4-cidrs=[0.0.0.0/0]" \
  --rule "direction=ingress,port=80,protocol=tcp,v4-cidrs=[0.0.0.0/0]" \
  --rule "direction=ingress,port=443,protocol=tcp,v4-cidrs=[0.0.0.0/0]" \
  --rule "direction=egress,protocol=any,v4-cidrs=[0.0.0.0/0]"

# Add rule to existing group
yc vpc security-group update-rules <id> \
  --add-rule "direction=ingress,port=3000,protocol=tcp,v4-cidrs=[0.0.0.0/0]"
```

### Profile Configuration

```bash
# Create and configure profile
yc config profile create russia
yc config profile activate russia
yc config set endpoint api.cloud.yandex.net:443
yc config set token <oauth-token>

# Get OAuth token: https://oauth.yandex.ru/authorize?response_type=token&client_id=1a6990aa636648e9b2ef855fa7bec2fb

# Set cloud and folder
yc resource-manager cloud list
yc config set cloud-id <id>
yc resource-manager folder list
yc config set folder-id <id>

# Run command with different profile (no switch)
yc compute instance list --profile turbocode-kz
```

## Image Families

| Family | Description |
|--------|-------------|
| ubuntu-2404-lts | Ubuntu 24.04 LTS |
| ubuntu-2204-lts | Ubuntu 22.04 LTS |
| debian-11 | Debian 11 |
| almalinux-9 | AlmaLinux 9 |
| container-optimized-image | Docker-optimized |

```bash
# Search images
yc compute image list --folder-id standard-images | grep ubuntu
```

## SSH Access

```bash
# Default user is yc-user
ssh yc-user@<EXTERNAL_IP>
ssh -i ~/.ssh/id_ed25519 yc-user@<EXTERNAL_IP>
```

## IAM & Service Accounts

```bash
# Create service account
yc iam service-account create --name my-robot

# Create API key
yc iam api-key create --service-account-name my-robot

# Create authorized key (for IAM tokens)
yc iam key create --service-account-name my-robot --output key.json

# Create static access key (for S3)
yc iam access-key create --service-account-name my-robot

# Assign role to folder
yc resource-manager folder add-access-binding <folder-id> \
  --role editor \
  --service-account-name my-robot
```

## Object Storage (S3)

```bash
# Yandex CLI - Bucket operations
yc storage bucket list
yc storage bucket get <name> --full
yc storage bucket update <name> --public-read

# AWS CLI - Setup (use ru-central1 region)
aws configure
# Always add: --endpoint-url=https://storage.yandexcloud.net

# AWS CLI - Object operations
aws --endpoint-url=https://storage.yandexcloud.net \
  s3 cp file.txt s3://bucket-name/
aws --endpoint-url=https://storage.yandexcloud.net \
  s3 ls --recursive s3://bucket-name
```

## Container Registry & Kubernetes

```bash
# Container Registry
yc container registry list
yc container registry create --name my-registry
yc container registry configure-docker  # Auth helper

# Push image
docker tag my-app cr.yandex/<registry-id>/my-app:latest
docker push cr.yandex/<registry-id>/my-app:latest

# Kubernetes Cluster
yc managed-kubernetes cluster list
yc managed-kubernetes cluster create \
  --name my-cluster \
  --network-name default \
  --zone kz1-a \
  --service-account-name k8s-cluster \
  --node-service-account-name k8s-nodes

# Node group
yc managed-kubernetes node-group create \
  --cluster-name my-cluster \
  --name worker-nodes \
  --fixed-size 3
```

## Managed Databases

```bash
# PostgreSQL
yc managed-postgresql cluster list
yc managed-postgresql cluster create \
  --name pg-cluster \
  --environment production \
  --network-name default \
  --resource-preset s2.micro \
  --disk-size 10 \
  --disk-type network-ssd \
  --postgresql-version 15

# MySQL
yc managed-mysql cluster create --name mysql-cluster

# MongoDB
yc managed-mongodb cluster create --name mongo-cluster

# Backups
yc managed-postgresql backup list
yc managed-postgresql cluster restore --backup-id <id>
```

## Snapshots

```bash
# Create snapshot from disk
yc compute snapshot create --name backup-$(date +%Y%m%d) --disk-id <disk-id>

# List snapshots
yc compute snapshot list

# Create VM from snapshot
yc compute instance create \
  --name restored-vm \
  --boot-disk snapshot-id=<snapshot-id>,size=50
```

## Security Services

```bash
# Lockbox (Secrets)
yc lockbox secret create --name db-password --payload '[{"key":"password","text_value":"secret123"}]'
yc lockbox payload get --name db-password --key password

# Certificate Manager
yc certificate-manager certificate list
yc certificate-manager certificate request \
  --name my-cert \
  --domains example.com,www.example.com
```

## DNS

```bash
# Create DNS zone
yc dns zone create --name my-zone --zone example.com. --public-visibility

# Add A record
yc dns zone add-records my-zone --record "@ 300 A 1.2.3.4"

# Add CNAME record
yc dns zone add-records my-zone --record "www 300 CNAME example.com."
```

## Load Balancers

```bash
# Network Load Balancer (Layer 4)
yc load-balancer network-load-balancer create \
  --name my-nlb \
  --listener name=http,port=80,target-port=8080,external-ip-version=ipv4

# Application Load Balancer (Layer 7)
yc application-load-balancer load-balancer create \
  --name my-alb \
  --network-name default
```

## Detailed References

- [references/installation.md](references/installation.md) - Installation, setup, authentication (all OS)
- [references/compute.md](references/compute.md) - VM operations, platforms, disk types
- [references/vpc.md](references/vpc.md) - Networking, security group rules
- [references/profiles.md](references/profiles.md) - Multi-region profile setup
- [references/iam.md](references/iam.md) - Service accounts, keys, roles, CI/CD
- [references/storage.md](references/storage.md) - Object Storage, S3 API, lifecycle
- [references/containers.md](references/containers.md) - Container Registry, Kubernetes
- [references/databases.md](references/databases.md) - PostgreSQL, MySQL, MongoDB, snapshots
- [references/security.md](references/security.md) - Lockbox secrets, Certificate Manager
- [references/dns-lb.md](references/dns-lb.md) - DNS zones, Load Balancers
