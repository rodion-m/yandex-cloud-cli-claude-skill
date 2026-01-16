# Yandex Cloud CLI Guide: Container Registry & Managed Kubernetes

Complete reference for managing Container Registry and Managed Kubernetes using Yandex Cloud CLI (`yc`).

**Последнее обновление:** 2026-01-16

---

## Table of Contents

- [Container Registry](#container-registry)
  - [Registry Management](#registry-management)
  - [Docker Authentication](#docker-authentication)
  - [Image Management](#image-management)
  - [Access Control](#access-control)
- [Managed Kubernetes](#managed-kubernetes)
  - [Prerequisites](#prerequisites)
  - [Cluster Management](#cluster-management)
    - [List, Get, Credentials](#list-clusters)
    - [Create (Zonal & Regional)](#create-cluster)
    - [Stop/Start, Version Management](#stop-and-start-cluster)
    - [Update, Delete](#update-cluster)
  - [Node Group Management](#node-group-management)
    - [List, Get](#list-node-groups)
    - [Create (Basic, Auto-scale, Taints, Preemptible, GPU, Multi-zone)](#create-node-group)
    - [Platform IDs](#platform-ids)
    - [Update, Delete](#update-node-group)
  - [Common Workflows](#common-workflows)

---

## Container Registry

### Registry Management

#### Create Registry

Create a registry with automated vulnerability scanning:
```bash
yc container registry create --name my-reg --secure
```

Create a registry without vulnerability scanning:
```bash
yc container registry create --name my-reg
```

**Registry naming requirements:**
- 2-63 characters long
- Lowercase Latin letters, numbers, and hyphens only
- Must start with a letter
- Cannot end with a hyphen

#### List Registries

```bash
yc container registry list
```

Returns registry ID, name, and folder ID in a formatted table.

#### Get Registry Details

```bash
yc container registry get <registry_id>
```

#### Delete Registry

```bash
yc container registry delete <registry_id>
```

**Note:** All users and service accounts with folder access can use newly created registries.

---

### Docker Authentication

#### Configure Docker Credential Helper (Recommended)

The most secure and convenient method:

```bash
yc container registry configure-docker
```

This command:
- Configures Docker to use the Yandex Cloud credential helper (`docker-credential-yc`)
- Saves settings in `~/.docker/config.json`
- Eliminates the need for manual `docker login` commands
- **Requirement:** Docker must run without `sudo`

**For alternate profiles:**
```bash
yc container registry configure-docker --profile <profile_name>
```

**To disable:**
Remove the Container Registry domain line from the `credHelpers` section in `~/.docker/config.json`.

#### OAuth Token Authentication

Use a 12-month lifetime OAuth token:

```bash
echo <OAuth_token> | docker login \
  --username oauth \
  --password-stdin \
  cr.yandex
```

#### IAM Token Authentication

Use shorter-lived IAM tokens (12 hours or less):

```bash
echo <IAM_token> | docker login \
  --username iam \
  --password-stdin \
  cr.yandex
```

**Minimum role required:** `container-registry.images.puller`

---

### Image Management

#### List Images

List all images in a repository:
```bash
yc container image list --repository-name=<registry_id>/<image_name>
```

List all images with JSON output:
```bash
yc container image list --format=json
```

#### Push Images

After authentication, use standard Docker commands:

```bash
# Tag image
docker tag <local_image>:tag cr.yandex/<registry_id>/<image_name>:tag

# Push image
docker push cr.yandex/<registry_id>/<image_name>:tag
```

#### Delete Images

Delete a specific image:
```bash
yc container image delete <image_id>
```

Bulk delete all images (use with caution):
```bash
yc container image list --format=json | jq -r .[].id | \
  while read id; do yc container image delete $id; done
```

---

### Access Control

#### Set IP Permissions

Configure IP-based access restrictions:

```bash
yc container registry set-ip-permissions <registry_name> \
  --pull <IP_address> \
  --push <IP_address>
```

**Flags:**
- `--pull` - Allow downloading Docker images from specified IP
- `--push` - Allow uploading Docker images from specified IP

**Warning:** This command replaces all current IP permissions; confirmation is required.

#### View IP Permissions

```bash
yc container registry list-ip-permissions <registry_name>
```

Displays existing access rules showing which actions (PULL/PUSH) are permitted for specific IP addresses.

---

## Managed Kubernetes

### Prerequisites

Before creating a Kubernetes cluster, ensure you have:

#### 1. Network & Subnets

- A network with subnets in the availability zones where the cluster will be created
- Cluster IPv4 range for pod addresses must NOT overlap with network subnets
- Service IPv4 range must NOT overlap with network subnets

#### 2. Service Accounts

Two service accounts are required:

**Cluster service account:**
- Role: `editor` on the folder where the cluster will be created
- Role: `editor` on the folder where the network resides
- Used to create cluster resources

**Node service account:**
- Role: `container-registry.images.puller` on the folder with the Docker image registry
- Used by nodes to access the Docker registry

#### 3. Billing & Resources

- Active billing account (status: `ACTIVE` or `TRIAL_ACTIVE`)
- Sufficient resources available in the cloud

---

### Cluster Management

#### List Clusters

```bash
# List all clusters in current folder
yc managed-kubernetes cluster list

# List clusters in specific folder
yc managed-kubernetes cluster list --folder-id <folder-id>
```

#### Get Cluster Details

```bash
# Get cluster by name
yc managed-kubernetes cluster get <cluster-name>

# Get cluster by ID
yc managed-kubernetes cluster get --id <cluster-id>
```

#### Get Cluster Credentials (kubeconfig)

```bash
# External access (public IP)
yc managed-kubernetes cluster get-credentials <cluster-name> --external

# Internal access (from within VPC)
yc managed-kubernetes cluster get-credentials <cluster-name> --internal

# Custom kubeconfig path
yc managed-kubernetes cluster get-credentials <cluster-name> \
  --external \
  --kubeconfig /custom/path/kubeconfig

# Force recreate credentials
yc managed-kubernetes cluster get-credentials <cluster-name> \
  --external \
  --force-recreate

# Custom context name
yc managed-kubernetes cluster get-credentials <cluster-name> \
  --external \
  --context-name my-prod-cluster
```

**Flags:**
- `--external` - Configure access using external (public) IP
- `--internal` - Configure access using internal IP (requires VPC access)
- `--kubeconfig <path>` - Custom kubeconfig location (default: `~/.kube/config`)
- `--force-recreate` - Force recreate even if credentials exist
- `--context-name <name>` - Custom kubectl context name

#### Create Cluster

##### Zonal Master (Single Zone)

Complete example with all common parameters:

```bash
yc managed-kubernetes cluster create \
  --name test-k8s \
  --network-name default \
  --zone ru-central1-a \
  --subnet-name default-a \
  --public-ip \
  --release-channel regular \
  --version 1.13 \
  --cluster-ipv4-range 10.1.0.0/16 \
  --service-ipv4-range 10.2.0.0/16 \
  --security-group-ids enpe5sdn7vs5mu6udl7i,enpj6c5ifh755o6evmu4 \
  --service-account-name default-sa \
  --node-service-account-name default-sa \
  --daily-maintenance-window start=22:00,duration=10h
```

**Key parameters:**
- `--name` - Cluster name
- `--network-name` - Network for the cluster
- `--zone` - Availability zone
- `--subnet-name` - Subnet in the zone
- `--public-ip` - Assign public IP to master
- `--release-channel` - Update channel (rapid/regular/stable)
- `--version` - Kubernetes version
- `--cluster-ipv4-range` - IPv4 range for pods
- `--service-ipv4-range` - IPv4 range for services
- `--security-group-ids` - Security group IDs (comma-separated)
- `--service-account-name` - Cluster service account
- `--node-service-account-name` - Node service account
- `--daily-maintenance-window` - Maintenance window

**Optional features:**
```bash
# Enable KMS encryption for secrets
--kms-key-id <key_id>
--kms-key-name <key_name>

# Enable Network Policies
--enable-network-policy
```

##### Regional Master (Three Zones, High Availability)

```bash
yc managed-kubernetes cluster create \
  --name prod-k8s-regional \
  --network-name production \
  --master-location zone=ru-central1-a,subnet-name=prod-subnet-a \
  --master-location zone=ru-central1-b,subnet-name=prod-subnet-b \
  --master-location zone=ru-central1-d,subnet-name=prod-subnet-d \
  --public-ip \
  --release-channel stable \
  --version 1.28 \
  --service-account-name k8s-prod-sa \
  --node-service-account-name k8s-prod-node-sa
```

**Regional Master:**
- `--master-location` - Must specify exactly 3 locations (one per AZ)
- Format: `zone=<zone>,subnet-name=<subnet>`
- Provides high availability across zones
- **Cannot use** with `--zone` parameter (mutually exclusive)

**Master Type Comparison:**

| Type | Availability Zones | Use Case | Downtime Risk |
|------|-------------------|----------|---------------|
| Zonal | 1 | Development, testing | Higher (single AZ) |
| Regional | 3 | Production | Lower (multi-AZ) |

#### Stop and Start Cluster

```bash
# Stop cluster (stops all node groups)
yc managed-kubernetes cluster stop <cluster-name>

# Start cluster (starts all node groups)
yc managed-kubernetes cluster start <cluster-name>
```

#### Version Management

```bash
# Update to specific Kubernetes version
yc managed-kubernetes cluster update <cluster-name> --version 1.28

# Update to latest revision
yc managed-kubernetes cluster update <cluster-name> --latest-revision
```

#### List Cluster Resources

```bash
# List all node groups in cluster
yc managed-kubernetes cluster list-node-groups <cluster-name>

# List all nodes across cluster
yc managed-kubernetes cluster list-nodes <cluster-name>

# List operations
yc managed-kubernetes cluster list-operations <cluster-name>
```

#### Update Cluster

```bash
yc managed-kubernetes cluster update <cluster_id> \
  --new-name <new_name> \
  --description <description> \
  --version 1.28
```

#### Delete Cluster

```bash
yc managed-kubernetes cluster delete <cluster-id>

# Async deletion
yc managed-kubernetes cluster delete <cluster-name> --async
```

---

### Node Group Management

#### List Node Groups

```bash
# List all node groups
yc managed-kubernetes node-group list

# List node groups for specific cluster
yc managed-kubernetes node-group list --cluster-name <cluster-name>
```

#### Get Node Group Details

```bash
# Get by name
yc managed-kubernetes node-group get <node-group-name>

# Get by ID
yc managed-kubernetes node-group get --id <node-group-id>
```

#### Create Node Group

##### Basic Node Group with Fixed Size

Create a node group with labels:

```bash
yc managed-kubernetes node-group create \
  --name k8s-labels-node \
  --cluster-name k8s-labels \
  --disk-type network-ssd \
  --fixed-size 1 \
  --node-labels environment=production,apps/tier=backend
```

**Key parameters:**
- `--name` - Node group name
- `--cluster-name` - Parent cluster name
- `--disk-type` - Disk type (network-ssd/network-hdd)
- `--fixed-size` - Fixed number of nodes
- `--node-labels` - Kubernetes labels (key=value,key=value)

##### Auto-Scaling Node Group

```bash
yc managed-kubernetes node-group create \
  --name autoscale-ng \
  --cluster-name my-cluster \
  --platform-id standard-v3 \
  --cores 4 \
  --memory 8 \
  --disk-type network-ssd \
  --disk-size 96 \
  --auto-scale min=2,max=10,initial=3 \
  --location zone=ru-central1-a,subnet-name=default-ru-central1-a
```

**Auto-scale parameters:**
- `min` - Minimum nodes
- `max` - Maximum nodes
- `initial` - Initial node count

##### Node Group with Taints

```bash
yc managed-kubernetes node-group create \
  --name tainted-ng \
  --cluster-name my-cluster \
  --platform-id standard-v3 \
  --cores 4 \
  --memory 8 \
  --disk-type network-ssd \
  --disk-size 64 \
  --fixed-size 2 \
  --location zone=ru-central1-a,subnet-name=default-ru-central1-a \
  --node-labels environment=production,tier=database \
  --node-taints dedicated=database:NoSchedule
```

**Taint effects:**
- `NoSchedule` - Prohibit new pods
- `PreferNoSchedule` - Avoid if possible
- `NoExecute` - Evict existing pods

##### Preemptible Nodes (Cost-Optimized)

```bash
yc managed-kubernetes node-group create \
  --name preemptible-ng \
  --cluster-name my-cluster \
  --platform-id standard-v3 \
  --preemptible \
  --cores 2 \
  --memory 4 \
  --disk-type network-ssd \
  --disk-size 32 \
  --fixed-size 3 \
  --location zone=ru-central1-a,subnet-name=default-ru-central1-a
```

**Preemptible nodes:**
- Significantly cheaper than regular VMs
- Can be stopped by Yandex Cloud with 30-second warning
- Suitable for fault-tolerant workloads

##### GPU Node Group

```bash
yc managed-kubernetes node-group create \
  --name gpu-ng \
  --cluster-name my-cluster \
  --platform-id gpu-standard-v3 \
  --gpus 1 \
  --cores 8 \
  --memory 96 \
  --disk-type network-ssd \
  --disk-size 256 \
  --fixed-size 1 \
  --location zone=ru-central1-a,subnet-name=default-ru-central1-a \
  --node-labels workload=ml,gpu=true \
  --node-taints nvidia.com/gpu=present:NoSchedule
```

**GPU requirements:**
- Platform: `gpu-standard-v3` or `gpu-standard-v2`
- Available zones: `ru-central1-a`, `ru-central1-b`
- Pre-installed NVIDIA drivers and CUDA
- Kubernetes 1.16+

##### Multi-Zone Node Group

```bash
yc managed-kubernetes node-group create \
  --name multi-zone-ng \
  --cluster-name my-cluster \
  --platform-id standard-v3 \
  --cores 4 \
  --memory 8 \
  --disk-type network-ssd \
  --disk-size 64 \
  --fixed-size 6 \
  --location zone=ru-central1-a,subnet-name=subnet-a \
  --location zone=ru-central1-b,subnet-name=subnet-b \
  --location zone=ru-central1-d,subnet-name=subnet-d
```

Nodes distributed evenly across specified zones for high availability.

#### Platform IDs

| Platform ID | CPU | Use Case |
|-------------|-----|----------|
| `standard-v2` | Intel Cascade Lake | General purpose |
| `standard-v3` | Intel Ice Lake | Modern general purpose (recommended) |
| `highfreq-v3` | High-freq Intel Ice Lake | CPU-intensive |
| `gpu-standard-v3` | NVIDIA GPU | ML/AI workloads |
| `gpu-standard-v2` | NVIDIA GPU (older) | Legacy GPU |

#### Update Node Group

**Basic settings:**
```bash
yc managed-kubernetes node-group update <node_group_id> \
  --new-name <new_name> \
  --description <description> \
  --version <kubernetes_version>
```

**Network configuration:**
```bash
yc managed-kubernetes node-group update <node_group_id> \
  --network-interface security-group-ids=[<security_group_ids>],ipv4-address=nat
```

**Scaling - Fixed size:**
```bash
yc managed-kubernetes node-group update <node_group_id> \
  --fixed-size <number_of_nodes>
```

**Scaling - Autoscaling:**
```bash
yc managed-kubernetes node-group update <node_group_id> \
  --auto-scale min=<min>,max=<max>,initial=<initial>
```

**Additional options:**
```bash
yc managed-kubernetes node-group update <node_group_id> \
  --service-account-id <id> \
  --node-name <template> \
  --auto-upgrade \
  --network-acceleration-type standard|software-accelerated
```

#### Node Group Labels (Cloud Labels)

**Add labels:**
```bash
yc managed-kubernetes node-group add-labels <node_group_name> \
  --labels <label_name>=<label_value>
```

**Update labels (replaces all existing):**
```bash
yc managed-kubernetes node-group update <node_group_name> \
  --labels <label_name>=<label_value>
```

**Remove labels:**
```bash
yc managed-kubernetes node-group remove-labels <node_group_name> \
  --labels <label_name>
```

#### Kubernetes Node Labels

**Add Kubernetes node labels:**
```bash
yc managed-kubernetes node-group add-node-labels \
  --id <node_group_id> \
  --labels <key>=<value>,...
```

**Remove Kubernetes node labels:**
```bash
yc managed-kubernetes node-group remove-node-labels \
  --id <node_group_id> \
  --labels <label_key>,...
```

#### List Node Group Resources

```bash
# List nodes in specific node group
yc managed-kubernetes node-group list-nodes <node-group-name>

# List operations for node group
yc managed-kubernetes node-group list-operations <node-group-name>
```

#### Delete Node Group

```bash
# Delete node group
yc managed-kubernetes node-group delete <node-group-name>

# Delete by ID
yc managed-kubernetes node-group delete --id <node-group-id>

# Async deletion
yc managed-kubernetes node-group delete <node-group-name> --async
```

---

## Common Workflows

### Workflow 1: Container Registry Setup

```bash
# 1. Create registry
yc container registry create --name my-project-registry

# 2. Configure Docker authentication
yc container registry configure-docker

# 3. Build and tag image
docker build -t cr.yandex/<registry_id>/my-app:v1.0 .

# 4. Push image
docker push cr.yandex/<registry_id>/my-app:v1.0

# 5. Verify
yc container image list --repository-name=<registry_id>/my-app
```

### Workflow 2: Kubernetes Cluster Setup

```bash
# 1. Create network and subnet (if needed)
yc vpc network create --name k8s-network
yc vpc subnet create \
  --name k8s-subnet \
  --network-name k8s-network \
  --zone ru-central1-a \
  --range 10.128.0.0/24

# 2. Create service accounts
yc iam service-account create --name k8s-cluster-sa
yc iam service-account create --name k8s-node-sa

# 3. Assign roles
yc resource-manager folder add-access-binding <folder_id> \
  --role editor \
  --subject serviceAccount:<cluster_sa_id>

yc resource-manager folder add-access-binding <folder_id> \
  --role container-registry.images.puller \
  --subject serviceAccount:<node_sa_id>

# 4. Create cluster
yc managed-kubernetes cluster create \
  --name production-k8s \
  --network-name k8s-network \
  --zone ru-central1-a \
  --subnet-name k8s-subnet \
  --public-ip \
  --release-channel stable \
  --service-account-id <cluster_sa_id> \
  --node-service-account-id <node_sa_id>

# 5. Create node group
yc managed-kubernetes node-group create \
  --name production-nodes \
  --cluster-name production-k8s \
  --disk-type network-ssd \
  --disk-size 50 \
  --fixed-size 3 \
  --node-labels environment=production

# 6. Get kubeconfig
yc managed-kubernetes cluster get-credentials production-k8s --external
```

### Workflow 3: Deploy App from Container Registry to Kubernetes

```bash
# 1. Build and push image to registry
docker build -t cr.yandex/<registry_id>/myapp:v1 .
docker push cr.yandex/<registry_id>/myapp:v1

# 2. Create Kubernetes secret for registry access (if needed)
kubectl create secret docker-registry yc-registry \
  --docker-server=cr.yandex \
  --docker-username=json_key \
  --docker-password="$(cat key.json)"

# 3. Deploy to Kubernetes
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      imagePullSecrets:
        - name: yc-registry
      containers:
        - name: myapp
          image: cr.yandex/<registry_id>/myapp:v1
          ports:
            - containerPort: 8080
EOF
```

---

## Resources

- [Yandex Cloud Container Registry Documentation](https://yandex.cloud/en/services/container-registry)
- [Yandex Cloud Managed Kubernetes Documentation](https://yandex.cloud/en/docs/managed-kubernetes/quickstart)
- [YC CLI Reference](https://yandex.cloud/en/docs/cli/cli-ref/)
- [Container Registry Authentication](https://yandex.cloud/en/docs/container-registry/operations/authentication)
- [Creating a Kubernetes Cluster](https://yandex.cloud/en/docs/managed-kubernetes/operations/kubernetes-cluster/kubernetes-cluster-create)

---

## Quick Reference

### Container Registry Commands

| Command | Description |
|---------|-------------|
| `yc container registry create --name <name>` | Create registry |
| `yc container registry list` | List registries |
| `yc container registry delete <id>` | Delete registry |
| `yc container registry configure-docker` | Configure Docker auth |
| `yc container image list --repository-name=<name>` | List images |
| `yc container image delete <id>` | Delete image |
| `yc container registry set-ip-permissions <name> --pull <ip> --push <ip>` | Set IP permissions |

### Kubernetes Commands

#### Cluster Operations

| Command | Description |
|---------|-------------|
| `yc managed-kubernetes cluster list` | List clusters |
| `yc managed-kubernetes cluster get <name>` | Get cluster details |
| `yc managed-kubernetes cluster get-credentials <name> --external` | Get kubeconfig (external access) |
| `yc managed-kubernetes cluster create` | Create cluster (zonal or regional) |
| `yc managed-kubernetes cluster update <id>` | Update cluster |
| `yc managed-kubernetes cluster start <name>` | Start cluster |
| `yc managed-kubernetes cluster stop <name>` | Stop cluster |
| `yc managed-kubernetes cluster delete <id>` | Delete cluster |
| `yc managed-kubernetes cluster list-node-groups <name>` | List node groups in cluster |
| `yc managed-kubernetes cluster list-nodes <name>` | List all nodes in cluster |
| `yc managed-kubernetes cluster list-operations <name>` | List cluster operations |

#### Node Group Operations

| Command | Description |
|---------|-------------|
| `yc managed-kubernetes node-group list` | List node groups |
| `yc managed-kubernetes node-group get <name>` | Get node group details |
| `yc managed-kubernetes node-group create` | Create node group |
| `yc managed-kubernetes node-group update <id>` | Update node group |
| `yc managed-kubernetes node-group delete <name>` | Delete node group |
| `yc managed-kubernetes node-group list-nodes <name>` | List nodes in group |
| `yc managed-kubernetes node-group add-labels` | Add cloud labels |
| `yc managed-kubernetes node-group add-node-labels` | Add Kubernetes labels |
| `yc managed-kubernetes node-group remove-node-labels` | Remove Kubernetes labels |
| `yc managed-kubernetes node-group list-operations <name>` | List node group operations |

---

## Notes

### Container Registry
- **Container Registry address:** `cr.yandex`
- **Credential helper:** Docker must run without `sudo` for `docker-credential-yc` to work
- **Vulnerability scanning:** Available with `--secure` flag; incurs additional charges
- **Repository format:** `<registry_id>/<image_name>`

### Kubernetes Clusters
- **Service accounts:** Two separate accounts required (cluster + nodes)
- **IP ranges:** Cluster and service IPv4 ranges must NOT overlap with network subnets
- **Master types:**
  - **Zonal** - Single AZ, for development/testing
  - **Regional** - 3 AZs, for production high availability
- **kubeconfig:** Default location `~/.kube/config`, use `--external` for public access
- **Versions:** Update channels: `rapid` (early access), `regular` (balanced), `stable` (production)

### Node Groups
- **Platform IDs:** Use `standard-v3` (Intel Ice Lake) for modern workloads
- **Preemptible nodes:** 70% cheaper but can be stopped by Yandex Cloud
- **GPU support:** Available in `ru-central1-a` and `ru-central1-b` zones
- **Taints:** Use `NoSchedule`, `PreferNoSchedule`, or `NoExecute`
- **Auto-scaling:** Configure with `min`, `max`, and `initial` parameters
- **Multi-zone:** Specify multiple `--location` flags for zone distribution
