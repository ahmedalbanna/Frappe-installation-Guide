## Frappe and ERPNext Installation Guide for Ubuntu

This guide will guide you through installing Frappe and ERPNext on your Ubuntu system. It's broken down into several steps, each with clear instructions and explanations.

### Prerequisites

Before you begin, ensure your Ubuntu system meets these prerequisites:

* **Python:** Version 3.6 or later
* **Node.js:** Version 14 or later
* **Redis:** Version 5
* **MariaDB:** Version 10.3.x (or PostgreSQL 9.5.x)
* **Yarn:** Version 1.12 or later
* **Pip:** Version 20 or later
* **wkhtmltopdf:** Version 0.12.5 (with patched Qt) for PDF generation
* **Cron:** For automated tasks like certificate renewals and backups
* **Nginx:** For proxying multi-tenant sites in production

### Step 1: Install Git

Install Git, a powerful version control system:

```bash
sudo apt-get install git
```

### Step 2: Install python-dev

Install `python-dev` for compiling Python C extensions:

```bash
sudo apt-get install python3-dev
```

### Step 3: Install setuptools and pip

Install `setuptools` and `pip`, Python's package managers:

```bash
sudo apt-get install python3-setuptools python3-pip
```

### Step 4: Install virtualenv

Install `virtualenv` to create isolated Python environments:

```bash
sudo apt-get install virtualenv
```

**Check your Python version:**

```bash
python3 -V
```

If your version is 3.8.x, run:

```bash
sudo apt install python3.8-venv
```

If your version is 3.10.x, run:

```bash
sudo apt install python3.10-venv
```

### Step 5: Install MariaDB 10.3

MariaDB is a powerful open-source relational database management system:

