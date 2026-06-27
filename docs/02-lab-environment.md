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
