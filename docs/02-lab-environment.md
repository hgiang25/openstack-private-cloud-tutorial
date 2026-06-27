# Lab Environment

## Overview

This document describes the infrastructure used to deploy the OpenStack Private Cloud laboratory.

Unlike a minimal all-in-one installation, this project follows a **multi-node deployment model** inspired by the official OpenStack architecture. Each virtual machine performs a dedicated role, allowing the environment to more closely resemble a production infrastructure while remaining suitable for learning and experimentation.

The entire platform is deployed on VMware Workstation using Ubuntu Server 22.04 LTS as the operating system for all nodes.

---

# Design Goals

The laboratory was designed with the following objectives:

- Separate OpenStack services into dedicated nodes.
- Simulate an enterprise private cloud deployment.
- Minimize service interference between components.
- Provide an environment for networking, storage, and orchestration experiments.
- Demonstrate horizontal scalability.
- Simplify troubleshooting by separating responsibilities.

---

# Physical Host

The OpenStack laboratory runs entirely on a single physical workstation.

Although production deployments normally use multiple physical servers, virtualization provides an efficient method for understanding OpenStack architecture without requiring dedicated hardware.

Example Host Specification

| Component             | Specification                               |
| --------------------- | ------------------------------------------- |
| CPU                   | Intel Core i7 / AMD Ryzen 7 (or equivalent) |
| Memory                | 32 GB RAM (Recommended 64 GB)               |
| Storage               | 1 TB NVMe SSD                               |
| Virtualization        | VMware Workstation Pro                      |
| Host Operating System | Windows 11                                  |

The host machine provides sufficient resources to execute multiple virtual machines simultaneously while maintaining acceptable performance.

---

# Why VMware Workstation?

Several virtualization platforms could be used for an OpenStack laboratory.

Examples include:

- VMware Workstation
- VMware ESXi
- VirtualBox
- KVM
- Hyper-V

VMware Workstation was selected because it provides:

- Stable virtual networking
- Flexible virtual switches
- Snapshot management
- Easy VM cloning
- High compatibility with Ubuntu Server

Snapshots significantly reduce recovery time during deployment and troubleshooting.

---

# Virtual Infrastructure

The environment consists of multiple Ubuntu Server virtual machines.

Each node has a clearly defined responsibility.

```text
                     VMware Workstation

                           │

        ┌────────────────────────────────────┐

        │                                    │

 Controller

 Compute-01

 Compute-02

 Compute-03

 Storage

 Swift

 Prometheus

        │                                    │

        └────────────────────────────────────┘
```

Each virtual machine communicates through dedicated virtual networks.

---

# Node Roles

## Controller Node

The Controller Node hosts the management plane of OpenStack.

Responsibilities include:

- Identity Management
- API Endpoints
- Scheduling
- Dashboard
- Networking Control Plane
- Image Service
- Orchestration
- Metadata Services

Installed Services

- Keystone
- Glance
- Nova API
- Nova Scheduler
- Nova Conductor
- Placement API
- Neutron Server
- Horizon
- Heat
- Cinder API
- Swift Proxy

Because nearly every OpenStack service depends on the Controller Node, it is considered the most critical component of the environment.

---

## Compute Nodes

Compute nodes execute virtual machine workloads.

Each Compute Node registers itself with Nova and reports available resources to the Placement service.

Responsibilities include:

- Hypervisor
- Virtual Machine Lifecycle
- CPU Scheduling
- Memory Allocation
- Virtual Networking

Installed Services

- nova-compute
- neutron-agent

Instances are distributed among compute nodes by Nova Scheduler according to available resources.

---

## Storage Node

Persistent block storage is provided by a dedicated storage node.

Responsibilities include:

- Volume Creation
- Snapshot
- Backup
- Restore

Installed Services

- cinder-volume
- cinder-backup

LVM is used as the backend storage driver.

---

## Swift Node

Swift provides object storage.

Unlike block storage, object storage stores files as objects inside containers.

Typical use cases include:

