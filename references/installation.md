# Yandex Cloud CLI - Installation and Setup Guide

## Installation

### Linux and macOS

#### Interactive Installation (Recommended)

```bash
curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash
```

This script will:
- Download and install the latest CLI version
- Add `yc` to your PATH
- Set up shell completion (bash/zsh)

**Installation locations:**
- Binary: `~/yandex-cloud/bin/yc`
- Shell config updates: `~/.bashrc` or `~/.zshrc`

#### Manual Installation

```bash
# Download
curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh -o install.sh

# Review the script (optional but recommended)
cat install.sh

# Execute with bash
bash install.sh

# Or with specific options
bash install.sh -i /custom/path -n  # -i: install path, -n: no PATH modification
```

**Apply PATH changes:**
```bash
source ~/.bashrc  # or ~/.zshrc for zsh users
```

### Windows

#### PowerShell Installation

**Run PowerShell as Administrator**, then execute:

```powershell
iex (New-Object System.Net.WebClient).DownloadString('https://storage.yandexcloud.net/yandexcloud-yc/install.ps1')
```

This will:
- Download and install the CLI
- Add to system PATH
- Create `%USERPROFILE%\yandex-cloud\bin\yc.exe`

**Alternative: Manual download**
1. Download from: `https://storage.yandexcloud.net/yandexcloud-yc/release/yc-windows-amd64.exe`
2. Rename to `yc.exe`
3. Move to desired location
4. Add directory to PATH manually

**Restart PowerShell** after installation to apply PATH changes.

### Docker

```bash
# Run in container
docker run -it --rm \
  -v ~/.config/yandex-cloud:/root/.config/yandex-cloud \
  cr.yandex/yc/cli:latest

# Or use as alias
alias yc='docker run -it --rm -v ~/.config/yandex-cloud:/root/.config/yandex-cloud cr.yandex/yc/cli:latest'
```

### Verification

```bash
yc version
```

Expected output:
```
Yandex Cloud CLI 0.xxx.x linux/amd64
```

---

## Initialization

### Interactive Initialization (Recommended for First-Time Setup)

```bash
# Russia region (default)
yc init

# Kazakhstan region
yc init --region=kz
```

The interactive wizard will guide you through:

1. **Authentication Method Selection**
   - OAuth token (recommended for personal use)
   - IAM token (short-lived, 12 hours)
   - Service account key (for automation)

2. **Cloud Selection**
   - Lists available clouds
   - Select default cloud

3. **Folder Selection**
   - Lists folders in selected cloud
   - Select default folder

4. **Zone Selection**
   - Russia: `ru-central1-a`, `ru-central1-b`, `ru-central1-c`, `ru-central1-d`
   - Kazakhstan: `kz1-a` (only zone currently available)

---

## Authentication Methods

### 1. OAuth Token (Personal Accounts)

**Step 1: Get OAuth Token**

**For Russia:**
```
https://oauth.yandex.ru/authorize?response_type=token&client_id=1a6990aa636648e9b2ef855fa7bec2fb
```

**For Kazakhstan:**
```
https://oauth.yandex.kz/authorize?response_type=token&client_id=1a6990aa636648e9b2ef855fa7bec2fb
```

**Step 2: Configure CLI**

```bash
# During yc init
# Or manually:
yc config set token <your-oauth-token>
```

**Token characteristics:**
- Long-lived (valid until manually revoked)
- Tied to your Yandex account
- Best for personal/development use
- Can be revoked at: https://passport.yandex.ru/profile

### 2. IAM Token (Short-Lived)

**Get IAM token from OAuth:**
```bash
yc iam create-token
```

**Use IAM token:**
```bash
yc config set token <iam-token>
```

**Token characteristics:**
- Valid for 12 hours
- Must be regenerated frequently
- Less common for CLI usage (more for API)

### 3. Service Account Key (Automation/CI/CD)

**Step 1: Create Service Account**
```bash
yc iam service-account create --name my-robot
```

**Step 2: Assign Roles**
```bash
yc resource-manager folder add-access-binding <folder-id> \
  --role editor \
  --service-account-name my-robot
```

