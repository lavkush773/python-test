# 🌍 HAProxy Geofencing & GeoIP Automated Updater

## 📖 Description

This project provides an **automated, scalable, and production-ready solution** for GeoIP-based traffic control in HAProxy.

It helps you:

* Update the **GeoIP database (`geoip.map`) automatically**
* Block traffic based on **domain + country**
* Use **regex-based rules** for flexible geo-blocking
* Run everything using **Bash + AWK only (no heavy dependencies)**

This setup is specifically designed for **production environments** where **domain-level geo restrictions** are required.

---

## 🚀 Features

* **Automated GeoIP Updates** – Fetches and updates GeoLite2 database automatically
* **Domain-Level Blocking** – Block countries per domain using regex
* **Backup Support** – Automatically backs up old `geoip.map` before update
* **IPv4 Optimized** – High-performance CIDR processing
* **Regex-Based Filtering** – Flexible and scalable blocking rules
* **Lightweight** – No external libraries required
* **Safe Reload** – Validates HAProxy config before reload

---

## 📋 Requirements

* **HAProxy** (v2.6 or later recommended)
* **Bash**
* **AWK**
* **curl**
* **unzip**
* **MaxMind Account** (GeoLite2 database)

---

## 🛠️ Installation

### 1️⃣ Setup Working Directory

```bash
mkdir -p ~/geoip-updater/temp
cd ~/geoip-updater
```

---

### 2️⃣ Create Update Script

```bash
nano update_geoip.sh
```

Paste your script inside and save it.

---

### 3️⃣ Set Executable Permission

```bash
chmod +x update_geoip.sh
```

---

## 🔑 MaxMind License Setup

To download the GeoLite2 database, you need a **MaxMind license key**.

### ▶️ Create License Key

Follow this guide:
https://www.youtube.com/watch?v=f8QsxwG8sLY

---

### ⚙️ Configure in Script

```bash
nano update_geoip.sh
```

Add:

```bash
ACCOUNT_ID="Your_Account_ID"
LICENSE_KEY="Your_MaxMind_License_Key"
```

These credentials allow the script to download the latest GeoLite2 database.

---

## 🚀 Usage

### ▶️ Run Manually

```bash
./update_geoip.sh
```

### ✅ Expected Output

* `geoip.map updated`
* `HAProxy reloaded successfully`

---

### ⏰ Automate with Cron

```bash
crontab -e
```

Example (runs every Wednesday & Saturday at 3:30 AM):

```bash
30 3 * * 3,6 /home/yourusername/geoip-updater/update_geoip.sh >> /home/yourusername/geoip-updater/update.log 2>&1
```

---

## ⚙️ Workflow

1. Download latest GeoLite2 CSV
2. Extract archive
3. Backup existing `geoip.map`
4. Process IPv4 data using AWK
5. Generate new `geoip.map`
6. Validate HAProxy configuration
7. Reload HAProxy
8. Cleanup temporary files

---

## 🛡️ Domain Geo-Blocking Setup

### 📁 Required Files

* `/etc/haproxy/geoip.map`
* `/etc/haproxy/domain_block.map`
* `/etc/haproxy/haproxy.cfg`

---

### 🔹 Step 1: Create Domain Block Map

```bash
sudo nano /etc/haproxy/domain_block.map
```

#### Format

```
^domain\.com-(COUNTRY1|COUNTRY2)$ 1
```

#### Example

```
^example\.com-(CN|IN|RU|US)$ 1
^test\.com-(IN|PK|BD|AF)$ 1
^mysite\.in-(US|GB|CN)$ 1
```

---

### 🔹 Step 2: Update HAProxy Configuration

```haproxy
frontend http_front
    bind *:80

    # Detect country from client IP
    http-request set-var(req.country) src,map_ip(/etc/haproxy/geoip.map)

    # Combine host + country
    http-request set-var-fmt(txn.host_country) "%[hdr(host),lower]-%[var(req.country)]"

    # Match against regex rules
    acl is_blocked var(txn.host_country),map_reg(/etc/haproxy/domain_block.map) -m found

    # Deny if matched
    http-request deny deny_status 403 if is_blocked

    default_backend http_back
```

---

### 🔹 Step 3: Validate & Reload

```bash
sudo haproxy -c -f /etc/haproxy/haproxy.cfg
sudo systemctl reload haproxy
```

---

## 🧠 How It Works

* HAProxy detects **country from client IP**
* Combines **domain + country** → `example.com-IN`
* Matches against **regex rules**
* If matched → request is **blocked (HTTP 403)**

---

## 🛠️ Maintenance

### ➕ Add New Country

```
(CN|IN) → (CN|IN|US)
```

---

### ➕ Add New Domain

```
^newdomain\.com-(CN|RU)$ 1
```

---

### 🔄 Apply Changes

```bash
sudo systemctl reload haproxy
```

---

## 📞 Support

For issues or improvements:

* Create a **merge request**
* Raise an **issue in the repository**

---

## 📄 Project Status

✅ **Active**
🚀 **Production Ready**