- Images
- Backup files
- Static web content
- Application assets
- Log archives

Installed Services

- swift-object
- swift-container
- swift-account

---

## Monitoring Node

Monitoring is implemented using Prometheus.

Responsibilities include:

- Resource monitoring
- CPU utilization
- Memory utilization
- Node metrics
- Alert generation

This node demonstrates how infrastructure metrics can be integrated into cloud automation workflows.

---

# Resource Planning

Proper resource allocation is essential for OpenStack.

Each service consumes CPU, memory, and storage differently.

Example allocation:

| Node       | vCPU | RAM  | Disk   |
| ---------- | ---- | ---- | ------ |
| Controller | 4    | 8 GB | 80 GB  |
| Compute-01 | 4    | 8 GB | 80 GB  |
| Compute-02 | 4    | 8 GB | 80 GB  |
| Compute-03 | 4    | 8 GB | 80 GB  |
| Storage    | 2    | 4 GB | 120 GB |
| Swift      | 2    | 4 GB | 120 GB |
| Prometheus | 2    | 2 GB | 40 GB  |

Actual allocations may vary depending on available hardware resources.

---

# Operating System

Ubuntu Server 22.04 LTS was selected because:

- Long-Term Support
- Stable package repositories
- Excellent OpenStack compatibility
- Strong community support
- Predictable release cycle

Using the same operating system across all nodes simplifies package management and maintenance.

---

# Software Components

The laboratory includes several supporting infrastructure services in addition to OpenStack.

| Component          | Purpose                    |
| ------------------ | -------------------------- |
| MariaDB            | Database backend           |
| RabbitMQ           | Message broker             |
| Memcached          | Authentication token cache |
| Etcd               | Distributed coordination   |
| Chrony             | Time synchronization       |
| Prometheus         | Monitoring                 |
| Apache HTTP Server | OpenStack API endpoints    |

Each component is essential for reliable operation of the cloud platform.

---

# Time Synchronization

Accurate time synchronization is mandatory.

Authentication tokens generated by Keystone depend on synchronized clocks.

All nodes synchronize their clocks using Chrony.

```text
Controller (NTP Server)

↓

Compute Nodes

↓

Storage

↓

Swift

↓

Prometheus
```

Even a small time drift may cause authentication failures between OpenStack services.

---

# DNS Resolution

Reliable hostname resolution simplifies service communication.

Each node should resolve:

- Controller
- Compute Nodes
- Storage
- Swift
- Prometheus

using either:

- `/etc/hosts`
- Internal DNS Server

Consistent hostname resolution improves maintainability and reduces configuration errors.

---

# Security Considerations

Although this is a laboratory deployment, several security practices are followed:

- Unique service credentials
- Principle of least privilege
- Network segmentation
- Security Groups
- Dedicated management interfaces
- SSH key authentication
- Firewall configuration
- Separate service accounts

These practices closely resemble production deployment recommendations from the OpenStack community.

---

# Environment Summary

The laboratory provides a realistic private cloud infrastructure capable of demonstrating the complete lifecycle of OpenStack resources.

By separating management, compute, networking, storage, monitoring, and orchestration services into dedicated virtual machines, the platform offers practical experience that closely resembles enterprise cloud deployments while remaining suitable for experimentation and learning.

The environment described in this document serves as the foundation for the deployment procedures presented in the following chapters.

# VMware Virtual Network Design

The OpenStack laboratory is deployed on VMware Workstation using multiple virtual network adapters to simulate a production environment.

Instead of connecting every virtual machine to a single virtual switch, different VMnet segments are used to isolate management traffic, external access, tenant communication, and storage traffic.

A typical VMware network layout is illustrated below.

```text
                    VMware Workstation

                     Virtual Network Editor

┌────────────────────────────────────────────────────┐

 VMnet0   Bridged Network
    │
    └────── Internet Access

 VMnet1   Host-Only
    │
    └────── Management Network

 VMnet2   Host-Only
    │
    └────── Provider Network

 VMnet3   Host-Only
    │
    └────── VXLAN Tunnel Network

 VMnet4   Host-Only
    │
    └────── Storage Network

└────────────────────────────────────────────────────┘
```

