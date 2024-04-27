# Deploying Roots/Bedrock WordPress to DigitalOcean

This provides a comprehensive guide on deploying Roots/Bedrock WordPress on a DigitalOcean Droplet using Nginx, PHP, MySQL, Let's Encrypt, UFW, and other essential tools.

## Table of Contents

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Step-by-Step Guide](#step-by-step-guide)
  - [Create a DigitalOcean Droplet](#step-1-create-a-digitalocean-droplet)
  - [Connect to Your Droplet](#step-2-connect-to-your-droplet)
  - [Update and Upgrade the System](#step-3-update-and-upgrade-the-system)
  - [Install Nginx, PHP, and MySQL](#step-4-install-nginx-php-and-mysql)
  - [Configure Nginx with Public IP Address](#step-5-configure-nginx-with-public-ip-address)
  - [Install Let's Encrypt for SSL](#step-6-install-lets-encrypt-for-ssl)
  - [Configure UFW Firewall](#step-7-configure-ufw-firewall)
  - [Install Composer, Node, and Yarn](#step-8-install-composer-node-and-yarn)
  - [Clone the Repository](#step-9-clone-the-repository)
  - [Install WordPress with Bedrock](#step-10-install-wordpress-with-bedrock)
  - [Finalize the Installation](#step-11-finalize-the-installation)
  - [Secure Your WordPress Installation](#step-12-secure-your-wordpress-installation)
  - [Database Configuration](#step-13-database-configuration)
  - [Verify Everything is Working](#step-14-verify-everything-is-working)
  - [Regular Maintenance](#step-15-regular-maintenance)
- [Conclusion](#conclusion)

## Introduction

Deploying Roots/Bedrock WordPress to DigitalOcean involves setting up a server environment, installing necessary software, and configuring services to ensure a secure and efficient WordPress site.

## Prerequisites

- A DigitalOcean account.
- Basic knowledge of terminal and SSH.
- An Ubuntu 20.04 LTS Droplet.

## Step-by-Step Guide

### Step 1: Create a DigitalOcean Droplet

1. Log in to your DigitalOcean account.
2. Click "Create" and select "Droplets".
3. Choose Ubuntu 20.04 LTS as the image.
4. Select the appropriate size for your Droplet.
5. Choose a datacenter region.
6. Add your SSH keys for secure access.
7. Click "Create Droplet".

### Step 2: Connect to Your Droplet

_Assuming your public SSH key has been added to your DigitalOcean Droplet's `~/.ssh/authorized_keys`._

```bash
ssh root@your_droplet_ip
```

### Step 3: Update and Upgrade the System

```bash
sudo apt update && sudo apt upgrade -y
```

### Step 4: Install Nginx, PHP, and MySQL

```bash
sudo apt install nginx -y
sudo apt install php8.2 php8.2-fpm php8.2-mysql php8.2-xml php8.2-curl php8.2-mbstring php8.2-zip php8.2-gd -y
sudo apt install mysql-server -y
sudo mysql_secure_installation
```

### Step 5: Configure Nginx with Public IP Address

1. Open the Nginx configuration file for your site:

    ```bash
    sudo nano /etc/nginx/sites-available/wpbedrock
    ```

2. Add the server block configuration.

    ```bash
    server {
        listen 80;
        server_name your_droplet_ip; # or use a domain (this will need additional DNS records configuration)
        root /var/www/wp-bedrock/web;
    
        index index.php index.html index.htm;
    
        location / {
            try_files $uri $uri/ /index.php?$args;
        }
    
        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
        }
    }
    ```

3. Enable the site by creating a symbolic link:

   ```bash
   sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/
   ```

5. Test the Nginx configuration and reload:

   ```bash
   sudo nginx -t
   sudo systemctl reload nginx
   ```

### Step 6: Install Let's Encrypt for SSL

```bash
# Install Certbot and the Nginx plugin:
sudo apt install certbot python3-certbot-nginx -y

# Obtain an SSL certificate for your domain or (your_droplet public IP address):
sudo certbot --nginx -d (your-domain.com OR your_droplet_ip)
```

Follow the prompts to configure SSL.

### Step 7: Configure UFW Firewall

```bash
# Enable UFW:
sudo ufw enable

# Allow Nginx, SSH, and other necessary services:
sudo ufw allow 'Nginx Full'
sudo ufw allow OpenSSH
```

### Step 8: Install Composer, Node, and Yarn

```bash
sudo apt install php
sudo apt install php-cli
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
sudo chmod +x /usr/local/bin/composer
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
npm install --global yarn
```

<details>
<summary><strong>If you encounter any issues install PHP with the above try this.</strong></summary>

If you need to install a specific version of PHP (in this case, version 8.2.3), you'll need to ensure that the appropriate PHP version is available in your package manager's repositories. Since PHP 8.2.3 might not be available by default, you may need to add a third-party repository or compile PHP from source.

Here's how you can install PHP 8.2.3 on Ubuntu:

1. **Add Ondřej Surý's PHP PPA**:

    ```bash
    sudo add-apt-repository ppa:ondrej/php
    sudo apt update
    ```

2. **Install PHP 8.2.3**:

    ```bash
    sudo apt install php8.2
    ```

3. **Install PHP CLI**:

    ```bash
    sudo apt install php8.2-cli
    ```

4. **To make PHP 8.2.3 the default version on your system**:
   you can use the `update-alternatives` command to set the default PHP version. Here's how you can do it:

    1. **List Available PHP Versions**:
    
        First, list the available PHP versions on your system:
    
        ```bash
        update-alternatives --display php
        ```
    
        This command will show you the available PHP versions along with their paths.
    
    2. **Set PHP 8.2.3 as Default**:
    
        Once you've confirmed that PHP 8.2.3 is installed and listed, you can set it as the default version using the following command:
    
        ```bash
        sudo update-alternatives --set php /usr/bin/php8.2
        ```
    
        This command sets the PHP 8.2.3 executable (`php8.2`) as the default version.
    
    3. **Verify Default PHP Version**:
    
        To verify that PHP 8.2.3 is now the default version, you can check the PHP version again:
    
        ```bash
        php -v
        ```
    
        This should display PHP 8.2.3 as the installed version.

6. **Retry Composer Installation**:

    After installing PHP 8.2.3, you can retry installing Composer using the same command:

    ```bash
    curl -sS https://getcomposer.org/installer | php
    ```

7. **Move Composer to `/usr/local/bin`**:

    Once Composer is successfully downloaded, move it to the `/usr/local/bin` directory:

    ```bash
    sudo mv composer.phar /usr/local/bin/composer
    ```

By following these steps, you should be able to install PHP 8.2.3 and Composer on your system. If you encounter any issues or need further assistance, feel free to ask!
</details>

### Step 9: Clone the Repository

_To be able to clone a repository you will need to add DigitalOcean Droplet's public SSH key if already exists and if not below are the steps._

```bash
# Navigate to the web root directory:
cd /var/www/
```

<details>
<summary><strong>Configuring Git</strong></summary>

```bash
# Replace username and email address in the following steps with the ones you use with your GitHub account.
git config --global user.name "USERNAME"
git config --global user.email "YOUR@EMAIL.com"
ssh-keygen -t ed25519 -C "YOUR@EMAIL.com"

# Next take the newly generated SSH key and add it to your GitHub account.
cat ~/.ssh/id_ed25519.pub

# Check and see if it worked:
ssh -T git@github.com

# You should get a message like this:
Hi USERNAME! You've successfully authenticated, but GitHub does not provide shell access.
```

</details>

```bash
# Clone the repository recursively:
sudo git clone --recursive git@github.com:your-username/repo-name.git wp-bedrock
```

### Step 10: Install WordPress with Bedrock

Configure `.env` file with your environment settings in wp-bedrock.

```bash
cd wp-bedrock
composer install --ignore-platform-reqs

# Assuming you have a theme that uses composer and npm or yarn to install dependencies
cd web/app/themes/sage-theme
composer install --ignore-platform-reqs
yarn install && yarn build:production
```

### Step 11: Finalize the Installation

_Visit your domain or Droplet's public IP address in a web browser to complete the WordPress installation._

### Step 12: Secure Your WordPress Installation

Set proper permissions for files and directories:

```bash
sudo find /var/www/wp-bedrock/ -type d -exec chmod 750 {} \;
sudo find /var/www/wp-bedrock/ -type f -exec chmod 640 {} \;
```

Ensure that sensitive files are protected.

### Step 13: Database Configuration

**Note** you should use the `.env`database credentials

```bash
# Log in to MySQL:
sudo mysql -u root -p

# Create a database for WordPress (DB_NAME):
CREATE DATABASE wordpress;

# Create a MySQL user and grant privileges (DB_USER, DB_HOST, and DB_PASSWORD):
GRANT ALL ON wordpress.* TO 'wordpressuser'@'localhost' IDENTIFIED BY 'password';
FLUSH PRIVILEGES;
EXIT;
```

### Step 14: Verify Everything is Working

Ensure that all services are running:

```bash
sudo systemctl status nginx
sudo systemctl status php8.2-fpm
sudo systemctl status mysql
```

Check your website by visiting your domain or Droplet's public IP address.

### Step 15: Regular Maintenance

```bash
sudo apt update && sudo apt upgrade
```

Regularly back up your WordPress site and database.
Monitor your server's performance and security.

## Conclusion

By following these steps, you should have a fully functional Roots/Bedrock WordPress installation running on a DigitalOcean Droplet with Nginx, PHP, MySQL, and Let's Encrypt SSL. Remember to replace placeholders like `your-domain.com`, `your_droplet_ip`, `wordpress`, `wordpressuser`, and `password` with your actual IP address and desired database and user names. Regular maintenance and monitoring are essential to ensure the performance and security of your WordPress site.