**Step 3: Create Authorized Key**
```bash
yc iam key create \
  --service-account-name my-robot \
  --output key.json \
  --description "CI/CD key"
```

**Step 4: Authenticate with Key**
```bash
yc config set service-account-key key.json
```

**Or use environment variable:**
```bash
export YC_SERVICE_ACCOUNT_KEY_FILE=/path/to/key.json
yc compute instance list  # Uses key automatically
```

### 4. Federated Account (SAML)

For organizations using identity federation:

```bash
yc init --federation-id=<federation-id>
```

This opens a browser for SSO authentication via your corporate identity provider.

### 5. Metadata Service (VM Internal Authentication)

When running inside a Yandex Cloud VM with attached service account:

```bash
yc config set use-metadata-service true
```

The CLI automatically retrieves credentials from the VM metadata service.

**Check metadata service availability:**
```bash
curl -H Metadata-Flavor:Google http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/token
```

---

## Profile Management

### Creating Profiles

```bash
# Create profile for Russia
yc config profile create russia
yc config profile activate russia
yc config set endpoint api.cloud.yandex.net:443
yc config set token <russia-oauth-token>
yc config set cloud-id <cloud-id>
yc config set folder-id <folder-id>

# Create profile for Kazakhstan
yc config profile create kazakhstan
yc config profile activate kazakhstan
yc config set endpoint api.yandexcloud.kz:443
yc config set token <kazakhstan-oauth-token>
yc config set cloud-id <cloud-id>
yc config set folder-id <folder-id>
yc config set compute-default-zone kz1-a
```

### Switching Profiles

```bash
# List profiles
yc config profile list

# Activate profile
yc config profile activate russia

# Use specific profile for one command (no switch)
yc compute instance list --profile kazakhstan
```

### Profile Configuration Files

**Linux/macOS:**
- Config: `~/.config/yandex-cloud/config.yaml`
- Credentials: `~/.config/yandex-cloud/credentials`

**Windows:**
- Config: `%USERPROFILE%\.config\yandex-cloud\config.yaml`

---

## Configuration Options

### View Current Configuration

```bash
yc config list
```

### Set Configuration Properties

```bash
# Authentication
yc config set token <token>
yc config set service-account-key <path-to-key.json>

# Cloud/Folder
yc config set cloud-id <cloud-id>
yc config set folder-id <folder-id>

# Region/Zone
yc config set compute-default-zone ru-central1-a
yc config set endpoint api.cloud.yandex.net:443

# Output format
yc config set format yaml  # or json, table

# Unset property
yc config unset token
```

---

## Shell Completion

### Bash

```bash
# Install completion
source <(yc completion bash)

# Add to ~/.bashrc for persistence
echo 'source <(yc completion bash)' >> ~/.bashrc
```

### Zsh

```bash
# Install completion
source <(yc completion zsh)

# Add to ~/.zshrc for persistence
echo 'source <(yc completion zsh)' >> ~/.zshrc
```

### Fish

```bash
yc completion fish | source

# Add to ~/.config/fish/config.fish
yc completion fish | source
```

---

## Updates

### Check for Updates

```bash
yc version --check-update
```

### Update CLI

**Linux/macOS:**
```bash
yc components update
```

**Windows:**
```powershell
# Re-run installation script
iex (New-Object System.Net.WebClient).DownloadString('https://storage.yandexcloud.net/yandexcloud-yc/install.ps1')
```

---

## Troubleshooting

### CLI Not Found After Installation

**Linux/macOS:**
```bash
# Reload shell configuration
source ~/.bashrc  # or ~/.zshrc

# Or manually add to PATH
export PATH="$HOME/yandex-cloud/bin:$PATH"
```

**Windows:**
- Restart PowerShell
- Verify PATH includes: `%USERPROFILE%\yandex-cloud\bin`

### Authentication Errors

```bash
# Error: "Authentication failed"
# Solution: Refresh OAuth token
yc init  # Re-authenticate

# Error: "IAM token expired"
# Solution: Generate new IAM token
yc iam create-token
yc config set token <new-iam-token>

# Error: "Service account key invalid"
# Solution: Verify key file and permissions
ls -la key.json  # Check file exists and is readable
yc config set service-account-key key.json
```

### Endpoint/Region Issues

