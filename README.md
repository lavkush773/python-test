Haproxy Geofencing
Description

This project provides an automated and scalable solution for:

Updating GeoIP database (geoip.map) for HAProxy
Blocking traffic based on domain and country
Using regex-based rules for flexible geo-blocking
Running with Bash and AWK only (no external dependencies)

This setup is designed for production environments where domain-level geo restrictions are required.

Features
Automated GeoIP database updates
Backup support before every update
IPv4 optimized processing
Regex-based domain blocking
Scalable and clean configuration
Lightweight (no external libraries required)
Requirements
HAProxy (2.6 or later recommended)
Bash
AWK
curl
unzip
MaxMind account
Installation
Setup Working Directory
mkdir -p ~/geoip-updater/temp
cd ~/geoip-updater
Create Script
nano update_geoip.sh

Paste your script and save.

Set Permissions
chmod +x update_geoip.sh
MaxMind License Setup

To download the GeoLite2 database, you need a MaxMind license key.

Create License Key

Follow this guide:
https://www.youtube.com/watch?v=f8QsxwG8sLY

Configure in Script
nano update_geoip.sh

Add:

ACCOUNT_ID="Your Account ID"
LICENSE_KEY="Your MaxMind License Key"
Usage
Run Script Manually
./update_geoip.sh
Expected Output
geoip.map updated
HAProxy reloaded successfully
Automate with Cron
crontab -e

Example:

30 3 * * 3,6 /home/yourusername/geoip-updater/update_geoip.sh >> /home/yourusername/geoip-updater/update.log 2>&1
Workflow
Download latest GeoLite2 CSV
Extract archive
Backup existing geoip.map
Process IPv4 data using AWK
Generate new geoip.map
Validate HAProxy configuration
Reload HAProxy
Cleanup temporary files
Domain Geo-Blocking Setup
Required Files
/etc/haproxy/geoip.map
/etc/haproxy/domain_block.map
/etc/haproxy/haproxy.cfg
Step 1: Create Domain Block Map
sudo nano /etc/haproxy/domain_block.map

Format:

^domain\.com-(COUNTRY1|COUNTRY2)$ 1

Example:

^example\.com-(CN|IN|RU|US)$ 1
^test\.com-(IN|PK|BD|AF)$ 1
^mysite\.in-(US|GB|CN)$ 1
Step 2: Update HAProxy Configuration
frontend http_front
    bind *:80

    http-request set-var(req.country) src,map_ip(/etc/haproxy/geoip.map)

    http-request set-var-fmt(txn.host_country) "%[hdr(host),lower]-%[var(req.country)]"

    acl is_blocked var(txn.host_country),map_reg(/etc/haproxy/domain_block.map) -m found

    http-request deny deny_status 403 if is_blocked

    default_backend http_back
Step 3: Validate and Reload
sudo haproxy -c -f /etc/haproxy/haproxy.cfg
sudo systemctl reload haproxy
How It Works
HAProxy detects country from client IP
Combines domain and country (example: example.com-IN)
Matches against regex rules
If matched, request is denied with HTTP 403
Maintenance
Add New Country
(CN|IN) → (CN|IN|US)
Add New Domain
^newdomain\.com-(CN|RU)$ 1
Reload After Changes
sudo systemctl reload haproxy
Support
Create a merge request
Raise an issue in the repository
Project Status

Active and production-ready
