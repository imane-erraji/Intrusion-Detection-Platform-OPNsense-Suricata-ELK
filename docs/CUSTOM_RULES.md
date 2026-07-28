# Custom Suricata Rules

This document explains the custom Suricata detection rules developed for this project, how they work, how they were tested, and the methodology used to create and validate them.

The objective is to demonstrate how custom signatures can be used to detect network activities that are specific to an organization's environment.

---

# Table of Contents

- Overview
- Why Custom Rules?
- Rule Anatomy
- Rule Header
- Rule Options
- Rule Identification
- Rule Categories
- Implemented Rules
- Rule Testing
- Validation
- False Positives
- Rule Development Workflow
- Best Practices
- Future Improvements

---

# Overview

Suricata includes thousands of community-maintained detection signatures through Emerging Threats Open (ET Open). While these signatures provide broad detection coverage, they cannot identify organization-specific behaviors or custom attack scenarios.

To address this limitation, this project includes several custom detection rules tailored to the laboratory environment.

The rules are stored in:

```

rules/custom.rules

```

---

# Why Custom Rules?

Custom rules allow analysts to:

- Detect organization-specific attacks
- Monitor sensitive assets
- Generate custom alerts
- Detect internal reconnaissance
- Reduce dependence on community signatures
- Improve detection coverage

---

# Rule Anatomy

A Suricata rule consists of two main parts:

```
Header
↓

Options
```

Example:

```text
alert icmp any any -> any any (
msg:"Custom ICMP Detection";
sid:1000001;
rev:1;
)
```

---

# Rule Header

The header defines:

- Action
- Protocol
- Source
- Destination

General format:

```text
action protocol source_ip source_port -> destination_ip destination_port
```

Example:

```text
alert tcp any any -> any 22
```

Meaning:

Generate an alert whenever TCP traffic targets port 22 (SSH).

---

# Rule Options

Options define how the alert behaves.

Example:

```text
(
msg:"SSH Connection";

sid:1000002;

rev:1;

priority:2;
)
```

---

# Common Keywords

| Keyword | Description |
|----------|-------------|
| msg | Alert message |
| sid | Signature ID |
| rev | Rule revision |
| priority | Alert severity |
| classtype | Attack category |
| content | String matching |
| nocase | Ignore letter case |
| threshold | Limit alert frequency |

---

# Signature IDs (SID)

Each custom rule must use a unique Signature ID.

Recommended ranges:

| Range | Usage |
|--------|-------|
| 1–999999 | Reserved |
| 1000000–1999999 | Local custom rules |
| 2000000+ | Third-party rules |

Example:

```text
sid:1000005;
```

---

# Rule 1 — ICMP Detection

## Objective

Detect ICMP traffic generated during laboratory testing.

Rule:

```text
alert icmp any any -> any any (
msg:"Custom ICMP Detection";
sid:1000001;
rev:1;
priority:3;
)
```

Detection Trigger

```
ping
```

Test Command

```bash
ping 192.168.100.10
```

Expected Result

- Suricata alert generated
- Event stored in Elasticsearch
- Visible in Kibana
- Telegram notification (if enabled)

---

# Rule 2 — SSH Connection Detection

Objective

Monitor inbound SSH connections.

Rule

```text
alert tcp any any -> any 22 (
msg:"SSH Connection Attempt";
flow:to_server;
sid:1000002;
rev:1;
priority:2;
)
```

Trigger

SSH connection attempt.

Example

```bash
ssh user@192.168.100.10
```

---

# Rule 3 — Nmap SYN Scan Detection

Objective

Identify TCP SYN scans performed with Nmap.

Example Rule

```text
alert tcp any any -> any any (
flags:S;
msg:"Possible Nmap SYN Scan";
sid:1000003;
rev:1;
priority:2;
)
```

Example Command

```bash
nmap -sS 192.168.100.10
```

---

# Rule 4 — HTTP User-Agent Detection

Objective

Detect requests generated using curl.

Rule

```text
alert http any any -> any any (
msg:"Curl User-Agent Detected";
content:"curl";
http_user_agent;
sid:1000004;
rev:1;
)
```

Example

```bash
curl http://192.168.100.10
```

---

# Rule 5 — Laboratory Test Rule

Purpose

Verify that the complete detection pipeline is functioning correctly.

Rule

```text
alert ip any any -> any any (
msg:"Laboratory Test Rule";
sid:1000005;
rev:1;
priority:3;
)
```

---

# Rule Development Workflow

```
Define Detection Objective

↓

Create Rule

↓

Validate Syntax

↓

Reload Suricata

↓

Generate Traffic

↓

Verify Alert

↓

Investigate in Kibana

↓

Optimize Rule
```

---

# Testing Procedure

After modifying the rule file:

Restart Suricata.

Example:

```bash
sudo service suricata restart
```

or through OPNsense:

```
Services

↓

Intrusion Detection

↓

Administration

↓

Restart
```

---

# Validation Checklist

Each rule should be validated by confirming:

- Rule loads successfully
- No syntax errors
- Alert generated
- Event stored in Elasticsearch
- Dashboard updated
- Telegram notification received

---

# False Positives

Custom rules should be reviewed regularly to minimize unnecessary alerts.

Potential causes include:

- Broad IP matching
- Generic content patterns
- High-volume protocols
- Normal administrative activity

Where appropriate, rules should include:

- Specific source or destination networks
- Protocol restrictions
- Thresholds
- Flow direction
- Additional content matching

---

# Rule Optimization

Recommended practices:

- Keep rules focused on a single behavior.
- Use descriptive alert messages.
- Assign unique Signature IDs.
- Increase rule specificity whenever possible.
- Test rules before deployment.
- Document every modification.

---

# Best Practices

- Store custom rules in a separate file.
- Use consistent naming conventions.
- Version-control all rule changes.
- Review rules after Suricata updates.
- Remove obsolete signatures.
- Document the purpose of every rule.

---

# Directory Structure

```
rules/
│
├── custom.rules
├── icmp.rules
├── ssh.rules
├── web.rules
└── README.md
```

---

# Future Improvements

Future versions of the project may include:

- DNS tunneling detection
- SMB attack detection
- RDP monitoring
- SQL Injection signatures
- XSS detection
- Command injection detection
- Reverse shell detection
- Beaconing detection
- PowerShell detection
- Sigma-to-Suricata rule conversion

---

# Conclusion

Developing custom Suricata rules extends the capabilities of the intrusion detection platform beyond default community signatures.

By creating, testing, and validating organization-specific rules, analysts can improve detection accuracy, reduce blind spots, and adapt the monitoring platform to evolving threats.

The custom rules implemented in this project demonstrate a practical approach to detection engineering and highlight the flexibility of Suricata in modern network security environments.