```bash
# Error: "Connection refused" or "Endpoint not found"
# Solution: Verify correct endpoint for region

# Russia
yc config set endpoint api.cloud.yandex.net:443

# Kazakhstan
yc config set endpoint api.yandexcloud.kz:443
```

### SSL Certificate Issues

```bash
# Error: "SSL certificate verify failed"
# Solution (temporary, not recommended for production):
export YC_SKIP_TLS_VERIFY=true

# Better: Update system CA certificates
# Ubuntu/Debian
sudo apt-get update && sudo apt-get install ca-certificates

# macOS
brew install ca-certificates
```

### Permission Denied Errors

```bash
# Error: "You don't have permission to perform this operation"
# Solution: Check role assignments
yc resource-manager folder list-access-bindings <folder-id>

# Add necessary role
yc resource-manager folder add-access-binding <folder-id> \
  --role <required-role> \
  --subject userAccount:<user-id>
```

### Debug Mode

Enable verbose output for troubleshooting:

```bash
yc --debug compute instance list
```

---

## Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `YC_TOKEN` | OAuth or IAM token | `export YC_TOKEN=<token>` |
| `YC_CLOUD_ID` | Default cloud ID | `export YC_CLOUD_ID=<id>` |
| `YC_FOLDER_ID` | Default folder ID | `export YC_FOLDER_ID=<id>` |
| `YC_SERVICE_ACCOUNT_KEY_FILE` | Service account key path | `export YC_SERVICE_ACCOUNT_KEY_FILE=~/key.json` |
| `YC_ENDPOINT` | API endpoint | `export YC_ENDPOINT=api.cloud.yandex.net:443` |
| `YC_PROFILE` | Active profile name | `export YC_PROFILE=production` |
| `YC_SKIP_TLS_VERIFY` | Skip TLS verification | `export YC_SKIP_TLS_VERIFY=true` |

Environment variables override config file settings and are useful for:
- CI/CD pipelines
- Temporary overrides
- Multi-account automation

---

## Quick Start Examples

### Personal Development Setup (Russia)

```bash
# 1. Install CLI
curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash
source ~/.bashrc

# 2. Initialize with OAuth
yc init
# Follow prompts, use OAuth token from oauth.yandex.ru

# 3. Verify
yc config list
yc compute instance list
```

### CI/CD Setup (GitHub Actions)

```bash
# 1. Create service account
yc iam service-account create --name github-actions

# 2. Assign role
yc resource-manager folder add-access-binding <folder-id> \
  --role editor \
  --service-account-name github-actions

# 3. Create key
yc iam key create \
  --service-account-name github-actions \
  --output github-key.json

# 4. Add to GitHub Secrets
# GitHub Repository → Settings → Secrets → New secret
# Name: YC_SA_KEY
# Value: <content of github-key.json>

# 5. Use in workflow (.github/workflows/deploy.yml):
# - name: Authenticate with Yandex Cloud
#   run: |
#     echo '${{ secrets.YC_SA_KEY }}' > key.json
#     yc config set service-account-key key.json
#     yc compute instance list
```

### Multi-Region Setup

```bash
# Russia profile
yc config profile create prod-russia
yc config profile activate prod-russia
yc init  # Select Russia cloud/folder

# Kazakhstan profile
yc config profile create prod-kazakhstan
yc config profile activate prod-kazakhstan
yc init --region=kz  # Select Kazakhstan cloud/folder

# Switch as needed
yc config profile activate prod-russia
```

---

## Security Best Practices

1. **OAuth Token Security**
   - Never commit tokens to version control
   - Revoke unused tokens at https://passport.yandex.ru/profile
   - Use service accounts for automation

2. **Service Account Keys**
   - Store key files securely (chmod 600 on Linux/macOS)
   - Use separate service accounts per application
   - Rotate keys periodically
   - Apply principle of least privilege (minimal roles)

3. **Profile Management**
   - Use separate profiles for development/staging/production
   - Avoid using admin roles for routine operations
   - Configure production profiles with read-only access when possible

4. **Audit and Monitoring**
   - Enable Cloud Audit Trails for compliance
   - Monitor service account activity
   - Set up alerts for unusual CLI usage patterns
