# OpenStack Deployment Guide

# Overview

This document describes the deployment procedure used to build the OpenStack Private Cloud laboratory.

The deployment follows the architecture presented in the previous chapters and is based on a multi-node environment running Ubuntu Server 22.04 LTS.

Rather than focusing solely on installation commands, this guide explains the rationale behind each deployment stage, the dependencies between services, and the validation steps required to ensure a stable cloud platform.

---

# Deployment Strategy

The deployment follows a layered approach.

Each layer provides the foundation for the next one.

```text
Infrastructure

↓

Operating System

↓

Network Configuration

↓

Time Synchronization

↓

Database

↓

Message Queue

↓

Identity Service

↓

Image Service

↓

Placement

↓

Compute

↓

Networking

↓

Dashboard

↓

Storage

↓

Orchestration

↓

Monitoring
```

Deploying services out of sequence often results in authentication failures, missing API endpoints, or communication errors.

---

# Phase 1 – Infrastructure Preparation

Before installing OpenStack services, prepare the virtual infrastructure.

Tasks include:

- Create all virtual machines.
- Allocate CPU, memory, and storage resources.
- Configure VMware virtual networks.
- Enable hardware virtualization.
- Verify Internet connectivity.
- Install Ubuntu Server 22.04 LTS.
- Update all packages.
- Configure static IP addresses.

At this stage, every node should be reachable through SSH.

---

# Phase 2 – Operating System Configuration

Every node should follow a consistent operating system configuration.

Recommended tasks include:

- Set the hostname.
- Configure `/etc/hosts`.
- Create an administrative user.
- Configure SSH key authentication.
- Disable unnecessary services.
- Configure firewall rules.
- Update package repositories.
- Install required utilities.

Example utilities:

- curl
- wget
- vim
- git
- net-tools
- bridge-utils
- chrony

Consistency across nodes greatly simplifies administration.

---

# Phase 3 – Network Configuration

Each network interface should be configured according to its designated purpose.

Example:

| Interface | Purpose    |
| --------- | ---------- |
| ens33     | Management |
| ens34     | Provider   |
| ens35     | VXLAN      |
| ens36     | Storage    |

Typical tasks include:

- Assign static IP addresses.
- Configure gateways.
- Configure DNS.
- Verify routing tables.
- Test connectivity between nodes.

Connectivity should be validated before continuing with service installation.

---

# Phase 4 – Time Synchronization

OpenStack relies heavily on accurate timestamps.

Chrony is configured on every node.

Deployment steps:

1. Configure the Controller as the primary NTP server.
2. Configure all remaining nodes as Chrony clients.
3. Synchronize clocks.
4. Verify synchronization status.

Example verification:

```bash
chronyc sources
chronyc tracking
```

Authentication problems are frequently caused by unsynchronized clocks.

---

# Phase 5 – Install Supporting Infrastructure

Before deploying OpenStack services, several supporting components must be installed.

## MariaDB

Responsibilities:

- Store persistent service data.
- Maintain service databases.
- Support schema migrations.

After installation:

- Secure the database.
- Create service databases.
- Create service users.
- Assign privileges.

---

## RabbitMQ

RabbitMQ provides asynchronous communication between distributed services.

Typical tasks:

- Install RabbitMQ.
- Create service accounts.
- Assign permissions.
- Verify messaging functionality.

---

## Memcached

Memcached improves Keystone performance by caching authentication tokens.

Deployment includes:

- Install package.
- Configure listening interface.
- Enable service.
- Verify connectivity.

---

## Etcd

Etcd provides distributed key-value storage.

Deployment tasks:

- Configure cluster membership.
- Configure peer communication.
- Verify cluster health.

Etcd is required by several networking components.

---

# Phase 6 – Keystone Deployment

Keystone should always be deployed before other OpenStack services.

Deployment steps:

1. Install Keystone packages.
2. Configure database connection.
3. Configure token provider.
4. Synchronize the database.
5. Bootstrap Keystone.
6. Create service endpoints.
7. Create projects.
8. Create users.
9. Create roles.

Verification:

```bash
openstack token issue
```

Successful token generation confirms that authentication is functioning correctly.

---

# Phase 7 – Glance Deployment

After Keystone becomes operational, deploy the Image Service.

Typical procedure:

1. Create the Glance database.
2. Create the Glance user.
3. Register the Image Service.
4. Configure API endpoints.
5. Install Glance packages.
6. Configure `glance-api.conf`.
7. Synchronize the database.
8. Start the service.

Validation:

```bash
openstack image list
```

A successful response indicates that the Image Service is operational.

---

# Phase 8 – Placement Deployment

