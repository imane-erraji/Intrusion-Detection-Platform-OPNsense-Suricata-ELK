# Troubleshooting Guide

This document provides solutions to the most common issues that may occur while deploying or operating the Network Intrusion Detection Platform.

The troubleshooting procedures are based on the project architecture and are intended to help identify, diagnose, and resolve configuration or operational problems.

---

# Table of Contents

- Overview
- General Verification
- OPNsense Issues
- Suricata Issues
- Logstash Issues
- Elasticsearch Issues
- Kibana Issues
- Telegram Issues
- VMware Networking Issues
- Detection Issues
- Performance Issues
- Useful Commands
- Conclusion

---

# Overview

Before troubleshooting a specific component, verify that all virtual machines are powered on and connected to the correct VMware virtual networks.

The platform depends on the following services:

- OPNsense
- Suricata
- Logstash
- Elasticsearch
- Kibana
- Telegram Notification Script

A failure in one component may affect the entire detection pipeline.

---

# General Verification

Verify that every virtual machine can communicate with the firewall.

Example:

```bash
ping 192.168.100.1
```

Verify Internet connectivity.

```bash
ping google.com
```

Confirm that all required services are running.

---

# OPNsense Issues

## Problem

Virtual machines cannot access the Internet.

### Possible Causes

- WAN interface disconnected
- Incorrect gateway
- NAT misconfiguration
- DNS configuration issue

### Resolution

Verify:

```
Interfaces
→ WAN
```

Check:

- Interface status
- Gateway
- IP address

Verify outbound NAT.

```
Firewall
→ NAT
→ Outbound
```

Automatic Outbound NAT should be enabled.

---

## Problem

LAN clients do not receive an IP address.

### Possible Causes

- DHCP disabled
- Incorrect DHCP range
- Wrong VMware network

### Resolution

Navigate to:

```
Services
→ DHCPv4
```

Verify:

- DHCP is enabled
- Correct address pool
- LAN interface selected

---

# Suricata Issues

## Problem

No alerts are generated.

### Possible Causes

- IDS disabled
- Rule set not installed
- Wrong monitoring interface
- No matching traffic

### Resolution

Navigate to:

```
Services
→ Intrusion Detection
```

Verify:

- IDS is enabled
- Rules downloaded successfully
- LAN interface selected

Generate test traffic.

```bash
nmap -sS 192.168.100.10
```

---

## Problem

Custom rules are ignored.

### Possible Causes

- Invalid rule syntax
- Duplicate SID
- Rule file not loaded

### Resolution

Verify:

```
rules/custom.rules
```

Restart Suricata after every modification.

Reload the rule set and confirm that no syntax errors are reported.

---

# Logstash Issues

## Problem

No logs are received.

### Possible Causes

- Syslog forwarding disabled
- Incorrect UDP port
- Logstash service stopped

### Resolution

Check service status.

```bash
sudo systemctl status logstash
```

Review the log file.

```bash
sudo journalctl -u logstash
```

Confirm that OPNsense forwards logs to the correct IP address and port.

---

## Problem

Pipeline configuration errors.

### Resolution

Validate the configuration.

```bash
sudo /usr/share/logstash/bin/logstash --config.test_and_exit -f /etc/logstash/conf.d/logstash.conf
```

Correct any reported syntax errors before restarting the service.

---

# Elasticsearch Issues

## Problem

Elasticsearch service does not start.

### Possible Causes

- Insufficient memory
- Incorrect configuration
- Disk space exhausted

### Resolution

Check service status.

```bash
sudo systemctl status elasticsearch
```

Inspect logs.

```bash
sudo journalctl -u elasticsearch
```

Verify the configuration file.

```
/etc/elasticsearch/elasticsearch.yml
```

---

## Problem

No indices are created.

### Resolution

Query the cluster.

```bash
curl http://localhost:9200/_cat/indices?v
```

If no Suricata index exists:

- Verify Logstash output
- Verify Elasticsearch connectivity
- Generate new test traffic

---

# Kibana Issues

## Problem

Kibana cannot be accessed.

### Resolution

Verify the service.

```bash
sudo systemctl status kibana
```

Confirm that port **5601** is listening.

```bash
ss -tuln | grep 5601
```

Check firewall rules if remote access is required.

---

## Problem

"No results found" in Discover.

### Possible Causes

- Incorrect data view
- No indexed events
- Wrong time filter

### Resolution

Verify the data view:

```
suricata-*
```

Adjust the time range to include recent events.

Generate new alerts and refresh Discover.

---

# Telegram Issues

## Problem

No notification is received.

### Possible Causes

- Invalid Bot Token
- Incorrect Chat ID
- Internet connectivity issue

### Resolution

Verify the environment file.

```
configs/telegram.env.example
```

Ensure the bot has received at least one message from the administrator before testing.

---

## Problem

Python script returns an authentication error.

### Resolution

Generate a new Bot Token through **BotFather** and update the configuration.

---

# VMware Networking Issues

## Problem

Virtual machines cannot communicate.

### Possible Causes

- Incorrect virtual network
- Interface disconnected
- IP conflict

### Resolution

Verify VMware network assignments.

| Interface | VMware Network |
|-----------|----------------|
| WAN | VMnet0 |
| LAN | VMnet1 |

Confirm that all internal machines are connected to the LAN network.

---

# Detection Issues

## Problem

Attack traffic is generated but no alerts appear.

### Verify

- IDS enabled
- Correct interface selected
- Rules updated
- Matching signatures available
- Attack executed successfully

Use:

```bash
nmap -sS 192.168.100.10
```

or

```bash
hydra -l administrator -P passwords.txt ssh://192.168.100.10
```

---

# Performance Issues

## Problem

High CPU utilization.

### Possible Causes

- Intensive attack simulation
- Large rule set
- Limited virtual machine resources

### Resolution

Increase virtual machine resources or reduce simultaneous attack traffic during testing.

---

# Useful Commands

## Check Services

```bash
sudo systemctl status elasticsearch
sudo systemctl status logstash
sudo systemctl status kibana
```

---

## Restart Services

```bash
sudo systemctl restart elasticsearch
sudo systemctl restart logstash
sudo systemctl restart kibana
```

---

## View Logs

```bash
sudo journalctl -u elasticsearch
sudo journalctl -u logstash
sudo journalctl -u kibana
```

---

## Verify Elasticsearch

```bash
curl http://localhost:9200
```

---

## Test Connectivity

```bash
ping 192.168.100.1
ping 192.168.100.20
```

---

# Conclusion

Most deployment issues can be resolved by verifying network connectivity, service status, configuration files, and log outputs.

Following a structured troubleshooting methodology simplifies the identification of configuration errors and helps restore the platform to a fully operational state while minimizing downtime.
