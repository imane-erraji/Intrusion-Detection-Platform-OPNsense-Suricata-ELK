# System Architecture

This document describes the architecture of the Network Intrusion Detection Platform, including its components, network topology, traffic flow, security zones, and event processing pipeline.

---

# Table of Contents

- Overview
- Design Objectives
- Architecture Overview
- Network Topology
- Infrastructure Components
- Data Flow
- Security Zones
- Event Lifecycle
- Design Decisions
- Network Services
- Communication Matrix
- Platform Advantages
- Current Limitations
- Future Improvements

---

# Overview

The platform was designed to emulate a lightweight Security Operations Center (SOC) using entirely open-source technologies.

Its primary objective is to detect malicious network activities, centralize security logs, visualize events in real time, and notify administrators whenever suspicious behavior is detected.

Unlike a traditional firewall-only deployment, this architecture integrates multiple security components into a unified monitoring platform capable of collecting, processing, storing, and analyzing network security events.

---

# Design Objectives

The architecture was designed around the following goals:

- Detect network attacks in real time
- Monitor all internal traffic
- Centralize security logs
- Visualize attack events
- Simplify incident investigation
- Provide instant administrator notifications
- Remain entirely open source
- Be reproducible in a virtual laboratory

---

# High-Level Architecture

```
                        Internet
                            │
                            │
                   ┌────────────────┐
                   │   OPNsense     │
                   │ Firewall + IDS │
                   └────────────────┘
                            │
                   Host-Only Network
                            │
      ┌───────────────┬───────────────┬───────────────┐
      │               │               │
      │               │               │
┌────────────┐  ┌────────────┐  ┌────────────┐
│ Ubuntu ELK │  │ Windows 10 │  │ Kali Linux │
│ SIEM       │  │ Victim      │  │ Attacker   │
└────────────┘  └────────────┘  └────────────┘
```

---

# Component Responsibilities

## OPNsense

OPNsense serves as the network perimeter firewall and routing gateway.

Responsibilities include:

- Packet filtering
- NAT
- DHCP
- Network routing
- Traffic inspection
- Suricata integration
- Log forwarding

---

## Suricata

Suricata is responsible for deep packet inspection.

It continuously analyzes network traffic and compares packets against thousands of intrusion detection signatures.

Responsibilities:

- Packet inspection
- Threat detection
- Signature matching
- Protocol analysis
- Alert generation
- Network visibility

---

## Logstash

Logstash acts as the ingestion layer of the platform.

Responsibilities:

- Receive logs
- Parse events
- Normalize data
- Extract fields
- Forward documents to Elasticsearch

---

## Elasticsearch

Elasticsearch stores all security events.

Responsibilities:

- Index creation
- Search engine
- Data storage
- Fast querying
- Aggregation
- Historical analysis

---

## Kibana

Kibana provides the visualization layer.

Responsibilities:

- Dashboards
- Charts
- Timelines
- Alert investigation
- Event filtering
- Search interface

---

## Telegram Bot

Telegram acts as the notification system.

Responsibilities:

- Alert delivery
- Remote monitoring
- Immediate administrator notification

---

# Network Topology

## External Network

```
VMnet0
```

Purpose

- Internet connectivity

Connected Component

- OPNsense WAN

---

## Internal Network

```
VMnet1
```

Purpose

Private laboratory network

Connected Components

- Ubuntu ELK
- Kali Linux
- Windows
- OPNsense LAN

---

# Security Zones

## WAN Zone

Characteristics

- Untrusted
- Internet-facing
- Incoming traffic

Security Controls

- Firewall Rules
- NAT
- Packet Filtering

---

## LAN Zone

Characteristics

Trusted internal network.

Contains:

- ELK Server
- Windows Victim
- Kali Attacker

Traffic is continuously inspected by Suricata.

---

# Event Processing Pipeline

The following diagram illustrates the lifecycle of a security event.

```
Network Packet

        │

        ▼

OPNsense

        │

Packet Inspection

        ▼

Suricata IDS

        │

Alert Generated

        ▼

Syslog

        │

UDP 514

        ▼

Logstash

        │

JSON Processing

        ▼

Elasticsearch

        │

Indexed Event

        ▼

Kibana Dashboard

        │

Critical Alert

        ▼

Telegram Bot
```

---

# Security Event Lifecycle

## Phase 1

Traffic enters the firewall.

---

## Phase 2

Suricata inspects packets.

---

## Phase 3

A rule matches malicious activity.

---

## Phase 4

An alert is generated.

---

## Phase 5

The alert is exported through Syslog.

---

## Phase 6

Logstash receives and parses the event.

---

## Phase 7

The event is indexed in Elasticsearch.

---

## Phase 8

Kibana visualizes the alert.

---

## Phase 9

Telegram sends a notification.

---

# Communication Matrix

| Source | Destination | Protocol | Purpose |
|---------|-------------|----------|----------|
| Windows | OPNsense | TCP/IP | Network Access |
| Kali | OPNsense | TCP/IP | Attack Simulation |
| OPNsense | Logstash | Syslog UDP 514 | Log Export |
| Logstash | Elasticsearch | TCP 9200 | Event Indexing |
| Kibana | Elasticsearch | REST API | Data Queries |
| Python Script | Telegram API | HTTPS | Alert Notification |

---

# Detection Workflow

The platform detects several attack categories.

| Attack | Detection Component |
|---------|--------------------|
| Port Scan | Suricata |
| SSH Brute Force | Suricata |
| ICMP Flood | Suricata |
| Custom Rules | Suricata |

All detected events are indexed by Elasticsearch and visualized through Kibana.

---

# Design Decisions

## Why OPNsense?

- Open source
- Enterprise-grade firewall
- Native Suricata integration
- Web management interface
- Active community support

---

## Why Suricata?

- High-performance IDS
- Multi-threaded architecture
- Extensive rule support
- Protocol awareness
- Community rule sets

---

## Why ELK Stack?

- Powerful search engine
- Flexible dashboards
- Large ecosystem
- Excellent visualization capabilities
- Widely adopted in SOC environments

---

## Why VMware Workstation?

VMware Workstation enables the creation of an isolated laboratory environment where attacks can be executed safely without affecting production systems.

---

# Platform Advantages

- Modular architecture
- Fully open source
- Easy to extend
- Scalable
- Real-time monitoring
- Centralized logging
- Reproducible laboratory
- Practical SOC workflow

---

# Current Limitations

The current implementation has several limitations:

- Single-node Elasticsearch deployment
- No high availability
- Signature-based detection only
- No endpoint detection agent
- No automatic incident response
- Limited attack coverage
- Manual dashboard creation

---

# Future Improvements

Potential enhancements include:

- Wazuh integration
- Zeek Network Monitor
- Elastic Security
- Docker Compose deployment
- Kubernetes deployment
- Multi-node Elasticsearch cluster
- Sigma rule support
- Threat Intelligence feeds
- SOAR integration
- AI-assisted anomaly detection

---

# Architecture Summary

The platform follows a layered security architecture where each component performs a dedicated function within the detection pipeline.

Traffic is first inspected by OPNsense and Suricata, security events are exported to Logstash, indexed by Elasticsearch, visualized through Kibana, and finally forwarded to Telegram for real-time notification.

This modular design simplifies maintenance, facilitates future enhancements, and reflects the architecture commonly found in modern Security Operations Centers (SOCs).
