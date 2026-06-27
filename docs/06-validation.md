# Validation Guide

# Overview

Deployment is only the first phase of building a cloud platform.

Before the environment can be considered operational, every infrastructure component must be validated to ensure that services communicate correctly, resources can be provisioned successfully, and workloads function as expected.

This document provides a structured validation procedure for the OpenStack laboratory.

Rather than testing individual services in isolation, the validation process confirms the complete end-to-end functionality of the cloud platform.

---

# Validation Objectives

The primary objectives are to verify:

- Infrastructure readiness
- Service availability
- Authentication
- Networking
- Compute resources
- Image management
- Storage
- Orchestration
- Monitoring
- External connectivity

Each validation stage builds upon the previous one.

---

# Validation Workflow

```text
Infrastructure

↓

Operating System

↓

Supporting Services

↓

OpenStack APIs

↓

Compute

↓

Networking

↓

Storage

↓

Orchestration

↓

Monitoring

↓

User Acceptance Test
```

Failures should be resolved before continuing to the next stage.

---

# Stage 1 – Infrastructure Validation

## Virtual Machines

Verify that all infrastructure nodes are powered on and accessible.

Expected nodes:

- Controller
- Compute01
- Compute02
- Compute03
- Storage
- Swift
- Monitor

Validation checklist:

- VM is running.
- CPU allocation is correct.
- Memory allocation is correct.
- Disk capacity is sufficient.
- Network adapters are connected.

---

## SSH Connectivity

Verify SSH access from the Controller to every node.

Example:

```bash
ssh compute01
ssh compute02
ssh storage
ssh swift
ssh monitor
```

Passwordless SSH using public keys is recommended for administrative tasks.

---

# Stage 2 – Operating System Validation

Verify the operating system configuration on every node.

Checklist:

- Correct hostname
- Static IP address
- DNS resolution
- Gateway configuration
- Package repositories
- Firewall status
- Disk space
- Memory availability

Useful commands:

```bash
hostnamectl
ip addr
ip route
df -h
free -h
```

---

# Stage 3 – Time Synchronization

Authentication relies on synchronized clocks.

Validation:

```bash
chronyc tracking
chronyc sources
timedatectl
```

Expected result:

- Synchronization active
- Small clock offset
- No time drift warnings

---

# Stage 4 – Supporting Infrastructure

## MariaDB

Verify:

- Service running
- Database connectivity
- Service databases exist

Example:

```bash
systemctl status mariadb
mysql -u root -p
```

---

## RabbitMQ

Verify:

- Service status
- Queue health
- User accounts

Example:

```bash
rabbitmqctl status
rabbitmqctl list_users
```

---

## Memcached

Verify:

```bash
systemctl status memcached
```

---

## Etcd

Verify cluster health.

Example:

```bash
etcdctl endpoint health
```

---

# Stage 5 – Keystone Validation

Authentication is the foundation of OpenStack.

Validation:

```bash
openstack token issue
```

Expected result:

- Token generated successfully
- Expiration time displayed
- Project scope returned

Also verify:

- Users
- Projects
- Roles
- Service Catalog

---

# Stage 6 – Service Registration

Every service should appear in Keystone.

Example:

```bash
openstack service list
```

Verify services:

- Keystone
- Glance
- Nova
- Placement
- Neutron
- Cinder
- Heat
- Swift

Missing services indicate incomplete deployment.

---

# Stage 7 – API Endpoint Validation

Verify API endpoint registration.

Example:

```bash
openstack endpoint list
```

Confirm:

- Public endpoints
- Internal endpoints
- Administrative endpoints

Endpoints should resolve correctly from every node.

---

# Stage 8 – Image Service Validation

Upload a test image.

Example:

```bash
openstack image create
```

Verify:

```bash
openstack image list
```

Expected:

- Image status = active
- Correct size
- Correct format

Images should be downloadable by compute nodes.

---

# Stage 9 – Compute Validation

Verify compute services.

