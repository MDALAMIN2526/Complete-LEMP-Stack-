# Complete LEMP Stack with phpMyAdmin, Multiple PHP Versions & SSL

## Table of Contents
1. [System Requirements](#system-requirements)
2. [Initial Setup](#initial-setup)
3. [Nginx Installation](#nginx-installation)
4. [MariaDB Installation](#mariadb-installation)
5. [PHP Installation](#php-installation)
6. [phpMyAdmin Installation](#phpmyadmin-installation)
7. [SSL Configuration](#ssl-configuration)
8. [NGINX ModSecurity](#nginx-modsecurity)
9. [Post-Installation](#post-installation)
10. [Troubleshooting](#troubleshooting)
11. [Maintenance](#maintenance)

## System Requirements
- Ubuntu 20.04/22.04 or Debian 10/11
- Minimum 1GB RAM (2GB recommended)
- Root or sudo privileges
- Domain name pointed to server IP (for SSL)

## Initial Setup
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y software-properties-common curl wget unzip
```

## Nginx Installation
```bash
sudo apt install -y nginx
sudo systemctl enable nginx
sudo systemctl start nginx
```

## MariaDB Installation
```bash
sudo apt install -y mariadb-server
sudo mysql_secure_installation
```
Follow prompts to set root password and secure installation.

## PHP Installation

### Add PHP Repository
```bash
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update
```

### Install PHP Versions Individually

#### PHP 8.0
```bash
sudo apt install -y php8.0 php8.0-fpm php8.0-mysql \
    php8.0-mbstring php8.0-xml php8.0-curl \
    php8.0-zip php8.0-gd php8.0-intl php8.0-bcmath \
    php8.0-soap php8.0-opcache
```

#### PHP 8.1
```bash
sudo apt install -y php8.1 php8.1-fpm php8.1-mysql \
    php8.1-mbstring php8.1-xml php8.1-curl \
    php8.1-zip php8.1-gd php8.1-intl php8.1-bcmath \
    php8.1-soap php8.1-opcache
```

#### PHP 8.2
```bash
sudo apt install -y php8.2 php8.2-fpm php8.2-mysql \
    php8.2-mbstring php8.2-xml php8.2-curl \
    php8.2-zip php8.2-gd php8.2-intl php8.2-bcmath \
    php8.2-soap php8.2-opcache
```

#### PHP 8.3
```bash
sudo apt install -y php8.3 php8.3-fpm php8.3-mysql \
    php8.3-mbstring php8.3-xml php8.3-curl \
    php8.3-zip php8.3-gd php8.3-intl php8.3-bcmath \
    php8.3-soap php8.3-opcache
```

### Verify PHP Installations
```bash
php8.0 -v
php8.1 -v
php8.2 -v
php8.3 -v
```

## phpMyAdmin Installation
```bash
sudo apt install -y phpmyadmin
```
During installation:
- Select "nginx" as web server
- Choose "Yes" to configure with dbconfig-common
- Set a password for phpMyAdmin database user

### Configure Nginx
```bash
sudo nano /etc/nginx/sites-available/phpmyadmin.conf
```

Add this configuration (adjust PHP version as needed):
```nginx
server {
    listen 80;
    server_name mysql.yourdomain.com;
    root /usr/share/phpmyadmin;
    index index.php;

    access_log /var/log/nginx/phpmyadmin.access.log;
    error_log /var/log/nginx/phpmyadmin.error.log;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.2-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

Enable the configuration:
```bash
sudo ln -s /etc/nginx/sites-available/phpmyadmin.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

## SSL Configuration

### Install Certbot
```bash
sudo apt install -y certbot python3-certbot-nginx
```

### Obtain SSL Certificate
```bash
sudo certbot --nginx -d mysql.yourdomain.com
```
Follow prompts to configure HTTPS and redirects.

### Auto-Renewal Test
```bash
sudo certbot renew --dry-run
```
##  NGINX ModSecurity
installing and configuring NGINX and ModSecurity as a Web Application Firewall (WAF)
### Update the System
```bash
sudo apt update && sudo apt upgrade -y
```

### Install Necessary Prerequisites
```bash
sudo apt install  git gcc build-essential libtool libpcre3 libpcre3-dev zlib1g zlib1g-dev libssl-dev -y
```
### Download and Compile ModSecurity
```bash
cd /usr/local/src
```
```bash
sudo git clone --depth 1 https://github.com/SpiderLabs/ModSecurity
```
```bash
cd ModSecurity
```
```bash
sudo git submodule init
```
```bash
sudo git submodule update
```
```bash
sudo ./build.sh
```
```bash
sudo ./configure
```
```bash
sudo make
```
```bash
sudo make install
```

### Download and Compile ModSecurity NGINX Connector
```bash
cd /usr/local/src
```
```bash
sudo git clone --depth 1 https://github.com/SpiderLabs/ModSecurity-nginx.git
```

### Download and Compile NGINX with ModSecurity
First, download the NGINX source code. Make sure to get the version matching your currently installed NGINX.
```bash
wget http://nginx.org/download/nginx-1.18.0.tar.gz
```
```bash
tar -zxvf nginx-1.18.0.tar.gz
```
```bash
cd nginx-1.18.0
```

### Compile NGINX with the ModSecurity module:
```bash
sudo ./configure --with-compat --add-dynamic-module=../ModSecurity-nginx
```
```bash
sudo make modules
```

### Move the compiled module to the NGINX modules directory:
```bash
sudo mkdir -p /etc/nginx/modules
```
```bash
sudo cp /usr/local/src/nginx-1.18.0/objs/ngx_http_modsecurity_module.so /etc/nginx/modules/

```

### Configure NGINX to Load ModSecurity Module
Edit the NGINX configuration to load the ModSecurity module:
```bash
sudo nano /etc/nginx/nginx.conf
```
Add the following line at the top:
```nginx
load_module modules/ngx_http_modsecurity_module.so;
```

### Configure ModSecurity
Move the ModSecurity configuration file to the appropriate directory:
```bash
sudo mv /usr/local/src/ModSecurity/modsecurity.conf-recommended /etc/nginx/modsecurity.conf
```
```bash
sudo mv /usr/local/src/ModSecurity/unicode.mapping /etc/nginx/
```

### Edit the ModSecurity configuration file to enable the engine:
```bash
sudo nano /etc/nginx/modsecurity.conf
```
```bash
sudo nginx -t
```
Change `SecRuleEngine DetectionOnly` to `SecRuleEngine On`.

### Download OWASP Core Rule Set
```bash
sudo git clone https://github.com/coreruleset/coreruleset /etc/nginx/owasp-crs
```
```bash
sudo cp /etc/nginx/owasp-crs/crs-setup.conf.example /etc/nginx/owasp-crs/crs-setup.conf
```
### Include the CRS configuration in your ModSecurity configuration:
```bash
echo 'Include /etc/nginx/owasp-crs/crs-setup.conf' | sudo tee -a /etc/nginx/modsecurity.conf
```
```bash
echo 'Include /etc/nginx/owasp-crs/rules/*.conf' | sudo tee -a /etc/nginx/modsecurity.conf
```
### Add WAF in  Nginx Site Configure 
```bash
sudo nano /etc/nginx/sites-available/phpmyadmin.conf
```

Add this configuration (adjust PHP version as needed):
```nginx
server {
    listen 80;
    server_name mysql.yourdomain.com;
    root /usr/share/phpmyadmin;
    index index.php;
# Load ModSecurity configuration
    modsecurity on;
    modsecurity_rules_file /etc/nginx/modsec/modsecurity.conf;

    access_log /var/log/nginx/phpmyadmin.access.log;
    error_log /var/log/nginx/phpmyadmin.error.log;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.2-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

Update the configuration:
```bash
sudo nginx -t
sudo systemctl reload nginx
```

## Post-Installation

### Secure phpMyAdmin
1. Change default path:
```bash
sudo mv /usr/share/phpmyadmin /usr/share/db-admin-xyz
```
Update Nginx configuration to match new path.

2. Add HTTP authentication:
```bash
sudo apt install -y apache2-utils
sudo htpasswd -c /etc/nginx/pma_pass admin
```

Add to Nginx config:
```nginx
location /db-admin-xyz {
    auth_basic "Admin Login";
    auth_basic_user_file /etc/nginx/pma_pass;
}
```

### Set Default PHP Version
```bash
sudo update-alternatives --set php /usr/bin/php8.2
```

### Optimize MariaDB
```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
Recommended settings for 2GB RAM:
```ini
[mysqld]
innodb_buffer_pool_size = 1G
innodb_log_file_size = 256M
query_cache_size = 64M
query_cache_type = 1
```

### Optimize PHP-FPM
For each PHP version:
```bash
sudo nano /etc/php/8.2/fpm/php.ini
```
Recommended settings:
```ini
memory_limit = 256M
upload_max_filesize = 64M
post_max_size = 64M
max_execution_time = 300
```

## Troubleshooting

### Common Issues
1. **502 Bad Gateway**: Check PHP-FPM is running:
   ```bash
   sudo systemctl status php8.2-fpm
   ```

2. **Missing PHP extensions**: Install required extensions:
   ```bash
   sudo apt install php8.2-[extension]
   ```

3. **Connection refused**: Verify MariaDB is running:
   ```bash
   sudo systemctl status mariadb
   ```

## Maintenance

### Update All Components
```bash
sudo apt update && sudo apt upgrade -y
```

### Backup Database
```bash
mysqldump -u root -p --all-databases > full_backup.sql
```

### Check SSL Auto-Renewal
```bash
sudo systemctl list-timers | grep certbot
```

### Monitor Resources
```bash
htop
df -h
free -h
```

This comprehensive guide for setting up a complete LEMP stack with Nginx, Multiple PHP versions, phpMyAdmin, WAF and SSL security.
