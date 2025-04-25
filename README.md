```markdown
# Firewall Monitor & Dashboard for macOS

A custom-built firewall monitoring solution for macOS that continuously captures, analyzes, and visualizes `pf` (Packet Filter) logs. Designed to run as a background daemon and power a real-time dashboard with actionable insights into incoming and outgoing network traffic.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Script Breakdown](#script-breakdown)
- [System Daemon Setup](#system-daemon-setup)
- [Dashboard](#dashboard)
- [Log Output](#log-output)
- [Troubleshooting & Challenges](#troubleshooting--challenges)
- [Next Steps](#next-steps)

---

## Overview

This project was created to provide real-time visibility into macOS's `pf` firewall activity, using a lightweight Python-based backend and a simple web dashboard frontend. The system is designed to be modular, extensible, and robust — surviving reboots and running independently without user interaction.

It’s ideal for:

- Home lab monitoring
- Self-hosted firewall analysis
- Visualizing incoming/outgoing connection attempts
- Learning firewall log parsing and threat analysis

---

## Features

- Real-time log capture using `tcpdump` from the `pf` firewall
- Automatic filtering and parsing of suspicious traffic
- Top source IP and port analysis
- JSON-friendly log outputs for front-end consumption
- LaunchDaemon support for always-on background monitoring
- Lightweight dashboard (HTML + JS) with log summaries
- Docker-friendly environment for isolated dashboard testing

---

## 🧱 Architecture

```
           ┌────────────────────────────────┐
           │      pf firewall (macOS)       │
           └────────────┬───────────────────┘
                        │ (real-time logs)
                        ▼
           ┌────────────────────────────────┐
           │   firewall_monitor.py          │
           │  - Runs tcpdump on pflog0      │
           │  - Writes to raw log file      │
           └────────────┬───────────────────┘
                        │ (log parsing)
                        ▼
           ┌────────────────────────────────┐
           │   analyze_pf_log.py            │
           │  - Summarizes IPs and ports    │
           │  - Highlights suspicious data  │
           └────────────┬───────────────────┘
                        │
        ┌───────────────┴───────────────┐
        ▼                               ▼
 /var/log/firewall_monitor/      Dashboard Frontend
 (raw, suspicious, summary)         (charts & tabs)
```

---

## Script Breakdown

### `firewall_monitor.py`

- Launches `tcpdump -n -ttt -i pflog0` via `subprocess`
- Filters and parses output in real time
- Stores data in `firewall.log` using log rotation
- Ensures long-running process stability

### `analyze_pf_log.py`

- Reads the `firewall.log`
- Extracts:
  - Most frequent **source IPs**
  - Most targeted **destination ports**
  - Suspicious patterns (e.g. repetitive access attempts)
- Outputs:
  - `summary.log` (for visualization)
  - `suspicious.log` (IP flags, unknown ports, etc.)

### `run_monitor.sh`

A shell script that runs both Python scripts in background mode.

```bash
#!/bin/bash
/opt/firewall_monitor/firewall_monitor.py &
sleep 2
/opt/firewall_monitor/analyze_pf_log.py &
```

---

## System Daemon Setup

To ensure the monitor runs on reboot and stays persistent, a **macOS LaunchDaemon** plist was created:

### Location

```bash
/Library/LaunchDaemons/com.firewall.monitor.plist
```

### Purpose

- Executes `run_monitor.sh` on system boot
- Logs output to a log directory
- Requires root/sudo permissions to install and manage

### Install Process

```bash
sudo cp com.firewall.monitor.plist /Library/LaunchDaemons/
sudo launchctl load /Library/LaunchDaemons/com.firewall.monitor.plist
```

Make sure the script is executable:

```bash
chmod +x /opt/firewall_monitor/run_monitor.sh
```

And create the log output folder:

```bash
sudo mkdir -p /var/log/firewall_monitor
sudo chown root:wheel /var/log/firewall_monitor
```

---

## Dashboard

### Functionality

A minimal front-end dashboard was developed to:

- Display summaries (`summary.log`)
- Show suspicious connections (`suspicious.log`)
- Allow filtering and tab-based navigation
- Enable auto-refresh to visualize live data

### Stack

- HTML + JavaScript frontend
- Log data fetched from a mounted directory (e.g. via Docker volume)
- Tested in a lightweight Docker container for portability

```bash
docker run -v /var/log/firewall_monitor:/app/logs -p 8080:80 firewall-dashboard
```

### Planned Tabs

- **Summary View**: Top IPs, ports, protocols
- **Suspicious View**: Filtered alerts
- **Raw View**: Complete raw logs (optional)

---

## Log Output

All logs are written to:

```bash
/var/log/firewall_monitor/
```

- `firewall.log`: Raw unprocessed logs
- `summary.log`: Top source IPs and destination ports
- `suspicious.log`: Highlighted entries based on heuristics (e.g. port scanning, unknown services)

All files are rotated and kept clean. Future updates may include auto-deletion of files older than 7 days.

---

## 🛠 Troubleshooting & Challenges

### Real-time Log Monitoring

- `pf` logs aren’t available through normal syslog on macOS.
- Required use of `tcpdump -i pflog0`, which doesn’t buffer well, so we implemented a `subprocess` line reader with error handling and timeouts.

### ⚙️ LaunchDaemon Behavior

- Needed to make sure script paths were **absolute**, not relative.
- The daemon failed silently without correct permissions on `/opt` and `/var/log`.
- Made sure the log directory was created with proper ownership (`root:wheel`).

### 🧪 Dashboard Testing

- Initially ran dashboard using a local web server.
- Later, moved it into a Docker container for portability and simplicity.
- Updated script paths to reflect the final install directory (`/opt/firewall_monitor/`).

---

## Next Steps

- [ ] **Dashboard Enhancements**
  - Chart.js visualizations
  - Tabbed interface: Raw | Suspicious | Summary
  - Auto-refresh using JS
  - Filtering: Exclude internal DNS (e.g. 192.168.1.74:53)

- [ ] **Log Management**
  - Auto-delete files older than 7 days
  - Zip/archive old logs for optional review

- [ ] **Security & Access**
  - Add simple HTTP auth for dashboard
  - Expose dashboard over SSH tunnel or VPN (e.g. Twingate)

- [ ] **Codebase Refactor**
  - Convert shell script logic into a Python controller
  - Unit testing for parsing logic

Author - Harrymush
