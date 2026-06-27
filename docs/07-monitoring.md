# Monitoring and Observability

# Overview

Monitoring is an essential component of every cloud platform.

Deploying OpenStack successfully is only the beginning. Administrators must continuously observe infrastructure health, detect failures early, analyze resource utilization, and respond to incidents before they impact users.

This laboratory implements infrastructure monitoring using Prometheus and Node Exporter to collect system metrics from every node within the OpenStack environment.

The monitoring platform provides visibility into:

- Infrastructure health
- Compute resources
- Storage capacity
- Network utilization
- Operating system metrics
- Service availability

The collected data enables proactive administration and capacity planning.

---

# Monitoring Architecture

The monitoring stack follows a pull-based architecture.

```text
                 +----------------------+
                 |     Prometheus       |
                 +----------+-----------+
                            |
          ------------------------------------------
          |         |          |          |         |
          ▼         ▼          ▼          ▼         ▼
     Controller  Compute1  Compute2  Storage  Monitor
          |         |          |          |         |
          ▼         ▼          ▼          ▼         ▼
     Node Exporter on every Linux host
```

Prometheus periodically scrapes metrics from every exporter.

No agents actively push metrics to the monitoring server.

---

# Monitoring Objectives

The monitoring system should provide visibility into:

- CPU utilization
- Memory utilization
- Disk capacity
- Filesystem usage
- Network throughput
- Process health
- Service availability
- System uptime

Monitoring should answer questions such as:

- Is the Controller overloaded?
- Which Compute Node has available resources?
- Is storage reaching capacity?
- Which network interface is saturated?
- Are OpenStack services still running?

---

# Monitoring Components

The monitoring platform consists of several independent components.

## Prometheus

Responsibilities:

- Metric collection
- Time-series database
- Query engine
- Alert evaluation
- Target discovery

Prometheus periodically scrapes metrics from configured targets.

---

## Node Exporter

Node Exporter exposes Linux operating system metrics.

Examples include:

- CPU
- Memory
- Load Average
- Filesystem
- Network
- Disk
- Kernel statistics

Every infrastructure node runs one Node Exporter instance.

---

## Optional Components

Although not mandatory for this laboratory, the following tools are commonly deployed together with Prometheus:

- Grafana
- Alertmanager
- Loki
- Tempo

These components extend visualization, alerting, and log aggregation capabilities.

---

# Metric Collection Workflow

```text
Linux Host

↓

Node Exporter

↓

HTTP Endpoint

↓

Prometheus Scraper

↓

Time-Series Database

↓

Dashboard / Alert
```

This workflow repeats automatically according to the configured scrape interval.

---

# Prometheus Configuration

Prometheus behavior is defined in the configuration file.

Typical configuration sections include:

- Global Settings
- Scrape Interval
- Evaluation Interval
- Alert Rules
- Target Definitions

Example scrape interval:

```
15 seconds
```

A shorter interval provides higher resolution but increases storage and CPU utilization.

---

# Scrape Targets

Each infrastructure node should appear as a scrape target.

Example targets:

| Target     | Purpose             |
| ---------- | ------------------- |
| Controller | Control Plane       |
| Compute01  | Compute Resources   |
| Compute02  | Compute Resources   |
| Compute03  | Compute Resources   |
| Storage    | Block Storage       |
| Swift      | Object Storage      |
| Monitor    | Monitoring Services |

Every target should report an **UP** status.

---

# Node Exporter Metrics

Node Exporter exposes hundreds of Linux metrics.

Examples include:

CPU Metrics

- CPU Usage
- Idle Time
- Context Switches
- Interrupts

Memory Metrics

- Total Memory
- Used Memory
- Available Memory
- Cached Memory
- Swap Usage

Filesystem Metrics

- Total Capacity
- Free Capacity
- Used Capacity
- Inodes

Disk Metrics

- Read Operations
- Write Operations
- Queue Length
- IO Time

Network Metrics

- Packets Received
- Packets Sent
- Errors
- Drops
- Bandwidth

These metrics provide a complete view of operating system performance.

---

# Infrastructure Metrics

