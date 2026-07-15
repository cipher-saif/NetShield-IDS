<div align="center">

```
    _   ___________________ __  ______________    ____ 
   / | / / ____/_  __/ ___// / / /  _/ ____/ /   / __ \
  /  |/ / __/   / /  \__ \/ /_/ // // __/ / /   / / / /
 / /|  / /___  / /  ___/ / __  // // /___/ /___/ /_/ / 
/_/ |_/_____/ /_/  /____/_/ /_/___/_____/_____/_____/  
```

**Hybrid Intrusion Detection & Prevention System**

*Signature-based precision, fused with anomaly-based instinct.*

[![Python](https://img.shields.io/badge/Python-3.8+-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![Flask](https://img.shields.io/badge/Flask-Dashboard-111111?style=flat-square&logo=flask&logoColor=white)](https://flask.palletsprojects.com/)
[![Snort](https://img.shields.io/badge/Snort-Signature%20Engine-A6192E?style=flat-square&logo=snort&logoColor=white)](https://www.snort.org/)
[![License](https://img.shields.io/badge/License-MIT-2E7D32?style=flat-square)](LICENSE)

</div>

<br>

> NetShield watches network traffic the way a SOC analyst would — cross-checking known attack signatures against behavioral anomalies — then surfaces everything on one live dashboard.

<br>

## Contents

- [Overview](#overview)
- [Core Capabilities](#core-capabilities)
- [Architecture](#architecture)
- [Getting Started](#getting-started)
- [Usage](#usage)
- [Detection Engine](#detection-engine)
- [IPS Auto-Blocking](#ips-auto-blocking)
- [Sample Output](#sample-output)
- [Configuration](#configuration)
- [Troubleshooting](#troubleshooting)
- [Tech Stack](#tech-stack)
- [License](#license)

<br>

## Overview

**NetShield IDS/IPS** is a lightweight, lab-friendly intrusion detection and prevention system. It runs Snort's signature-based engine alongside a custom statistical anomaly detector, unifies both alert streams, and displays everything on a real-time web dashboard — with optional automatic IP blocking for high-severity threats.

Built for home labs, coursework, and SOC-style demos where a hybrid detection story is needed without standing up a full enterprise stack.

<br>

## Core Capabilities

| Capability | Description |
|---|---|
| PCAP Traffic Analysis | Parses offline network captures via Tshark |
| Snort Integration | Signature-based detection with 30+ custom rules |
| Anomaly Detection | Statistical thresholds for DoS/DDoS, ARP spoofing, malware beaconing, DNS tunneling |
| Unified Alert System | Merges Snort and anomaly alerts with severity classification |
| Real-Time Dashboard | Live alerts, charts, and attack timeline |
| Security Reports | Downloadable HTML reports with full analytics |
| IPS Auto-Blocking | Optional automatic firewall blocking for high-severity IPs |

<br>

## Architecture

```
netshield-ids/
├── backend/                    core detection engines
│   ├── pcap_parser.py          Tshark PCAP parsing
│   ├── snort_engine.py         Snort integration
│   ├── anomaly_detector.py     custom anomaly detection
│   ├── alert_manager.py        unified alert processing
│   ├── report_generator.py     HTML report generation
│   ├── ips_blocker.py          auto-blocking (Windows Firewall)
│   └── config.py               configuration & thresholds
├── dashboard/                  web interface
│   ├── app.py                  Flask web server
│   ├── static/                 CSS theme + live JS charts
│   └── templates/               dashboard HTML
├── snort_rules/
│   └── custom.rules            30+ Snort detection rules
├── sample_pcaps/                sample captures
├── data/                        alert storage & logs
├── main.py                     CLI entry point
└── requirements.txt
```

<br>

## Getting Started

**Prerequisites**

```
Python 3.8+
Snort        snort --version
Tshark       tshark --version   (comes with Wireshark)
```

**Install**

```powershell
# 1. Clone the repo
git clone https://github.com/Dazai022/Netshield-IDS.git
cd Netshield-IDS

# 2. Create a virtual environment (recommended)
python -m venv venv
.\venv\Scripts\Activate

# 3. Install dependencies
pip install -r requirements.txt
```

Point `backend/config.py` at your local binaries if they're not on PATH:

```python
SNORT_PATH  = "snort"      # or "C:\\Snort\\bin\\snort.exe"
TSHARK_PATH = "tshark"     # or "C:\\Program Files\\Wireshark\\tshark.exe"
```

<br>

## Usage

**Web Dashboard** *(recommended)*

```powershell
python main.py dashboard
```

Open **http://127.0.0.1:5000** — drag-and-drop PCAP upload, live severity-coded alert table, attack timeline, one-click report generation, IPS toggle.

**Command Line**

```powershell
python main.py analyze sample.pcap
python main.py analyze sample.pcap --report output_report.html
```

<br>

## Detection Engine

**Signature-Based — Snort**

30+ custom rules covering SQL injection, XSS, port scans, brute force, command injection, directory traversal, C2 traffic, and file-upload attacks. PCAP → Snort → parsed alerts → severity by Snort priority.

**Anomaly-Based — Custom**

| Threat | Trigger |
|---|---|
| High packet rate | >1000 pkts/sec from one IP |
| SYN flood | >80% SYN without ACK |
| DDoS | >50 unique sources targeting one host |
| ARP spoofing | One MAC claiming multiple IPs |
| HTTP beaconing | Periodic requests, ±5s interval |
| DNS tunneling | Domains >50 chars or >3.5 Shannon entropy |

**Alert Management**

Both streams are unified, classified High / Medium / Low, persisted to JSON, and pushed to the dashboard in real time.

Full breakdown in [`DETECTION_LOGIC.md`](DETECTION_LOGIC.md).

<br>

## IPS Auto-Blocking

Optional. Watches the alert stream and blocks source IPs for high-severity threats via Windows Firewall (`netsh advfirewall`), respecting a whitelist.

```python
# backend/config.py
AUTO_BLOCKING_ENABLED = True   # off by default
```

Requires administrator privileges — review alerts before enabling in production.

<br>

## Sample Output

```
============================================================
  NetShield IDS/IPS - Traffic Analysis
============================================================
Analyzing: sample.pcap

[1/4] Parsing PCAP file with Tshark...
      Parsed 15,432 packets
      Unique source IPs: 47

[2/4] Running Snort signature-based detection...
      Snort detected 12 threats

[3/4] Running custom anomaly detection...
      Anomaly detector found 5 suspicious patterns

[4/4] Generating results...
      Total alerts: 17   High: 8   Medium: 6   Low: 3
```

<br>

## Configuration

All thresholds live in `backend/config.py`:

```python
DOS_PACKET_RATE_THRESHOLD = 1000   # packets/sec
SYN_FLOOD_RATIO            = 0.8   # 80% SYN packets
DDOS_SOURCE_THRESHOLD      = 50    # unique sources
DNS_TUNNEL_LENGTH          = 50    # domain length
DNS_ENTROPY_THRESHOLD      = 3.5   # Shannon entropy
BEACON_INTERVAL_VARIANCE   = 5     # seconds

AUTO_BLOCKING_ENABLED = False
WHITELIST_IPS = ["127.0.0.1", "192.168.1.1"]
DASHBOARD_PORT = 5000
```

<br>

## Troubleshooting

| Issue | Fix |
|---|---|
| `Snort not found` | Install Snort or update `SNORT_PATH` in `config.py` |
| `Tshark not found` | Install Wireshark or update `TSHARK_PATH` |
| No alerts detected | Confirm PCAP has real attack traffic; lower thresholds for testing |
| IPS blocking fails | Run as Administrator; check Windows Firewall is on and IP isn't whitelisted |

<br>

## Tech Stack

Python 3.8+ · Snort · Tshark · Flask · HTML5 / CSS3 / JS · Chart.js · Windows Firewall (netsh)

<br>

## License

Released under the MIT License — see [`LICENSE`](LICENSE). Built for academic and educational use, free to fork, extend, and learn from.

<div align="center">

<br>

</div>
