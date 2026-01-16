# VPC Networking Reference

## Networks

```bash
# Create network
yc vpc network create --name my-network --description "My network"

# Get network details
yc vpc network get <network-id>
yc vpc network get --name my-network

# Delete network
yc vpc network delete <network-id>
```

## Subnets

```bash
# Create subnet
yc vpc subnet create \
  --name my-subnet \
  --zone kz1-a \
  --network-name my-network \
  --range 10.1.0.0/24

# With DNS options
yc vpc subnet create \
  --name my-subnet \
  --zone kz1-a \
  --network-id <network-id> \
  --range 10.1.0.0/24 \
  --domain-name internal.local \
  --domain-name-servers 10.1.0.2

# List subnets
yc vpc subnet list

# Get subnet by name
yc vpc subnet get --name my-subnet
```

## Security Groups

### Create Security Group

```bash
yc vpc security-group create \
  --name web-server \
  --network-id <network-id> \
  --rule "direction=ingress,port=22,protocol=tcp,v4-cidrs=[YOUR_IP/32]" \
  --rule "direction=ingress,port=80,protocol=tcp,v4-cidrs=[0.0.0.0/0]" \
  --rule "direction=ingress,port=443,protocol=tcp,v4-cidrs=[0.0.0.0/0]" \
  --rule "direction=egress,protocol=any,v4-cidrs=[0.0.0.0/0]"
```

### Rule Syntax

```
direction=<ingress|egress>,
port=<port>,                    # Single port
from-port=<start>,to-port=<end>, # Port range
protocol=<tcp|udp|icmp|any>,
v4-cidrs=[<cidr1>,<cidr2>]
```

### Common Security Rules

```bash
# SSH from specific IP
--rule "direction=ingress,port=22,protocol=tcp,v4-cidrs=[203.0.113.0/32]"

# HTTP/HTTPS from anywhere
--rule "direction=ingress,port=80,protocol=tcp,v4-cidrs=[0.0.0.0/0]"
--rule "direction=ingress,port=443,protocol=tcp,v4-cidrs=[0.0.0.0/0]"

# Custom port range
--rule "direction=ingress,from-port=8000,to-port=9000,protocol=tcp,v4-cidrs=[0.0.0.0/0]"

# ICMP (ping)
--rule "direction=ingress,protocol=icmp,v4-cidrs=[0.0.0.0/0]"

# All outbound
--rule "direction=egress,protocol=any,v4-cidrs=[0.0.0.0/0]"

# Internal network only
--rule "direction=ingress,protocol=any,v4-cidrs=[10.0.0.0/8]"
```

### Update Security Group Rules

```bash
# Add rule
yc vpc security-group update-rules <sg-id> \
  --add-rule "direction=ingress,port=3306,protocol=tcp,v4-cidrs=[10.0.0.0/8]"

# Delete rule by ID
yc vpc security-group update-rules <sg-id> \
  --delete-rule-id <rule-id>

# Get rule IDs
yc vpc security-group get <sg-id> --format json | jq '.rules[].id'
```

### Apply to VM

```bash
# At creation
yc compute instance create \
  --network-interface subnet-name=my-subnet,security-group-ids=<sg-id> \
  ...

# Update existing VM
yc compute instance update-network-interface <instance-id> \
  --network-interface-index 0 \
  --security-group-ids <sg-id1>,<sg-id2>
```

## Static Addresses

```bash
# Reserve external IP
yc vpc address create \
  --name my-static-ip \
  --external-ipv4 zone=kz1-a

# List addresses
yc vpc address list

# Assign to VM (at creation)
--network-interface subnet-name=my-subnet,nat-address=<reserved-ip>
```

## Route Tables

```bash
# Create route table
yc vpc route-table create \
  --name my-routes \
  --network-id <network-id> \
  --route destination=0.0.0.0/0,next-hop=10.1.0.1

# Attach to subnet
yc vpc subnet update <subnet-id> \
  --route-table-id <route-table-id>
```
