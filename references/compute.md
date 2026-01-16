# Compute Operations Reference

## VM Creation Options

### Full VM Create Syntax

```bash
yc compute instance create \
  --name <name> \
  --zone <zone> \
  --platform standard-v3 \
  --cores <num> \
  --core-fraction <50|100> \
  --memory <size>G \
  --network-interface subnet-name=<subnet>,nat-ip-version=ipv4,security-group-ids=<sg-id> \
  --create-boot-disk image-folder-id=standard-images,image-family=<family>,size=<gb>,type=<disk-type> \
  --ssh-key ~/.ssh/id_ed25519.pub \
  --metadata-from-file user-data=cloud-init.yaml
```

### Zones

| Region | Zones |
|--------|-------|
| Russia | ru-central1-a, ru-central1-b, ru-central1-c, ru-central1-d |
| Kazakhstan | kz1-a (kz1-b, kz1-c planned) |

### Disk Types

| Type | Description | Use Case |
|------|-------------|----------|
| network-hdd | Standard HDD | Archives, backups |
| network-ssd | Standard SSD | General workloads |
| network-ssd-io-m3 | SSD with 3 replicas | High reliability |
| network-ssd-nonreplicated | Non-replicated SSD | Max performance |

### Platforms

| Platform | Description |
|----------|-------------|
| standard-v1 | Intel Broadwell |
| standard-v2 | Intel Cascade Lake |
| standard-v3 | Intel Ice Lake (recommended) |
| highfreq-v3 | High-frequency Ice Lake |

### Core Fractions

- `--core-fraction 50` - Burstable (50% baseline)
- `--core-fraction 100` - Dedicated (100% baseline)

## VM Operations

### Attach/Detach Disk

```bash
# Attach
yc compute instance attach-disk <instance-id> \
  --disk-id <disk-id> \
  --device-name data

# Detach
yc compute instance detach-disk <instance-id> \
  --disk-id <disk-id>
```

### Network Interface

```bash
# Add public IP to existing VM
yc compute instance add-one-to-one-nat <instance-id> \
  --network-interface-index 0

# Remove public IP
yc compute instance remove-one-to-one-nat <instance-id> \
  --network-interface-index 0

# Update security groups
yc compute instance update-network-interface <instance-id> \
  --network-interface-index 0 \
  --security-group-ids <sg-id>
```

### Metadata and Cloud-Init

```bash
# Pass cloud-init config
yc compute instance create \
  --metadata-from-file user-data=cloud-init.yaml \
  ...

# Pass SSH keys via metadata
yc compute instance create \
  --metadata "ssh-keys=yc-user:$(cat ~/.ssh/id_ed25519.pub)" \
  ...
```

## Disk Operations

```bash
# Create disk from snapshot
yc compute disk create \
  --name restored-disk \
  --zone kz1-a \
  --source-snapshot-id <snapshot-id>

# Create snapshot
yc compute snapshot create \
  --name backup-$(date +%Y%m%d) \
  --disk-id <disk-id>

# List snapshots
yc compute snapshot list
```

## Images

```bash
# List available images
yc compute image list --folder-id standard-images

# Filter by family
yc compute image list --folder-id standard-images | grep ubuntu

# Get latest image from family
yc compute image get-latest-from-family ubuntu-2404-lts --folder-id standard-images

# Create custom image from disk
yc compute image create \
  --name my-image \
  --source-disk-id <disk-id>
```

## Instance Groups (autoscaling)

```bash
# List instance groups
yc compute instance-group list

# Get instance group
yc compute instance-group get <group-id>

# List instances in group
yc compute instance-group list-instances <group-id>
```