Using dedicated virtual networks significantly simplifies troubleshooting and better reflects the physical separation commonly found in enterprise deployments.

---

# Virtual Machine Inventory

The following table summarizes every virtual machine participating in the laboratory.

| Node       | Role           | Main Services                              |
| ---------- | -------------- | ------------------------------------------ |
| controller | Control Plane  | Keystone, Nova API, Neutron, Horizon, Heat |
| compute01  | Compute        | nova-compute                               |
| compute02  | Compute        | nova-compute                               |
| compute03  | Compute        | nova-compute                               |
| storage    | Block Storage  | Cinder                                     |
| swift      | Object Storage | Swift                                      |
| monitor    | Monitoring     | Prometheus                                 |

Each node has a clearly defined responsibility and minimizes unnecessary service overlap.

---

# Network Interface Assignment

Each node is connected to one or more virtual networks.

Example design:

| Node       | Management | Provider | Tunnel | Storage |
| ---------- | ---------- | -------- | ------ | ------- |
| Controller | ens33      | ens34    | ens35  | ens36   |
| Compute01  | ens33      | ens34    | ens35  | -       |
| Compute02  | ens33      | ens34    | ens35  | -       |
| Compute03  | ens33      | ens34    | ens35  | -       |
| Storage    | ens33      | -        | -      | ens34   |
| Swift      | ens33      | -        | -      | ens34   |
| Prometheus | ens33      | -        | -      | -       |

Not every node requires access to every network segment. Services should only connect to the networks necessary for their operation, following the principle of least privilege.

---

# IP Address Plan

A consistent addressing scheme improves deployment, troubleshooting, and future expansion.

Example addressing plan:

| Device     | Management IP |
| ---------- | ------------- |
| Controller | 192.168.10.10 |
| Compute01  | 192.168.10.21 |
| Compute02  | 192.168.10.22 |
| Compute03  | 192.168.10.23 |
| Storage    | 192.168.10.30 |
| Swift      | 192.168.10.40 |
| Prometheus | 192.168.10.50 |

Example network allocation:

| Network    | CIDR            | Purpose             |
| ---------- | --------------- | ------------------- |
| Management | 192.168.10.0/24 | OpenStack Services  |
| Provider   | 192.168.20.0/24 | External Access     |
| Tunnel     | 10.10.10.0/24   | VXLAN Encapsulation |
| Storage    | 172.16.10.0/24  | Storage Traffic     |

The exact IP ranges may vary depending on the available infrastructure.

---

# Hostname Convention

Consistent naming simplifies administration.

Recommended hostnames:

```text
controller

compute01

compute02

compute03

storage

swift

monitor
```

Service configuration files should reference hostnames rather than changing IP addresses whenever possible.

---

# Resource Sizing

Proper sizing is critical for OpenStack.

### Minimum Laboratory

| Component  | vCPU | RAM  |
| ---------- | ---- | ---- |
| Controller | 2    | 4 GB |
| Compute    | 2    | 4 GB |
| Storage    | 2    | 2 GB |

Suitable for basic functional testing.

---

### Recommended Laboratory

| Component  | vCPU | RAM  |
| ---------- | ---- | ---- |
| Controller | 4    | 8 GB |
| Compute    | 4    | 8 GB |
| Storage    | 2    | 4 GB |
| Swift      | 2    | 4 GB |
| Monitoring | 2    | 2 GB |

Recommended for this project.

---

### Production

Production environments typically require:

- Redundant controllers
- High Availability
- Dedicated storage clusters
- Hardware RAID
- SSD storage
- 10GbE networking
- Redundant power
- Enterprise monitoring

This laboratory intentionally focuses on architectural concepts rather than production-scale capacity.

---

# BIOS Requirements

Before creating virtual machines, hardware virtualization must be enabled.

