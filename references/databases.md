# Yandex Cloud CLI Reference: Managed Databases & Compute Snapshots

Comprehensive guide to Yandex Cloud CLI commands for managing databases (PostgreSQL, MySQL, MongoDB) and VM disk snapshots.

**Last Updated:** 2026-01-16

---

## Table of Contents

- [Managed PostgreSQL](#managed-postgresql)
- [Managed MySQL](#managed-mysql)
- [Managed MongoDB](#managed-mongodb)
- [Compute Snapshots](#compute-snapshots)
- [Backup & Restore Workflows](#backup--restore-workflows)

---

## Managed PostgreSQL

### Create Cluster

```bash
yc managed-postgresql cluster create \
  --name <cluster-name> \
  --environment <prestable|production> \
  --network-name <network-name> \
  --host zone-id=<zone>,subnet-id=<subnet-id> \
  --resource-preset <host-class> \
  --user name=<username>,password=<password> \
  --database name=<db-name>,owner=<username> \
  --disk-size <size-in-gb> \
  --disk-type <network-hdd|network-ssd|network-ssd-nonreplicated|local-ssd> \
  --security-group-ids <group-id> \
  --deletion-protection=<true|false>
```

**Key Parameters:**

| Parameter | Description | Example |
|-----------|-------------|---------|
| `--name` | Cluster name (unique in folder) | `my-postgres` |
| `--environment` | Environment type | `production` or `prestable` |
| `--network-name` | VPC network name | `default` |
| `--host` | Host configuration | `zone-id=ru-central1-a,subnet-id=e9b...` |
| `--resource-preset` | VM class | `s2.micro`, `s2.small`, `s2.medium` |
| `--disk-size` | Storage size in GB | `20`, `50`, `100` |
| `--disk-type` | Disk type | `network-ssd` (recommended) |
| `--postgresql-version` | PostgreSQL version | `14`, `15`, `16` |

**Note:** If an availability zone contains 2+ subnets, specify `subnet-id` explicitly.

### Backup Operations

#### List Backups

```bash
yc managed-postgresql backup list
```

#### Get Backup Info

```bash
yc managed-postgresql backup get <backup-id>
```

#### Create Manual Backup

```bash
yc managed-postgresql cluster backup <cluster-name-or-id>
```

### Restore from Backup

#### Restore to Latest State

```bash
yc managed-postgresql cluster restore \
  --backup-id <backup-id> \
  --name <new-cluster-name> \
  --environment production \
  --network-name <network-name> \
  --host zone-id=<zone>,subnet-id=<subnet-id> \
  --disk-size <size-in-gb> \
  --disk-type network-ssd \
  --resource-preset <host-class>
```

#### Point-in-Time Recovery (PITR)

Restore cluster to any point in time between oldest backup and most recent WAL archive:

```bash
yc managed-postgresql cluster restore \
  --backup-id <backup-id> \
  --time "2026-01-15T14:30:00Z" \
  --name restored-cluster \
  --environment production \
  --network-name default \
  --host zone-id=ru-central1-a,subnet-id=<subnet-id> \
  --disk-size 50 \
  --resource-preset s2.micro
```

**Time Format:** `yyyy-mm-ddThh:mm:ssZ` (ISO 8601, UTC)

### Cluster Management

```bash
# List clusters
yc managed-postgresql cluster list

# Get cluster info
yc managed-postgresql cluster get <cluster-name>

# Start cluster
yc managed-postgresql cluster start <cluster-name>

# Stop cluster
yc managed-postgresql cluster stop <cluster-name>

# Delete cluster
yc managed-postgresql cluster delete <cluster-name>
```

### Database & User Management

```bash
# List databases
yc managed-postgresql database list --cluster-name <cluster-name>

# Create database
yc managed-postgresql database create <db-name> \
  --cluster-name <cluster-name> \
  --owner <username>

# List users
yc managed-postgresql user list --cluster-name <cluster-name>

# Create user
yc managed-postgresql user create <username> \
  --cluster-name <cluster-name> \
  --password <password>
```

---

## Managed MySQL

### Create Cluster

```bash
yc managed-mysql cluster create \
  --name <cluster-name> \
  --mysql-version <version> \
  --environment <prestable|production> \
  --network-name <network-name> \
  --security-group-ids <group-id> \
  --host zone-id=<zone>,subnet-id=<subnet-id> \
  --resource-preset <host-class> \
  --disk-type <disk-type> \
  --disk-size <size-in-gb> \
  --user name=<username>,password=<password> \
  --database name=<db-name> \
  --deletion-protection=<true|false>
```

**Example:**

```bash
yc managed-mysql cluster create \
  --name my-mysql \
  --mysql-version 8.0 \
  --environment production \
  --network-name default \
  --security-group-ids enp6saqnq4ie244g67sb \
  --host zone-id=ru-central1-c,subnet-id=b0rcctk2rvtr8efcch64 \
  --resource-preset s2.micro \
  --disk-type network-ssd \
  --disk-size 20 \
  --user name=user1,password="user1user1" \
  --database name=db1 \
  --deletion-protection=true
```

**MySQL Versions:** `5.7`, `8.0`

### Backup & Restore

MySQL supports Point-in-Time Recovery (PITR) similar to PostgreSQL:

```bash
# List backups
yc managed-mysql backup list

# Create backup
yc managed-mysql cluster backup <cluster-name>

# Restore cluster
yc managed-mysql cluster restore \
  --backup-id <backup-id> \
  --name <new-cluster-name> \
  --environment production \
  --network-name <network-name> \
  --host zone-id=<zone>,subnet-id=<subnet-id> \
  --resource-preset s2.micro \
  --disk-size 20

# Restore to specific time
yc managed-mysql cluster restore \
  --backup-id <backup-id> \
  --time "2026-01-15T14:30:00Z" \
  --name restored-mysql
```

### Cluster Management

```bash
# List clusters
yc managed-mysql cluster list

# Start failover (switch master)
yc managed-mysql cluster start-failover <cluster-name>

# Update cluster
yc managed-mysql cluster update <cluster-name> \
  --disk-size 50

# Delete cluster
yc managed-mysql cluster delete <cluster-name>
```

---

## Managed MongoDB

### Create Cluster

```bash
yc managed-mongodb cluster create \
  --name <cluster-name> \
  --environment <prestable|production> \
  --network-name <network-name> \
  --mongodb-version <version> \
  --host zone-id=<zone>,subnet-id=<subnet-id> \
  --mongod-resource-preset <host-class> \
  --mongod-disk-type <disk-type> \
  --mongod-disk-size <size-in-gb> \
  --database name=<db-name> \
  --user name=<username>,password=<password> \
  --security-group-ids <group-id> \
  --deletion-protection=<true|false> \
  --performance-diagnostics enabled=<true|false>
```

**Key MongoDB-Specific Parameters:**

| Parameter | Description | Values |
|-----------|-------------|--------|
| `--mongodb-version` | MongoDB version | `4.4`, `5.0`, `6.0`, `7.0` |
| `--mongod-resource-preset` | Host class for mongod | `s2.micro`, `s2.small`, etc. |
| `--mongod-disk-type` | Disk type | `network-ssd`, `local-ssd` |
| `--mongod-disk-size` | Storage size in GB | `20`, `50`, `100` |
| `--host` | Advanced host config | `zone-id=...,assign-public-ip=true,hidden=false,priority=1.0` |

**Example:**

```bash
yc managed-mongodb cluster create \
  --name mymongo \
  --mongodb-version 6.0 \
  --environment production \
  --network-name default \
  --host zone-id=ru-central1-a,subnet-id=e9b0c7qv... \
  --mongod-resource-preset s2.small \
  --mongod-disk-type network-ssd \
  --mongod-disk-size 30 \
  --database name=mydb \
  --user name=admin,password="SecurePass123" \
  --deletion-protection=true
```

### Cluster Management

```bash
# List clusters
yc managed-mongodb cluster list

# Get cluster info
yc managed-mongodb cluster get <cluster-name>

# Update cluster
yc managed-mongodb cluster update <cluster-name> \
  --mongod-disk-size 50 \
  --performance-diagnostics enabled=true

# Delete cluster
yc managed-mongodb cluster delete <cluster-name>
```

---

## Compute Snapshots

Disk snapshots are point-in-time copies of disk file systems for backups and versioning.

### Create Snapshot

```bash
yc compute snapshot create <snapshot-name> \
  --name <snapshot-name> \
  --description "<description>" \
  --disk-id <disk-id>
```

**Example:**

```bash
yc compute snapshot create first-snapshot \
  --name first-snapshot \
  --description "Backup before system upgrade" \
  --disk-id fhm4aq4hvq5g3nepvt9b
```

**Important Notes:**

- Snapshots are created **asynchronously** with `CREATING` status
- Only data already written to disk is included
- Snapshots contain only disk data, not VM configuration

### List Snapshots

```bash
yc compute snapshot list
```

**Output Fields:** ID, NAME, PRODUCT IDS, STATUS, DESCRIPTION

### Get Snapshot Info

```bash
yc compute snapshot get <snapshot-id>
```

### Delete Snapshot

```bash
yc compute snapshot delete <snapshot-id>
```

### Update Snapshot Metadata

```bash
yc compute snapshot update <snapshot-id> \
  --name new-name \
  --description "Updated description"
```

### Manage Labels

```bash
# Add labels
yc compute snapshot add-labels <snapshot-id> \
  --labels env=production,app=webapp

# Remove labels
yc compute snapshot remove-labels <snapshot-id> \
  --labels env,app
```

### List Operations

```bash
yc compute snapshot list-operations <snapshot-id>
```

---

## Backup & Restore Workflows

### VM Disk Backup Workflow

#### 1. Create Snapshot

```bash
# Get disk ID from VM
yc compute instance get turbocode-vm1

# Create snapshot
yc compute snapshot create backup-$(date +%Y%m%d-%H%M%S) \
  --disk-id <disk-id> \
  --description "Automated backup before deployment"
```

#### 2. Verify Snapshot

```bash
yc compute snapshot list
```

#### 3. Restore from Snapshot

**Option A: Create New VM from Snapshot**

```bash
yc compute instance create \
  --name restored-instance \
  --zone ru-central1-a \
  --public-ip \
  --create-boot-disk snapshot-name=<snapshot-name> \
  --ssh-key ~/.ssh/id_rsa.pub
```

**Option B: Create VM with Multiple Disks from Snapshots**

```bash
yc compute instance create \
  --name multi-disk-vm \
  --zone ru-central1-a \
  --public-ip \
  --create-boot-disk snapshot-name=boot-snapshot \
  --create-disk snapshot-name=data-snapshot \
  --ssh-key ~/.ssh/id_rsa.pub
```

**Important:** You cannot restore a boot disk on an existing VM. Create a new VM instead.

### Database Backup Workflow

#### PostgreSQL Production Backup

```bash
# 1. Create manual backup before maintenance
yc managed-postgresql cluster backup prod-postgres

# 2. List backups to get backup-id
yc managed-postgresql backup list

# 3. If needed, restore to new cluster
yc managed-postgresql cluster restore \
  --backup-id <backup-id> \
  --name postgres-restore-$(date +%Y%m%d) \
  --environment production \
  --network-name default \
  --host zone-id=ru-central1-a,subnet-id=$SUBNET_ID \
  --resource-preset s2.small \
  --disk-size 50
```

#### MySQL Point-in-Time Recovery

```bash
# Restore to specific time (e.g., before bad migration)
yc managed-mysql cluster restore \
  --backup-id <backup-id> \
  --time "2026-01-16T10:45:00Z" \
  --name mysql-pitr-recovery \
  --environment production \
  --network-name default \
  --host zone-id=ru-central1-b,subnet-id=$SUBNET_ID \
  --resource-preset s2.small \
  --disk-size 30
```

#### MongoDB Backup Strategy

```bash
# Regular manual backup
yc managed-mongodb cluster backup prod-mongo

# List and verify
yc managed-mongodb backup list

# Restore if needed
yc managed-mongodb cluster restore \
  --backup-id <backup-id> \
  --name mongo-restore \
  --environment production \
  --network-name default \
  --mongod-resource-preset s2.small \
  --mongod-disk-size 50
```

### Snapshot Scheduling

Yandex Cloud supports **automatic snapshot schedules** via the API and Console.

**API Reference:** `SnapshotSchedule` service

```bash
# List snapshot schedules
GET https://compute.api.cloud.yandex.net/compute/v1/snapshotSchedules
```

**Schedule Properties:**

- `schedulePolicy.startAt` - Start time
- `schedulePolicy.expression` - Cron expression
- `retentionPeriod` or `snapshotCount` - Retention settings
- `snapshotSpec` - Snapshot attributes

**Note:** CLI command `yc compute snapshot-schedule create` may be available. Check with:

```bash
yc compute snapshot-schedule --help
```

### Common Patterns

#### Pre-Deployment Snapshot

```bash
#!/bin/bash
# Create snapshot before deployment
DISK_ID="fhm4aq4hvq5g3nepvt9b"
SNAPSHOT_NAME="pre-deploy-$(date +%Y%m%d-%H%M%S)"

yc compute snapshot create $SNAPSHOT_NAME \
  --disk-id $DISK_ID \
  --description "Automated snapshot before deployment"

# Wait for snapshot to complete
while true; do
  STATUS=$(yc compute snapshot get $SNAPSHOT_NAME --format json | jq -r '.status')
  if [ "$STATUS" = "READY" ]; then
    echo "Snapshot ready: $SNAPSHOT_NAME"
    break
  fi
  echo "Waiting for snapshot... (status: $STATUS)"
  sleep 10
done
```

#### Database Pre-Migration Backup

```bash
#!/bin/bash
# Backup database before migration
CLUSTER_NAME="prod-postgres"

echo "Creating backup for $CLUSTER_NAME..."
yc managed-postgresql cluster backup $CLUSTER_NAME

# Get latest backup ID
BACKUP_ID=$(yc managed-postgresql backup list --format json | \
  jq -r '.[0].id')

echo "Backup created: $BACKUP_ID"
echo "Run migration with confidence!"
```

#### Automated Cleanup (Old Snapshots)

```bash
#!/bin/bash
# Delete snapshots older than 30 days
RETENTION_DAYS=30

yc compute snapshot list --format json | \
  jq -r --arg days "$RETENTION_DAYS" \
  '.[] | select(.createdAt | fromdateiso8601 < (now - ($days | tonumber * 86400))) | .id' | \
while read SNAPSHOT_ID; do
  echo "Deleting old snapshot: $SNAPSHOT_ID"
  yc compute snapshot delete $SNAPSHOT_ID
done
```

---

## Additional Resources

### CLI Help Commands

```bash
# PostgreSQL
yc managed-postgresql --help
yc managed-postgresql cluster create --help

# MySQL
yc managed-mysql --help
yc managed-mysql cluster create --help

# MongoDB
yc managed-mongodb --help
yc managed-mongodb cluster create --help

# Snapshots
yc compute snapshot --help
yc compute snapshot create --help
```

### Common Host Classes

| Class | vCPUs | RAM | Use Case |
|-------|-------|-----|----------|
| `s2.micro` | 2 | 8 GB | Development, testing |
| `s2.small` | 4 | 16 GB | Small production workloads |
| `s2.medium` | 8 | 32 GB | Medium production workloads |
| `s2.large` | 12 | 48 GB | Large production workloads |
| `s3-c2-m8` | 2 | 8 GB | Burstable performance |

### Disk Types

| Type | Description | Use Case |
|------|-------------|----------|
| `network-hdd` | Network HDD | Large infrequent access |
| `network-ssd` | Network SSD | Standard production (recommended) |
| `network-ssd-nonreplicated` | Fast non-replicated SSD | High IOPS temporary data |
| `local-ssd` | Local SSD | Maximum performance |

### Availability Zones

| Zone | Region | Location |
|------|--------|----------|
| `ru-central1-a` | Russia (Central) | Moscow |
| `ru-central1-b` | Russia (Central) | Moscow |
| `ru-central1-c` | Russia (Central) | Moscow |
| `kz1-a` | Kazakhstan | Almaty |

---

## Sources

- [Yandex Cloud: Creating PostgreSQL clusters](https://cloud.yandex.com/docs/managed-postgresql/operations/cluster-create)
- [Yandex Cloud: PostgreSQL CLI Reference](https://cloud.yandex.com/en/docs/cli/cli-ref/managed-services/managed-postgresql/)
- [Yandex Cloud: PostgreSQL Backups](https://cloud.yandex.com/en/docs/managed-postgresql/operations/cluster-backups)
- [Yandex Cloud: MySQL CLI Reference](https://cloud.yandex.com/en/docs/cli/cli-ref/managed-services/managed-mysql/cluster/create)
- [Yandex Cloud: Creating MySQL clusters](https://cloud.yandex.com/en/docs/managed-mysql/operations/cluster-create)
- [Yandex Cloud: MongoDB CLI Reference](https://cloud.yandex.com/en/docs/cli/cli-ref/managed-services/managed-mongodb/cluster/create)
- [Yandex Cloud: Creating MongoDB clusters](https://cloud.yandex.com/en/docs/managed-mongodb/operations/cluster-create)
- [Yandex Cloud: Creating disk snapshots](https://cloud.yandex.com/en/docs/compute/operations/disk-control/create-snapshot)
- [Yandex Cloud: Compute snapshots](https://cloud.yandex.com/en/docs/compute/concepts/snapshot)
- [Yandex Cloud: VM from snapshots](https://cloud.yandex.com/en/docs/compute/operations/vm-create/create-from-snapshots)
- [Yandex Cloud: Snapshot Schedule API](https://yandex.cloud/en/docs/compute/api-ref/SnapshotSchedule/list)

---

**Generated:** 2026-01-16 by Claude Code
