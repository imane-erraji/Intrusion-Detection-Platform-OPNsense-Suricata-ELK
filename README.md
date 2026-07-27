# Network Intrusion Detection Platform using OPNsense, Suricata and ELK Stack

A practical cybersecurity project focused on designing and implementing a virtualized Network Intrusion Detection and Monitoring Platform capable of detecting, collecting, and visualizing network security events in real time.

The platform combines **OPNsense** as the firewall and gateway, **Suricata IDS** for intrusion detection, and the **Elastic Stack (Elasticsearch, Logstash, and Kibana)** for centralized log management and security event visualization. A Telegram notification system was also integrated to provide instant alerts whenever a critical security event is detected.

This project was developed as my Final Year Project (PFE) at the École Nationale des Sciences Appliquées during the 2025–2026 academic year.

---

## Project Objectives

The primary objective of this project is to build a complete open-source intrusion detection platform capable of:

- Monitoring network traffic in real time.
- Detecting suspicious activities using Suricata IDS.
- Centralizing security logs using the Elastic Stack.
- Visualizing security events through Kibana dashboards.
- Simulating common cyberattacks in a controlled laboratory environment.
- Sending automatic Telegram notifications when attacks are detected.

---

## Technologies Used

| Category | Technology |
|----------|------------|
| Firewall | OPNsense 26.1.6 |
| IDS | Suricata |
| SIEM | Elasticsearch 8.19.16 |
| Log Processing | Logstash 8.19.16 |
| Dashboard | Kibana 8.19.16 |
| Operating System | Ubuntu Server 24.04 LTS |
| Attacker Machine | Kali Linux |
| Victim Machine | Windows 10 |
| Virtualization | VMware Workstation |
| Notifications | Telegram Bot API |

---

## Platform Architecture

Insert the architecture diagram here.

```text

images/architecture/network-topology.png
```

The entire laboratory environment is deployed on VMware Workstation. OPNsense acts as the gateway between the internal network and the external network while Suricata continuously inspects network traffic. Security events are forwarded to Logstash, indexed by Elasticsearch, and visualized in Kibana dashboards. Critical alerts are automatically transmitted to Telegram to notify the administrator in real time.

---

## Virtual Infrastructure

The laboratory contains four virtual machines.

| Machine | Operating System | Role |
|----------|------------------|------|
| OPNsense | OPNsense 26.1.6 | Firewall + IDS |
| ELK Server | Ubuntu Server 24.04 | Elasticsearch, Logstash and Kibana |
| Kali Linux | Kali Rolling | Attack simulation |
| Windows 10 | Windows 10 | Victim machine |

---

## Features

- OPNsense firewall configuration
- Suricata Intrusion Detection System
- Custom Suricata detection rules
- Centralized log collection
- Elasticsearch indexing
- Logstash processing pipeline
- Kibana dashboards
- Attack simulation using Nmap
- SSH brute-force simulation using Hydra
- ICMP detection
- Automatic Telegram alerts
- Real-time monitoring

---

## Repository Structure

```text
Intrusion-Detection-Platform/
│
├── configs/
├── diagrams/
├── docs/
├── images/
├── rules/
├── scripts/
├── README.md
└── LICENSE
```

---

## Documentation

The complete implementation process is available inside:

```
docs/PFE_Report.pdf
```

Additional installation instructions are available in:

```
docs/INSTALLATION.md
```

---

## License

This project is released under the MIT License.

---

## Author

**Imane Erraji**

Engineering Student – Network and Systems Security

École Nationale des Sciences Appliquées

Morocco