Placement manages resource inventories for compute scheduling.

Deployment tasks:

- Create the Placement database.
- Register the Placement service.
- Configure API endpoints.
- Install packages.
- Synchronize the database.
- Verify API availability.

Validation:

```bash
placement-status upgrade check
```

Placement must be fully operational before deploying Nova.

---

# Phase 9 – Nova Deployment

Nova consists of several components deployed across different nodes.

Controller:

- Nova API
- Scheduler
- Conductor

Compute Nodes:

- nova-compute

Deployment procedure:

1. Configure database access.
2. Configure RabbitMQ.
3. Configure Keystone authentication.
4. Configure Placement.
5. Configure Glance.
6. Synchronize the database.
7. Start Nova services.
8. Register Compute Nodes.

Verification:

```bash
openstack compute service list
```

Each compute node should report an **up** status.

---

# Deployment Validation

At the end of each phase, validation is mandatory.

Administrators should never continue to the next phase without confirming that the previous phase completed successfully.

Typical validation includes:

- Service status
- API endpoint availability
- Database connectivity
- Authentication
- Message queue communication
- Log inspection

## Incremental validation dramatically reduces troubleshooting complexity later in the deployment process.

# Phase 10 – Neutron Deployment

## Overview

Neutron provides the networking foundation of the OpenStack cloud.

Unlike traditional virtualization platforms where networking is configured directly on the hypervisor, OpenStack delegates all networking operations to Neutron.

Before virtual machines can communicate, Neutron must successfully configure:

- Virtual Networks
- Subnets
- Routers
- DHCP
- Metadata
- Security Groups
- Floating IPs
- Overlay Networks

Because networking affects every workload, Neutron is one of the most complex OpenStack services.

---

## Components

The deployment consists of several cooperating services.

Controller Node

- neutron-server
- ML2 Plugin
- L3 Agent
- DHCP Agent
- Metadata Agent

Compute Nodes

- Linux Bridge Agent
- VXLAN Agent

---

## Deployment Workflow

```text
Install Packages

↓

Configure Database

↓

Configure RabbitMQ

↓

Configure Keystone

↓

Configure ML2 Plugin

↓

Configure Linux Bridge

↓

Configure VXLAN

↓

Configure DHCP

↓

Configure Metadata

↓

Restart Services

↓

Validate Agents
```

Each stage should be validated before continuing.

---

## ML2 Plugin Configuration

The Modular Layer 2 (ML2) plugin determines how tenant networks are implemented.

Typical configuration includes:

- Type Drivers
- Tenant Network Types
- Mechanism Drivers
- Extension Drivers

Example network types:

- flat
- vlan
- vxlan

Mechanism driver:

- linuxbridge

The laboratory uses VXLAN as the tenant network technology.

---

## Linux Bridge Configuration

Each compute node requires bridge mappings between physical interfaces and virtual bridges.

Responsibilities include:

- Interface Bridging
- MAC Learning
- Frame Forwarding
- VXLAN Integration

Incorrect bridge mappings are among the most common causes of connectivity issues.

---

## VXLAN Configuration

Overlay networking requires every compute node to communicate through the tunnel network.

Example workflow

```text
VM

↓

Linux Bridge

↓

VXLAN Interface

↓

Physical Network

↓

Remote Compute

↓

Linux Bridge

↓

VM
```

VXLAN identifiers uniquely separate tenant traffic.

---

## Agent Registration

After services start, verify that every agent successfully registers.

Example

```bash
openstack network agent list
```

Expected agents include:

- DHCP Agent
- L3 Agent
- Metadata Agent
- Linux Bridge Agent

All agents should report **Alive = True**.

---

# Phase 11 – Horizon Deployment

## Overview

Horizon provides the graphical management interface for OpenStack.

Although all cloud operations can be performed through APIs or the CLI, Horizon simplifies administration and demonstrations.

---

## Deployment Tasks

- Install Horizon packages.
- Configure local settings.
- Configure Keystone integration.
- Configure session backend.
- Configure allowed hosts.
- Restart Apache.

---

## Validation

Access:

```
http://controller/
```

Verify:

- Login page loads.
- Authentication succeeds.
- Dashboard displays resources.
- Images appear.
- Networks appear.
- Compute services appear.

---

# Phase 12 – Cinder Deployment

## Overview

Cinder provides persistent block storage.

Unlike ephemeral disks, Cinder volumes survive instance deletion.

---

## Deployment Steps

1. Create database.
2. Register service.
3. Configure API.
4. Configure Scheduler.
5. Configure Volume service.
6. Configure LVM backend.
7. Restart services.

---