```bash
openstack compute service list
```

Expected:

- Enabled
- Up

Verify hypervisors.

```bash
openstack hypervisor list
```

Confirm:

- CPU resources
- Memory resources
- Local storage

Every compute node should report available resources.

---

# Stage 10 – Networking Validation

Create a test network.

Validation includes:

- Network creation
- Subnet creation
- Router creation
- Gateway configuration

Verify:

```bash
openstack network list
openstack subnet list
openstack router list
```

All resources should appear successfully.

---

# Stage 11 – Security Group Validation

Create or inspect Security Groups.

Verify:

- SSH
- ICMP
- HTTP
- HTTPS

Example:

```bash
openstack security group rule list
```

Improper firewall rules are among the most common causes of failed connectivity.

---

# Stage 12 – Instance Validation

Launch a virtual machine.

Example:

```bash
openstack server create
```

Expected lifecycle:

```text
BUILD

↓

ACTIVE
```

Confirm:

- Image boots
- IP assigned
- SSH key injected
- Instance accessible

---

# Stage 13 – Floating IP Validation

Allocate a Floating IP.

Associate it with the test instance.

Verify:

- Public IP assigned
- Ping successful
- SSH successful

Traffic path:

```text
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

---

# Stage 14 – Storage Validation

Create a Cinder volume.

Verify:

```bash
openstack volume list
```

Attach the volume.

Confirm:

- Attachment successful
- Device detected inside VM
- Filesystem creation possible

---

# Stage 15 – Object Storage Validation

Upload an object to Swift.

Example:

```bash
swift upload
```

Verify:

- Upload succeeds
- Object listed
- Download successful

---

# Stage 16 – Heat Validation

Deploy a simple orchestration template.

Validation:

```bash
openstack stack create
```

Verify:

```bash
openstack stack list
```

Expected:

- CREATE_COMPLETE

---

# Stage 17 – Monitoring Validation

Open Prometheus.

Verify:

- Targets = UP
- Metrics collected
- Queries return data

Example metrics:

- CPU
- Memory
- Disk
- Network

---

# Stage 18 – End-to-End User Acceptance Test

The final validation simulates a complete user workflow.

Scenario:

1. User logs into Horizon.
2. Upload image.
3. Create network.
4. Create subnet.
5. Create router.
6. Launch instance.
7. Assign Floating IP.
8. SSH into instance.
9. Create volume.
10. Attach volume.
11. Verify Internet access.

Successful completion confirms that all major OpenStack services are functioning together.

---

# Validation Checklist

| Component      | Status |
| -------------- | ------ |
| Infrastructure | ✓      |
| Network        | ✓      |
| Time Sync      | ✓      |
| Database       | ✓      |
| RabbitMQ       | ✓      |
| Keystone       | ✓      |
| Glance         | ✓      |
| Placement      | ✓      |
| Nova           | ✓      |
| Neutron        | ✓      |
| Horizon        | ✓      |
| Cinder         | ✓      |
| Swift          | ✓      |
| Heat           | ✓      |
| Prometheus     | ✓      |

Every component should be validated before the environment is considered production-ready.

---

# Common Validation Failures

| Symptom                 | Possible Cause   |
| ----------------------- | ---------------- |
| Login fails             | Keystone         |
| Image upload fails      | Glance           |
| Instance stuck in BUILD | Nova / Placement |
| No IP address           | DHCP Agent       |
| Cannot reach Internet   | Neutron Router   |
| SSH fails               | Security Group   |
| Volume attachment fails | Cinder           |
| Stack creation fails    | Heat             |
| Metrics missing         | Prometheus       |

Validation should always begin by identifying the first failed dependency rather than troubleshooting every service simultaneously.

---

# Best Practices

- Validate incrementally after each deployment phase.
- Record validation results.
- Automate repetitive tests where possible.
- Keep baseline screenshots for comparison.
- Repeat validation after updates or upgrades.
- Document every anomaly and its resolution.
- Perform periodic health checks even after deployment.

---

# Conclusion

Validation is an essential stage of every OpenStack deployment.

By systematically verifying infrastructure, core services, networking, storage, orchestration, and monitoring, administrators can ensure that the cloud platform is stable, functional, and ready for real-world workloads.

A disciplined validation process not only reduces deployment risk but also provides confidence that future maintenance and scaling activities can be performed on a reliable foundation.

# Advanced Validation and Operations Runbook

## Health Check Matrix

Routine health checks should be performed daily to ensure that the OpenStack control plane remains operational.

The following matrix summarizes recommended validation commands.

| Component  | Validation Command                     | Expected Result              |
| ---------- | -------------------------------------- | ---------------------------- |
| Keystone   | `openstack token issue`                | Token successfully generated |
| Glance     | `openstack image list`                 | Images displayed             |
| Nova       | `openstack compute service list`       | All services Up              |
| Placement  | `openstack resource provider list`     | Providers listed             |
| Neutron    | `openstack network agent list`         | Agents Alive                 |
| Horizon    | Access Web Dashboard                   | Login page accessible        |
| Cinder     | `openstack volume service list`        | Services Up                  |
| Swift      | `swift stat`                           | Proxy responds               |
| Heat       | `openstack orchestration service list` | Engine Up                    |
| Prometheus | Web UI                                 | Targets Up                   |

Administrators should automate these checks whenever possible.

---

# Infrastructure Health Verification

Before investigating OpenStack services, verify the operating system itself.

Recommended commands:

```bash
uptime
hostnamectl
free -h
df -h
ip addr
ip route
systemctl --failed
```

Expected outcome:

- No failed system services
- Sufficient free memory
- Available disk capacity
- Correct routing table
- Expected IP addresses

---

# Service Health Verification

Every OpenStack service should be verified independently.

Example:

```bash
systemctl status apache2
systemctl status mariadb
systemctl status rabbitmq-server
systemctl status memcached
systemctl status etcd
systemctl status nova-api
systemctl status nova-scheduler
systemctl status neutron-server
systemctl status cinder-api
```

All services should report:

- Active (running)
- Enabled
- No restart loop

---

# API Endpoint Validation

Every registered endpoint should be reachable.

Example:

```bash
openstack endpoint list
```

Verify:

- Public URL
- Internal URL
- Admin URL

Each endpoint should respond without authentication or connectivity errors.

---

# Database Validation

Verify database availability.

Example:

```bash
mysql -u root -p
SHOW DATABASES;
```

Confirm that databases exist for:

- keystone
- glance
- nova
- neutron
- placement
- cinder
- heat

Unexpected schema errors usually indicate failed migrations.

---

# RabbitMQ Validation

RabbitMQ forms the communication backbone of OpenStack.

Verification:

```bash
rabbitmqctl status
rabbitmqctl list_users
rabbitmqctl list_queues
```

Expected:

- Running node
- No queue backlog
- Required service users present

Large queue backlogs may indicate stalled compute or networking services.

---

# Compute Validation

Each compute node should periodically report its resources.

Validation:

```bash
openstack hypervisor list
openstack hypervisor stats show
```

Confirm:

- CPU inventory
- Memory inventory
- Disk inventory
- Hypervisor status

Unavailable hypervisors require immediate investigation.

---

# Instance Lifecycle Validation

Create a temporary test instance.

Observe lifecycle transitions.

```text
BUILD

