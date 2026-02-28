# 🔍 Nmap Port Scan Detection using ELK Stack

![Security](https://img.shields.io/badge/Security-Blue%20Team-blue)
![ELK](https://img.shields.io/badge/Stack-ELK%208.x-005571?logo=elastic)
![Platform](https://img.shields.io/badge/Platform-Ubuntu%20%7C%20Kali%20Linux-orange)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

---

## 📌 Project Overview

This project demonstrates a **real-world SOC-level detection pipeline** that identifies Nmap port scanning activity using the **ELK Stack** (Elasticsearch, Logstash/Filebeat, Kibana).

The lab simulates an attacker performing network reconnaissance from **Kali Linux**, while the **Ubuntu Server** captures, indexes, and alerts on suspicious scanning behavior using UFW firewall logs and Kibana threshold-based rules.

> ⚡ Built as a hands-on Blue Team exercise to understand log ingestion, detection engineering, and SIEM alerting fundamentals.

---

## 🏗️ Lab Architecture

```
┌─────────────────────┐              ┌──────────────────────────────────┐
│   Kali Linux VM     │              │        Ubuntu Server VM          │
│   (Attacker)        │              │        (Victim / Monitor)        │
│                     │   Nmap Scan  │                                  │
│  nmap -sS -p 1-500  │─────────────▶│  UFW Firewall (blocks traffic)   │
│  192.168.10.5       │              │        ↓                         │
│                     │              │  Filebeat (collects /var/log/ufw)│
└─────────────────────┘              │        ↓                         │
                                     │  Elasticsearch (indexes logs)    │
                                     │        ↓                         │
                                     │  Kibana (detects + visualizes)   │
                                     └──────────────────────────────────┘
```

**Detection Flow:**
```
Nmap Scan → UFW Blocks Traffic → Filebeat Collects Logs →
Elasticsearch Indexes → Kibana Rule Triggers → Alert Generated
```

---

## 🛠️ Tech Stack

| Component | Tool | Version |
|-----------|------|---------|
| SIEM | Elasticsearch | 7.17.20 |
| Dashboard | Kibana | 7.17.20 |
| Log Shipper | Filebeat | 7.17.20 |
| Firewall | UFW (Uncomplicated Firewall) | - |
| Attacker | Nmap | 7.98 |
| Attacker OS | Kali Linux | 2024+ |
| Victim OS | Ubuntu Server | 22.04 LTS |
| Network | VirtualBox Host-Only Adapter | - |

---

## ⚙️ Setup & Configuration

### 1️⃣ Network Setup (VirtualBox)

Both VMs connected via **Host-Only Network**:

```
Kali Linux  →  192.168.56.101  (Attacker)
Ubuntu      →  192.168.56.102  (Victim / Monitor)
```

### 2️⃣ UFW Firewall Configuration (Ubuntu)

UFW configured to log all blocked inbound traffic:

```bash
sudo ufw enable
sudo ufw logging on
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw status verbose
```

Logs are written to:
```
/var/log/ufw.log
```

### 3️⃣ Filebeat Configuration

Filebeat monitors UFW logs and ships them to Elasticsearch:

```yaml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/ufw.log

output.elasticsearch:
  hosts: ["localhost:9200"]
```

Data is indexed into: `filebeat-*`

Verify logs in Kibana Discover using:
```
message:"UFW BLOCK"
```

### 4️⃣ Kibana Detection Rule

**Rule Type:** Elasticsearch Query Rule

**Index:** `filebeat-*`

**Time Field:** `@timestamp`

**Query (DSL):**
```json
{
  "query": {
    "match_phrase": {
      "message": "UFW BLOCK"
    }
  }
}
```

**Threshold Condition:**
- Trigger when count is **above 5**
- Within **1 minute window**
- Group alerts by `source.ip`

---

## 🧪 Attack Simulation

From **Kali Linux**, execute the following Nmap scan:

```bash
# Basic SYN scan (100 ports)
sudo nmap -sS 192.168.56.102 -p 1-100

# Larger scan to trigger HIGH severity alert
sudo nmap -sS 192.168.56.102 -p 1-500

# Aggressive scan
sudo nmap -A 192.168.56.102
```

---

## 📊 Kibana Visualization

Created using **Kibana Lens**:

| Setting | Value |
|---------|-------|
| Chart Type | Vertical Stacked Bar |
| X-Axis | `@timestamp` (per minute) |
| Y-Axis | Unique count of `destination.port` |
| Breakdown | `source.ip` (Top values) |

This visualization reveals:
- One IP hitting many different ports rapidly
- Clear port scanning behavior pattern
- Timeline of attack activity

---

## ✅ Detection Results

| Field | Value |
|-------|-------|
| Alert Name | Nmap Port Scan Detection |
| Rule Type | Elasticsearch Query |
| Status | ✅ Active |
| Alert Triggered | query matched |
| Detection Time | 28 Feb 2026 @ 18:49:33 |
| Duration | 00:00:04 |

**Alert triggered successfully** — detection pipeline confirmed working end-to-end.

---

## 🧠 Detection Logic Explained

Port scanning behavior has these characteristics that we detect:

```
✔ One source IP
✔ Rapid connection attempts in short time window
✔ Multiple different destination ports
✔ UFW blocks appear in bursts
```

The Kibana rule detects an **abnormal frequency of blocked connection attempts** from a single IP, which is the signature pattern of network reconnaissance.

This simulates basic **SOC Tier 1 detection logic** for identifying reconnaissance activity.

---

## 🖼️ Screenshots

| Screenshot | Description |
|------------|-------------|
| `screenshots/alert_triggered.png` | Kibana alert — status Active, query matched |
| `screenshots/rule_ok.png` | Rule monitoring screen — status OK |
| `screenshots/visualization.png` | Lens chart showing ports scanned per IP over time |
| `screenshots/rule_creation.png` | Rule creation with filebeat-* index and @timestamp |

---

## 🎯 Skills Demonstrated

- ✅ Log ingestion with Filebeat
- ✅ UFW firewall log monitoring
- ✅ Elasticsearch querying (DSL)
- ✅ KQL filtering in Kibana Discover
- ✅ Threshold-based alerting and rule creation
- ✅ Kibana Lens visualization
- ✅ Detection engineering fundamentals
- ✅ Blue Team / SOC workflow simulation
- ✅ Virtual lab setup (VirtualBox networking)

---

## 🚀 Outcome

Successfully built a **working SIEM detection pipeline** capable of identifying Nmap port scanning activity in a controlled lab environment.

This project demonstrates practical understanding of:
- Network reconnaissance detection
- Firewall log analysis
- Alert engineering in Kibana
- ELK stack deployment and troubleshooting
- Real-world Blue Team workflows

---

## 📂 Future Improvements

- [ ] Add email / Slack connector for real-time alert notifications
- [ ] Create a full dashboard combining scan timeline + alert history
- [ ] Implement anomaly-based detection using Kibana ML jobs
- [ ] Correlate port scans with failed SSH login attempts
- [ ] Add GeoIP enrichment to visualize attacker locations on a map
- [ ] Integrate Suricata IDS for deeper packet-level detection

---

## 📁 Project Structure

```
nmap-elk-detection/
│
├── filebeat/
│   └── filebeat.yml           # Filebeat configuration
│
├── kibana/
│   └── detection_rule.json    # Exported Kibana alert rule
│
├── screenshots/
│   ├── alert_triggered.png    # Alert active screenshot
│   ├── rule_ok.png            # Rule status OK
│   ├── visualization.png      # Lens chart
│   └── rule_creation.png      # Rule setup
│
└── README.md
```

---

## 👤 Author

**Your Name**
- 🔗 GitHub: [@yourusername](https://github.com/yourusername)
- 💼 LinkedIn: [linkedin.com/in/yourprofile](https://linkedin.com/in/yourprofile)

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).

---

> 💡 *This project was built for educational purposes as part of a cybersecurity home lab setup.*
