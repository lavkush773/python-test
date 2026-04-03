# HAProxy Geofencing & GeoIP Automated Updater

This project provides an automated and scalable solution for managing GeoIP-based traffic control in HAProxy. It allows you to update the MaxMind GeoIP database automatically and implement flexible geo-blocking rules based on domain names and countries.

## 🚀 Features

- **Automated Updates:** Automatically fetches and updates the GeoIP database (`geoip.map`).
- **Domain-Level Blocking:** Block specific countries for specific domains using regex-based rules.
- **Scalable & Lightweight:** Written in Bash and AWK; no heavy external dependencies required.
- **Reliable:** Includes backup support before every update and validates HAProxy configuration before reloading.
- **IPv4 Optimized:** High-performance processing of CIDR ranges.

## 📋 Requirements

- **HAProxy** (Version 2.6 or later recommended)
- **Bash Shell**
- **AWK**
- **curl** & **unzip**
- **MaxMind Account** (Free GeoLite2 database)

## 🛠️ Installation

### 1. Setup Working Directory
```
mkdir -p ~/geoip-updater/temp

cd ~/geoip-updater

2. Create the Update Script

Create a file named update_geoip.sh:


nano update_geoip.sh

Paste your update logic into this file and save it.

3. Set Executable Permissions


chmod +x update_geoip.sh

🔑 MaxMind License Setup

To download the GeoLite2 database, you need a MaxMind account and a license key.
Create License Key:
Sign up at MaxMind.com and generate a license key.
Configure the Script:
Open update_geoip.sh and add your credentials:


ACCOUNT_ID="Your_Account_ID"
LICENSE_KEY="Your_MaxMind_License_Key"
🚀 Usage
Manual Update
Run the script manually to test the setup:


./update_geoip.sh

Automation with Cron
To keep the database updated automatically, add a cron job:


crontab -e

Add the following line (runs every Wednesday and Saturday at 3:30 AM):


30 3 * * 3,6 /home/yourusername/geoip-updater/update_geoip.sh >> /home/yourusername/geoip-updater/update.log 2>&1

🛡️ Domain Geo-Blocking Configuration

Step 1: Create Domain Block Map

Create /etc/haproxy/domain_block.map. This file maps a domain-country combination to a block action (1).

Format: ^domain\.com-(COUNTRY_CODE1|COUNTRY_CODE2)$ 1

Example:


^example\.com-(CN|IN|RU|US)$ 1
^test\.com-(IN|PK|BD|AF)$ 1
^mysite\.in-(US|GB|CN)$ 1

Step 2: Update HAProxy Configuration (haproxy.cfg)

Add the following logic to your frontend:


frontend http_front
    bind *:80

    # 1. Look up the country code from the client IP
    http-request set-var(req.country) src,map_ip(/etc/haproxy/geoip.map)

    # 2. Create a combined string of "host-country"
    http-request set-var-fmt(txn.host_country) "%[hdr(host),lower]-%[var(req.country)]"

    # 3. Check if this combination exists in the block map using regex
    acl is_blocked var(txn.host_country),map_reg(/etc/haproxy/domain_block.map) -m found

    # 4. Deny request if blocked
    http-request deny deny_status 403 if is_blocked

    default_backend http_back
    
Step 3: Validate and Reload


sudo haproxy -c -f /etc/haproxy/haproxy.cfg
sudo systemctl reload haproxy

⚙️ Workflow

Download: Fetches the latest GeoLite2 CSV.
Process: AWK processes IPv4 data into HAProxy's .map format.
Backup: Saves the old geoip.map.
Verify: HAProxy checks the new config for errors.
Reload: HAProxy reloads without dropping connections.
Cleanup: Removes temporary files.

🛠️ Maintenance

Add Country: Edit domain_block.map and add to the regex: (CN|IN) → (CN|IN|US).
Add Domain: Add a new line in domain_block.map for the new domain.

Apply Changes: Run sudo systemctl reload haproxy.

📄 Project Status

Status: Active & Production Ready.
Support: Raise an issue in the repository for bugs or feature requests.
