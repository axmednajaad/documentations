# Frappe/ERPNext Version 15 Installation Guide for Ubuntu 22.04 LTS

A complete guide to install Frappe/ERPNext version 15 in Ubuntu 22.04 LTS with production setup.

## Pre-requisites
- Python 3.11+
- Node.js 18+
- Redis 5 (caching and real time updates)
- MariaDB 10.3.x / Postgres 9.5.x (to run database driven apps)
- yarn 1.12+ (js dependency manager)
- pip 20+ (py dependency manager)
- wkhtmltopdf (version 0.12.5 with patched qt) (for pdf generation)
- cron (bench's scheduled jobs: automated certificate renewal, scheduled backups)
- NGINX (proxying multitenant sites in production)

## System Preparation

### Step 1: Update System
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl software-properties-common -y
```

### Step 2: Install Python 3.11
```bash
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt update
sudo apt install python3.11 python3.11-venv python3.11-dev python3.11-full -y
python3.11 --version
```

### Step 3: Install Git
```bash
sudo apt-get install git -y
```

### Step 4: Install Python Development Tools
```bash
sudo apt-get install python3-dev python3-setuptools python3-pip -y
```

## Database Installation

### Step 5: Install MariaDB
```bash
sudo apt install mariadb-server -y
sudo mysql_secure_installation
```

During MariaDB secure installation:
- Enter current password for root: (PRESS ENTER)
- Switch to unix_socket authentication: Y
- Change the root password: Y (set a secure password)
- Remove anonymous users: Y
- Disallow root login remotely: Y
- Remove test database and access to it: Y
- Reload privilege tables: Y

### Step 6: Install MySQL Development Files
```bash
sudo apt-get install libmysqlclient-dev -y
```

### Step 7: Configure MariaDB for Frappe
```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Add the following configuration:
```ini
[server]
user = mysql
pid-file = /run/mysqld/mysqld.pid
socket = /run/mysqld/mysqld.sock
basedir = /usr
datadir = /var/lib/mysql
tmpdir = /tmp
lc-messages-dir = /usr/share/mysql
bind-address = 127.0.0.1
query_cache_size = 16M
log_error = /var/log/mysql/error.log

[mysqld]
innodb-file-format=barracuda
innodb-file-per-table=1
innodb-large-prefix=1
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

[mysql]
default-character-set = utf8mb4
```

Restart MariaDB:
```bash
sudo service mysql restart
```

## Other Dependencies

### Step 8: Install Redis
```bash
sudo apt-get install redis-server -y
```

### Step 9: Install Node.js 18
```bash
curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
source ~/.bashrc
nvm install 18
nvm use 18
```

### Step 10: Install Yarn
```bash
sudo npm install -g yarn
```

### Step 11: Install wkhtmltopdf
```bash
sudo apt-get install xvfb libfontconfig wkhtmltopdf -y
```

## Install Frappe Bench (System-wide)

### Step 12: Install Frappe Bench as Root
```bash
sudo -H pip3 install frappe-bench
bench --version
```

## Create Dedicated User

### Step 13: Create Frappe User
```bash
# Create new user
sudo adduser frappe
sudo usermod -aG sudo frappe
sudo usermod -aG www-data frappe

# Switch to new user
su - frappe
```

### Step 14: Install User-specific Dependencies
```bash
# Install NVM and Node.js for the frappe user
curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
source ~/.bashrc
nvm install 18
nvm use 18

# Install Yarn for the frappe user
npm install -g yarn

# Verify installations
node --version      # Should show v18.x
python3.11 --version # Should show Python 3.11.x
bench --version     # Should show bench version (system-wide installation)
```

## Initialize Bench and Create Site

### Step 15: Initialize Bench
```bash
bench init frappe-bench --frappe-branch version-15 --python python3.11
cd frappe-bench
```

### Step 16: Create a Site
```bash
bench new-site dcode.com
bench use dcode.com
bench --site dcode.com add-to-hosts
bench --site dcode.com migrate
```

### Step 17: Install ERPNext
```bash
# Install payments app
bench get-app payments

# Install ERPNext version 15
bench get-app erpnext --branch version-15
# OR
bench get-app https://github.com/frappe/erpnext --branch version-15

# Install ERPNext on the site
bench --site dcode.com install-app erpnext

# Install Rasiin Design if needed
bench get-app  https://github.com/Rasiintech/rasiin_design.git

# Install RASIIN DESIGN on the site
bench --site dcode.com install-app rasiin_design

```

### Step 18: Start Development Server
```bash
bench start
```

## Production Setup

### Step 19: Setup Production Environment
```bash
# Exit back to the root shell
exit

sudo bench setup production frappe

# The command above needs to know the user. It will find the bench in /home/frappe/frappe-bench.
# Restart the services
sudo bench restart
# Or
bench restart
```

If `bench restart` doesn't work, run the setup command again and answer Yes to all questions:
```bash
sudo bench setup production frappe
```

If JS and CSS files are not loading on the login window, run:
```bash
sudo chmod o+x /home/frappe
```

### Step 20: SSL Certificate for HTTPS
```bash
sudo apt install certbot python3-certbot-nginx
certbot -d {domain_name} --register-unsafely-without-email
```

### Step 21: Auto-renew SSL Certificate
```bash
sudo certbot renew --dry-run
```


## bench restart not works Some Issue may you face nodejs version says 12 when you run this command : sudo /usr/bin/node -v  solve this below steps :
``` 
sudo apt-get remove -y nodejs libnode-dev
sudo apt-get purge  -y nodejs libnode-dev
sudo apt-get autoremove -y

curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

sudo /usr/bin/node -v

# Refresh the supervisor
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl restart frappe-bench-node-socketio

# now run those command to verify
sudo /usr/bin/node -v  # it should say v18.20.8
bench restart   # it will work insha allah

```


This guide provides a complete setup for Frappe/ERPNext version 15 on Ubuntu 22.04 LTS, including production deployment with NGINX, SSL, and proper security configurations.
