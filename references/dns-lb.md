# Yandex Cloud CLI: DNS and Load Balancers Guide

Comprehensive reference for managing DNS zones, records, and load balancers using Yandex Cloud CLI.

**Last updated:** 2026-01-16

---

## Table of Contents

1. [DNS Management](#dns-management)
   - [DNS Zones](#dns-zones)
   - [DNS Records](#dns-records)
   - [Common Record Types](#common-record-types)
2. [Network Load Balancer (L4)](#network-load-balancer-l4)
   - [Creating Network Load Balancers](#creating-network-load-balancers)
   - [Health Checks](#health-checks)
   - [Listeners](#listeners)
3. [Application Load Balancer (L7)](#application-load-balancer-l7)
   - [Target Groups](#target-groups)
   - [Backend Groups](#backend-groups)
   - [HTTP Routers](#http-routers)
   - [Load Balancer Creation](#load-balancer-creation)

---

## DNS Management

### DNS Zones

#### Create a Public DNS Zone

```bash
yc dns zone create \
  --name <zone_name> \
  --zone <domain_zone>. \
  --public-visibility
```

**Parameters:**
- `--name` - Zone name (must be unique within a folder)
- `--zone` - Domain zone name (must end with a trailing dot `.`)
- `--public-visibility` - Makes the zone publicly accessible

**Example:**

```bash
yc dns zone create \
  --name my-public-zone \
  --zone example.com. \
  --public-visibility
```

#### Create a Private DNS Zone

```bash
yc dns zone create \
  --name <zone_name> \
  --zone <domain_zone>. \
  --private-visibility=true \
  --network-ids="<network_1_ID>","<network_2_ID>"
```

**Example:**

```bash
yc dns zone create \
  --name test-zone \
  --zone testing. \
  --private-visibility=true \
  --network-ids="enp1234567890abcd"
```

**Notes:**
- Cannot create top-level domain (TLD) zones
- Domain zone name must end with a trailing dot (`.`)
- For private zones, specify network IDs where the zone will be accessible

#### List DNS Zones

```bash
yc dns zone list
```

#### Delete a DNS Zone

```bash
yc dns zone delete --name <zone_name>
```

---

### DNS Records

#### Add DNS Records

```bash
yc dns zone add-records \
  --name <zone_name> \
  --record "<name> <TTL> <type> <data>"
```

**Record Format:** `"name TTL type data"`

**Parameters:**
- `name` - Record name (subdomain or `@` for root)
- `TTL` - Time to live in seconds (e.g., 600)
- `type` - Record type (A, AAAA, CNAME, TXT, MX, etc.)
- `data` - Record data (IP address, domain, text, etc.)

#### Replace Records

Replace all records of a specific name and type:

```bash
yc dns zone replace-records \
  --name <zone_name> \
  --record "<name> <TTL> <type> <data>"
```

#### List Records

```bash
yc dns zone list-records --name <zone_name>
```

---

### Common Record Types

#### A Record (IPv4 Address)

Points a domain name to an IPv4 address.

```bash
yc dns zone add-records \
  --name test-zone \
  --record "test-vm-1 600 A 10.128.0.5"
```

**Wildcard A Record:**

```bash
yc dns zone add-records \
  --name your-zone \
  --record "*.alb.example.com. 60 A 51.250.10.15"
```

#### AAAA Record (IPv6 Address)

Points a domain name to an IPv6 address.

```bash
yc dns zone add-records \
  --name test-zone \
  --record "www 600 AAAA 2001:0db8:85a3:0000:0000:8a2e:0370:7334"
```

#### CNAME Record (Canonical Name)

Creates an alias from one domain name to another.

```bash
yc dns zone add-records \
  --name test-zone \
  --record "www 600 CNAME example.com."
```

**Example for ACME challenge:**

```bash
yc dns zone add-records \
  --name test-zone \
  --record "_acme-challenge 600 CNAME fpd1234567890.cm.yandexcloud.net."
```

**Notes:**
- Target domain must end with a trailing dot (`.`)
- Cannot create CNAME at the zone apex (root domain)

#### TXT Record (Text)

Stores arbitrary text data, often used for verification and SPF records.

```bash
yc dns zone add-records \
  --name test-zone \
  --record "@ 600 TXT \"v=spf1 include:_spf.yandex.net ~all\""
```

**Limitations:**
- Maximum 255 characters per line
- Maximum 1024 characters total
- Use quotes (`"`) if the text contains semicolons (`;`)

#### MX Record (Mail Exchange)

Specifies mail servers for the domain.

```bash
yc dns zone add-records \
  --name test-zone \
  --record "@ 600 MX \"10 mx.yandex.net.\""
```

**Format:** `"priority mail_server"`

#### NS Record (Name Server)

Delegates a subdomain to different name servers.

```bash
yc dns zone add-records \
  --name test-zone \
  --record "subdomain 600 NS ns1.example.com."
```

#### SRV Record (Service)

Defines the location of services.

```bash
yc dns zone add-records \
  --name test-zone \
  --record "_service._tcp 600 SRV \"10 20 5060 sipserver.example.com.\""
```

**Format:** `"priority weight port target"`

---

## Network Load Balancer (L4)

Network Load Balancer operates at Layer 4 (TCP/UDP) and distributes traffic across backend resources.

### Creating Network Load Balancers

#### Basic Network Load Balancer

```bash
yc load-balancer network-load-balancer create \
  --region-id ru-central1 \
  --name test-load-balancer \
  --listener name=test-listener,external-ip-version=ipv4,port=80
```

**Parameters:**
- `--region-id` - Region ID (e.g., `ru-central1`)
- `--name` - Load balancer name
- `--listener` - Listener configuration (see below)

#### Network Load Balancer with Target Group

```bash
yc load-balancer network-load-balancer create \
  --region-id ru-central1 \
  --name test-load-balancer-3 \
  --listener name=test-listener,external-ip-version=ipv4,port=80 \
  --target-group target-group-id=b7rjtf12qdeehrj31hri,healthcheck-name=http,healthcheck-interval=2s,healthcheck-timeout=1s,healthcheck-unhealthythreshold=2,healthcheck-healthythreshold=2,healthcheck-http-port=80
```

#### Internal Network Load Balancer

For load balancers without external IP addresses:

```bash
yc load-balancer network-load-balancer create \
  --region-id ru-central1 \
  --name internal-lb \
  --type internal \
  --listener name=internal-listener,port=80,internal-subnet-id=<subnet_id>
```

**Parameters:**
- `--type internal` - Creates an internal load balancer
- `internal-subnet-id` - Subnet ID for the internal listener

---

### Health Checks

Health checks determine if backend instances are ready to receive traffic.

**Health Check Parameters:**

| Parameter | Description | Example |
|-----------|-------------|---------|
| `healthcheck-name` | Name of health check | `http` |
| `healthcheck-interval` | Interval between checks | `2s` |
| `healthcheck-timeout` | Timeout for check response | `1s` |
| `healthcheck-unhealthythreshold` | Failed checks before marking unhealthy | `2` |
| `healthcheck-healthythreshold` | Successful checks before marking healthy | `2` |
| `healthcheck-http-port` | Port for HTTP health checks | `80` |
| `healthcheck-http-path` | Path for HTTP health checks | `/health` |

**Example:**

```bash
--target-group target-group-id=<id>,\
  healthcheck-name=http,\
  healthcheck-interval=2s,\
  healthcheck-timeout=1s,\
  healthcheck-unhealthythreshold=2,\
  healthcheck-healthythreshold=2,\
  healthcheck-http-port=80,\
  healthcheck-http-path=/health
```

---

### Listeners

Listeners define how the load balancer accepts incoming traffic.

**Listener Parameters:**

| Parameter | Description | Example |
|-----------|-------------|---------|
| `name` | Listener name | `test-listener` |
| `external-ip-version` | IP version for external listener | `ipv4` or `ipv6` |
| `port` | Port number | `80` or `443` |
| `internal-subnet-id` | Subnet ID for internal listener | `e2l123...` |
| `target-port` | Backend target port | `8080` |

**External IPv4 Listener:**

```bash
--listener name=web-listener,external-ip-version=ipv4,port=80
```

**External IPv6 Listener:**

```bash
--listener name=web-listener-v6,external-ip-version=ipv6,port=443
```

**Internal Listener:**

```bash
--listener name=internal-listener,port=80,internal-subnet-id=e2l123456789
```

#### Manage Network Load Balancers

**List load balancers:**

```bash
yc load-balancer network-load-balancer list
```

**Get load balancer details:**

```bash
yc load-balancer network-load-balancer get --name <lb_name>
```

**Delete load balancer:**

```bash
yc load-balancer network-load-balancer delete --name <lb_name>
```

**Start/Stop load balancer:**

```bash
yc load-balancer network-load-balancer start --name <lb_name>
yc load-balancer network-load-balancer stop --name <lb_name>
```

---

## Application Load Balancer (L7)

Application Load Balancer operates at Layer 7 (HTTP/HTTPS) and provides advanced routing, TLS termination, and content-based routing.

### Target Groups

Target groups define the backend resources that will receive traffic.

#### Create Target Group

```bash
yc alb target-group create <target_group_name> \
  --target subnet-name=<subnet_name>,ip-address=<internal_ip>
```

**Example:**

```bash
yc alb target-group create web-servers \
  --target subnet-name=default-ru-central1-a,ip-address=10.128.0.5 \
  --target subnet-name=default-ru-central1-b,ip-address=10.129.0.5
```

**Parameters:**
- `subnet-name` - Name of the subnet where the target is located
- `ip-address` - Internal IP address of the target

#### List Target Groups

```bash
yc alb target-group list
```

---

### Backend Groups

Backend groups contain backends (target groups) and health check configurations.

#### Create Backend Group with HTTP Backend

```bash
yc alb backend-group create <backend_group_name>

yc alb backend-group add-http-backend \
  --backend-group-name <backend_group_name> \
  --name <backend_name> \
  --weight <backend_weight> \
  --port <backend_port> \
  --target-group-id <target_group_id> \
  --panic-threshold 90 \
  --http-healthcheck healthy-threshold=2,unhealthy-threshold=2,timeout=1s,interval=3s,path=/
```

**Example:**

```bash
yc alb backend-group create web-backend-group

yc alb backend-group add-http-backend \
  --backend-group-name web-backend-group \
  --name backend-1 \
  --weight 100 \
  --port 80 \
  --target-group-id enp1234567890 \
  --panic-threshold 90 \
  --http-healthcheck healthy-threshold=2,unhealthy-threshold=2,timeout=1s,interval=3s,path=/health
```

**Health Check Parameters:**

| Parameter | Description | Example |
|-----------|-------------|---------|
| `healthy-threshold` | Successful checks to mark healthy | `2` |
| `unhealthy-threshold` | Failed checks to mark unhealthy | `2` |
| `timeout` | Health check timeout | `1s` |
| `interval` | Interval between checks | `3s` |
| `path` | HTTP path for health check | `/health` |

**Backend Parameters:**

| Parameter | Description | Example |
|-----------|-------------|---------|
| `--name` | Backend name | `backend-1` |
| `--weight` | Traffic distribution weight | `100` |
| `--port` | Backend port | `80` |
| `--target-group-id` | ID of target group | `enp123...` |
| `--panic-threshold` | Panic mode threshold (%) | `90` |

---

### HTTP Routers

HTTP routers define routing rules for incoming traffic.

#### Create HTTP Router

```bash
yc alb http-router create <http_router_name>
```

#### Add Virtual Host

```bash
yc alb virtual-host create <virtual_host_name> \
  --http-router-name <http_router_name> \
  --authority example.com
```

#### Add HTTP Route

```bash
yc alb virtual-host append-http-route <route_name> \
  --http-router-name <http_router_name> \
  --virtual-host-name <virtual_host_name> \
  --prefix-path-match / \
  --backend-group-name <backend_group_name> \
  --request-timeout 60s \
  --request-idle-timeout 60s
```

**Example:**

```bash
yc alb http-router create web-router

yc alb virtual-host create web-vhost \
  --http-router-name web-router \
  --authority example.com,www.example.com

yc alb virtual-host append-http-route default-route \
  --http-router-name web-router \
  --virtual-host-name web-vhost \
  --prefix-path-match / \
  --backend-group-name web-backend-group \
  --request-timeout 60s \
  --request-idle-timeout 60s
```

**Routing Parameters:**

| Parameter | Description | Example |
|-----------|-------------|---------|
| `--prefix-path-match` | Match URL path prefix | `/` or `/api` |
| `--exact-path-match` | Match exact URL path | `/login` |
| `--regex-path-match` | Match URL path by regex | `^/api/.*` |
| `--backend-group-name` | Backend group to route to | `web-backend-group` |
| `--request-timeout` | Request timeout | `60s` |
| `--request-idle-timeout` | Idle timeout | `60s` |

---

### Load Balancer Creation

#### Create Application Load Balancer

```bash
yc alb load-balancer create <load_balancer_name> \
  --network-name <network_name> \
  --location subnet-name=<subnet_name>,zone=ru-central1-a
```

**Example:**

```bash
yc alb load-balancer create web-alb \
  --network-name default \
  --location subnet-name=default-ru-central1-a,zone=ru-central1-a \
  --location subnet-name=default-ru-central1-b,zone=ru-central1-b
```

#### Add Listener to Load Balancer

```bash
yc alb load-balancer add-listener <load_balancer_name> \
  --listener-name <listener_name> \
  --http-router-id <http_router_id> \
  --external-ipv4-endpoint port=80
```

**Example:**

```bash
yc alb load-balancer add-listener web-alb \
  --listener-name http-listener \
  --http-router-id e2l123456789 \
  --external-ipv4-endpoint port=80
```

**HTTPS Listener with TLS:**

```bash
yc alb load-balancer add-listener web-alb \
  --listener-name https-listener \
  --http-router-id e2l123456789 \
  --external-ipv4-endpoint port=443 \
  --tls-certificate-id fpd123456789
```

**Listener Parameters:**

| Parameter | Description | Example |
|-----------|-------------|---------|
| `--listener-name` | Name of listener | `http-listener` |
| `--http-router-id` | HTTP router ID | `e2l123...` |
| `--external-ipv4-endpoint` | External IPv4 endpoint | `port=80` |
| `--external-ipv6-endpoint` | External IPv6 endpoint | `port=80` |
| `--internal-ipv4-endpoint` | Internal IPv4 endpoint | `port=80,subnet-id=...` |
| `--tls-certificate-id` | TLS certificate ID | `fpd123...` |

#### Manage Application Load Balancers

**List load balancers:**

```bash
yc alb load-balancer list
```

**Get load balancer details:**

```bash
yc alb load-balancer get --name <lb_name>
```

**Delete load balancer:**

```bash
yc alb load-balancer delete --name <lb_name>
```

---

## Quick Reference

### DNS Commands

```bash
# Create public DNS zone
yc dns zone create --name <name> --zone <domain>. --public-visibility

# Add A record
yc dns zone add-records --name <zone> --record "hostname 600 A <ip>"

# Add CNAME record
yc dns zone add-records --name <zone> --record "www 600 CNAME example.com."

# List records
yc dns zone list-records --name <zone>
```

### Network Load Balancer Commands

```bash
# Create network load balancer with health check
yc load-balancer network-load-balancer create \
  --region-id ru-central1 \
  --name <name> \
  --listener name=listener,external-ip-version=ipv4,port=80 \
  --target-group target-group-id=<id>,healthcheck-name=http,healthcheck-interval=2s,healthcheck-timeout=1s

# List load balancers
yc load-balancer network-load-balancer list
```

### Application Load Balancer Commands

```bash
# Create target group
yc alb target-group create <name> --target subnet-name=<subnet>,ip-address=<ip>

# Create backend group
yc alb backend-group create <name>
yc alb backend-group add-http-backend --backend-group-name <name> --name backend-1 --port 80 --target-group-id <id>

# Create HTTP router
yc alb http-router create <name>
yc alb virtual-host create <vhost> --http-router-name <router> --authority example.com
yc alb virtual-host append-http-route default --http-router-name <router> --virtual-host-name <vhost> --prefix-path-match / --backend-group-name <backend>

# Create load balancer
yc alb load-balancer create <name> --network-name <network> --location subnet-name=<subnet>,zone=ru-central1-a
yc alb load-balancer add-listener <lb> --listener-name http --http-router-id <router-id> --external-ipv4-endpoint port=80
```

---

## Additional Resources

- [Yandex Cloud DNS Documentation](https://cloud.yandex.com/en/docs/dns/)
- [Network Load Balancer Documentation](https://cloud.yandex.com/en/docs/network-load-balancer/)
- [Application Load Balancer Documentation](https://cloud.yandex.com/en/docs/application-load-balancer/)
- [YC CLI Reference](https://cloud.yandex.com/en/docs/cli/cli-ref/)

---

## Notes

- All domain names in DNS records must end with a trailing dot (`.`)
- DNS zone names must be unique within a folder
- Load balancer names must be unique within a folder
- Health checks are crucial for high availability
- Use `--help` flag with any command for detailed parameter information
- Network Load Balancer operates at Layer 4 (TCP/UDP)
- Application Load Balancer operates at Layer 7 (HTTP/HTTPS)
