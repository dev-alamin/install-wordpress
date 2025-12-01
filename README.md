# Guide to Install WordPress with on Ubuntu

This guide will walk you through the step-by-step process of setting up a web development environment on Ubuntu Linux. It includes the installation of Ngninx/Apache2, PHP, MariaDB/MySQL, phpMyAdmin, the configuration of virtual hosts for different websites, fixing permission issues, and installing WP-CLI for WordPress management.

## Update Ubuntu before going further
Begin your WordPress installation on your Ubuntu server smoothly and successfully by running the standard update command. Running this command updates your system, minimizing the risk of conflicts during the WordPress installation process, as WordPress is extensive and intricate.

To execute the standard update command, open your terminal and input the following command:
```bash
sudo apt update && sudo apt upgrade
```

## 1. Install Apache2

To install Apache2, run the following commands:

```bash
sudo apt update
sudo apt install apache2
```

Verify the Apache2 installation:

```bash
sudo systemctl status apache2
```

## Or Install Nginx

To install Apache2, run the following commands:

```bash
sudo apt install nginx
```

Verify the Nginx installation:

```bash
systemctl status nginx
```

## Install Some Utility 

Ensure you have installed the following packages before proceeding:

```bash
sudo apt install curl git wget unzip zip
```

## Configure UFW Firewall for Nginx and WordPress

Securing your Nginx server and allowing access to default web ports requires configuring the UFW firewall. Nginx offers profiles for quick UFW configuration.

Ensure UFW is installed on your Ubuntu system with the command:

```bash
sudo apt install ufw
```

Before allowing some necessary portfol, allow ssh to be connected and avoiding unnecessary hassle.

```bash
sudo ufw allow ssh
```

Now after installation and allowing SSH, enable UFW on your system. By default, UFW will deny all incoming and allow all outgoing connections. To enable the firewall, use the following command:

```bash
sudo ufw enable
```

To see the available Nginx profiles, run the following command | Or depending on the server configure and requirement, allow specificly:

```bash
sudo ufw allow 'Nginx Full'
```

```bash
sudo ufw allow 'Nginx Secure'
```

```bash
sudo ufw allow 'Nginx HTTP'
```

## Install MariaDB on Ubuntu
To install MariaDB, execute the following command in your terminal.

```bash
sudo apt install mariadb-server mariadb-client
```

```bash
systemctl status mariadb
```

```bash
sudo systemctl enable mariadb --now
```

## Secure MariaDB with Security Script
To begin, launch the mysql_secure_installation script by executing the following command:

```bash
sudo mysql_secure_installation
```

## 2. Install PHP

To install PHP and necessary modules, run the following commands:

```bash
sudo apt install php php-fpm php-mbstring php-bcmath php-xml php-mysql php-common php-gd php-cli php-curl php-zip php-imagick php-ldap php-intl
```

Verify the PHP installation:

```bash
php -v
```

## Create a Database for WordPress on Ubuntu
First, create a database to run WordPress on your server. Since you previously installed MariaDB during the LEMP stack setup, use it to create a new WordPress database. Open your terminal, enter the following command, and access the MariaDB shell as root to begin the process:

```bash
sudo mariadb -u root
```

```bash
CREATE DATABASE WORDPRESSDB;
```

```bash
CREATE USER 'WPUSER'@localhost IDENTIFIED BY 'PASSWORD';
```

```bash
GRANT ALL PRIVILEGES ON WORDPRESSDB.* TO WPUSER@localhost IDENTIFIED BY 'PASSWORD';
```

```bash
FLUSH PRIVILEGES;
```

```bash
EXIT;
```

## Set WordPress Configuration Files
Setting the WordPress configuration files is an essential step in the installation process. It involves renaming the sample wp-config.php file and entering the necessary configuration details.

```bash
cd /var/www/html/wordpress/
```

```bash
sudo cp wp-config-sample.php wp-config.php
```

```bash
sudo nano wp-config.php
```

