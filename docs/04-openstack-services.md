# OpenStack Services

# Overview

OpenStack is designed as a collection of loosely coupled services rather than a single monolithic application.

Each service performs a specific responsibility and communicates with other components through REST APIs, message queues, databases, and authentication services.

This modular architecture allows organizations to scale individual services independently while maintaining a unified cloud platform.

The laboratory environment implements the core OpenStack services required to provide Infrastructure-as-a-Service (IaaS) capabilities.

---

# Service Architecture

The following diagram illustrates the relationship between major OpenStack services.

```text
                           Horizon
                               │
                               ▼
                         Keystone API
                               │
      ┌─────────────┬──────────┴──────────────┬───────────────┐
      ▼             ▼                         ▼               ▼
   Glance         Nova                    Neutron         Cinder
      │             │                         │               │
      │             ▼                         ▼               ▼
      │      Compute Nodes              Virtual Network    Volumes
      │             │
      ▼             ▼
 Virtual Machine Images
```

All OpenStack services authenticate through Keystone before accessing protected resources.

---

# Keystone (Identity Service)

## Purpose

Keystone is the identity and authentication service of OpenStack.

Every request to the cloud platform is authenticated and authorized by Keystone before reaching other services.

Without Keystone, users, administrators, and services cannot access cloud resources.

---

## Responsibilities

Keystone provides:

- Authentication
- Authorization
- Identity Management
- Token Generation
- Service Catalog
- API Endpoint Registration

---

## Authentication Workflow

```text
User

↓

Horizon

↓

Keystone

↓

Authentication Token

↓

Nova / Neutron / Cinder / Glance
```

Once authenticated, the issued token is presented to all OpenStack APIs until it expires.

---

## Main Components

- Identity
- User
- Project (Tenant)
- Role
- Domain
- Token
- Service Catalog

These objects form the foundation of OpenStack's Role-Based Access Control (RBAC) model.

---

# Glance (Image Service)

## Purpose

Glance stores and manages virtual machine images.

Before a virtual machine can be launched, Nova retrieves the selected operating system image from Glance.

---

## Responsibilities

- Image Repository
- Image Metadata
- Version Control
- Image Distribution
- Image Validation

---

## Supported Image Formats

Examples include:

- qcow2
- raw
- vhd
- vmdk
- iso

QCOW2 is commonly used due to its support for compression and copy-on-write functionality.

---

## Image Boot Workflow

```text
User

↓

Nova

↓

Glance

↓

Image Download

↓

Compute Node

↓

Instance Boot
```

---

# Nova (Compute Service)

## Purpose

Nova manages the lifecycle of virtual machine instances.

It coordinates scheduling, provisioning, resizing, migration, and deletion of compute resources.

---

## Main Components

### Nova API

Receives user requests from Horizon or the OpenStack CLI.

---

### Nova Scheduler

Selects the most appropriate Compute Node based on available resources.

---

### Nova Conductor

Coordinates communication between the API layer and compute services.

---

### Nova Compute

Runs directly on Compute Nodes.

Responsibilities include:

- Creating instances
- Managing hypervisor resources
- Reporting resource usage
- Instance lifecycle management

---

## Instance Creation Workflow

```text
User

↓

Nova API

↓

Scheduler

↓

Placement

↓

RabbitMQ

↓

Nova Compute

↓

Virtual Machine
```

---

# Placement Service

## Purpose

Placement tracks available compute resources.

Instead of relying solely on Nova, modern OpenStack deployments use Placement to determine where workloads should run.

---

## Managed Resources

- CPU
- RAM
- Disk
- Custom Resource Classes

Placement enables efficient scheduling and resource accounting across multiple compute nodes.

---

# Neutron (Networking Service)

## Purpose

Neutron provides Software Defined Networking (SDN) capabilities.

It allows tenants to create isolated virtual networks while administrators maintain centralized network control.

---