Verify that:

- Intel VT-x or AMD-V is enabled.
- Nested virtualization is available if required.
- Virtualization extensions are exposed to guest operating systems.

Failure to enable hardware virtualization may prevent compute services from launching instances.

---

# Storage Planning

The environment separates operating system disks from OpenStack storage.

Example:

```text
Controller

80 GB
└── Operating System

Storage Node

40 GB
└── Operating System

120 GB
└── Cinder Volume Group

Swift Node

40 GB
└── Operating System

120 GB
└── Swift Object Storage
```

Separating service data from the operating system improves maintainability and allows storage backends to scale independently.

---

# Time Synchronization Strategy

OpenStack services exchange authentication tokens with strict timestamp validation.

Chrony is configured to synchronize every node with the Controller.

```text
Controller

↓

Compute Nodes

↓

Storage

↓

Swift

↓

Prometheus
```

Clock drift can lead to Keystone authentication failures, expired tokens, and service communication issues.

---

# Name Resolution

Reliable hostname resolution is essential.

Each node should resolve all infrastructure hostnames either through:

- `/etc/hosts`
- Internal DNS server

Example:

```text
192.168.10.10 controller

192.168.10.21 compute01

192.168.10.22 compute02

192.168.10.23 compute03

192.168.10.30 storage

192.168.10.40 swift

192.168.10.50 monitor
```

Consistent hostname resolution reduces dependency on static IP references within configuration files.

---

# Deployment Preparation Checklist

Before installing OpenStack services, verify the following:

- VMware virtual machines created
- Ubuntu Server installed
- Static IP addresses configured
- Hostnames assigned
- DNS resolution verified
- Internet connectivity confirmed
- Chrony synchronized
- SSH access available
- Required repositories configured
- Firewall rules reviewed
- Package updates completed

Completing this checklist significantly reduces installation issues during later deployment stages.

---

# Best Practices

The following practices are recommended throughout the project:

- Assign a single responsibility to each node.
- Avoid mixing management and tenant traffic.
- Document all IP allocations.
- Use descriptive hostnames.
- Create VMware snapshots before major configuration changes.
- Keep all nodes on the same Ubuntu LTS release.
- Synchronize time before installing OpenStack.
- Validate connectivity after configuring each network.
- Record every configuration change in version control.

Following these practices results in a more maintainable and reproducible deployment process.

# Infrastructure Topology

This section provides a detailed view of the infrastructure layout used throughout the laboratory.

Unlike logical architecture diagrams, the following topology focuses on the relationship between virtual machines, network interfaces, and infrastructure services.

