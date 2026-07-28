# Attack Simulations

This document describes the penetration testing scenarios used to validate the intrusion detection platform.

All attacks were executed inside an isolated VMware laboratory for educational and research purposes.

---

# Table of Contents

- Laboratory Overview
- Test Environment
- Detection Workflow
- Attack Scenario 1 - Network Discovery
- Attack Scenario 2 - Port Scanning
- Attack Scenario 3 - SSH Brute Force
- Attack Scenario 4 - ICMP Flood
- Attack Scenario 5 - Custom Rule Validation
- Event Analysis
- MITRE ATT&CK Mapping
- Detection Summary
- Lessons Learned

---

# Laboratory Overview

The platform was validated by simulating common reconnaissance and intrusion techniques from a Kali Linux attacker machine targeting a Windows 10 victim.

Every generated alert was expected to pass through the complete detection pipeline:

```

Kali

↓

OPNsense

↓

Suricata

↓

Logstash

↓

Elasticsearch

↓

Kibana

↓

Telegram

```

---

# Test Environment

| Machine | Role | IP Address |
|----------|------|------------|
| Kali Linux | Attacker | 192.168.100.30 |
| Windows 10 | Victim | 192.168.100.10 |
| OPNsense | Firewall + IDS | 192.168.100.1 |
| Ubuntu ELK | SIEM | 192.168.100.20 |

---

# Detection Workflow

Every attack follows the same processing pipeline.

```
Attack

↓

Network Traffic

↓

Suricata Detection

↓

EVE JSON

↓

Logstash

↓

Elasticsearch

↓

Kibana

↓

Telegram Notification
```

---

# Scenario 1 — Host Discovery

## Objective

Verify network visibility and confirm that ICMP traffic is monitored.

---

## Tool

```
ping
```

---

## Command

```bash
ping 192.168.100.10
```

---

## Expected Result

The victim responds successfully.

---

## Detection

If an ICMP detection rule is enabled, Suricata generates an alert.

---

## Evidence

Screenshot placeholder:

```
images/attacks/ping.png
```

---

# Scenario 2 — Port Scan

## Objective

Detect reconnaissance activity using Nmap.

---

## Tool

```
Nmap
```

---

## Command

```bash
nmap -sS 192.168.100.10
```

---

## Attack Description

A TCP SYN scan sends SYN packets to identify open ports without completing the TCP handshake.

This technique is frequently used during the reconnaissance phase of cyber attacks.

---

## Expected Detection

Suricata should generate alerts such as:

```
ET SCAN Nmap Scripting Engine User-Agent

ET SCAN Potential SSH Scan

ET SCAN Potential Port Scan
```

---

## Kibana Validation

Verify:

- Source IP
- Destination IP
- Signature
- Timestamp
- Protocol

---

## Telegram Alert

Example:

```
⚠️ Port Scan Detected

Source:
192.168.100.30

Target:
192.168.100.10

Priority:
High
```

---

## Screenshot

```
images/attacks/nmap-scan.png
```

---

# Scenario 3 — SSH Brute Force

## Objective

Detect password guessing attempts.

---

## Tool

```
Hydra
```

---

## Command

```bash
hydra -l administrator -P passwords.txt ssh://192.168.100.10
```

---

## Attack Description

Hydra performs multiple authentication attempts using a username and a password list.

---

## Expected Detection

Suricata should identify repeated SSH authentication attempts.

Typical signatures include:

```
ET SCAN SSH Brute Force

Potential SSH Login Attempt
```

---

## Validation

Confirm:

- Alert in Kibana
- Indexed event
- Telegram notification

---

## Screenshot

```
images/attacks/hydra.png
```

---

# Scenario 4 — ICMP Flood

## Objective

Validate detection of abnormal ICMP traffic.

---

## Tool

```
ping
```

---

## Command

Linux

```bash
ping -f 192.168.100.10
```

---

Windows

```cmd
ping -t 192.168.100.10
```

---

## Expected Detection

Large volumes of ICMP traffic should generate alerts depending on the active rule set.

---

## Screenshot

```
images/attacks/icmp-flood.png
```

---

# Scenario 5 — Custom Rule Validation

## Objective

Validate custom Suricata signatures.

---

## Rule

Example:

```text
alert icmp any any -> any any (
msg:"Custom ICMP Rule";
sid:1000001;
rev:1;
)
```

---

## Procedure

1. Enable the rule.
2. Restart Suricata.
3. Generate ICMP traffic.
4. Verify alert creation.

---

## Expected Result

Alert appears in:

- Suricata
- Elasticsearch
- Kibana
- Telegram

---

# Event Analysis

Each generated event should contain:

| Field | Description |
|---------|-------------|
| Timestamp | Event time |
| Source IP | Attacker |
| Destination IP | Victim |
| Signature | Triggered rule |
| Protocol | Network protocol |
| Severity | Alert priority |

---

# MITRE ATT&CK Mapping

| Attack | Technique | ATT&CK ID |
|----------|-----------|-----------|
| Host Discovery | Network Service Discovery | T1046 |
| Port Scan | Active Scanning | T1595 |
| SSH Brute Force | Password Guessing | T1110 |
| ICMP Flood | Network DoS | T1498 |

---

# Detection Summary

| Attack | Detected | Kibana | Telegram |
|----------|-----------|----------|-----------|
| Ping | Yes | Yes | Optional |
| Port Scan | Yes | Yes | Yes |
| SSH Brute Force | Yes | Yes | Yes |
| ICMP Flood | Yes | Yes | Yes |
| Custom Rule | Yes | Yes | Yes |

---

# Lessons Learned

The attack simulations demonstrate the effectiveness of integrating OPNsense, Suricata, and the Elastic Stack into a unified monitoring platform.

The platform successfully detected reconnaissance activities, authentication attacks, and custom signatures while providing centralized visibility and real-time notifications.

These experiments also highlight the importance of continuously updating detection rules, monitoring event quality, and validating alerts through realistic attack scenarios.

---

# References

- Suricata Documentation
- OPNsense Documentation
- Elastic Documentation
- MITRE ATT&CK Framework
- Nmap Reference Guide
- THC Hydra Documentation

---

# Disclaimer

All attacks described in this document were executed in a controlled virtual laboratory for educational purposes only. No production systems or third-party infrastructure were targeted.