## Create Nginx Server Block For WordPress
To install WordPress through the web UI, you must first configure your Nginx server block. It is essential to pay attention to the settings listed below, particularly “try_files $uri $uri/ /index.php?$args;”, as omitting the “?$args” may cause issues with the REST API of WordPress.

To create a new server configuration file, use the following command, replacing “example.com” with your domain name:

```bash
sudo nano /etc/nginx/sites-available/example.com.conf
```

Use this nginx config carefully: 

```nginxconf
server {
  listen 80;
  listen [::]:80;
  server_name www.example.com example.com;
  root /var/www/html/wordpress;
  index index.php index.html index.htm index.nginx-debian.html;

  location / {
    try_files $uri $uri/ /index.php?$args;
  }

  location ~* /wp-sitemap.*\.xml {
    try_files $uri $uri/ /index.php$is_args$args;
  }

  client_max_body_size 100M;

  location ~ \.php$ {
    fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
    include snippets/fastcgi-php.conf;
    fastcgi_buffer_size 128k;
    fastcgi_buffers 4 128k;
    fastcgi_intercept_errors on;
  }

  gzip on;
  gzip_comp_level 6;
  gzip_min_length 1000;
  gzip_proxied any;
  gzip_disable "msie6";
  gzip_types application/atom+xml application/geo+json application/javascript application/x-javascript application/json application/ld+json application/manifest+json application/rdf+xml application/rss+xml application/xhtml+xml application/xml font/eot font/otf font/ttf image/svg+xml text/css text/javascript text/plain text/xml;

  location ~* \.(?:css(\.map)?|js(\.map)?|jpe?g|png|gif|ico|cur|heic|webp|tiff?|mp3|m4a|aac|ogg|midi?|wav|mp4|mov|webm|mpe?g|avi|ogv|flv|wmv)$ {
    expires 90d;
    access_log off;
  }

  location ~* \.(?:svgz?|ttf|ttc|otf|eot|woff2?)$ {
    add_header Access-Control-Allow-Origin "*";
    expires 90d;
    access_log off;
  }

  location ~ /\.ht {
    access_log off;
    log_not_found off;
    deny all;
  }
}
```

To finish off this the Nginx server block configuration, you will need to enable the Nginx configuration file from “sites-available” by creating a symlink to “sites-enabled” using the following command, replacing “example.conf” with your configuration file name:

```bash
sudo ln -s /etc/nginx/sites-available/example.com.conf /etc/nginx/sites-enabled/
```

Check if nginx has any issue

```bash
sudo nginx -t
```

Restart Nginx to reflect the changes you made:

```bash
sudo systemctl restart nginx
```

## Additionally you can block unwanted flooting bot like Semrush..

Put it just inside server blcok

```bash
if ($http_user_agent ~* (semrushbot|ahrefsbot|petalbot|baiduspider|mj12bot|dotbot)) {
    return 403;
}
```

## PHP.ini Configuration

Adjusting your PHP configuration is crucial for optimal WordPress use. To accommodate WordPress media files, you should increase the maximum upload size, post size, and memory limit. You can also adjust the maximum execution time and input variables.

To do this, first open your php.ini file using your terminal. The location may differ depending on your PHP version number.

```bash
sudo nano /etc/php/8.3/fpm/php.ini
```

##increase this to the maximum file size you want to upload, recommended 50 to 100MB## 
 upload_max_filesize = 100MB

##increase this to the maximum post size you want to allow, recommended 50 to 100MB##
 post_max_size = 100MB

##increase this to the maximum execution time, recommended 150 to 300 seconds##
 max_execution_time = 300

##increase this to the maximum GET/POST/COOKIE input variables, recommended 5000 to 10000##
max_input_vars = 5000

##increase this to the maximum memory limit, recommended 256MB or 512MB. Note that you should ensure your system has enough RAM before raising this.##
memory_limit = 256M

After changing the PHP configuration file, restart the PHP-FPM server to implement the new settings. The restart command varies with the PHP version. Identify and use your PHP version number in the command, even if the examples below do not list it.