1. **Download MariaDB repository:** Visit [https://downloads.mariadb.org/mariadb/repositories/#mirror=piconets](https://downloads.mariadb.org/mariadb/repositories/#mirror=piconets)

2. **For Ubuntu 20.04:**
   ```bash
   sudo apt-get install software-properties-common
   sudo apt-key adv --fetch-keys 'https://mariadb.org/mariadb_release_signing_key.asc'
   sudo add-apt-repository 'deb [arch=amd64,arm64,ppc64el] https://ftp.icm.edu.pl/pub/unix/database/mariadb/repo/10.3/ubuntu focal main'
   sudo apt update
   sudo apt install mariadb-server
   ```

3. **For Ubuntu 18.04:**
   ```bash
   sudo apt-get install software-properties-common dirmngr apt-transport-https
   sudo apt-key adv --fetch-keys 'https://mariadb.org/mariadb_release_signing_key.asc'
   sudo add-apt-repository 'deb [arch=amd64,arm64,ppc64el] https://mirrors.aliyun.com/mariadb/repo/10.3/ubuntu bionic main'
   sudo apt update
   sudo apt install mariadb-server
   ```

**Important:** You will be prompted to set the MySQL root password during installation. If you don't see this prompt, you can manually initialize MariaDB using:

```bash
sudo mysql_secure_installation
```

### Step 6: Install MariaDB development files

```bash
sudo apt-get install libmysqlclient-dev
```

### Step 7: Configure MariaDB for Unicode

1. **Edit the MariaDB configuration file:**

   ```bash
   sudo nano /etc/mysql/my.cnf
   ```

2. **Add the following lines:**

   ```
   [mysqld]
   character-set-client-handshake = FALSE
   character-set-server = utf8mb4
   collation-server = utf8mb4_unicode_ci

   [mysql]
   default-character-set = utf8mb4
   ```

3. **Save the file (Ctrl+X, Y, Enter).**

4. **Restart the MariaDB service:**

   ```bash
   sudo service mysql restart
   ```

### Step 8: Install Redis

Redis is a high-performance in-memory data store used for caching and real-time updates:

```bash
sudo apt-get install redis-server
```

### Step 9: Install Node.js 20.x

Node.js is a JavaScript runtime environment used for server-side development:

```bash
sudo apt-get install curl
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### Step 10: Install Yarn

Yarn is a package manager for JavaScript projects:

```bash
sudo npm install -g yarn
```

**If you encounter errors, try these steps:**

```bash
sudo npm uninstall -g yarn
sudo rm -rf /usr/bin/yarn
```

### Step 11: Install wkhtmltopdf

wkhtmltopdf converts HTML pages to PDFs:

```bash
sudo apt-get install xvfb libfontconfig wkhtmltopdf
```

### Step 12: Install Frappe-bench

**For a server environment:**

```bash
pip3 install frappe-bench
```

**For local development:**

```bash
sudo -H pip3 install frappe-bench
```

**Important:** After installing `frappe-bench`, log out and log back into your terminal before proceeding.

Verify the installation:

```bash
bench --version
```

### Step 13: Initialize the Frappe Bench

1. **Create a directory for your Frappe installation:**

   ```bash
   mkdir frappe-bench
   cd frappe-bench
   ```

2. **Initialize the Frappe bench:**

   ```bash
   bench init frappe-bench --frappe-branch version-15
   ```

   (Replace `version-15` with the desired Frappe version if necessary)

3. **Start the bench:**

   ```bash
   bench start
   ```

### Step 14: Create a new Frappe site

```bash
bench new-site dcode.com
```

### Step 15: Install ERPNext

1. **Get the ERPNext application:**

   ```bash
   bench get-app erpnext --branch version-15
   ```

   Or, from a specific GitHub repository:

   ```bash
   bench get-app https://github.com/frappe/erpnext --branch version-15
   ```

2. **Install ERPNext on your site:**

   ```bash
   bench --site dcode.com install-app erpnext
   ```

3. **Start the bench:**

   ```bash
   bench start
   ```

### Optional Step: Production Server Setup

**This step is only necessary for production server setups.**

1. **Create a new user:**

   ```bash
   sudo adduser dcode-frappe
   sudo usermod -aG sudo dcode-frappe
   su - dcode-frappe
   python3 -m venv /path/to/my_env
   source /path/to/my_env/bin/activate
   ```

2. **Follow steps 12 to 15.**

3. **Set up production:**

   ```bash
   sudo bench setup production dcode-frappe
   bench restart
   ```

4. **Open the server IP in your web browser:** You can now access your production server.

### Port Configuration for Multiple Sites

1. **Switch off DNS-based multitenancy:**

   ```bash
   bench config dns_multitenant off
   ```

2. **Create a new site:**

   ```bash
   bench new-site site2name
   ```

3. **Set a port for the new site:**

   ```bash
   bench set-nginx-port site2name 82
   ```

4. **Regenerate the Nginx configuration:**

   ```bash
   bench setup nginx
   ```

5. **Reload Nginx and Supervisor:**

   ```bash
   sudo service nginx reload
   sudo service supervisor restart
   ```

### Other Important Commands

* **Obtain a Let's Encrypt certificate for your site:**

   ```bash
   sudo add-apt-repository ppa:certbot/certbot
   sudo apt update
   sudo apt install python-certbot-nginx
   sudo certbot --nginx -d example.com
   ```

* **Install the Frappe application locally:**

   ```bash
   ./env/bin/python -m pip install -q -U -e /apps/frappe
   ```

* **Stop production services and start in development mode:**

   ```bash
   sudo service supervisor stop
   sudo service redis stop
   sudo service nginx stop
   ```

* **Start development mode (ensure you have a `Procfile` in the `frappe-bench` directory):**

   ```bash
   bench setup procfile
   bench start
   ```

* **Restart production mode:**

   ```bash
   sudo service supervisor start
   sudo service redis start
   sudo service nginx start
   ```

* **Check the status of services:**

   ```bash
   sudo service supervisor status
   sudo service redis status
   sudo service nginx status
   ```

### Adding a New User to MySQL

1. **Connect to MySQL:**

   ```bash
   mysql -u root -p
   ```

2. **Create the new user:**

   ```sql
   CREATE USER 'new_user'@'localhost' IDENTIFIED BY 'password';
   GRANT ALL PRIVILEGES ON * . * TO 'new_user'@'localhost';
   FLUSH PRIVILEGES;
   ```

3. **Grant all privileges to the `admin` user:**

   ```sql
   grant all on *.* to 'admin@localhost' identified by 'admin' with grant option;
   ```

4. **Set the `root` user password:**

   ```sql
   SET PASSWORD FOR 'root'@'localhost' = PASSWORD('mypass');
   ```

### Configuring the Server

* **Enable DNS-based multitenancy:**

   ```bash
   bench config dns_multitenant on
   ```

* **Set the site name in `sites/currentsite.txt`:**

   ```bash
   nano sites/currentsite.txt
   ```

   Write "banna.com" in the file.

### Restarting Services

* **MariaDB:**

   ```bash
   sudo service mysql restart
   ```

* **Redis:**

   ```bash
   sudo service redis-server {start|stop|restart|force-reload|status}
   ```

* **Configure Redis port (in `/etc/redis/redis.conf`):**
   * Change the port number to 11000 and comment out the original port 6379.
   * Test the new port:
     ```bash
     redis-cli -h localhost -p 11001 ping
     ```

* **Supervisor:**

   ```bash
   sudo nano /etc/supervisor/supervisord.conf
   chown= yourfrappeusername:yourfrappeusername
   sudo -A systemctl restart supervisor
   ```

### Installing Supervisor and Nginx

* **Supervisor:**

   ```bash
   sudo apt -y install supervisor
   bench setup supervisor
   sudo ln -s pwd/config/supervisor.conf /etc/supervisor/conf.d/frappe-bench.conf
   ```

* **Nginx:**

   ```bash
   sudo apt -y install nginx
   bench setup nginx
   sudo ln -s pwd/config/nginx.conf /etc/nginx/conf.d/frappe-bench.conf
   ```

### Adding a Site to the Host System

```bash
sudo nano /etc/hosts
```

Add the following line:

```
127.0.0.1	frappe.localhost
```

### Creating a New Site

```bash
bench new-site --db-name [] --db-password [] --db-type mariadb --db-host 127.0.0.1 --db-port 3306 --mariadb-root-username root --mariadb-root-password [] --no-mariadb-socket --admin-password Admin --install-app erpnext [].cybererps.com
```

### Dropping a Site

```bash
bench drop-site --mariadb-root-username admin@localhost --mariadb-root-password admin --force banna.com
```

### Activating and Deactivating the Python Environment

* **Activate:**

   ```bash
   source env/bin/activate
   ```

* **Deactivate:**

   ```bash
   source env/bin/deactivate
   ```

### Exploring Site Commands

```bash
bench --site library.test console
bench --site library.test mariadb
bench --site library.test backup
```

### Updating the Frappe App

```bash
bench switch-to-branch version-15 frappe erpnext --upgrade
git pull upstream version-15
bench get-app --branch=version-15 frappe
```

### Adding a Frontend

```bash
bench add-frappe-ui
```

### ERPNext Configuration

Set the following environment variables:

```
ERPNEXT_API_KEY = 'ad43b3e0a015a82'
ERPNEXT_API_SECRET = 'd550eb2103b2a9b'
ERPNEXT_URL = 'http://dev.local:8000'
ERPNEXT_VERSION = 15
```

### Disabling CSRF Protection

```bash
bench --site frappe.localhost set-config ignore_csrf 1
```

### Changing the MySQL Root Password

1. **Stop the MySQL service:**

   ```bash
   sudo systemctl stop mysql
   ```

2. **Start MySQL in safe mode:**

   ```bash
   sudo mysqld_safe --skip-grant-tables --skip-networking &
   ```

3. **Connect to MySQL as the root user:**

   ```bash
   mysql -u root
   ```

4. **Update the root password:**

   ```sql
   USE mysql;
   UPDATE user SET authentication_string=PASSWORD('new_password') WHERE User='root';
   FLUSH PRIVILEGES;
   ```

5. **Exit MySQL:**

   ```sql
   EXIT;
   ```

6. **Kill the safe mode process:**

   ```bash
   sudo killall mysqld_safe
   ```

7. **Start the MySQL service:**

   ```bash
   sudo systemctl start mysql
   ```

### Network Check

```bash
sudo apt install net-tools
sudo netstat -tuln | grep '13000\|11000'
sudo lsof -i :13000
sudo kill 27381
```

### Remember:

* Replace placeholders like `[]` with your specific values.
* Use the correct Frappe and ERPNext versions for your needs.
* Refer to the official Frappe and ERPNext documentation for more detailed instructions and troubleshooting guides.
* This guide provides a starting point for installation. Adapt the steps to suit your specific needs.

Good luck!
