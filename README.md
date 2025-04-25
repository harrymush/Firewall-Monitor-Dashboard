
```markdown
# Firewall Monitor & Dashboard

A Python-based firewall log monitoring and analysis system for macOS, built to run continuously in the background as a system daemon. It collects, processes, and exposes firewall data for local or remote dashboard viewing.

---

## 📦 Project Structure

```bash
firewall_monitor/
├── firewall_monitor.py        # Collects and parses real-time pf firewall logs
├── analyze_pf_log.py         # Analyzes collected log data (top IPs, ports, etc.)
├── run_monitor.sh            # Shell script to run both Python scripts together
├── logs/                     # Log output directory (e.g. /var/log/firewall_monitor/)
└── README.md                 # This file
```

---

## 🔧 How It Works

### 1. **Monitor Script**
The core of the system consists of two Python scripts:

- `firewall_monitor.py`: Reads `pf` firewall logs in real time using `subprocess` and writes to a rotating log file.
- `analyze_pf_log.py`: Parses the logs to highlight top source IPs, targeted ports, and suspicious activity.

They are executed together using a wrapper script:

```bash
./run_monitor.sh
```

### 2. **Dashboard**
A minimal web dashboard was created and tested using a temporary Docker container that:

- Mounts the log files
- Reads and visualizes output from the analyzer
- Provides a basic front-end for real-time monitoring

### 3. **System Daemon (LaunchDaemon)**
To automate log monitoring:

- The scripts were moved to `/opt/firewall_monitor/` for persistence.
- A macOS LaunchDaemon `.plist` was created at:

```bash
/Library/LaunchDaemons/com.firewall.monitor.plist
```

This runs the monitor on startup and ensures it stays running in the background.

### 4. **Log Output**
Logs are stored at:

```bash
/var/log/firewall_monitor/
```

Including:

- `firewall.log`: Raw log output from pf
- `suspicious.log`: Highlighted suspicious activity
- `summary.log`: Top IPs, ports, and trends

---

## 🚀 Setup Summary

1. Clone or copy `firewall_monitor/` to `/opt/`
2. Make `run_monitor.sh` executable:
   ```bash
   chmod +x /opt/firewall_monitor/run_monitor.sh
   ```
3. Create the log directory:
   ```bash
   sudo mkdir -p /var/log/firewall_monitor
   sudo chown root:wheel /var/log/firewall_monitor
   ```
4. Install the `com.firewall.monitor.plist` to `/Library/LaunchDaemons/`
5. Load the service:
   ```bash
   sudo launchctl load /Library/LaunchDaemons/com.firewall.monitor.plist
   ```

---

## 📈 Next Steps

- **Dashboard Enhancements**:
  - Add interactive charts using Chart.js
  - Implement tabbed views (Summary, Suspicious, Raw)
  - Auto-refresh frontend data
  - Filter out common traffic (e.g. from local DNS server or port 53)

- **Data Source Linking**:
  - Update dashboard backend to use logs from `/var/log/firewall_monitor/`
  - Ensure paths in Docker container or web app reflect the final script location (`/opt/firewall_monitor/`)

- **Security and Access**:
  - Add basic auth to dashboard
  - Consider SSH tunneling or VPN for remote access

- **Log Management**:
  - Ensure auto-deletion of logs older than 7 days
  - Add archiving option if needed

---

## 🧠 Notes

- This project is designed for educational and monitoring purposes.
- Tested on macOS using the native `pf` firewall and LaunchDaemon system.

---

## 🛠️ Author

Harrymush – Cybersecurity, Python, and Network Monitoring enthusiast  
```