Infrastructure monitoring focuses on physical and virtual resource utilization.

Important metrics include:

CPU

- Usage %
- Idle %
- System %
- User %

Memory

- Utilization
- Available RAM
- Swap

Storage

- Disk Space
- Disk Latency
- Filesystem Usage

Network

- Bandwidth
- Packet Loss
- Interface Errors

These metrics help identify infrastructure bottlenecks before they affect workloads.

---

# Service Monitoring

Operating system metrics alone are insufficient.

Critical OpenStack services should also be monitored.

Examples include:

- Apache
- MariaDB
- RabbitMQ
- Memcached
- Keystone
- Nova
- Neutron
- Glance
- Cinder
- Heat

Recommended checks include:

- Service running
- Restart count
- Process availability
- Response time

Monitoring service health reduces incident detection time.

---

# Validation

Prometheus should be accessible through the web interface.

Example:

```
http://monitor:9090
```

Verify:

- All targets appear.
- Targets are UP.
- Queries return results.
- No scrape failures.
- No configuration errors.

## A healthy monitoring platform forms the foundation for operational visibility.

# PromQL Fundamentals

Prometheus stores metrics as time-series data.

Administrators retrieve information using PromQL (Prometheus Query Language).

Understanding PromQL enables engineers to investigate performance issues, identify resource bottlenecks, and build meaningful dashboards.

---

# CPU Monitoring

Overall CPU utilization

```promql
100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

This query calculates the percentage of CPU actively used during the previous five minutes.

Recommended thresholds

| Usage  | Status   |
| ------ | -------- |
| < 60%  | Normal   |
| 60–80% | Moderate |
| 80–90% | High     |
| >90%   | Critical |

Persistent CPU usage above 90% may indicate resource saturation or excessive workload scheduling.

---

# Memory Monitoring

Memory utilization

```promql
100 * (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes))
```

Expected ranges

| Usage  | Status   |
| ------ | -------- |
| <70%   | Healthy  |
| 70–85% | Observe  |
| >85%   | Warning  |
| >95%   | Critical |

High memory utilization can lead to swapping and degraded virtual machine performance.

---

# Filesystem Monitoring

Filesystem usage

```promql
100 -
(
node_filesystem_avail_bytes
/
node_filesystem_size_bytes
*100
)
```

Recommended threshold

```
80%
```

Critical threshold

```
90%
```

Disk exhaustion can prevent OpenStack services from writing logs or storing instance data.

---

# Network Monitoring

Receive throughput

```promql
rate(node_network_receive_bytes_total[5m])
```

Transmit throughput

```promql
rate(node_network_transmit_bytes_total[5m])
```

Network monitoring helps identify:

- Bandwidth saturation
- Packet loss
- Unexpected traffic spikes
- Interface congestion

---

# Disk I/O Monitoring

Read operations

```promql
rate(node_disk_reads_completed_total[5m])
```

Write operations

```promql
rate(node_disk_writes_completed_total[5m])
```

Disk monitoring is especially important for:

- MariaDB
- RabbitMQ
- Glance
- Cinder

Storage latency directly affects cloud responsiveness.

---

# Load Average

Current system load

```promql
node_load5
```

Interpretation

Load should generally remain below the number of available CPU cores.

Example

8 CPU cores

Healthy load:

```
<8
```

Sustained values significantly above the core count suggest CPU contention.

---

# Monitoring OpenStack Services

Infrastructure metrics alone cannot determine cloud health.

Critical services should also be monitored.

Examples

- Keystone
- Glance
- Nova API
- Nova Scheduler
- Nova Compute
- Placement
- Neutron Server
- Neutron Agents
- Cinder API
- Horizon
- RabbitMQ
- MariaDB

Recommended health indicators include:

- Process availability
- Service restart count
- Response latency
- API reachability

---

# Alerting Strategy

Monitoring without alerts requires administrators to constantly observe dashboards.

Alerting enables automated notification when predefined conditions occur.

Typical alert workflow

```text
Metric

↓

Prometheus Rule

↓

Alertmanager

↓

Email / Slack / Microsoft Teams

↓

