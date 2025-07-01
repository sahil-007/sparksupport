# sparksupport
This repo is for spark support machine test

# Compiling LAMP Stack from Source & WordPress Deployment (Machine Test Challenge)

This repository documents my step-by-step process of manually compiling and configuring a LAMP stack (Linux, Apache, MySQL, PHP) **without using the OS package manager**, as part of a surprise machine test during a technical interview. The final goal was to host a **WordPress website** on a custom domain (`sahil07.shop`) over port 80.

##  Objective

✅ Compile **Apache 2.4.64** from source  
✅ Compile **PHP 8.3.7** from source  
✅ Install **MySQL 8.4.0** using binary tarball  
✅ Download and configure **WordPress**  
✅ Serve WordPress on **port 80** mapped to `sahil07.shop` (via `/etc/hosts`)  
✅ Complete all steps **without using OS repositories like `apt` or `yum`**

---

---

## System Requirements

- Ubuntu 20.04 / 22.04 Virtual Machine
- Root or sudo privileges
- Basic internet access for downloading source packages
- Approx. 2GB RAM or more

---
##  Setup Process

## Step 1: Install Required Dependencies
Start by updating the package list and installing essential build tools and libraries needed for compilation.

```bash
sudo apt update
sudo apt install build-essential libpcre3 libpcre3-dev libssl-dev tar wget unzip libxml2-dev libsqlite3-dev libcurl4-openssl-dev pkg-config libpng-dev libjpeg-dev libonig-dev libzip-dev libexpat1-dev cmake -y
```

## Step 2: Compile Apache HTTPD from Source
Apache requires the Apache Portable Runtime (APR) and APR-util. We'll download, compile, and install Apache with support for modules, rewrites, and SSL.

```bash
cd /usr/local/src
sudo wget https://downloads.apache.org/httpd/httpd-2.4.63.tar.gz
sudo tar -xzf httpd-2.4.63.tar.gz
cd httpd-2.4.63

# Download and unpack APR and APR-util
cd srclib
sudo wget https://downloads.apache.org/apr/apr-1.7.6.tar.gz
sudo wget https://downloads.apache.org/apr/apr-util-1.6.3.tar.gz
sudo tar -xzf apr-1.7.6.tar.gz
sudo tar -xzf apr-util-1.6.3.tar.gz
sudo mv apr-1.7.6 apr
sudo mv apr-util-1.6.3 apr-util
cd ..

# Configure and compile Apache
sudo ./configure --enable-so --enable-rewrite --enable-ssl --with-included-apr --prefix=/usr/local/apache2
sudo make
sudo make install

# Set ServerName in the configuration
sudo nano /usr/local/apache2/conf/httpd.conf
```

In `httpd.conf`, add or uncomment the line:
```
ServerName localhost
```

## Step 3: Start Apache and Test
Launch Apache and verify it works by checking `localhost` in your browser, where you should see "It Works!"

```bash
sudo /usr/local/apache2/bin/apachectl start
```

- **Test**: Open a browser and navigate to `http://localhost`. You should see the default Apache page.

## Step 4: Compile PHP from Source
Now, compile PHP and integrate it with Apache for dynamic web content.

```bash
cd /usr/local/src
sudo wget https://www.php.net/distributions/php-8.3.7.tar.gz
sudo tar -xzf php-8.3.7.tar.gz
cd php-8.3.7

# Configure and compile PHP
sudo ./configure --with-apxs2=/usr/local/apache2/bin/apxs --with-mysqli --with-openssl --enable-mbstring --with-zlib --with-curl
sudo make
sudo make install

# Link PHP to Apache
sudo cp php.ini-development /usr/local/lib/php.ini
sudo nano /usr/local/apache2/conf/httpd.conf
```

In `httpd.conf`, add this line at the end:
```
AddType application/x-httpd-php .php
```

Replace the `DirectoryIndex` line with:
```
DirectoryIndex index.php index.html
```

