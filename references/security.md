# Yandex Cloud Lockbox & Certificate Manager CLI Guide

Comprehensive guide for managing secrets and SSL/TLS certificates using Yandex Cloud CLI.

**Последнее обновление:** 2026-01-16

---

## Table of Contents

- [Lockbox (Secrets Management)](#lockbox-secrets-management)
  - [Creating Secrets](#creating-secrets)
  - [Managing Secret Versions](#managing-secret-versions)
  - [Retrieving Secret Values](#retrieving-secret-values)
  - [Listing and Managing Secrets](#listing-and-managing-secrets)
  - [Access Control](#access-control)
- [Certificate Manager](#certificate-manager)
  - [Creating Certificates](#creating-certificates)
  - [Let's Encrypt Managed Certificates](#lets-encrypt-managed-certificates)
  - [Domain Validation](#domain-validation)
  - [Certificate Lifecycle](#certificate-lifecycle)
- [Security Best Practices](#security-best-practices)
- [Common Use Cases](#common-use-cases)

---

## Lockbox (Secrets Management)

Yandex Cloud Lockbox is a service for securely storing sensitive data like passwords, API keys, and tokens. All secrets are encrypted using Yandex KMS and replicated across three availability zones.

### Creating Secrets

#### Basic Secret Creation

```bash
yc lockbox secret create \
  --name <secret_name> \
  --description <secret_description> \
  --payload "<array_with_secret_contents>" \
  --cloud-id <cloud_ID> \
  --folder-id <folder_ID> \
  --deletion-protection
```

#### Parameters

| Parameter | Purpose | Required |
|-----------|---------|----------|
| `--name` | Identifier for the secret | Yes |
| `--description` | Additional context | No |
| `--payload` | Secret content in YAML/JSON format | No |
| `--cloud-id` | Cloud where secret resides | No |
| `--folder-id` | Folder containing the secret | No |
| `--deletion-protection` | Prevents accidental deletion | No |
| `--kms-key-id` | Custom KMS key for encryption | No |

#### Payload Format

Secrets support multiple key-value pairs. Use base64 encoding for binary data:

```bash
# Simple text secret
yc lockbox secret create \
  --name lockbox-secret \
  --payload '[{"key": "password", "textValue": "p@$$w0rd"}]'

# Multiple values (text + binary)
yc lockbox secret create \
  --name sample-secret \
  --description "Sample secret with text and binary data" \
  --payload "[{'key': 'username', 'text_value': 'myusername'},{'key': 'avatar', 'binary_value': $(base64 -w 0 ./avatar.jpg)}]" \
  --deletion-protection
```

#### Real-World Example: Database Credentials

```bash
yc lockbox secret create \
  --name mongodb-credentials \
  --description "Production MongoDB credentials" \
  --payload '[
    {"key": "host", "text_value": "localhost:27017"},
    {"key": "username", "text_value": "admin"},
    {"key": "password", "text_value": "SecureP@ssw0rd123"},
    {"key": "database", "text_value": "production"}
  ]' \
  --folder-id b1qt6g8ht345******** \
  --deletion-protection
```

### Managing Secret Versions

Lockbox uses versioning to manage secret lifecycle. Each update creates a new version.

#### Add New Version

```bash
yc lockbox secret add-version <secret_name> \
  --description <version_description> \
  --payload "<array_with_version_contents>" \
  --base-version-id <existing_version_id>
```

**Example: Update API Key**

```bash
yc lockbox secret add-version api-keys \
  --description "Rotated API key - Jan 2026" \
  --payload '[{"key": "api_key", "text_value": "new-key-value-2026"}]'
```

#### Schedule Version Deletion

Schedule version destruction (e.g., 1 week = 168h):

```bash
yc lockbox secret schedule-version-destruction <secret_name> \
  --version-id <version_id> \
  --pending-period 168h
```

#### Cancel Version Deletion

```bash
yc lockbox secret cancel-version-destruction <secret_name> \
  --version-id <version_id>
```

#### List All Versions

```bash
yc lockbox secret list-versions <secret_name>
```

### Retrieving Secret Values

#### Get Secret Payload

```bash
# Get current version
yc lockbox payload get --id <secret_id>

# Get specific version
yc lockbox payload get \
  --id e6qetpqfe8vv******** \
  --version-id e6qqr7k79ecm********

# Get single key from secret
yc lockbox payload get \
  --id <secret_id> \
  --key password
```

**Example: Use in Script**

```bash
#!/bin/bash
# Retrieve database password from Lockbox
DB_PASSWORD=$(yc lockbox payload get \
  --id e6qetpqfe8vv12345678 \
  --key password \
  --format json | jq -r '.entries[0].textValue')

# Use in connection string
mongosh "mongodb://admin:${DB_PASSWORD}@localhost:27017/production"
```

### Listing and Managing Secrets

#### List All Secrets

```bash
# List all secrets in current folder
yc lockbox secret list

# List secrets in specific folder
yc lockbox secret list --folder-id <folder_id>
```

#### Get Secret Details

```bash
yc lockbox secret get <secret_name>
yc lockbox secret get --id <secret_id>
```

#### Update Secret Metadata

```bash
yc lockbox secret update <secret_name> \
  --new-name <new_name> \
  --description "Updated description"
```

#### Delete Secret

```bash
# Delete secret (must disable deletion-protection first)
yc lockbox secret delete --name <secret_name>

# Force delete with ID
yc lockbox secret delete --id <secret_id>
```

**Example:**

```bash
yc lockbox secret delete --name airflow/connections/mysqldb
```

#### Activate/Deactivate Secret

```bash
# Deactivate secret (prevents access)
yc lockbox secret deactivate <secret_name>

# Reactivate secret
yc lockbox secret activate <secret_name>
```

### Access Control

#### Assign IAM Roles

```bash
# Grant payload viewer access
yc lockbox secret add-access-binding <secret_name> \
  --role lockbox.payloadViewer \
  --service-account-id <sa_id>

# Grant admin access
yc lockbox secret add-access-binding <secret_name> \
  --role lockbox.admin \
  --user-account-id <user_id>
```

#### Remove Access

```bash
yc lockbox secret remove-access-binding <secret_name> \
  --role lockbox.payloadViewer \
  --service-account-id <sa_id>
```

#### List Access Bindings

```bash
yc lockbox secret list-access-bindings --name <secret_name>
```

#### Set Complete Access Policy

```bash
yc lockbox secret set-access-bindings <secret_name> \
  --access-binding role=lockbox.payloadViewer,service-account-id=<sa_id_1> \
  --access-binding role=lockbox.admin,user-account-id=<user_id>
```

---

## Certificate Manager

Yandex Certificate Manager allows you to obtain and manage TLS/SSL certificates for HTTPS access to your resources. It supports both Let's Encrypt managed certificates and user-uploaded certificates.

### Creating Certificates

#### Import User Certificate

```bash
yc certificate-manager certificate create \
  --name <certificate_name> \
  --chain <path_to_certificate_chain_file> \
  --key <path_to_private_key_file>
```

**Example:**

```bash
yc certificate-manager certificate create \
  --name my-ssl-cert \
  --chain /path/to/fullchain.pem \
  --key /path/to/privkey.pem \
  --description "Production SSL certificate"
```

#### Verify Certificate Content

```bash
yc certificate-manager certificate content \
  --id <certificate_id> \
  --chain <path_to_save_chain> \
  --key <path_to_save_key>
```

### Let's Encrypt Managed Certificates

Certificate Manager can automatically obtain and renew Let's Encrypt certificates with 90-day validity periods.

#### Request Managed Certificate (HTTP Challenge)

```bash
yc certificate-manager certificate request \
  --name <certificate_name> \
  --domains <domain_list> \
  --challenge http
```

**Example:**

```bash
yc certificate-manager certificate request \
  --name contextron-cert \
  --domains contextron.ru,www.contextron.ru \
  --challenge http \
  --description "Let's Encrypt certificate for contextron.ru"
```

#### Request Managed Certificate (DNS Challenge)

```bash
yc certificate-manager certificate request \
  --name <certificate_name> \
  --domains <domain_list> \
  --challenge dns
```

**Example (Wildcard Certificate):**

```bash
yc certificate-manager certificate request \
  --name wildcard-cert \
  --domains "*.example.com,example.com" \
  --challenge dns \
  --description "Wildcard certificate for all subdomains"
```

> **Note:** Wildcard certificates (e.g., `*.example.com`) can only be validated using DNS challenge.

### Domain Validation

After requesting a Let's Encrypt certificate, you must prove domain ownership.

#### Check Certificate Status

```bash
yc certificate-manager certificate get <certificate_name>
```

Certificate statuses:
- `Validating` - Domain rights check in progress
- `Issued` - Certificate successfully issued
- `Invalid` - Validation failed
- `Renewing` - Certificate renewal in progress
- `Renewal_failed` - Renewal failed

#### HTTP Challenge (HTTP-01)

For HTTP validation, create a file on your web server:

```bash
# The challenge details are shown when you get the certificate
yc certificate-manager certificate get <certificate_name>

# Create the validation file on your web server
# Location: http://<domain>/.well-known/acme-challenge/<token>
echo "<validation_string>" > /var/www/html/.well-known/acme-challenge/<token>
```

**Automatic HTTP Validation Setup:**

Configure an HTTP redirect on your web server to automatically pass validation during renewals:

```nginx
# Nginx configuration
location /.well-known/acme-challenge/ {
    root /var/www/certbot;
}
```

#### DNS Challenge (DNS-01)

For DNS validation, add a TXT record to your DNS zone:

```bash
# Get challenge details
yc certificate-manager certificate get <certificate_name>

# Add TXT record to DNS (example for Yandex Cloud DNS)
yc dns zone add-records <zone_name> \
  --record "_acme-challenge.<domain>. 600 TXT <validation_string>"
```

**CNAME Delegation for Automatic Renewal:**

Create a CNAME record once for automatic validation during renewals:

```bash
# One-time CNAME setup
yc dns zone add-records <zone_name> \
  --record "_acme-challenge.<domain>. 600 CNAME <cm_delegation_domain>"
```

### Certificate Lifecycle

#### List Certificates

```bash
# List all certificates
yc certificate-manager certificate list

# List certificates in specific folder
yc certificate-manager certificate list --folder-id <folder_id>
```

#### Get Certificate Details

```bash
yc certificate-manager certificate get <certificate_name>
yc certificate-manager certificate get --id <certificate_id>
```

#### Update Certificate Metadata

```bash
yc certificate-manager certificate update <certificate_name> \
  --new-name <new_name> \
  --description "Updated description"
```

#### Delete Certificate

```bash
yc certificate-manager certificate delete <certificate_name>
yc certificate-manager certificate delete --id <certificate_id>
```

#### Automatic Renewal

Let's Encrypt certificates are automatically renewed by Certificate Manager:

- Renewal process starts **30 days before expiration**
- Requires passing the same domain validation challenge (HTTP or DNS)
- CNAME delegation or HTTP redirects enable automatic validation
- Certificate status changes to `Renewing` during renewal
- If validation fails, status becomes `Renewal_failed`

**Monitor Renewal Status:**

```bash
# Check if renewal is needed
yc certificate-manager certificate get <certificate_name> \
  --format json | jq '.status, .notAfter'
```

---

## Security Best Practices

### Lockbox Security

1. **Use Deletion Protection**
   ```bash
   yc lockbox secret create --deletion-protection ...
   ```
   Enable for all production secrets to prevent accidental deletion.

2. **Custom KMS Encryption**
   ```bash
   yc lockbox secret create \
     --kms-key-id <kms_key_id> \
     --name sensitive-secret \
     ...
   ```
   Use custom KMS keys for enhanced security and audit control.

3. **Granular IAM Roles**
   - `lockbox.payloadViewer` - Read secret values only
   - `lockbox.editor` - Manage secret versions
   - `lockbox.admin` - Full control including access policies

   **Principle of Least Privilege:**
   ```bash
   # Service accounts should only have payloadViewer
   yc lockbox secret add-access-binding production-db \
     --role lockbox.payloadViewer \
     --service-account-id <app_sa_id>
   ```

4. **KMS Key Management**
   - Store KMS keys in a dedicated folder separate from application resources
   - Assign `kms.keys.encrypterDecrypter` role (not `kms.admin`) for secret encryption
   - Enable key rotation for compliance

   ```bash
   # Assign minimal KMS permissions
   yc kms symmetric-key add-access-binding <key_name> \
     --role kms.keys.encrypterDecrypter \
     --service-account-id <lockbox_sa_id>
   ```

5. **Secret Versioning**
   - Always add new versions instead of modifying existing ones
   - Schedule old versions for deletion after rotation period
   - Keep version history for audit trails

   ```bash
   # Rotate secret and schedule old version deletion
   yc lockbox secret add-version api-key --payload '[...]'
   yc lockbox secret schedule-version-destruction api-key \
     --version-id <old_version_id> \
     --pending-period 168h  # 1 week grace period
   ```

6. **Secret Deactivation**
   ```bash
   # Deactivate compromised secrets immediately
   yc lockbox secret deactivate <secret_name>
   ```
   Deactivated secrets cannot be accessed but remain recoverable.

### Certificate Manager Security

1. **Use Let's Encrypt for Public Sites**
   - Free, automated certificate management
   - Automatic renewals reduce manual errors
   - Industry-standard validation

2. **DNS Challenge for Internal/Wildcard Certs**
   - Use DNS-01 challenge for wildcard certificates
   - Better for internal services without public HTTP access
   - CNAME delegation simplifies renewals

3. **Monitor Certificate Expiration**
   ```bash
   # Create monitoring script
   #!/bin/bash
   yc certificate-manager certificate list --format json | \
     jq -r '.[] | select(.status == "Renewal_failed" or .status == "Invalid") | .name'
   ```

4. **Separate Certificates by Environment**
   - Production, staging, and development in different folders
   - Use naming convention: `<env>-<service>-cert`
   - Example: `prod-api-cert`, `staging-web-cert`

5. **Backup Imported Certificates**
   ```bash
   # Export certificate for backup
   yc certificate-manager certificate content \
     --id <cert_id> \
     --chain /backup/certs/chain-$(date +%Y%m%d).pem \
     --key /backup/certs/key-$(date +%Y%m%d).pem
   ```

6. **Access Control**
   - Restrict certificate access to deployment service accounts
   - Use `certificate-manager.certificates.downloader` role for applications
   - Limit `certificate-manager.admin` to operations team

---

## Common Use Cases

### Use Case 1: Storing Application Secrets

**Scenario:** Store database credentials for a Node.js application

```bash
# 1. Create secret
yc lockbox secret create \
  --name app-db-credentials \
  --payload '[
    {"key": "DB_HOST", "text_value": "rc1a-xyz.mdb.yandexcloud.net"},
    {"key": "DB_PORT", "text_value": "27018"},
    {"key": "DB_NAME", "text_value": "production"},
    {"key": "DB_USER", "text_value": "app_user"},
    {"key": "DB_PASSWORD", "text_value": "SecurePassword123!"}
  ]' \
  --deletion-protection

# 2. Grant service account access
yc lockbox secret add-access-binding app-db-credentials \
  --role lockbox.payloadViewer \
  --service-account-id <app_sa_id>

# 3. Retrieve in application startup script
SECRET_ID="e6qetpqfe8vv12345678"
yc lockbox payload get --id $SECRET_ID --format json > /tmp/secrets.json
export $(jq -r '.entries[] | "\(.key)=\(.textValue)"' /tmp/secrets.json)
rm /tmp/secrets.json

# 4. Use in application
node server.js  # DB_HOST, DB_PASSWORD, etc. available as env vars
```

### Use Case 2: API Key Rotation

**Scenario:** Rotate API keys every 90 days

```bash
# 1. Create initial secret
yc lockbox secret create \
  --name third-party-api-key \
  --payload '[{"key": "api_key", "text_value": "initial-key-2026-01"}]'

# 2. Rotate after 90 days
yc lockbox secret add-version third-party-api-key \
  --payload '[{"key": "api_key", "text_value": "rotated-key-2026-04"}]' \
  --description "Q2 2026 rotation"

# 3. List versions to get old version ID
yc lockbox secret list-versions third-party-api-key

# 4. Schedule old version deletion (7 day grace period)
yc lockbox secret schedule-version-destruction third-party-api-key \
  --version-id <old_version_id> \
  --pending-period 168h
```

### Use Case 3: SSL Certificate for Web Application

**Scenario:** Obtain Let's Encrypt certificate for `example.com`

```bash
# 1. Request certificate (HTTP challenge)
yc certificate-manager certificate request \
  --name example-com-cert \
  --domains example.com,www.example.com \
  --challenge http \
  --description "Production certificate for example.com"

# 2. Get validation requirements
yc certificate-manager certificate get example-com-cert

# Output shows challenge details:
# domain: example.com
# http_challenge:
#   url: http://example.com/.well-known/acme-challenge/abc123xyz
#   content: abc123xyz.def456uvw

# 3. Create validation file on web server
ssh web-server
mkdir -p /var/www/html/.well-known/acme-challenge
echo "abc123xyz.def456uvw" > /var/www/html/.well-known/acme-challenge/abc123xyz

# 4. Wait for validation (usually takes 1-5 minutes)
watch -n 10 'yc certificate-manager certificate get example-com-cert | grep status'

# 5. Once status is "Issued", configure automatic renewal redirect in Nginx
cat >> /etc/nginx/sites-available/example.com <<EOF
location /.well-known/acme-challenge/ {
    root /var/www/certbot;
}
EOF

nginx -t && systemctl reload nginx
```

### Use Case 4: Wildcard Certificate with DNS Challenge

**Scenario:** Create wildcard certificate for all subdomains

```bash
# 1. Request wildcard certificate
yc certificate-manager certificate request \
  --name wildcard-example-cert \
  --domains "*.example.com,example.com" \
  --challenge dns

# 2. Get DNS challenge details
yc certificate-manager certificate get wildcard-example-cert

# Output shows:
# domain: example.com
# dns_challenge:
#   name: _acme-challenge.example.com
#   type: TXT
#   value: "challenge-string-xyz123"

# 3. Add TXT record to DNS
yc dns zone add-records example-com-zone \
  --record "_acme-challenge.example.com. 600 TXT challenge-string-xyz123"

# 4. Verify DNS propagation
dig TXT _acme-challenge.example.com +short

# 5. Wait for validation
watch -n 10 'yc certificate-manager certificate get wildcard-example-cert | grep status'

# 6. (Optional) Setup CNAME for automatic renewal
yc dns zone add-records example-com-zone \
  --record "_acme-challenge.example.com. 600 CNAME <cm_delegation_domain>"
```

### Use Case 5: Kubernetes Secret from Lockbox

**Scenario:** Inject Lockbox secrets into Kubernetes pods

```bash
# 1. Create secret in Lockbox
yc lockbox secret create \
  --name k8s-app-config \
  --payload '[
    {"key": "REDIS_URL", "text_value": "redis://redis:6379"},
    {"key": "SESSION_SECRET", "text_value": "random-secret-key"}
  ]'

# 2. Grant Kubernetes service account access
K8S_SA_ID=$(yc iam service-account get k8s-sa --format json | jq -r '.id')
yc lockbox secret add-access-binding k8s-app-config \
  --role lockbox.payloadViewer \
  --service-account-id $K8S_SA_ID

# 3. Use External Secrets Operator in Kubernetes
cat <<EOF | kubectl apply -f -
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: yc-lockbox-store
spec:
  provider:
    yandexlockbox:
      auth:
        authorizedKeySecretRef:
          name: yc-sa-key
          key: authorized-key
      apiEndpoint: lockbox.api.cloud.yandex.net:443

---
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: app-config
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: yc-lockbox-store
    kind: SecretStore
  target:
    name: app-config-secret
    creationPolicy: Owner
  data:
  - secretKey: REDIS_URL
    remoteRef:
      key: k8s-app-config
      property: REDIS_URL
  - secretKey: SESSION_SECRET
    remoteRef:
      key: k8s-app-config
      property: SESSION_SECRET
EOF

# 4. Reference secret in pod
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:latest
    envFrom:
    - secretRef:
        name: app-config-secret
EOF
```

### Use Case 6: CI/CD Pipeline Integration

**Scenario:** Use GitHub Actions to deploy with Lockbox secrets

```bash
# 1. Create deployment secrets
yc lockbox secret create \
  --name github-actions-deploy \
  --payload '[
    {"key": "SSH_PRIVATE_KEY", "text_value": "-----BEGIN OPENSSH PRIVATE KEY-----\n..."},
    {"key": "DEPLOY_HOST", "text_value": "94.131.91.159"},
    {"key": "DEPLOY_USER", "text_value": "deploy"}
  ]'

# 2. Create service account for GitHub Actions
yc iam service-account create --name github-actions-sa
SA_ID=$(yc iam service-account get github-actions-sa --format json | jq -r '.id')

# 3. Grant access to secret
yc lockbox secret add-access-binding github-actions-deploy \
  --role lockbox.payloadViewer \
  --service-account-id $SA_ID

# 4. Create authorized key for service account
yc iam key create \
  --service-account-name github-actions-sa \
  --output github-actions-key.json

# 5. Add to GitHub Secrets
gh secret set YC_SA_JSON_CREDENTIALS < github-actions-key.json

# 6. Use in GitHub Actions workflow
cat > .github/workflows/deploy.yml <<'EOF'
name: Deploy
on: [push]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Install YC CLI
        run: |
          curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash
          echo "$HOME/yandex-cloud/bin" >> $GITHUB_PATH

      - name: Authenticate with Yandex Cloud
        run: |
          echo '${{ secrets.YC_SA_JSON_CREDENTIALS }}' > key.json
          yc config profile create sa-profile
          yc config set service-account-key key.json
          yc config set cloud-id <cloud_id>
          yc config set folder-id <folder_id>

      - name: Get deployment secrets
        run: |
          yc lockbox payload get --name github-actions-deploy --format json > secrets.json
          echo "SSH_PRIVATE_KEY=$(jq -r '.entries[] | select(.key=="SSH_PRIVATE_KEY") | .textValue' secrets.json)" >> $GITHUB_ENV
          echo "DEPLOY_HOST=$(jq -r '.entries[] | select(.key=="DEPLOY_HOST") | .textValue' secrets.json)" >> $GITHUB_ENV
          echo "DEPLOY_USER=$(jq -r '.entries[] | select(.key=="DEPLOY_USER") | .textValue' secrets.json)" >> $GITHUB_ENV
          rm secrets.json

      - name: Deploy application
        run: |
          echo "$SSH_PRIVATE_KEY" > deploy_key
          chmod 600 deploy_key
          ssh -i deploy_key -o StrictHostKeyChecking=no $DEPLOY_USER@$DEPLOY_HOST 'bash deploy.sh'
EOF
```

---

## Additional Resources

- [Yandex Lockbox Documentation](https://cloud.yandex.com/en/docs/lockbox/)
- [Yandex Certificate Manager Documentation](https://cloud.yandex.com/en/docs/certificate-manager/)
- [Yandex KMS Documentation](https://cloud.yandex.com/en/docs/kms/)
- [External Secrets Operator - Yandex Lockbox](https://external-secrets.io/latest/provider/yandex-lockbox/)
- [cert-manager webhook for Yandex Cloud DNS](https://github.com/yandex-cloud/cert-manager-webhook-yandex)

---

## Quick Reference

### Lockbox Commands

```bash
# Secrets
yc lockbox secret create --name <name> --payload '[...]'
yc lockbox secret list
yc lockbox secret get <name>
yc lockbox secret update <name>
yc lockbox secret delete <name>
yc lockbox secret activate <name>
yc lockbox secret deactivate <name>

# Versions
yc lockbox secret add-version <name> --payload '[...]'
yc lockbox secret list-versions <name>
yc lockbox secret schedule-version-destruction <name> --version-id <id> --pending-period <duration>
yc lockbox secret cancel-version-destruction <name> --version-id <id>

# Payload
yc lockbox payload get --id <secret_id>
yc lockbox payload get --id <secret_id> --version-id <version_id>
yc lockbox payload get --id <secret_id> --key <key>

# Access
yc lockbox secret add-access-binding <name> --role <role> --service-account-id <sa_id>
yc lockbox secret remove-access-binding <name> --role <role> --service-account-id <sa_id>
yc lockbox secret list-access-bindings <name>
```

### Certificate Manager Commands

```bash
# Certificates
yc certificate-manager certificate create --name <name> --chain <file> --key <file>
yc certificate-manager certificate request --name <name> --domains <domains> --challenge <http|dns>
yc certificate-manager certificate list
yc certificate-manager certificate get <name>
yc certificate-manager certificate update <name>
yc certificate-manager certificate delete <name>
yc certificate-manager certificate content --id <id> --chain <file> --key <file>

# Common options
--folder-id <folder_id>
--description <text>
--format json
```

### IAM Roles

**Lockbox:**
- `lockbox.payloadViewer` - Read secret values
- `lockbox.editor` - Manage secrets and versions
- `lockbox.admin` - Full access including IAM

**Certificate Manager:**
- `certificate-manager.certificates.downloader` - Download certificates
- `certificate-manager.editor` - Manage certificates
- `certificate-manager.admin` - Full access

**KMS:**
- `kms.keys.encrypterDecrypter` - Encrypt/decrypt with key
- `kms.admin` - Manage keys

---

**Last Updated:** 2026-01-16
