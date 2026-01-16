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

- **Compute Operations** — VM lifecycle management, disk operations, snapshots
- **Networking (VPC)** — Networks, subnets, security groups with rule syntax
- **Multi-Region Support** — Russia and Kazakhstan endpoints and profiles
- **Profile Management** — Switch between environments seamlessly
- **Quick Reference** — Common commands and patterns at your fingertips

## Installation

### Option 1: Install via Claude Code CLI

```bash
claude skill install yandex-cloud-cli-claude-skill
```

### Option 2: Manual Installation

1. Clone this repository:
```bash
git clone https://github.com/rodion-m/yandex-cloud-cli-claude-skill.git
```

2. Copy to your Claude skills directory:
```bash
cp -r yandex-cloud-cli-claude-skill ~/.claude/skills/yandex-cloud-cli
```

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

| Operation | Command |
|-----------|---------|
| List VMs | `yc compute instance list` |
| Get VM details | `yc compute instance get <id>` |
| Start/Stop VM | `yc compute instance start/stop <id>` |
| List disks | `yc compute disk list` |
| Resize disk | `yc compute disk resize <id> --size 100` |
| List networks | `yc vpc network list` |
| List security groups | `yc vpc security-group list` |
| Switch profile | `yc config profile activate <name>` |

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
    ├── compute.md        # VM operations, platforms, disk types
    ├── vpc.md            # Networking, security group rules
    └── profiles.md       # Multi-region profile configuration
```

## Requirements

- [Claude Code CLI](https://claude.ai/code)
- [Yandex Cloud CLI](https://yandex.cloud/docs/cli/quickstart) installed and configured

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

<p align="center">
  Made with Claude Code
</p>
