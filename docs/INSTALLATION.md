# Installation Guide

This guide explains how to deploy the complete Network Intrusion Detection Platform from scratch.

The deployment includes:

- OPNsense Firewall
- Suricata IDS
- Elasticsearch
- Logstash
- Kibana
- Telegram Notifications
- Kali Linux (Attacker)
- Windows 10 (Victim)

---

# Lab Topology

```
                    Internet
                        │
                        │
                VMnet0 (Bridged)
                        │
                +----------------+
                |   OPNsense     |
                | Firewall + IDS |
                +----------------+
                        │
                VMnet1 (LAN)
                        │
      ---------------------------------------
      │                 │                  │
      │                 │                  │
+-------------+   +-------------+   +-------------+
| ELK Server  |   | Windows 10  |   | Kali Linux  |
| Ubuntu      |   | Victim      |   | Attacker    |
+-------------+   +-------------+   +-------------+
```

---

# Hardware Requirements

Minimum recommended specifications.

| Resource | Value |
|----------|-------|
| CPU | 4 Cores |
| RAM | 16 GB |
| Storage | 80 GB SSD |
| Hypervisor | VMware Workstation 17 |
| Internet | Required |

---

# Software Versions

| Software | Version |
|-----------|----------|
| VMware Workstation | 17.x |
| OPNsense | 26.1.6 |
| Ubuntu Server | 24.04 LTS |
| Elasticsearch | 8.19.16 |
| Logstash | 8.19.16 |
| Kibana | 8.19.16 |
| Kali Linux | Rolling |
| Windows | Windows 10 |

---

# Virtual Machines

Create four virtual machines.

| Machine | RAM | CPU | Disk |
|----------|-----|-----|------|
| OPNsense | 2 GB | 2 | 20 GB |
| Ubuntu ELK | 6 GB | 2 | 50 GB |
| Kali Linux | 2 GB | 2 | 30 GB |
| Windows 10 | 4 GB | 2 | 40 GB |

---

# Network Configuration

## OPNsense

| Interface | Network |
|------------|----------|
| WAN | VMnet0 (Bridged) |
| LAN | VMnet1 (Host-Only) |

---

## Ubuntu ELK

```
IP Address : 192.168.100.20

Gateway : 192.168.100.1

DNS : 8.8.8.8
```

---

## Windows 10

```
IP Address : DHCP

Gateway : 192.168.100.1
```

---

## Kali Linux

```
IP Address : DHCP

Gateway : 192.168.100.1
```

---

# Step 1 — Install OPNsense

1. Create a new virtual machine.
2. Attach the OPNsense ISO.
3. Install using default settings.
4. Reboot.
5. Assign interfaces.

Expected result:

```
WAN → em0

LAN → em1
```

---

# Step 2 — Configure LAN

Navigate to

```
Interfaces

↓

LAN
```

Configure:

```
IPv4

192.168.100.1/24
```

Enable DHCP Server.

Range:

```
192.168.100.100

↓

192.168.100.200
```

Apply changes.

---

# Step 3 — Enable Internet Access

Navigate to

```
Firewall

↓

NAT

↓

Outbound
```

Use Automatic Outbound NAT.

Verify that Windows and Kali can access the Internet.

```
ping google.com
```

---

# Step 4 — Install Suricata

Navigate to

```
System

↓

Firmware

↓

Plugins
```

Install

```
os-suricata
```

After installation

```
Services

↓

Intrusion Detection
```

Enable:

- IDS

(Optional)

- IPS

---

# Step 5 — Download Detection Rules

Navigate to

```
Services

↓

Intrusion Detection

↓

Download
```

Enable

- Emerging Threats Open

Click

```
Download & Update Rules
```

---

# Step 6 — Configure Monitoring Interface

Select

```
LAN
```

Enable

```
Promiscuous Mode
```

Enable

```
Block Offenders (optional)
```

Save.

---