## Responsibilities

- Virtual Networks
- Routers
- DHCP
- Security Groups
- Floating IPs
- Metadata Services
- Network Isolation

---

## Major Components

- Neutron Server
- L3 Agent
- DHCP Agent
- Metadata Agent
- Linux Bridge Agent

---

## Packet Flow

```text
VM

↓

Linux Bridge

↓

VXLAN Tunnel

↓

Router

↓

Floating IP

↓

Internet
```

---

# Horizon (Dashboard)

## Purpose

Horizon provides a web-based graphical interface for managing OpenStack resources.

Rather than interacting with APIs directly, administrators and users can perform cloud operations through the dashboard.

---

## Features

- Instance Management
- Network Management
- Storage Management
- User Administration
- Security Groups
- Images
- Floating IPs

Horizon communicates exclusively with OpenStack APIs and does not manage infrastructure directly.

---

# Cinder (Block Storage)

## Purpose

Cinder provides persistent block storage for virtual machines.

Unlike ephemeral instance disks, Cinder volumes remain available after an instance is stopped or deleted.

---

## Features

- Volume Creation
- Volume Attachment
- Snapshots
- Backup
- Restore

---

## Backend

This laboratory uses LVM as the storage backend.

Production environments often replace LVM with distributed storage solutions such as Ceph.

---

## Volume Attachment Workflow

```text
User

↓

Cinder API

↓

Volume Created

↓

Nova

↓

Compute Node

↓

Virtual Machine
```

---

# Swift (Object Storage)

## Purpose

Swift stores unstructured data as objects.

Unlike block storage, object storage is optimized for scalability and durability rather than direct attachment to virtual machines.

---

## Typical Use Cases

- Image archives
- Application assets
- Backup files
- Static websites
- Log storage

---

## Components

- Proxy Service
- Account Service
- Container Service
- Object Service

Objects are distributed across storage devices to improve resilience.

---

# Heat (Orchestration)

## Purpose

Heat automates infrastructure deployment using templates.

Instead of manually creating resources, administrators define cloud infrastructure as code.

---

## Managed Resources

- Instances
- Networks
- Routers
- Volumes
- Security Groups
- Floating IPs

---

## Workflow

```text
Template

↓

Heat

↓

Nova

↓

Neutron

↓

Cinder

↓

Running Infrastructure
```

---

# Supporting Infrastructure Services

Several additional services support the OpenStack control plane.

## MariaDB

Stores persistent data for all OpenStack services.

---

## RabbitMQ

Provides asynchronous messaging between distributed services.

---

## Memcached

Caches authentication tokens to reduce Keystone load.

---

## Etcd

Stores distributed coordination data for services such as Neutron.

---

## Chrony

Maintains time synchronization across all nodes.

---

# Service Dependency Overview

The following dependency chain summarizes the relationship between services.

```text
Chrony

↓

MariaDB

↓

RabbitMQ

↓

Memcached

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

Cinder

↓

Swift

↓

Heat

↓

Horizon
```

Deploying services in this order minimizes dependency-related failures.

---

# Summary

OpenStack achieves flexibility and scalability by separating cloud functionality into specialized services.

Each component focuses on a single responsibility while collaborating through well-defined APIs, authentication mechanisms, and messaging systems.

Understanding how Keystone, Glance, Nova, Placement, Neutron, Horizon, Cinder, Swift, and Heat interact is essential for designing, deploying, troubleshooting, and operating a production-ready private cloud environment.

# Service Interaction Workflow

OpenStack services rarely operate independently.

A single user request may involve multiple services communicating through REST APIs, asynchronous message queues, authentication tokens, and shared databases.

Understanding these interactions is essential for troubleshooting distributed cloud environments.

---

# Virtual Machine Provisioning Workflow

The following sequence illustrates the complete lifecycle of launching a virtual machine.