```text
                                 Physical Host
                     Windows 11 + VMware Workstation
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  VMnet0 (Bridged)      → Internet / External Access                        │
│  VMnet1 (Host-Only)    → Management Network                               │
│  VMnet2 (Host-Only)    → Provider Network                                 │
│  VMnet3 (Host-Only)    → VXLAN / Tunnel Network                           │
│  VMnet4 (Host-Only)    → Storage Network                                  │
│                                                                            │
│ ┌──────────────────────────────────────────────────────────────────────┐   │
│ │ Controller Node                                                     │   │
│ │---------------------------------------------------------------------│   │
│ │ ens33 → Management Network                                          │   │
│ │ ens34 → Provider Network                                            │   │
│ │ ens35 → Tunnel Network                                              │   │
│ │ ens36 → Storage Network                                             │   │
│ └──────────────────────────────────────────────────────────────────────┘   │
│                                                                            │
│ ┌──────────────────────────────────────────────────────────────────────┐   │
│ │ Compute01                                                           │   │
│ │ ens33 → Management                                                  │   │
│ │ ens34 → Provider                                                    │   │
│ │ ens35 → Tunnel                                                      │   │
│ └──────────────────────────────────────────────────────────────────────┘   │
│                                                                            │
│ ┌──────────────────────────────────────────────────────────────────────┐   │
│ │ Compute02                                                           │   │
│ │ ens33 → Management                                                  │   │
│ │ ens34 → Provider                                                    │   │
│ │ ens35 → Tunnel                                                      │   │
│ └──────────────────────────────────────────────────────────────────────┘   │
│                                                                            │
│ ┌──────────────────────────────────────────────────────────────────────┐   │
│ │ Compute03                                                           │   │
│ │ ens33 → Management                                                  │   │
│ │ ens34 → Provider                                                    │   │
│ │ ens35 → Tunnel                                                      │   │
│ └──────────────────────────────────────────────────────────────────────┘   │
│                                                                            │
│ ┌──────────────────────────────────────────────────────────────────────┐   │
│ │ Storage Node                                                        │   │
│ │ ens33 → Management                                                  │   │
│ │ ens34 → Storage                                                     │   │
│ └──────────────────────────────────────────────────────────────────────┘   │
│                                                                            │
│ ┌──────────────────────────────────────────────────────────────────────┐   │
│ │ Swift Node                                                          │   │
│ │ ens33 → Management                                                  │   │
│ │ ens34 → Storage                                                     │   │
│ └──────────────────────────────────────────────────────────────────────┘   │
│                                                                            │
│ ┌──────────────────────────────────────────────────────────────────────┐   │
│ │ Prometheus                                                          │   │
│ │ ens33 → Management                                                  │   │
│ └──────────────────────────────────────────────────────────────────────┘   │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

---

# Infrastructure Communication Matrix

The following matrix summarizes how OpenStack components communicate with each other.

| Source         | Destination   | Purpose               |
| -------------- | ------------- | --------------------- |
| Horizon        | Keystone      | User Authentication   |
| Keystone       | MariaDB       | Identity Database     |
| Nova API       | Keystone      | Token Validation      |
| Nova API       | RabbitMQ      | RPC Messaging         |
| Nova Scheduler | Placement     | Resource Selection    |
| Nova Compute   | RabbitMQ      | Instance Management   |
| Nova Compute   | Glance        | Image Download        |
| Nova Compute   | Neutron       | Network Allocation    |
| Nova Compute   | Cinder        | Volume Attachment     |
| Neutron        | RabbitMQ      | Network Events        |
| Heat           | Nova          | Instance Provisioning |
| Heat           | Neutron       | Network Creation      |
| Heat           | Cinder        | Volume Creation       |
| Prometheus     | Node Exporter | Metrics Collection    |

Understanding these communication paths greatly simplifies troubleshooting because service failures often originate from dependency failures rather than application errors.

---

# Service Dependency Graph

OpenStack components must be deployed in a specific order.

The following dependency graph illustrates the relationship between infrastructure services.

```text
Ubuntu Server

↓

Network Configuration

↓

Chrony

↓

MariaDB

↓

RabbitMQ

↓

Memcached

↓

Etcd

↓

Keystone

↓

Glance

↓

Placement

↓

Nova

↓

Neutron

↓

Horizon

↓

Cinder

↓

Swift

↓

Heat

↓