```bash
sudo systemctl restart php8.3-fpm
```

## 4. Install phpMyAdmin (For local dev environemnt mostly)

To install phpMyAdmin, run the following commands:

```bash
sudo apt install phpmyadmin
sudo ln -s /usr/share/phpmyadmin /var/www/html/phpmyadmin
```

Access phpMyAdmin by visiting `http://localhost/phpmyadmin` in your web browser.

### Grant Privileges to phpMyAdmin User

If you encounter any permission issues while creating a new database using phpMyAdmin, execute the following SQL commands:

```sql
GRANT ALL PRIVILEGES ON database_name.* TO 'phpmyadmin'@'localhost';
FLUSH PRIVILEGES;
```

## 5. Configure Virtual Hosts If You Use Apahce2

To create different domains for different sites, you can use virtual hosts in Apache. Follow these steps:

1. Create a new virtual host configuration file:

   ```bash
   sudo nano /etc/apache2/sites-available/example.com.conf
   ```

2. Add the following content to the file, replacing `example.com` with your desired domain name:

   ```apacheconf
   <VirtualHost *:80>
       ServerAdmin webmaster@example.com
       ServerName example.com
       ServerAlias www.example.com
       DocumentRoot /var/www/html/example.com
       ErrorLog ${APACHE_LOG_DIR}/error.log
       CustomLog ${APACHE_LOG_DIR}/access.log combined

         <Directory /var/www/example>
              Options Indexes FollowSymLinks
              AllowOverride All
              Require all granted
          </Directory>

   </VirtualHost>
   ```

3. Save and close the file.

4. Enable the virtual host configuration:

   ```bash
   sudo a2ensite example.com.conf
   ```

5. Restart Apache to apply the changes:

   ```bash
   sudo systemctl restart apache2
   ```

6. Create the directory for your new site:

   ```bash
   sudo mkdir /var/www/html/example.com
   ```

7. Set the necessary permissions for the directory:

   ```bash
   sudo chown -R www-data:www-data /var/www/html/example.com
   sudo chmod -R 777 /var/www/
   ```

8. Add an entry in your `/etc/hosts` file to map the domain to your localhost:

   ```bash
   sudo nano /etc/hosts
   ```

   Add the following line:

   ```
   127.0.0.1 example.com www.example.com
   ```

   Save and close the file.

9. Now you can access your site by visiting `http://example.com` or `http://www.example.com` in your web browser.

Repeat steps 1 to 9 for each new site you want to create with a different domain name. Each virtual host configuration file should have a unique name and corresponding directory.

## 6. Fix Permission Issues



To fix permission issues, run the following commands:

```bash
sudo chown -R www-data:www-data /var/www/yoursite
sudo find /var/www/yoursite -type d -exec chmod 755 {} \;
sudo find /var/www/yoursite -type f -exec chmod 644 {} \;
sudo chmod 600 /var/www/yoursite/wp-config.php
```

## 7. Install WP-CLI

To install WP-CLI, run the following commands:

```bash
sudo apt install curl
curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
chmod +x wp-cli.phar
sudo mv wp-cli.phar /usr/local/bin/wp
```

Verify the WP-CLI installation:

```bash
wp --info
```

## 8. Install WordPress using WP-CLI

To install WordPress using WP-CLI, navigate to the document root and run the following command:

```bash
wp core download --path=/var/www/html
wp core config --path=/var/www/html --dbname=your_database_name --dbuser=your_username --dbpass=your_password --dbhost=localhost --dbprefix=wp_
wp core install --path=/var/www/html --url=http://localhost --title="Your Site Title" --admin_user=admin --admin_password=admin --admin_email=admin@example.com
```

Replace `your_database_name`, `your_username`, `your_password`, and other parameters with your actual values.

Access your WordPress site by visiting `http://localhost` in your web browser.

---

Feel free to customize and expand this guide further based on your specific requirements and preferences.
