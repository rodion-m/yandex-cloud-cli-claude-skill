---
name: yandex-cloud-cli
description: Yandex Cloud CLI (yc) command reference for managing cloud infrastructure. Use when working with Yandex Cloud resources: VMs (compute instances), disks, networks (VPC), security groups, profiles, or any yc commands. Triggers on tasks involving Yandex Cloud management, VM creation/configuration, network setup, or multi-region (Russia/Kazakhstan) cloud operations.
---

# Yandex Cloud CLI

## Quick Reference

```bash
# Installation
curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash

# Initialize (interactive)
yc init              # Russia region
yc init --region=kz  # Kazakhstan region

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

## Detailed References

- [references/compute.md](references/compute.md) - Full compute operations, VM creation options
- [references/vpc.md](references/vpc.md) - Networking, security group rules
- [references/profiles.md](references/profiles.md) - Multi-region profile setup
