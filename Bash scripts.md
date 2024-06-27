automate common Frappe tasks.  

**1. Install Packages Script (install_frappe_packages.sh)**

```bash
#!/bin/bash

# Install required packages
sudo apt-get update
sudo apt-get install -y git python3-dev python3-setuptools python3-pip virtualenv \
  libmysqlclient-dev redis-server nodejs yarn xvfb libfontconfig wkhtmltopdf \
  software-properties-common dirmngr apt-transport-https

# Install MariaDB
sudo apt update
sudo apt install mariadb-server

# Configure MariaDB for unicode (optional)
# Define the text to add for each section
TEXT_TO_ADD_MYSQLD="
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
"

TEXT_TO_ADD_MYSQL="
default-character-set = utf8mb4
"

# Add text to my.cnf, creating sections if they don't exist
CONFIG_FILE="/etc/mysql/my.cnf"
if [ ! -f "$CONFIG_FILE" ]; then
  echo "Error: Configuration file '$CONFIG_FILE' not found."
  exit 1
fi

# Function to add text to a section if it doesn't already exist
add_text_to_section() {
  local section_name="$1"
  local text_to_add="$2"
  local config_file="$3"

  if grep -q "\[$section_name\]" "$config_file"; then
    # Section exists, check if text already exists
    if ! grep -q "$text_to_add" "$config_file"; then
      # Text is missing, add it
      sed -i "/\[$section_name\]/a $text_to_add" "$config_file"
      echo "Added text to [$section_name] section in '$config_file'."
    fi
  else
    # Section doesn't exist, create it and add the text
    echo "[$section_name]" >> "$config_file"
    echo "$text_to_add" >> "$config_file"
    echo "Created [$section_name] section and added text to '$config_file'."
  fi
}

# Add text to [mysqld] and [mysql] sections
add_text_to_section "mysqld" "$TEXT_TO_ADD_MYSQLD" "$CONFIG_FILE"
add_text_to_section "mysql" "$TEXT_TO_ADD_MYSQL" "$CONFIG_FILE"
# Create a virtual environment:
python3 -m venv my_frappe_env
source my_frappe_env/bin/activate
# Install Frappe-bench
pip3 install frappe-bench

# Optional: Install Supervisor if it's not already installed
sudo apt install supervisor

# Optional: Install Nginx if it's not already installed
sudo apt install nginx

echo "Packages installed successfully!"
```

**2. Initialize Frappe Bench Script (init_frappe_bench.sh)**

```bash
#!/bin/bash

# Input:
#   - Frappe branch (optional): Default to version-15
FRAPPE_BRANCH="${1:-version-15}"


# Initialize the Frappe bench
bench init frappe-bench --frappe-branch ${FRAPPE_BRANCH}

# Start the bench
cd frappe-bench
source env/bin/activate
bench start

echo "Frappe bench initialized successfully!"
```

**3. Create New Site Script (create_frappe_site.sh)**

```bash
#!/bin/bash

# Input:
#   - Site name: Required
#   - Database type (optional): Default to mariadb
#   - Database host (optional): Default to localhost
#   - Database port (optional): Default to 3306
#   - Database username (optional): Default to root
#   - Database password (optional): Prompt for input if not provided
#   - Admin password (optional): Prompt for input if not provided
SITE_NAME="$1"
DB_TYPE="${2:-mariadb}"
DB_HOST="${3:-localhost}"
DB_PORT="${4:-3306}"
DB_USERNAME="${5:-root}"
DB_PASSWORD="${6}"
ADMIN_PASSWORD="${7}"

# Prompt for database password if not provided
if [ -z "$DB_PASSWORD" ]; then
  read -p "Enter database password: " DB_PASSWORD
fi

# Prompt for admin password if not provided
if [ -z "$ADMIN_PASSWORD" ]; then
  read -p "Enter admin password: " ADMIN_PASSWORD
fi

# Create the new site
bench new-site --db-name ${SITE_NAME} --db-password ${DB_PASSWORD} \
  --db-type ${DB_TYPE} --db-host ${DB_HOST} --db-port ${DB_PORT} \
  --mariadb-root-username ${DB_USERNAME} --mariadb-root-password ${DB_PASSWORD} \
  --no-mariadb-socket --admin-password ${ADMIN_PASSWORD}

echo "New site '${SITE_NAME}' created successfully!"
```

**4. Configure Supervisor Script (config_supervisor.sh)**

```bash
#!/bin/bash

# Input:
#   - Site name: Required
SITE_NAME="$1"

# Configure supervisor for the site
bench --site ${SITE_NAME} setup supervisor

# Create a symbolic link to the supervisor configuration file
sudo ln -s pwd/config/supervisor.conf /etc/supervisor/conf.d/frappe-bench.conf

echo "Supervisor configured for site '${SITE_NAME}'!"
```

**5. Configure Nginx Script (config_nginx.sh)**

```bash
#!/bin/bash

# Input:
#   - Site name: Required
SITE_NAME="$1"

# Configure Nginx for the site
bench --site ${SITE_NAME} setup nginx

# Create a symbolic link to the Nginx configuration file
sudo ln -s pwd/config/nginx.conf /etc/nginx/conf.d/frappe-bench.conf

echo "Nginx configured for site '${SITE_NAME}'!"
```

**6. Configure Redis Script (config_redis.sh)**

```bash
#!/bin/bash

# Input:
#   - Site name: Required
SITE_NAME="$1"

# Configure Redis for the site (optional, but recommended for production)
bench --site ${SITE_NAME} setup redis

# Edit the Redis configuration file if needed:
# sudo nano /etc/redis/redis.conf

echo "Redis configured for site '${SITE_NAME}'!"
```

**7. Configure Production Site Script (config_production_site.sh)**

```bash
#!/bin/bash

# Input:
#   - Site name: Required
#   - Frappe user: Required (e.g., dcode-frappe)
SITE_NAME="$1"
FRAPPE_USER="$2"

# Configure the production site
sudo bench --site ${SITE_NAME} setup production ${FRAPPE_USER}

echo "Production site '${SITE_NAME}' configured successfully!"
```

**How to use the scripts:**

1. **Make the scripts executable:**
   ```bash
   chmod +x install_frappe_packages.sh
   chmod +x init_frappe_bench.sh 
   chmod +x create_frappe_site.sh 
   chmod +x config_supervisor.sh 
   chmod +x config_nginx.sh 
   chmod +x config_redis.sh
   chmod +x config_production_site.sh
   ```

2. **Run the scripts:**
   * `./install_frappe_packages.sh` (installs packages)
   * `./init_frappe_bench.sh [frappe_branch]` (initializes the bench)
   * `./create_frappe_site.sh [site_name] [db_type] [db_host] [db_port] [db_username] [db_password] [admin_password]` (creates a new site)
   * `./config_supervisor.sh [site_name]` (configures supervisor)
   * `./config_nginx.sh [site_name]` (configures Nginx)
   * `./config_redis.sh [site_name]` (configures Redis)
   * `./config_production_site.sh [site_name] [frappe_user]` (configures the production site)

**Remember:** 
* Adjust the script inputs (e.g., site name, branch, passwords, etc.) as needed. 
* Read the comments in the scripts to understand how they work.
* Consult the official Frappe and ERPNext documentation for any further details or advanced configurations. 

Let me know if you have any other questions or need further help! 
