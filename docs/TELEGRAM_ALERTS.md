# Telegram Alert Integration

This document describes the implementation of the Telegram notification system used in the Network Intrusion Detection Platform.

The notification service enables administrators to receive real-time alerts whenever Suricata detects suspicious network activity, allowing for faster awareness and incident response without continuously monitoring the Kibana dashboard.

---

# Table of Contents

- Overview
- Objectives
- Notification Workflow
- System Architecture
- Requirements
- Telegram Bot Creation
- Bot Configuration
- Chat ID Retrieval
- Python Notification Script
- Alert Format
- Testing
- Troubleshooting
- Security Recommendations
- Future Improvements

---

# Overview

While Kibana provides powerful visualization and investigation capabilities, it requires an analyst to actively monitor the dashboard.

To improve operational awareness, this project integrates the Telegram Bot API to automatically send notifications whenever critical security events are detected.

This integration provides a lightweight and platform-independent alerting mechanism that works on desktop and mobile devices.

---

# Objectives

The Telegram integration aims to:

- Notify administrators immediately after a security event is detected
- Reduce the delay between detection and awareness
- Improve incident response time
- Provide remote visibility into the laboratory environment
- Complement Kibana dashboards with push notifications

---

# Notification Workflow

The notification process follows the sequence below.

```
Network Traffic

        │

        ▼

Suricata IDS

        │

Generate Alert

        ▼

Logstash

        │

Process Event

        ▼

Python Alert Script

        │

Telegram Bot API

        ▼

Telegram Chat

        │

Administrator
```

---

# Components

| Component | Purpose |
|-----------|---------|
| Suricata | Generates security alerts |
| Logstash | Processes event data |
| Python Script | Formats and sends notifications |
| Telegram Bot API | Delivers messages |
| Administrator | Receives alerts |

---

# Requirements

Before enabling Telegram notifications, ensure that the following are available:

- Telegram account
- Internet connectivity
- Telegram Bot Token
- Telegram Chat ID
- Python 3

---

# Creating a Telegram Bot

1. Open Telegram.
2. Search for **@BotFather**.
3. Start a conversation.
4. Execute the following command:

```
/newbot
```

5. Choose a bot name.
6. Choose a unique username ending with `bot`.

Example:

```
IntrusionDetectionBot
```

Bot username:

```
intrusion_detection_platform_bot
```

BotFather returns an API token similar to:

```text
123456789:AAEXAMPLE_TOKEN_REPLACE_WITH_YOURS
```

Store this token securely.

---

# Retrieving the Chat ID

Start a conversation with the newly created bot.

Send any message, for example:

```
Hello
```

Retrieve the Chat ID using the Telegram Bot API.

The returned response contains:

```json
{
  "message": {
    "chat": {
      "id": 123456789
    }
  }
}
```

Save the Chat ID for the notification script.

---

# Environment Configuration

Store sensitive values in an environment file.

Example:

```text
BOT_TOKEN=YOUR_BOT_TOKEN
CHAT_ID=YOUR_CHAT_ID
```

Never commit real tokens to GitHub.

Use the template file:

```
configs/telegram.env.example
```

---

# Python Notification Script

The notification script performs the following tasks:

1. Receive the alert information.
2. Format the message.
3. Authenticate with the Telegram Bot API.
4. Send the notification.
5. Log success or failure.

Suggested location:

```
scripts/telegram_alert.py
```

---

# Message Structure

Each notification should include relevant information for quick analysis.

Example:

```text
🚨 Intrusion Detection Alert

Rule:
ET SCAN Potential Nmap Scan

Severity:
High

Source IP:
192.168.100.30

Destination IP:
192.168.100.10

Protocol:
TCP

Timestamp:
2026-05-18 14:21:35
```

This format provides enough information for an administrator to understand the event without opening Kibana immediately.

---

# Alert Trigger

Notifications should be sent only for events that meet predefined criteria.

Examples:

- High severity alerts
- Port scanning
- SSH brute-force attempts
- Custom Suricata rules
- ICMP flood detection

Sending notifications for every event may generate excessive noise.

---

# Testing the Integration

Generate network activity using one of the attack simulations.

Example:

```bash
nmap -sS 192.168.100.10
```

Verify the following:

- Suricata generates an alert.
- Logstash processes the event.
- Elasticsearch indexes the document.
- Kibana displays the alert.
- Telegram receives the notification.

---

# Expected Output

Example notification:

```text
🚨 Intrusion Detection Alert

Signature:
Possible Nmap SYN Scan

Priority:
High

Source:
192.168.100.30

Target:
192.168.100.10

Protocol:
TCP

Status:
Alert Successfully Processed
```

---

# Error Handling

The notification script should detect and report common issues.

Examples:

| Issue | Possible Cause |
|--------|----------------|
| Notification not received | Invalid Bot Token |
| Unauthorized | Incorrect API credentials |
| Chat not found | Incorrect Chat ID |
| Connection failed | Internet unavailable |
| Timeout | Telegram API temporarily unavailable |

Errors should be written to the application log for troubleshooting.

---

# Security Recommendations

To improve security:

- Do not hard-code API tokens.
- Store credentials in environment variables.
- Exclude sensitive files using `.gitignore`.
- Rotate compromised tokens immediately.
- Restrict bot usage to trusted administrators.
- Avoid including sensitive information such as passwords or confidential data in notifications.

---

# Future Improvements

Potential enhancements include:

- Multiple notification recipients
- Alert severity filtering
- HTML or Markdown message formatting
- Interactive buttons
- Daily summary reports
- Integration with Microsoft Teams
- Slack notifications
- Discord notifications
- Email alerts
- Webhook-based notification system

---

# Conclusion

The Telegram integration extends the capabilities of the intrusion detection platform by providing immediate and accessible notifications whenever suspicious network activity is detected.

Combined with OPNsense, Suricata, and the Elastic Stack, the notification service completes the monitoring workflow by ensuring that critical events reach the administrator in near real time.

This feature improves operational awareness and demonstrates how open-source technologies can be combined to build a practical and responsive network security monitoring solution.
