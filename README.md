# Apache-Honeypot
A simple PHP/Apache Honeypot that blocks automated scanners on IP

Step 1: Upload all files to your web-root folder, usually /www/

Step 2: Modify the PHP file to add your preferences such as UNIQUEID, and add your own IP.

> UNIQUEID is required, so that scanners cannot guess the location of the honeypot!

Step 3:

```bash
# Install required packages
sudo apt update
sudo apt install fail2ban

# Enable Apache modules
sudo a2enmod rewrite

# Create log file
sudo touch /var/www/tmp/UNIQUEID-honeypot/UNIQUEID-honeypot.log
sudo chown www-data:www-data /var/www/tmp/UNIQUEID-honeypot/UNIQUEID-honeypot.log
```

Step 4: Next step is to create fail2ban filters for IP banning:

### `/etc/fail2ban/filter.d/honeypot.conf` (Fail2ban Filter)

> sudo nano /etc/fail2ban/filter.d/honeypot.conf

```conf
# Fail2Ban filter for PHP honeypot strikes

[Definition]

# Match lines where strike is 3 or higher
failregex = ^\[\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}\] IP: <HOST> \| Strike: ([3-9]|\d{2,}) \|

# Ignore lower strikes (1 or 2)
ignoreregex = Strike: [12] \|
```

---

In this file, be sure to edit:
1. Path to log file:  /var/www/tmp/UNIQUEID-honeypot/UNIQUEID-honeypot.log
2. IP: Your IP
3. Bantime, 30600 = ~24 hours.

> sudo nano /etc/fail2ban/jail.d/honeypot.conf
### `/etc/fail2ban/jail.d/honeypot.conf` (Fail2ban Jail)

```conf
[honeypot]
enabled = true
port = http,https
filter = honeypot
logpath = /var/www/tmp/UNIQUEID-honeypot/UNIQUEID-honeypot.log
maxretry = 1
findtime = 3600
bantime = 86400
action = iptables-multiport[name=honeypot, port="http,https", protocol=tcp]
ignoreip = 127.0.0.1/8 ::1 YOUR.IP.HERE
```

### 5. Restart services
```
sudo systemctl restart apache2
sudo systemctl restart fail2ban
```
### 6. Verify setup
```
sudo fail2ban-client status honeypot
```
