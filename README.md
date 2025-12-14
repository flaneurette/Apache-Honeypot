# Apache-Honeypot
A simple PHP/Apache Honeypot that blocks automated scanners on IP

Step 1: Upload all files to your web-root folder, usually /www/
> 1.1: Rename honeypot-strike.php to something unique.

> 1.2: Edit the .htaccess in the root, and rename: honeypot-strike.php to something unique.

Step 2: Modify the PHP file to add your preferences such as UNIQUE_KEY, and add your own IP.

> UNIQUE_KEY is required, so that scanners cannot guess the location of the honeypot!

Step 3:

```bash
# Install required packages
sudo apt update
sudo apt install fail2ban

# Enable Apache modules
sudo a2enmod rewrite

# Create log file
sudo touch /var/www/tmp/UNIQUE_KEY-honeypot/UNIQUE_KEY-honeypot.log
sudo chown www-data:www-data /var/www/tmp/UNIQUE_KEY-honeypot/UNIQUE_KEY-honeypot.log
sudo chmod 0700 -R /var/www/tmp/UNIQUE_KEY-honeypot/
```

Step 4: Next step is to create fail2ban filters for IP banning:

### `/etc/fail2ban/filter.d/honeypot.conf` (Fail2ban Filter)

> sudo nano /etc/fail2ban/filter.d/honeypot.conf

```conf
# Fail2Ban filter for PHP honeypot strikes

[Definition]
failregex = ^ IP: <HOST> \| Strike: (?:[3-9]|\d{2,}) \|
ignoreregex = 
datepattern = {^LN-BEG}\[%%Y-%%m-%%d %%H:%%M:%%S\]
```

---

In this file, be sure to edit:
1. Path to log file:  /var/www/tmp/UNIQUE_KEY-honeypot/UNIQUE_KEY-honeypot.log
2. IP: Your IP
3. Bantime, 86400 = ~24 hours.

> sudo nano /etc/fail2ban/jail.d/honeypot.conf
### `/etc/fail2ban/jail.d/honeypot.conf` (Fail2ban Jail)

```conf
[honeypot]
enabled = true
port = http,https
filter = honeypot
logpath = /var/www/tmp/UNIQUE_KEY-honeypot/UNIQUE_KEY-honeypot.log
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