↓

ACTIVE

↓

STOPPED

↓

ACTIVE

↓

REBOOT

↓

ACTIVE

↓

DELETED
```

Every transition should complete successfully.

Unexpected ERROR states require log analysis.

---

# Network Validation

Networking should be validated beyond simple ping tests.

Verify:

- DHCP
- Routing
- Security Groups
- Floating IP
- Metadata
- VXLAN connectivity

Commands:

```bash
openstack network list
openstack subnet list
openstack router list
openstack floating ip list
```

Network namespaces should also be inspected.

```bash
ip netns
```

Expected namespaces include:

- qrouter-\*
- qdhcp-\*

---

# Storage Validation

Persistent storage validation includes:

- Volume creation
- Volume attachment
- Filesystem creation
- Mount verification
- Volume deletion

Commands:

```bash
openstack volume list
lsblk
mount
```

Volumes should remain available after instance reboot.

---

# Object Storage Validation

Validate Swift by performing the following sequence:

1. Create container.
2. Upload object.
3. List object.
4. Download object.
5. Delete object.

Each operation confirms proper object storage functionality.

---

# Monitoring Validation

Verify Prometheus targets.

Open:

```
http://monitor:9090
```

Expected:

- Controller node = UP
- Compute nodes = UP
- Storage node = UP

Queries should return metrics immediately.

---

# Acceptance Criteria

The deployment should only be considered successful if all acceptance criteria are met.

| Requirement    | Acceptance Criteria             |
| -------------- | ------------------------------- |
| Authentication | Users can authenticate          |
| Dashboard      | Horizon accessible              |
| Images         | Upload successful               |
| Compute        | Instance launches               |
| Networking     | Internet connectivity available |
| Storage        | Volume attachment successful    |
| Object Storage | Upload/download successful      |
| Monitoring     | Metrics visible                 |
| Orchestration  | Heat stack deployed             |

Failure of any critical requirement should block production use.

---

# Failure Injection Tests

Controlled failure testing increases confidence in the deployment.

Recommended scenarios:

### RabbitMQ Restart

Expected:

- Temporary RPC interruption
- Automatic service recovery

---

### Compute Node Shutdown

Expected:

- Hypervisor marked Down
- Existing workloads unaffected
- Scheduler excludes unavailable host

---

### Apache Restart

Expected:

- Temporary Horizon outage
- APIs recover automatically

---

### Controller Reboot

Expected:

- Services restart successfully
- Authentication restored
- APIs available

Failure injection helps verify operational resilience.

---

# Disaster Recovery Validation

Administrators should periodically test recovery procedures.

Examples:

- Restore database backup
- Restore VMware snapshot
- Recover failed compute node
- Restart networking services
- Rejoin RabbitMQ
- Verify API endpoints

Recovery procedures should be documented and repeatable.

---

# Log Analysis

Most deployment issues can be diagnosed through log files.

Typical locations:

| Service  | Log Directory     |
| -------- | ----------------- |
| Keystone | /var/log/keystone |
| Nova     | /var/log/nova     |
| Neutron  | /var/log/neutron  |
| Glance   | /var/log/glance   |
| Cinder   | /var/log/cinder   |
| Heat     | /var/log/heat     |
| Apache   | /var/log/apache2  |

Useful commands:

```bash
journalctl -xe
tail -f /var/log/nova/nova-api.log
tail -f /var/log/neutron/server.log
```

Logs should always be reviewed before modifying configuration files.

---

# Validation Report Template

Each deployment should conclude with a validation report.

Example fields:

- Deployment Date
- OpenStack Release
- Ubuntu Version
- Number of Nodes
- Validation Engineer
- Passed Tests
- Failed Tests
- Known Issues
- Corrective Actions
- Final Status

Maintaining historical validation reports simplifies future upgrades and troubleshooting.

---

# Operational Best Practices

To maintain a healthy OpenStack environment:

- Perform daily health checks.
- Monitor disk utilization.
- Monitor RabbitMQ queue depth.
- Verify Chrony synchronization.
- Review failed systemd services.
- Test backups regularly.
- Rotate logs.
- Apply security updates during maintenance windows.
- Revalidate the platform after every upgrade.
- Document operational changes.

---

# Conclusion

Validation is not a one-time activity performed after deployment.

Instead, it should become a continuous operational process integrated into routine system administration.

By combining infrastructure verification, service health checks, functional testing, failure injection, and recovery validation, administrators can maintain a reliable and resilient OpenStack environment throughout its lifecycle.
