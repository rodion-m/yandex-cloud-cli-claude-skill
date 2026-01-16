<h1 align="center">
  <br>
  <img src="https://storage.yandexcloud.net/cloud-www-assets/constructor/stardust/images/header/yandex-cloud-logo.svg" alt="Yandex Cloud" width="200">
  <br>
  Yandex Cloud CLI Skill for Claude Code
  <br>
</h1>

<p align="center">
  <strong>A Claude Code skill that provides comprehensive Yandex Cloud CLI command reference and guidance</strong>
</p>

<p align="center">
  <a href="#installation">Installation</a> •
  <a href="#features">Features</a> •
  <a href="#usage">Usage</a> •
  <a href="#commands">Commands</a> •
  <a href="#license">License</a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Claude_Code-Skill-blueviolet?style=flat-square" alt="Claude Code Skill">
  <img src="https://img.shields.io/badge/Yandex_Cloud-CLI-red?style=flat-square" alt="Yandex Cloud">
  <img src="https://img.shields.io/badge/License-MIT-green?style=flat-square" alt="MIT License">
  <img src="https://img.shields.io/badge/Regions-Russia_|_Kazakhstan-blue?style=flat-square" alt="Regions">
</p>

---

## Overview

This skill extends [Claude Code](https://claude.ai/code) with specialized knowledge for working with [Yandex Cloud CLI (`yc`)](https://yandex.cloud/docs/cli/). It provides instant access to command references, best practices, and multi-region configuration guidance.

## Features

- **Compute Operations** — VM lifecycle management, disk operations, snapshots, images
- **Networking (VPC)** — Networks, subnets, security groups with rule syntax
- **IAM & Service Accounts** — Authentication, API keys, role assignments, CI/CD automation
- **Object Storage** — S3-compatible storage, bucket management, lifecycle policies
- **Container Services** — Container Registry, Managed Kubernetes clusters
- **Managed Databases** — PostgreSQL, MySQL, MongoDB with backup/restore
- **Security Services** — Lockbox secrets management, Certificate Manager
- **DNS Management** — DNS zones, record sets (A, CNAME, TXT, MX, etc.)
- **Load Balancers** — Network (L4) and Application (L7) load balancing
- **Multi-Region Support** — Russia and Kazakhstan endpoints and profiles
- **Profile Management** — Switch between environments seamlessly
- **Quick Reference** — Common commands and patterns at your fingertips

## Installation

### User Scope (Available in All Projects)

```bash
# Clone and copy to user skills directory
git clone https://github.com/rodion-m/yandex-cloud-cli-claude-skill.git
cp -r yandex-cloud-cli-claude-skill ~/.claude/skills/yandex-cloud-cli
```

### Project Scope (Available Only in This Repository)

```bash
# Clone and copy to project skills directory
git clone https://github.com/rodion-m/yandex-cloud-cli-claude-skill.git
mkdir -p .claude/skills
cp -r yandex-cloud-cli-claude-skill .claude/skills/yandex-cloud-cli
```

### Verify Installation

The skill will be automatically loaded when Claude Code detects relevant Yandex Cloud tasks.

## Usage

Once installed, Claude Code will automatically use this skill when you ask about Yandex Cloud operations. Examples:

```
> Create a new VM in Kazakhstan with Ubuntu 24.04
> How do I add a security group rule for port 3000?
> Switch to my production profile and list all disks
> What's the command to resize a disk without downtime?
```

## Commands

### Quick Reference

| Category | Operation | Command |
|----------|-----------|---------|
| **Compute** | List VMs | `yc compute instance list` |
| | Start/Stop VM | `yc compute instance start/stop <id>` |
| | Create snapshot | `yc compute snapshot create --disk-id <id>` |
| **Storage** | List buckets | `yc storage bucket list` |
| | Upload to S3 | `aws --endpoint-url=https://storage.yandexcloud.net s3 cp file.txt s3://bucket/` |
| **IAM** | Create service account | `yc iam service-account create --name my-sa` |
| | Create API key | `yc iam api-key create --service-account-name my-sa` |
| **Containers** | List registries | `yc container registry list` |
| | Configure Docker | `yc container registry configure-docker` |
| **Kubernetes** | List clusters | `yc managed-kubernetes cluster list` |
| | Create node group | `yc managed-kubernetes node-group create` |
| **Databases** | Create PostgreSQL | `yc managed-postgresql cluster create` |
| | List backups | `yc managed-postgresql backup list` |
| **Security** | Create secret | `yc lockbox secret create` |
| | Get secret value | `yc lockbox payload get --name <name>` |
| **DNS** | Create zone | `yc dns zone create` |
| | Add record | `yc dns zone add-records <zone> --record "..."` |
| **Networking** | List networks | `yc vpc network list` |
| | Create security group | `yc vpc security-group create` |
| **Profiles** | Switch profile | `yc config profile activate <name>` |

### Region Endpoints

| Region | API Endpoint | Console |
|--------|--------------|---------|
| Russia | `api.cloud.yandex.net:443` | [console.yandex.cloud](https://console.yandex.cloud) |
| Kazakhstan | `api.yandexcloud.kz:443` | [kz.console.yandex.cloud](https://kz.console.yandex.cloud) |

## Skill Structure

```
yandex-cloud-cli/
├── SKILL.md              # Main skill file with quick reference
└── references/
    ├── installation.md   # Installation, setup, authentication (all OS)
    ├── compute.md        # VM operations, platforms, disk types
    ├── vpc.md            # Networking, security group rules
    ├── profiles.md       # Multi-region profile configuration
    ├── iam.md            # Service accounts, keys, roles, CI/CD
    ├── storage.md        # Object Storage, S3 API, lifecycle policies
    ├── containers.md     # Container Registry, Kubernetes
    ├── databases.md      # PostgreSQL, MySQL, MongoDB, snapshots
    ├── security.md       # Lockbox secrets, Certificate Manager
    └── dns-lb.md         # DNS zones, Load Balancers
```

## Requirements

- [Claude Code CLI](https://claude.ai/code)
- [Yandex Cloud CLI](https://yandex.cloud/docs/cli/quickstart) — See installation guide below

### Installing Yandex Cloud CLI

**Linux/macOS:**
```bash
curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash
source ~/.bashrc  # or ~/.zshrc
yc init
```

**Windows (PowerShell as Administrator):**
```powershell
iex (New-Object System.Net.WebClient).DownloadString('https://storage.yandexcloud.net/yandexcloud-yc/install.ps1')
# Restart PowerShell, then run:
yc init
```

**For complete installation, authentication, and troubleshooting guide, see [references/installation.md](references/installation.md)**

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

<p align="center">
  Made with Claude Code
</p>