Restart Apache:
```bash
sudo /usr/local/apache2/bin/apachectl restart
```

- **Test PHP**: Create a test file and check it in your browser.
```bash
echo "<?php phpinfo(); ?>" | sudo tee /usr/local/apache2/htdocs/info.php
sudo /usr/local/apache2/bin/apachectl restart
```
- Visit `http://localhost/info.php` to see PHP configuration details.

## Step 5: Install MySQL (via Binary Tarball)
Set up MySQL to handle database operations for WordPress.

```bash
cd /usr/local/src
sudo wget https://dev.mysql.com/get/Downloads/MySQL-8.4/mysql-8.4.0-linux-glibc2.28-x86_64.tar.xz
sudo tar -xf mysql-8.4.0-linux-glibc2.28-x86_64.tar.xz
sudo mv mysql-8.4.0-linux-glibc2.28-x86_64 /usr/local/mysql

# Create MySQL user and initialize data directory
sudo groupadd mysql
sudo useradd -r -g mysql -s /bin/false mysql
cd /usr/local/mysql
sudo mkdir mysql-files
sudo chown mysql:mysql mysql-files
sudo chmod 750 mysql-files

# Install additional dependencies
sudo apt update
sudo apt install libaio1 libaio-dev

# Initialize MySQL
sudo bin/mysqld --initialize --user=mysql
```

- **Note**: The initialization outputs a temporary root password (e.g., `ByiSwdrj=k`). Save this!
- Start MySQL:
```bash
sudo bin/mysqld_safe --user=mysql &
```
- **Tip**: It may appear stuck. Open a new terminal tab (don’t press Ctrl+C).

## Step 6: Download and Set Up WordPress
Install WordPress in Apache’s web root and configure its database.

```bash
cd /usr/local/apache2/htdocs
sudo wget https://wordpress.org/latest.zip
sudo unzip latest.zip
sudo mv wordpress/* .
sudo rm -r wordpress latest.zip
sudo chown -R www-data:www-data /usr/local/apache2/htdocs

# Create the WordPress database
/usr/local/mysql/bin/mysql -u root -p
```

In the MySQL prompt, run:
```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Password@123';
CREATE DATABASE wordpress;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'wp_pass';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

Configure WordPress:
```bash
sudo cp wp-config-sample.php wp-config.php
sudo nano wp-config.php
```

Edit `wp-config.php` to update:
- `DB_NAME` to `wordpress`
- `DB_USER` to `wpuser`
- `DB_PASSWORD` to `wp_pass`

## Step 7: Simulate a Domain (e.g., sahil07.shop)
Simulate a domain for local testing by editing the hosts file.

```bash
sudo nano /etc/hosts
```

Add this line:
```
127.0.0.1    sahil07.shop
```

- **Test**: Navigate to `http://sahil07.shop` in your browser to access WordPress and complete the setup via the web interface.

## Conclusion
You’ve now compiled and configured Apache, PHP, and MySQL from source, and set up WordPress! This stack is ready for local development or testing. Explore `http://localhost/info.php` for PHP details and `http://sahil07.shop` for WordPress.

## Notes
- **Security**: The passwords (`Password@123`, `wp_pass`) are examples. Use strong, unique passwords in production.
- **Troubleshooting**: Ensure Apache and MySQL are running. Check logs in `/usr/local/apache2/logs` and `/usr/local/mysql/data` if issues arise.
- **Portfolio**: This project showcases skills in server setup, source compilation, and web stack integration.

---
Live WordPress site available at http://sahil07.shop (expires on 05/07/25)

## 🙋‍♂️ About Me

I'm **Sahil TK**, a passionate DevOps & Cloud Engineer.  
🔗 [LinkedIn](https://linkedin.com/in/sahil-tk-8733b5282/) | 🌐 [Portfolio](https://github.com/sahil-007/sparksupport-machine-test)

Give this repo a ⭐ if you found it useful!
