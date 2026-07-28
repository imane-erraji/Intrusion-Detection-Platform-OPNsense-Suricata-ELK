# Performance Evaluation

This document presents the performance evaluation of the Network Intrusion Detection Platform developed using OPNsense, Suricata, and the Elastic Stack.

The objective is to assess the platform's behavior under normal operation and during simulated attack scenarios while identifying potential bottlenecks and opportunities for improvement.

---

# Table of Contents

- Overview
- Evaluation Objectives
- Test Environment
- Evaluation Metrics
- Test Scenarios
- Resource Utilization
- Detection Performance
- Dashboard Responsiveness
- Observations
- Limitations
- Recommendations
- Conclusion

---

# Overview

The platform was evaluated after the complete deployment of all components to verify that it could:

- Inspect network traffic
- Detect malicious activities
- Process security events
- Store alerts
- Display dashboards
- Send Telegram notifications

without service interruption.

---

# Evaluation Objectives

The evaluation focused on the following aspects:

- Platform stability
- Resource consumption
- Alert generation
- Event processing
- Dashboard responsiveness
- End-to-end detection workflow

---

# Test Environment

| Component | Specification |
|-----------|---------------|
| Hypervisor | VMware Workstation |
| Firewall | OPNsense 26.1.6 |
| IDS | Suricata |
| SIEM | Elasticsearch, Logstash, Kibana |
| Attacker | Kali Linux |
| Victim | Windows 10 |

---

# Evaluation Metrics

The following metrics were monitored:

| Metric | Description |
|---------|-------------|
| CPU Usage | Processor utilization |
| Memory Usage | RAM consumption |
| Disk Usage | Storage utilization |
| Alert Detection | Successful alert generation |
| Event Processing | Delivery of events to Elasticsearch |
| Dashboard Availability | Accessibility of Kibana |
| Notification Delay | Time required to send Telegram alerts |

---

# Test Scenario 1 — Normal Network Activity

## Objective

Evaluate platform behavior during regular network usage.

Activities included:

- Web browsing
- DNS resolution
- ICMP communication
- SSH connections

### Result

The platform remained stable and generated only expected events.

---

# Test Scenario 2 — Port Scanning

## Tool

Nmap

```bash
nmap -sS 192.168.100.10
```

### Expected Result

- Suricata detects scan
- Logstash receives alert
- Elasticsearch indexes event
- Kibana updates dashboard
- Telegram notification is generated

### Observation

The complete detection pipeline operated successfully.

---

# Test Scenario 3 — SSH Brute Force

Tool:

Hydra

```bash
hydra -l administrator -P passwords.txt ssh://192.168.100.10
```

### Observation

Repeated authentication attempts generated multiple Suricata alerts that were successfully processed and displayed in Kibana.

---

# Test Scenario 4 — ICMP Traffic

Command:

```bash
ping -f 192.168.100.10
```

### Observation

The platform maintained normal operation while logging ICMP events according to the configured detection rules.

---

# Resource Utilization

The following observations were made during testing.

| Component | Resource Usage |
|-----------|----------------|
| OPNsense | Low CPU and memory usage under laboratory traffic |
| Suricata | Increased CPU utilization during attack simulations |
| Elasticsearch | Highest memory consumer due to indexing operations |
| Logstash | Stable resource usage during log processing |
| Kibana | Responsive during dashboard visualization |

---

# Detection Performance

All attack scenarios completed the expected detection workflow.

| Test | Detected | Indexed | Visualized | Notification |
|------|----------|----------|-------------|--------------|
| Host Discovery | Yes | Yes | Yes | Optional |
| Port Scan | Yes | Yes | Yes | Yes |
| SSH Brute Force | Yes | Yes | Yes | Yes |
| ICMP Detection | Yes | Yes | Yes | Yes |
| Custom Rules | Yes | Yes | Yes | Yes |

---

# Dashboard Responsiveness

Kibana dashboards updated shortly after new events were indexed.

Observed capabilities:

- Real-time event visualization
- Search by signature
- Filtering by IP address
- Timeline analysis
- Protocol statistics

---

# Platform Stability

Throughout the evaluation:

- All virtual machines remained operational.
- Elasticsearch and Kibana remained available.
- Log forwarding functioned correctly.
- No critical service failures were observed after the final configuration.

---

# Limitations

The evaluation was conducted in a controlled virtual laboratory.

The results should be interpreted with the following limitations:

- Limited hardware resources
- Single-node Elasticsearch deployment
- Small number of virtual machines
- Simulated attack traffic only
- No high-volume production traffic

---

# Recommendations

Future evaluations may include:

- Larger network environments
- Multiple simultaneous attackers
- High-volume traffic generation
- Long-term monitoring
- Performance comparison with IPS mode enabled
- Evaluation using additional detection rules

---

# Conclusion

The performance evaluation demonstrates that the platform successfully achieved its intended objectives within the scope of the project.

The integration of OPNsense, Suricata, Logstash, Elasticsearch, Kibana, and Telegram provided a stable and effective monitoring environment capable of detecting, processing, and visualizing network security events in real time.

Although the evaluation was performed in a laboratory environment, the platform architecture can serve as a foundation for further research, experimentation, and future enhancements in network security monitoring.