## LVM Backend

The laboratory uses Logical Volume Manager.

Advantages

- Easy setup
- Native Linux support
- Snapshot capability

Limitations

- Single storage node
- No replication

Production environments often replace LVM with Ceph.

---

## Validation

Example

```bash
openstack volume service list
```

All services should report:

```
enabled

up
```

---

# Phase 13 – Swift Deployment

## Overview

Swift provides distributed object storage.

Unlike block storage, objects are accessed through REST APIs.

---

## Deployment Workflow

```text
Install Packages

↓

Configure Rings

↓

Configure Storage Devices

↓

Configure Proxy

↓

Configure Object Server

↓

Configure Container Server

↓

Configure Account Server

↓

Restart Services
```

---

## Ring Builder

Swift organizes objects using ring files.

Ring builders determine:

- Object placement
- Replication
- Storage device selection

Proper ring configuration is essential for balanced storage utilization.

---

## Validation

Example

```bash
swift stat
```

Verify:

- Proxy responds.
- Rings load correctly.
- Storage nodes communicate.

---

# Phase 14 – Heat Deployment

## Overview

Heat introduces Infrastructure as Code (IaC) capabilities.

Instead of manually creating resources, administrators define templates.

Heat automatically provisions:

- Networks
- Routers
- Instances
- Volumes
- Security Groups

---

## Deployment Tasks

- Create database.
- Register service.
- Configure Keystone.
- Install packages.
- Configure Heat API.
- Configure Engine.
- Synchronize database.

---

## Validation

Example

```bash
openstack orchestration service list
```

The engine should report:

```
up
```

---

# Phase 15 – Monitoring Deployment

## Overview

Infrastructure monitoring is deployed after all OpenStack services become operational.

The laboratory uses Prometheus to collect infrastructure metrics.

---

## Components

- Prometheus Server
- Node Exporter

Optional integrations:

- Grafana
- Alertmanager

---

## Metrics

Collected metrics include:

- CPU utilization
- Memory usage
- Disk usage
- Filesystem utilization
- Network throughput
- System load

---

## Validation

Access

```
http://monitor:9090
```

Confirm:

- Targets are UP.
- Metrics are collected.
- Queries return data.

---

# Post-Deployment Validation

Deployment is not complete until the environment has been validated.

Recommended validation checklist:

✓ Keystone authentication

✓ Service endpoints

✓ Database connectivity

✓ RabbitMQ communication

✓ Image upload

✓ Network creation

✓ Router creation

✓ Floating IP allocation

✓ Instance launch

✓ SSH access

✓ Internet connectivity

✓ Volume attachment

✓ Dashboard access

✓ Monitoring metrics

Every successful validation increases confidence that the environment is functioning correctly.

---

# Rollback Strategy

Major deployment failures should not be corrected by manually modifying random configuration files.

Instead:

1. Stop affected services.
2. Review recent configuration changes.
3. Restore configuration backups.
4. Restore VMware snapshot if necessary.
5. Restart services.
6. Repeat validation.

Using VMware snapshots significantly reduces recovery time.

---

# Common Deployment Issues

| Problem                       | Possible Cause        | Resolution                |
| ----------------------------- | --------------------- | ------------------------- |
| Keystone authentication fails | Time synchronization  | Verify Chrony             |
| Database migration fails      | MariaDB configuration | Check credentials         |
| Compute node missing          | RabbitMQ              | Verify messaging          |
| Instance stuck in BUILD       | Placement or Nova     | Check scheduler           |
| VM has no IP                  | DHCP Agent            | Restart agent             |
| No Internet                   | Router or Floating IP | Verify Neutron            |
| Volume attachment fails       | LVM backend           | Check Cinder              |
| Horizon unavailable           | Apache                | Restart web server        |
| Object upload fails           | Swift Rings           | Verify ring configuration |
| Metrics unavailable           | Prometheus            | Verify scrape targets     |

---

# Deployment Best Practices

To improve deployment reliability:

- Validate every deployment phase.
- Keep configuration files under version control.
- Use descriptive hostnames.
- Synchronize clocks before installing services.
- Backup databases before upgrades.
- Test networking after each configuration change.
- Monitor logs continuously.
- Avoid changing multiple services simultaneously.
- Document every configuration adjustment.
- Create VMware snapshots before major milestones.

---

# Conclusion

Deploying OpenStack requires more than installing packages.

A successful deployment depends on understanding service dependencies, validating each phase, and following a structured implementation process.

By separating deployment into incremental stages, administrators reduce complexity, simplify troubleshooting, and create a cloud platform that is easier to maintain, scale, and upgrade over time.