```text
User

↓

Horizon Dashboard

↓

Keystone Authentication

↓

Nova API

↓

Placement API

↓

Nova Scheduler

↓

RabbitMQ

↓

Nova Compute

↓

Glance

↓

Neutron

↓

Cinder (Optional)

↓

Hypervisor

↓

Running Instance
```

Every component participates in a specific stage of the provisioning process.

---

# Step 1 — User Authentication

The user logs in through the Horizon Dashboard.

Horizon forwards the authentication request to Keystone.

Keystone validates:

- Username
- Password
- Domain
- Project
- Assigned Roles

If authentication succeeds, Keystone returns a scoped authentication token.

This token is attached to every subsequent API request.

---

# Step 2 — Instance Request

The user selects:

- Image
- Flavor
- Network
- Security Group
- Key Pair

Horizon sends these parameters to the Nova API.

At this stage, Nova has not yet selected a compute node.

---

# Step 3 — Resource Validation

Nova forwards the scheduling request to the Placement service.

Placement evaluates available resources across all registered compute nodes.

Typical resource checks include:

- Available vCPUs
- Available RAM
- Local Disk Capacity
- Resource Allocation Ratios
- Custom Resource Classes

Placement returns a list of candidate compute nodes.

---

# Step 4 — Scheduling

Nova Scheduler selects the most appropriate compute node.

Scheduling decisions may consider:

- Resource availability
- Affinity policies
- Anti-affinity rules
- Availability Zones
- Host aggregates

The selected node receives the provisioning request through RabbitMQ.

---

# Step 5 — Image Retrieval

Nova Compute downloads the requested operating system image from Glance.

The image is cached locally when possible to reduce deployment time.

Supported formats include:

- QCOW2
- RAW
- VHD
- VMDK

Caching frequently used images significantly improves provisioning performance.

---

# Step 6 — Network Allocation

Nova requests networking resources from Neutron.

Neutron performs several tasks:

- Create virtual interfaces
- Allocate MAC addresses
- Allocate IP addresses
- Connect the instance to the tenant network
- Apply Security Groups
- Configure DHCP
- Configure Metadata access

At this point, the instance becomes logically connected to the virtual network.

---

# Step 7 — Volume Attachment (Optional)

If the selected flavor includes persistent block storage, Nova contacts Cinder.

Cinder creates or attaches an existing volume.

The volume is exported to the compute node and attached to the virtual machine.

Persistent volumes remain available independently of the virtual machine lifecycle.

---

# Step 8 — Instance Boot

The hypervisor creates the virtual machine.

Cloud-init retrieves configuration data from the Metadata Service.

Typical initialization tasks include:

- Configure hostname
- Inject SSH public keys
- Configure users
- Execute initialization scripts
- Configure networking

The instance transitions from BUILD to ACTIVE after successful initialization.

---

# REST API Communication Model

OpenStack services communicate primarily through RESTful APIs.

Example interactions:

| Source  | Destination | Purpose              |
| ------- | ----------- | -------------------- |
| Horizon | Keystone    | Authentication       |
| Nova    | Placement   | Resource Discovery   |
| Nova    | Glance      | Image Retrieval      |
| Nova    | Neutron     | Networking           |
| Nova    | Cinder      | Volume Attachment    |
| Heat    | Nova        | Instance Creation    |
| Heat    | Neutron     | Network Provisioning |

REST APIs provide loose coupling between services, enabling independent scaling and maintenance.

---

# RabbitMQ Communication

Not all communication occurs synchronously.

Many internal operations rely on asynchronous messaging through RabbitMQ.

Example workflow:

```text
Nova API

↓

RabbitMQ

↓

Nova Compute

↓

Response Queue

↓

Nova API
```

Advantages include:

- Loose coupling
- Improved scalability
- Retry mechanisms
- Distributed execution

Without RabbitMQ, compute nodes cannot receive provisioning requests.

---

# Database Responsibilities

Each major OpenStack service maintains its own database schema.

