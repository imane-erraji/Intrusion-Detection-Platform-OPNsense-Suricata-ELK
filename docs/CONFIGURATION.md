# Platform Configuration

This document provides a detailed explanation of the configuration performed on each component of the Network Intrusion Detection Platform.

The platform consists of:

- OPNsense Firewall
- Suricata IDS
- Logstash
- Elasticsearch
- Kibana
- Telegram Notification System

Each section explains the configuration, the purpose behind it, and the expected outcome.

---

# Table of Contents

- OPNsense Configuration
- Network Interfaces
- Firewall Rules
- NAT Configuration
- DHCP Configuration
- Suricata Configuration
- Logging Configuration
- Logstash Pipeline
- Elasticsearch Configuration
- Kibana Configuration
- Telegram Notification System
- Validation Checklist

---

# OPNsense Configuration

OPNsense acts as the network gateway, firewall, and Intrusion Detection System (IDS).

Version used:

```

26.1.6

```

---

# Network Interfaces

Two virtual network adapters were configured.

| Interface | VMware Network | Purpose |
|------------|---------------|----------|
| WAN | VMnet0 (Bridged) | Internet Access |
| LAN | VMnet1 (Host-Only) | Internal Laboratory |

---

## LAN Interface

```

IPv4 Address

192.168.100.1/24

```

Purpose

- Default Gateway
- DHCP Server
- Internal Routing

---

## WAN Interface

Configured automatically using DHCP from the physical network.

Purpose

- Internet connectivity
- Rule updates
- Package installation

---

# DHCP Server

The DHCP service is enabled on the LAN interface.

Configuration:

| Parameter | Value |
|-----------|-------|
| Network | 192.168.100.0/24 |
| Gateway | 192.168.100.1 |
| DNS | 8.8.8.8 |
| Range Start | 192.168.100.100 |
| Range End | 192.168.100.200 |

This configuration automatically assigns IP addresses to the Windows and Kali virtual machines.

---

# Firewall Rules

A default rule allows outbound communication from the LAN network while keeping the WAN interface protected.

Example policy:

| Source | Destination | Action |
|---------|-------------|--------|
| LAN Net | Any | Allow |
| WAN | LAN | Deny |

This ensures that internal hosts can reach the Internet while unsolicited traffic from external networks is blocked.

---

# Network Address Translation (NAT)

Outbound NAT is configured in **Automatic Mode**.

Purpose:

- Translate private IP addresses to the public interface.
- Allow Internet access for internal virtual machines.

Verification:

```bash
ping google.com
```

Expected Result:

```

Successful replies

```

---

# DNS Configuration

DNS resolution is provided through:

```

8.8.8.8

```

Alternative DNS servers may also be configured depending on the deployment environment.

---

# Suricata Configuration

Suricata is deployed directly on the OPNsense firewall.

Version:

```

Integrated OPNsense Plugin

```

---

## Enabled Interfaces

Monitoring Interface:

```

LAN

```

The LAN interface was selected because all attack simulations occur inside the laboratory network.

---

## Detection Mode

The project uses:

```

IDS Mode

```

Packet inspection is enabled while traffic continues to flow normally.

IPS mode may also be enabled if active blocking is required.

---

## Rule Sources

The following rule source is enabled:

- Emerging Threats Open (ET Open)

Rule updates are performed directly from the OPNsense web interface.

---

## Custom Rules

Additional detection rules were created for:

- ICMP Detection
- SSH Brute Force
- Port Scan Detection
- Laboratory Testing

Location:

```

rules/custom.rules

```

---

## Alert Logging

Suricata generates alerts using the EVE JSON format.

Important log file:

```

/var/log/suricata/eve.json

```

This file is continuously processed by Logstash.

---

# Remote Logging

OPNsense forwards logs to the ELK Server using Syslog.

Configuration:

| Parameter | Value |
|-----------|-------|
| Protocol | UDP |
| Port | 514 |
| Destination | Ubuntu ELK Server |

Purpose:

- Centralized logging
- Long-term storage
- Security monitoring

---

# Logstash Configuration

Logstash acts as the event processing engine.

Pipeline file:

```

configs/logstash.conf

```

Responsibilities:

- Receive Syslog events
- Parse JSON
- Normalize fields
- Filter unwanted data
- Send documents to Elasticsearch

Typical pipeline stages:

```

Input

↓

Filter

↓

Output

```

---

## Input

Receives incoming Syslog messages.

Example:

```ruby
input {
  udp {
    port => 514
  }
}
```

---

## Filter

Extracts useful information.

Examples:

- Source IP
- Destination IP
- Alert Signature
- Protocol
- Timestamp

---

## Output

Processed events are sent to Elasticsearch.

Example:

```ruby
output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "suricata-%{+YYYY.MM.dd}"
  }
}
```

---

# Elasticsearch Configuration

Version:

```

8.19.16

```

Purpose:

- Store security events
- Index alerts
- Provide fast search capabilities

Main configuration file:

```

/etc/elasticsearch/elasticsearch.yml

```

Key settings:

```yaml
network.host: 0.0.0.0

http.port: 9200

discovery.type: single-node
```

Verification:

```bash
curl localhost:9200
```

---

# Kibana Configuration

Version:

```

8.19.16

```

Configuration file:

```

/etc/kibana/kibana.yml

```

Key settings:

```yaml
server.port: 5601

server.host: "0.0.0.0"
```

Purpose:

- Dashboard creation
- Event visualization
- Search interface
- Timeline analysis

---

# Data View

Kibana Data View:

```

suricata-*

```

Timestamp field:

```

@timestamp

```

---

# Dashboard Components

The project dashboard includes:

- Total Alerts
- Alerts by Signature
- Source IP Distribution
- Destination IP Distribution
- Protocol Statistics
- Timeline of Events
- Top Attack Types

---

# Telegram Notification System

Telegram provides instant administrator notifications.

Requirements:

- Telegram Bot
- Bot Token
- Chat ID

Configuration file:

```

configs/telegram.env.example

```

Environment Variables:

```text
BOT_TOKEN=xxxxxxxxxxxxxxxxxxxxxxxx
CHAT_ID=xxxxxxxx
```

Notification Flow:

```

Suricata Alert

↓

Logstash

↓

Python Script

↓

Telegram API

↓

Administrator

```

---

# File Structure

Configuration files are organized as follows:

```
configs/
│
├── elasticsearch.yml
├── kibana.yml
├── logstash.conf
├── suricata.yaml
├── telegram.env.example
└── opnsense-settings.md
```

---

# Configuration Validation

Verify the following after completing the configuration.

| Component | Expected Status |
|-----------|-----------------|
| WAN Interface | Connected |
| LAN Interface | Operational |
| DHCP | Running |
| Suricata | Enabled |
| Rule Updates | Successful |
| Syslog | Forwarding |
| Logstash | Running |
| Elasticsearch | Running |
| Kibana | Accessible |
| Telegram | Sending Alerts |

---

# Security Considerations

The current deployment is intended for laboratory and educational use.

For production environments, consider implementing:

- TLS encryption between components
- Elasticsearch authentication
- Kibana role-based access control
- Dedicated VLANs
- Firewall segmentation
- High Availability (HA)
- Automated backups
- Certificate-based authentication
- Regular rule updates
- Threat Intelligence feeds

---

# Summary

The platform configuration establishes a complete security monitoring pipeline where OPNsense routes traffic, Suricata inspects packets, Logstash processes events, Elasticsearch stores indexed data, Kibana visualizes security information, and Telegram delivers real-time notifications.

Each component performs a dedicated role while remaining modular, making the platform easier to maintain, extend, and adapt to future enhancements.