Administrator
```

---

# Example Alert Rules

## High CPU Utilization

Condition

```
CPU > 90%

Duration

10 minutes
```

Severity

```
Warning
```

---

## Critical Memory Usage

Condition

```
Memory >95%

Duration

5 minutes
```

Severity

```
Critical
```

---

## Disk Capacity

Condition

```
Filesystem >85%
```

Severity

```
Warning
```

---

## Instance Down

Condition

```
Node Exporter unreachable
```

Severity

```
Critical
```

---

## RabbitMQ Failure

Condition

RabbitMQ unavailable.

Impact

- Instance creation fails.
- Scheduler communication interrupted.
- Compute services become unavailable.

---

## Keystone Failure

Impact

- User authentication fails.
- APIs reject requests.
- Horizon login unavailable.

Immediate administrator intervention is required.

---

# Alert Severity Levels

| Severity | Description               |
| -------- | ------------------------- |
| Info     | Informational only        |
| Warning  | Attention recommended     |
| Critical | Immediate action required |

Using severity levels helps prioritize operational response.

---

# Dashboard Design

Effective dashboards should provide an overview before detailed analysis.

Recommended sections

## Infrastructure Overview

- Total CPU
- Total Memory
- Total Disk
- System Uptime

---

## Compute Resources

- Hypervisors
- Active Instances
- CPU Allocation
- Memory Allocation

---

## Storage

- Disk Capacity
- Volume Usage
- Storage IOPS

---

## Networking

- Bandwidth
- Packet Errors
- Interface Status

---

## OpenStack Services

Current status of

- Keystone
- Nova
- Neutron
- Glance
- Cinder
- Heat

Color-coded status indicators improve operational visibility.

---

# Capacity Planning

Monitoring historical metrics supports future infrastructure planning.

Administrators should evaluate:

- CPU growth
- Memory growth
- Storage growth
- Network growth

Capacity reports assist with:

- Hardware procurement
- Cluster expansion
- Budget planning

Capacity planning should rely on long-term trends rather than instantaneous measurements.

---

# Performance Baseline

A baseline defines expected system behavior under normal operating conditions.

Example baseline

| Metric       | Expected |
| ------------ | -------- |
| CPU          | <50%     |
| Memory       | <70%     |
| Disk         | <60%     |
| Network      | Stable   |
| API Response | <500 ms  |

Performance deviations become easier to identify once a baseline has been established.

---

# SLA, SLO, and Error Budget

Modern cloud operations are often guided by service reliability objectives.

## SLA (Service Level Agreement)

Defines the availability commitment provided to users.

Example

99.9% monthly availability.

---

## SLO (Service Level Objective)

Defines the operational target maintained by the engineering team.

Examples

- API response time
- Service availability
- Recovery time

---

## Error Budget

Represents the acceptable amount of service disruption within a given period.

Rather than striving for absolute perfection, engineering teams balance reliability with feature delivery by managing the available error budget.

---

# Troubleshooting Using Metrics

Monitoring data assists administrators during incident response.

Example workflow

```text
User reports slow VM

↓

Check CPU

↓

Check Memory

↓

Check Disk

↓

Check Network

↓

Review Logs

↓

Identify Root Cause
```

This systematic approach reduces troubleshooting time and avoids unnecessary configuration changes.

---

# Monitoring Best Practices

To maintain reliable observability:

- Monitor every infrastructure node.
- Monitor both infrastructure and application services.
- Configure meaningful alert thresholds.
- Review alert noise regularly.
- Retain historical metrics for trend analysis.
- Synchronize monitoring with centralized logging.
- Document alert response procedures.
- Validate monitoring after upgrades.
- Regularly test alert delivery mechanisms.
- Continuously refine dashboards as the environment evolves.

---

# Conclusion

Monitoring is a continuous operational process rather than a one-time deployment task.

By combining infrastructure metrics, service health monitoring, alerting, capacity planning, and structured troubleshooting, administrators gain comprehensive visibility into the OpenStack environment.

A mature monitoring strategy enables proactive maintenance, improves operational stability, and supports informed decision-making as the cloud infrastructure grows.