| Service  | Database Purpose     |
| -------- | -------------------- |
| Keystone | Identity Information |
| Glance   | Image Metadata       |
| Nova     | Instance Metadata    |
| Neutron  | Networking Resources |
| Cinder   | Volume Metadata      |
| Heat     | Stack Definitions    |

Although databases are separate, services cooperate through authenticated APIs rather than direct database access.

---

# Service Registration

Every OpenStack service registers itself with Keystone.

Registration includes:

- Service Name
- Service Type
- Public Endpoint
- Internal Endpoint
- Administrative Endpoint

This Service Catalog allows clients to dynamically discover API locations.

---

# Instance State Machine

Virtual machines transition through several operational states.

```text
BUILD

↓

ACTIVE

↓

STOPPED

↓

STARTING

↓

ACTIVE

↓

REBOOT

↓

ACTIVE

↓

SHELVED

↓

ACTIVE

↓

DELETED
```

Possible error states include:

- ERROR
- VERIFY_RESIZE
- RESCUED
- PAUSED
- SUSPENDED

Administrators should understand these states when diagnosing provisioning failures.

---

# Authentication Flow

Every service validates authentication tokens before processing requests.

```text
User

↓

Keystone

↓

Token

↓

Nova

↓

Token Validation

↓

Authorized Request
```

Expired or invalid tokens immediately terminate request processing.

---

# High-Level Service Dependencies

```text
Chrony
   │
MariaDB
   │
RabbitMQ
   │
Memcached
   │
Keystone
   │
Glance
   │
Placement
   │
Nova
   │
Neutron
   │
Cinder
   │
Swift
   │
Heat
   │
Horizon
```

A failure in a lower-layer service typically propagates upward through the dependency chain.

---

# Failure Impact Matrix

| Failed Service | Observable Symptoms                         |
| -------------- | ------------------------------------------- |
| Chrony         | Authentication token errors                 |
| MariaDB        | API failures, database connection errors    |
| RabbitMQ       | Instances remain in BUILD state             |
| Keystone       | Login failure, API authorization errors     |
| Glance         | Instances cannot boot from images           |
| Placement      | Scheduler cannot allocate compute resources |
| Nova           | Instance lifecycle operations fail          |
| Neutron        | Network connectivity unavailable            |
| Cinder         | Volume creation and attachment fail         |
| Swift          | Object storage unavailable                  |
| Heat           | Stack deployments fail                      |
| Horizon        | Web dashboard unavailable                   |

This matrix helps administrators quickly identify probable root causes based on observed symptoms.

---

# Troubleshooting Strategy

When troubleshooting OpenStack, avoid inspecting services randomly.

A recommended workflow is:

1. Verify network connectivity between nodes.
2. Confirm time synchronization.
3. Check database availability.
4. Verify RabbitMQ messaging.
5. Validate Keystone authentication.
6. Confirm service registration.
7. Inspect service logs.
8. Verify agent health.
9. Test API endpoints.
10. Retry the operation.

Following a structured approach significantly reduces diagnosis time in distributed environments.

---

# Operational Best Practices

To improve reliability and maintainability:

- Monitor all API endpoints.
- Keep service versions consistent across nodes.
- Synchronize system clocks continuously.
- Backup databases regularly.
- Monitor RabbitMQ queue health.
- Use HTTPS for production API endpoints.
- Enable centralized logging.
- Test failover scenarios periodically.
- Document every configuration change.
- Validate service health after updates.

---

# Conclusion

OpenStack's strength lies in its modular architecture.

Each service performs a specialized function while collaborating through authentication, messaging, and REST APIs.

A clear understanding of these interactions enables administrators to deploy, operate, troubleshoot, and scale complex cloud infrastructures with confidence.

Rather than viewing OpenStack as a collection of independent services, it should be understood as an integrated distributed system where reliability depends on the health and coordination of every component.
