# Frappe-installation-Guide
## The complete guide to install Frappe and ERPNext in your Ubuntu system

### Pre-requisites 

      Python 3.6+
      Node.js 14+
      Redis 5                                       (caching and real time updates)
      MariaDB 10.3.x / Postgres 9.5.x               (to run database driven apps)
      yarn 1.12+                                    (js dependency manager)
      pip 20+                                       (py dependency manager)
      wkhtmltopdf (version 0.12.5 with patched qt)  (for pdf generation)
      cron                                          (bench's scheduled jobs: automated certificate renewal, scheduled backups)
      NGINX                                         (proxying multitenant sites in production)



### STEP 1 Install git
Git is the most commonly used version control system. Git tracks the changes you make to files, 
so you have a record of what has been done, and you can revert to specific versions should you ever need to. 
Git also makes collaboration easier, allowing changes by multiple people to all be merged into one source.
    
    sudo apt-get install git

### STEP 2 install python-dev
python-dev is the package that contains the header files for the Python C API, 
which is used by lxml because it includes Python C extensions for high performance.

    sudo apt-get install python3-dev

### STEP 3 Install setuptools and pip (Python's Package Manager).
Setuptools is a collection of enhancements to the Python distutils that allow developers 
to more easily build and distribute Python packages, especially ones that have 
dependencies on other packages. Packages built and distributed using setuptools 
look to the user like ordinary Python packages based on the distutils.

pip is a package manager for Python.  It's a tool that allows you to install and manage 
additional libraries and dependencies that are not distributed as part of the standard library.

    sudo apt-get install python3-setuptools python3-pip

### STEP 4 Install virtualenv
virtualenv is a tool for creating isolated Python environments containing their own copy of
python , pip , and their own place to keep libraries installed from PyPI.
It's designed to allow you to work on multiple projects with different dependencies 
at the same time on the same machine.
    
    sudo apt-get install virtualenv
    
  CHECK PYTHON VERSION 
  
    python3 -V
  
  IF VERSION IS 3.8.X RUN
  
    sudo apt install python3.8-venv

  IF VERSION IS 3.10.X RUN
  
     sudo apt install python3.10-venv

### STEP 5 Install MariaDB 10.3 stable package
MariaDB is developed as open source software and as a relational database it provides an SQL interface 
for accessing data.

open this link

    https://downloads.mariadb.org/mariadb/repositories/#mirror=piconets
 
For ubuntu 20.04

    sudo apt-get install software-properties-common
    sudo apt-key adv --fetch-keys 'https://mariadb.org/mariadb_release_signing_key.asc'
    sudo add-apt-repository 'deb [arch=amd64,arm64,ppc64el] https://ftp.icm.edu.pl/pub/unix/database/mariadb/repo/10.3/ubuntu focal main'
    sudo apt update
    sudo apt install mariadb-server
    
 For ubuntu 18.04
 
    sudo apt-get install software-properties-common dirmngr apt-transport-https
    sudo apt-key adv --fetch-keys 'https://mariadb.org/mariadb_release_signing_key.asc'
    sudo add-apt-repository 'deb [arch=amd64,arm64,ppc64el] https://mirrors.aliyun.com/mariadb/repo/10.3/ubuntu bionic main'
    sudo apt update
    sudo apt install mariadb-server
     
IMPORTANT :During this installation you'll be prompted to set the MySQL root password.
If you are not prompted for the same You can initialize the MySQL server setup by executing 
the following command
    
    sudo mysql_secure_installation
    
### STEP 6  MySQL database development files

    sudo apt-get install libmysqlclient-dev

### STEP 7 Edit the mariadb configuration ( unicode character encoding )

    sudo nano /etc/mysql/my.cnf

add this to the my.cnf file

    [mysqld]
    character-set-client-handshake = FALSE
    character-set-server = utf8mb4
    collation-server = utf8mb4_unicode_ci

    [mysql]
    default-character-set = utf8mb4

Now press (Ctrl-X) to exit

    sudo service mysql restart

### STEP 8 install Redis
Redis is an open source (BSD licensed), in-memory data structure store, used as a database, 
cache, and message broker.
    
    sudo apt-get install redis-server

### STEP 9 install Node.js 20.X package
Node.js is an open source, cross-platform runtime environment for developing server-side and 
networking applications. Node.js applications are written in JavaScript, and can be run within the Node.js
runtime on OS X, Microsoft Windows, and Linux.

    sudo apt-get install curl
    curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
    sudo apt-get install -y nodejs

### STEP 10  install Yarn
Yarn is a JavaScript package manager that aims to be speedy, deterministic, and secure. 
See how easy it is to drop yarn in where you were using npm before, and get faster, more reliable installs.
Yarn is a package manager for JavaScript.
    
    sudo npm install -g yarn
    
if show error reset yarn

    sudo npm uninstall -g yarn
    sudo rm -rf /usr/bin/yarn

### STEP 11 install wkhtmltopdf
Wkhtmltopdf is an open source simple and much effective command-line shell utility that enables 
user to convert any given HTML (Web Page) to PDF document or an image (jpg, png, etc)

    sudo apt-get install xvfb libfontconfig wkhtmltopdf
    
### if you have to setup production server go to Step 16 else continue 

### STEP 12 install frappe-bench

for server

    pip3 install frappe-bench
for local

      sudo -H pip3 install frappe-bench
IMPORTANT: you may wish to log out and log back into your terminal 
before next step and You must login.
    
    bench --version
    
### STEP 13 initilise the frappe bench & install frappe latest version 

    bench init frappe-bench --frappe-branch version-15
    
    cd frappe-bench/
    bench start
    
### STEP 14 create a site in frappe bench 
    
    bench new-site dcode.com

### STEP 15 install ERPNext latest version in bench & site

    bench get-app erpnext --branch version-15
    ###OR
    bench get-app https://github.com/frappe/erpnext --branch version-15

    bench --site dcode.com install-app erpnext
    
    bench start


#
#
#
### Optional step for cratetind production setup

### STEP 16  Create a new user

    sudo adduser dcode-frappe
    sudo usermod -aG sudo dcode-frappe
    su - dcode-frappe
    python3 -m venv /path/to/my_env
    source /path/to/my_env/bin/activate
    
### Follow the steps from Step 12 to Step 15

### Step 17 setup production
    
    sudo bench setup production dcode-frappe
    bench restart
    
  Open the 0.0.0.0 or server IP in web browser and login to production server
    
    
    
    
  
  #### Port cofiguration for multiple site
  
  
      bench use sitename
  
 

    Switch off DNS based multitenancy (once)

      bench config dns_multitenant off

    Create a new site

      bench new-site site2name

    Set port

       bench set-nginx-port site2name 82

    Re generate nginx config

       bench setup nginx

    Reload nginx

      sudo service nginx reload
      

    Reload supervisor
      sudo service supervisor restart
      
      
    
### others

      sudo add-apt-repository ppa:certbot/certbot
      sudo apt update
      sudo apt install python-certbot-nginx
      sudo certbot --nginx -d example.com
      
      ./env/bin/python -m pip install -q -U -e /apps/frappe
    
    

## If you installed with --production option and you want to stop production services, and start in develop mode

      sudo service supervisor stop
      sudo service redis stop
      sudo service nginx stop
###  To start in develop mode, you need to have Procfile in the frappe-bench directory.

      bench setup procfile
      
      
### To start develop server:   
      bench start
### To restart the production mode agin

      sudo service supervisor start
      sudo service redis start
      sudo service nginx start
 
### To check on the status of services - mainly whether they are active or not

      sudo service supervisor status
      sudo service redis status
      sudo service nginx status
    
## add new user SQL
 mysql -u root -p
mysql>
CREATE USER 'new_user'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON * . * TO 'new_user'@'localhost';
FLUSH PRIVILEGES;

grant all on *.* to 'admin@localhost' identified by 'admin' with grant option;
ET PASSWORD FOR 'root'@'localhost' = PASSWORD('mypass');

## config Server
bench config dns_multitenant on
nano sites/currentsite.txt  write "banna.com"
## command 
sudo service mysql restart
sudo service redis-server {start|stop|restart|force-reload|status}
sudo nano /etc/redis/redis.conf
update port number to 11000. comment original port 6379
redis-cli -h localhost -p 11001 ping
##
sudo nano /etc/supervisor/supervisord.conf
chown= yourfrappeusername:yourfrappeusername
sudo -A systemctl restart supervisor

##
Supervisor:
sudo apt -y install supervisor
bench setup supervisor
sudo ln -s pwd/config/supervisor.conf /etc/supervisor/conf.d/frappe-bench.conf

Nginx :
sudo apt -y install nginx
bench setup nginx
sudo ln -s pwd/config/nginx.conf /etc/nginx/conf.d/frappe-bench.conf

## Add Site name into Host System
$ sudo nano /etc/hosts
$ 127.0.0.1	frappe.localhost

## CREATE NEW SITE
bench new-site --db-name [] --db-password [] --db-type mariadb --db-host 127.0.0.1 --db-port 3306 --mariadb-root-username root --mariadb-root-password [] --no-mariadb-socket --admin-password Admin --install-app erpnext [].cybererps.com

## Drop Site 
bench drop-site --mariadb-root-username admin@localhost --mariadb-root-password admin --force banna.com

## Source Node
https://deb.nodesource.com/

## Ative Python 
source env/bin/activate

## Explore Site Commands:
 bench --site library.test console
 bench --site library.test mariadb
 bench --site library.test backup

## Update Frappe App
bench switch-to-branch version-15 frappe erpnext --upgrade
git pull upstream version-15
bench get-app --branch=version-15 frappe

## add Frontend
bench add-frappe-ui

# ERPNext related configs
ERPNEXT_API_KEY = 'ad43b3e0a015a82'
ERPNEXT_API_SECRET = 'd550eb2103b2a9b'
ERPNEXT_URL = 'http://dev.local:8000'
ERPNEXT_VERSION = 15

## Set Security False into site for frappe-ui
bench --site frappe.localhost set-config ignore_csrf 1


## change user root mysql password
sudo systemctl stop mysql
sudo mysqld_safe --skip-grant-tables --skip-networking &
mysql -u root
USE mysql;
UPDATE user SET authentication_string=PASSWORD('new_password') WHERE User='root';
FLUSH PRIVILEGES;
EXIT;
sudo killall mysqld_safe
sudo systemctl start mysql
