# Network Design

## Overview

Networking is one of the most critical components of an OpenStack deployment. Unlike traditional virtualization platforms where virtual machines are attached directly to a virtual switch, OpenStack introduces a software-defined networking (SDN) model that separates management, tenant traffic, storage traffic, and external connectivity.

A properly designed network architecture provides:

- Secure communication between OpenStack services
- Tenant isolation
- High scalability
- Flexible virtual networking
- External Internet access
- Efficient east-west traffic
- Reliable service discovery

This laboratory follows the networking concepts recommended by the OpenStack Architecture Guide while remaining suitable for a VMware-based learning environment.

---

# Network Architecture

The infrastructure is divided into multiple logical networks.

```text
                                    Internet
                                        │
                                        │
                              External Network
                                        │
                              Provider Network
                                        │
                     ┌────────────────────────────┐
                     │      Controller Node       │
                     └─────────────┬──────────────┘
                                   │
───────────────────────────────────┼──────────────────────────────────
                                   │
          Management Network       │
                                   │
        ┌────────────┬─────────────┴─────────────┬────────────┐
        │            │                           │            │
  Compute01     Compute02                  Compute03     Storage
        │                                            │
        └────────────────────────────────────────────┘

                     VXLAN Tunnel Network

────────────────────────────────────────────────────────────

                     Storage Network

────────────────────────────────────────────────────────────

                      Swift Storage Network
```

Instead of allowing all traffic to share a single network, different traffic types are isolated into dedicated logical segments.

This approach improves:

- Performance
- Security
- Troubleshooting
- Scalability

---

# Network Types

The laboratory uses four primary network categories.

| Network    | Purpose                         |
| ---------- | ------------------------------- |
| Management | OpenStack service communication |
| Provider   | Internet connectivity           |
| Tenant     | Virtual machine communication   |
| Storage    | Storage synchronization         |

Each network serves a unique function.

---

# Management Network

## Purpose

The Management Network carries communication between OpenStack services.

Examples include:

- Keystone authentication
- Nova RPC
- Neutron API
- Cinder API
- RabbitMQ
- MariaDB
- Memcached
- Etcd

Without the management network, OpenStack services cannot communicate.

---

## Characteristics

Typical characteristics include:

- Private IP addresses
- No Internet exposure
- Low latency
- Stable connectivity

Example traffic

```text
Nova API

↓

RabbitMQ

↓

nova-compute

↓

Placement

↓

Database
```

Management traffic should always remain isolated from tenant traffic.

---

# Provider Network

## Purpose

The Provider Network connects virtual machines to external networks.

This network is responsible for:

- Internet access
- Floating IP addresses
- External routing
- Public connectivity

Virtual machines access the Internet through Neutron routers connected to the Provider Network.

---

## Floating IP

A Floating IP is a public IP address dynamically associated with a virtual machine.

Example

```text
Internet

↓

Floating IP

↓

Neutron Router

↓

Tenant Network

↓

Virtual Machine
```

Advantages

- Public access
- Dynamic assignment
- Easy migration
- NAT functionality

---

# Tenant Network

Tenant Networks are isolated virtual networks created by OpenStack users.

Each project can create its own networks without affecting other tenants.

Example

```text
Project A

10.0.0.0/24

──────────────────

Project B

10.0.1.0/24
```

Isolation is implemented using VXLAN.

---

# VXLAN

Virtual Extensible LAN (VXLAN) is an overlay networking technology.

Instead of relying on VLAN IDs, VXLAN encapsulates Layer-2 traffic inside Layer-3 packets.

Advantages include:

- Millions of isolated networks
- Better scalability
- Multi-host communication
- Cloud-native networking

Traffic flow

```text
VM

↓

Linux Bridge

↓

VXLAN Tunnel

↓

Compute Node

↓

Destination VM
```

---

# Linux Bridge

This laboratory uses Linux Bridge as the Layer-2 switching mechanism.

Responsibilities include:

- Bridge virtual interfaces
- Connect VM interfaces
- Forward Ethernet frames
- Integrate physical interfaces

Each instance interface is attached to a Linux bridge before reaching the physical network.

---

# Neutron Components

Networking services are coordinated by Neutron.

Major components include:

## Neutron Server

Responsible for:

- API requests
- Network database
- Scheduling
- Network orchestration

---

## L3 Agent

Provides:

- Routing
- NAT
- Floating IP
- Gateway services

---

## DHCP Agent

Automatically assigns:

- IP address
- Gateway
- DNS server
- Lease information

---

## Metadata Agent

Provides metadata services to instances.

Examples

- SSH keys
- Cloud-init
- Instance metadata

---

# Security Groups

Security Groups function as stateful virtual firewalls.

Example rules

| Protocol | Port | Purpose |
| -------- | ---- | ------- |
| ICMP     | All  | Ping    |
| TCP      | 22   | SSH     |
| TCP      | 80   | HTTP    |
| TCP      | 443  | HTTPS   |

Without appropriate rules, instances remain inaccessible despite having valid IP addresses.

---

# Virtual Router

Each tenant network connects to the external network through a virtual router.

```text
Tenant Network

↓

Neutron Router

↓

Provider Network

↓

Internet
```

The router performs:

- Routing
- NAT
- Floating IP translation

---

# Network Interfaces

Each OpenStack node may contain multiple network interfaces.

Example

| Interface | Purpose    |
| --------- | ---------- |
| ens33     | Management |
| ens34     | Provider   |
| ens35     | VXLAN      |
| ens36     | Storage    |

Separating traffic prevents congestion and improves reliability.

---

# IP Address Planning

A structured addressing plan simplifies management.

Example

| Network    | CIDR            |
| ---------- | --------------- |
| Management | 192.168.10.0/24 |
| Provider   | 192.168.20.0/24 |
| VXLAN      | 10.10.10.0/24   |
| Storage    | 172.16.10.0/24  |

Production environments typically reserve separate subnets for API endpoints and storage replication.

---

# Network Traffic Flow

The following example demonstrates how an external client accesses a virtual machine.

```text
Client

↓

Internet

↓

Provider Network

↓

Floating IP

↓

Neutron Router

↓

Tenant Network

↓

Virtual Machine
```

Internal communication between two virtual machines follows a different path.

```text
VM A

↓

Linux Bridge

↓

VXLAN Tunnel

↓

Destination Compute Node

↓

Linux Bridge

↓

VM B
```

---

# Common Networking Issues

| Problem                          | Possible Cause          |
| -------------------------------- | ----------------------- |
| Cannot ping VM                   | Security Group          |
| No Internet                      | Missing Floating IP     |
| DHCP failure                     | DHCP Agent stopped      |
| Metadata unavailable             | Metadata Agent failure  |
| Instance unreachable             | Router misconfiguration |
| Cross-node communication failure | VXLAN configuration     |

Systematic troubleshooting should begin by identifying the affected network layer before inspecting OpenStack services.

---

# Best Practices

Recommended practices include:

- Separate management traffic from tenant traffic.
- Use dedicated interfaces for storage replication.
- Minimize exposure of management services.
- Allocate Floating IPs only when required.
- Apply the principle of least privilege to Security Groups.
- Maintain consistent IP addressing.
- Document network topology.
- Monitor bandwidth utilization.
- Validate connectivity after every configuration change.

---

# Summary

Networking is the backbone of an OpenStack cloud.

By separating management, provider, tenant, and storage traffic into dedicated logical networks, the platform achieves better scalability, security, and operational stability.

Understanding how Neutron, Linux Bridge, VXLAN, routers, DHCP, metadata services, and Security Groups interact is essential for successfully deploying and operating an OpenStack environment.