# Step 7 — Install Ubuntu Server

Install Ubuntu Server 24.04 LTS.

Update packages.

```bash
sudo apt update

sudo apt upgrade -y
```

---

# Step 8 — Install Elasticsearch

Download Elasticsearch.

Install package.

```bash
sudo dpkg -i elasticsearch-8.19.16-amd64.deb
```

Enable service.

```bash
sudo systemctl daemon-reload

sudo systemctl enable elasticsearch

sudo systemctl start elasticsearch
```

Verify.

```bash
curl http://localhost:9200
```

Expected output

```json
{
"name":"elk-server",
"cluster_name":"elasticsearch",
"version":"8.19.16"
}
```

---

# Step 9 — Install Kibana

```bash
sudo dpkg -i kibana-8.19.16-amd64.deb
```

Enable.

```bash
sudo systemctl enable kibana

sudo systemctl start kibana
```

Verify.

```
http://SERVER-IP:5601
```

---

# Step 10 — Install Logstash

```bash
sudo dpkg -i logstash-8.19.16-amd64.deb
```

Copy pipeline.

```
configs/logstash.conf
```

Restart.

```bash
sudo systemctl restart logstash
```

Verify.

```bash
sudo systemctl status logstash
```

Status should be

```
active (running)
```

---

# Step 11 — Configure Log Forwarding

On OPNsense

Navigate to

```
System

↓

Settings

↓

Logging
```

Configure

```
Remote Server

↓

Ubuntu ELK
```

Protocol

```
UDP
```

Port

```
514
```

Save.

---

# Step 12 — Configure Kibana

Open

```
Stack Management

↓

Data Views
```

Create

```
logstash-*
```

Timestamp field

```
@timestamp
```

---

# Step 13 — Configure Telegram

Create a Bot using BotFather.

Obtain

- Bot Token
- Chat ID

Update

```
configs/telegram.env.example
```

Run the notification script.

```bash
python3 telegram_alert.py
```

Expected message

```
Platform Successfully Connected
```

---

# Step 14 — Validate Deployment

Verify the following.

| Component | Expected Status |
|-----------|-----------------|
| OPNsense | Running |
| Suricata | Running |
| Elasticsearch | Running |
| Logstash | Running |
| Kibana | Running |
| Telegram | Operational |

---

# Functional Testing

Run the following tests.

## Ping

```bash
ping 192.168.100.20
```

Expected

```
Reachable
```

---

## Port Scan

```bash
nmap -sS 192.168.100.10
```

Expected

```
Suricata Alert Generated
```

---

## SSH Brute Force

```bash
hydra -l test -P passwords.txt ssh://192.168.100.10
```

Expected

```
Alert Visible in Kibana
```

---

# Verification Checklist

- OPNsense routes traffic correctly
- Internet access is available
- Suricata generates alerts
- Logstash receives logs
- Elasticsearch indexes events
- Kibana displays dashboards
- Telegram receives notifications

---

# Troubleshooting

## Elasticsearch does not start

```bash
sudo journalctl -u elasticsearch
```

Check:

- Memory allocation
- Java installation
- Disk space

---

## Kibana is unreachable

```bash
sudo systemctl status kibana
```

Verify port

```
5601
```

---

## No Suricata Alerts

Check

```
Services

↓

Intrusion Detection

↓

Alerts
```

Verify:

- Rules are downloaded
- Monitoring interface is enabled
- Traffic is flowing

---

## Logstash receives no logs

Check

```bash
sudo systemctl status logstash
```

Inspect

```bash
sudo tail -f /var/log/logstash/logstash-plain.log
```

Verify UDP port 514 is open.

---

# Next Step

Once the platform is operational, continue with:

- `docs/CONFIGURATION.md` for detailed component configuration.
- `docs/ATTACK_SIMULATIONS.md` to reproduce the penetration testing scenarios.
- `docs/DASHBOARDS.md` to build and customize Kibana visualizations.