Prometheus
```

Each service relies on one or more previously configured components.

Attempting to deploy services out of sequence typically results in authentication failures, database errors, or unavailable API endpoints.

---

# OpenStack API Endpoints

The Controller Node exposes multiple REST APIs used by clients and internal services.

| Service    | Default Port | Protocol     |
| ---------- | -----------: | ------------ |
| Keystone   |         5000 | HTTP/HTTPS   |
| Glance     |         9292 | HTTP         |
| Nova       |         8774 | HTTP         |
| Placement  |         8778 | HTTP         |
| Neutron    |         9696 | HTTP         |
| Cinder     |         8776 | HTTP         |
| Heat API   |         8004 | HTTP         |
| Heat CFN   |         8000 | HTTP         |
| Horizon    |     80 / 443 | HTTP / HTTPS |
| Prometheus |         9090 | HTTP         |

When troubleshooting service availability, these endpoints should be verified before investigating higher-level issues.

---

# Firewall Considerations

During deployment, communication between nodes must not be unintentionally blocked.

At a minimum, the following categories of traffic should be permitted:

- Management API traffic
- Database communication
- Message queue traffic
- VXLAN encapsulation
- SSH administration
- ICMP for diagnostics

Production deployments should replace permissive firewall rules with carefully defined allowlists.

---

# Installation Order Rationale

The installation order is not arbitrary.

Each component depends on infrastructure that has already been deployed.

### MariaDB

Every OpenStack service stores configuration data in a database.

Without MariaDB, services cannot initialize their schemas.

---

### RabbitMQ

RabbitMQ acts as the messaging backbone for asynchronous communication.

Nova, Neutron, Cinder, and other services exchange RPC messages through RabbitMQ.

---

### Keystone

Authentication is required before any service can register API endpoints.

Consequently, Keystone must be available before Glance, Nova, Neutron, Cinder, Swift, or Heat.

---

### Glance

Compute nodes require Glance to download operating system images before launching virtual machines.

---

### Placement

Nova Scheduler depends on Placement to determine which compute node has sufficient resources.

---

### Nova

Only after authentication, messaging, image services, and placement are available can compute services successfully launch instances.

---

### Neutron

Networking services provide virtual networks, routers, DHCP, metadata services, and floating IPs required by virtual machines.

---

### Horizon

The dashboard is installed after core services become operational so that it can immediately communicate with all API endpoints.

---

### Cinder

Persistent block storage depends on authentication, database connectivity, and messaging infrastructure.

---

### Swift

Object storage integrates with Keystone authentication while maintaining independent storage services.

---

### Heat

Heat is deployed near the end because it orchestrates resources managed by nearly every other OpenStack component.

---

# Failure Scenarios

Understanding failure behavior is essential for infrastructure engineers.

## MariaDB Failure

Possible Symptoms

- Keystone authentication fails
- APIs return HTTP 500
- Database migrations fail

Troubleshooting

- Verify database service status
- Confirm database connectivity
- Check service credentials

---

## RabbitMQ Failure

Possible Symptoms

- Instances remain in BUILD state
- Nova Compute becomes unavailable
- RPC timeout errors

Troubleshooting

- Verify RabbitMQ service
- Inspect message queues
- Check AMQP connectivity

---

## Keystone Failure

Possible Symptoms

- Login fails
- Token validation errors
- Horizon unavailable

Troubleshooting

- Verify Keystone API
- Check Fernet keys
- Confirm endpoint registration

---

## Neutron Failure

Possible Symptoms

- DHCP unavailable
- Floating IP inaccessible
- Metadata service unavailable
- Virtual machines lose connectivity

Troubleshooting

- Verify Neutron agents
- Inspect bridge mappings
- Confirm router namespaces

---

## Cinder Failure

Possible Symptoms

- Volume creation fails
- Volume attachment timeout
- Backup unavailable

Troubleshooting

- Verify LVM backend
- Inspect volume groups
- Check cinder-volume service

---

## Time Synchronization Failure

Possible Symptoms

- Expired tokens
- Authentication rejected
- Service registration failures

Troubleshooting

- Verify Chrony synchronization
- Compare timestamps across nodes
- Restart affected services after synchronization

---

# Operational Best Practices

The following practices significantly improve deployment reliability.

- Deploy services incrementally and validate each stage.
- Create VMware snapshots before major configuration changes.
- Maintain consistent package versions across all nodes.
- Record every configuration change in version control.
- Monitor service health continuously.
- Separate infrastructure networks whenever possible.
- Avoid exposing management interfaces directly to external networks.
- Document firewall rules and network topology.
- Test backup and recovery procedures regularly.
- Perform periodic updates while preserving service compatibility.

---

# Conclusion

A successful OpenStack deployment depends not only on installing software but also on careful infrastructure planning.

By defining clear node responsibilities, separating network traffic, understanding service dependencies, and following recommended deployment practices, administrators can build a stable and maintainable private cloud platform suitable for experimentation, education, and future expansion.
