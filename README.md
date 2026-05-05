# moodle-server-deployment
Azure VM Moodle Server with HTTPS

## Overview
This documentation provides step-by-step instructions to deploy a Moodle Learning Management System on an Azure Virtual Machine using Nginx, a custom domain, and HTTPS secured via Let's Encrypt SSL.
This guide is designed to allow rapid re-deployment of the server in case of failure or deletion.

## Architecture

- Cloud Provider: Microsoft Azure
- OS: Ubuntu 22.04 LTS
- Web Server: Nginx
- Application: Moodle
- Database: MySQL
- SSL: Let's Encrypt (Certbot)
- Domain: <your-domain.com>

## Prerequisites

- Azure account
- Domain name
- SSH client
- Basic Linux command knowledge

## Deployment Workflow

1. Create Azure account
2. Create Virtual Machine
3. Connect to VM via SSH
4. Install Nginx
5. Configure domain
6. Enable HTTPS (SSL)
7. Install MariaDB
8. Install PHP
9. Install Moodle
10. Configure Nginx for Moodle

# Step 1: Create Azure Account
https://portal.azure.com

# Step 2: Create Virtual Machine

A Linux virtual machine was created with the following configuration:

- OS: Ubuntu 22.04 LTS
- Size: B1s
- Public IP: Enabled

Inbound ports opened:
- 22 (SSH)
- 80 (HTTP)
- 443 (HTTPS)

# Step 3: Connect to Virtual Machine

ssh azureuser@[your-public-ip]


# Step 4: Install Nginx

Before configuring the server, the Nginx web server must be installed to handle HTTP requests.

sudo apt update
sudo apt install nginx -y

Once installed, start and enable the service to ensure it runs on boot.

sudo systemctl start nginx
sudo systemctl enable nginx


# Step 5: Domain Registration and Configuration (Cloudflare)

A domain name was registered using Cloudflare and configured to point to the Azure Virtual Machine. This allows access to the server using a domain name instead of a public IP address.

## Step 1: Register a Domain

1. Log in to the Cloudflare dashboard:
   https://dash.cloudflare.com

2. From the sidebar, click:
   - **Domain Registration**

3. Search for your desired domain name.

4. Select an available domain and click **Purchase**.

5. Complete the payment process.

Once completed, the domain will appear in your Cloudflare dashboard.


## Step 2: Add Domain to Cloudflare (if not automatically added)

1. Click **Add a Site**
2. Enter your domain name (e.g., yourdomain.com)
3. Select the free plan (or as required)
4. Click **Continue**

Cloudflare will scan for existing DNS records.

### Step 3: Configure DNS Records

1. Go to the **DNS** tab
2. Click **Add record**

Enter the following:

- Type: A  
- Name: @  
- IPv4 address: <your-public-ip>  
- Proxy status: DNS only (important for SSL setup)

Click **Save**

### Step 4: Add www Record (Optional but recommended)

Click **Add record** again:

- Type: A  
- Name: www  
- IPv4 address: <your-public-ip>  
- Proxy status: DNS only  

Click **Save**

### Step 5: Verify DNS Propagation

Wait a few minutes for DNS to propagate.

Test connectivity using SSH:

ssh azureuser@yourdomain.com


# Step 6: HTTPS Configuration using Let's Encrypt

To generate the certificate, navigate to:
https://certbot.eff.org/
Select:
Web server: Nginx
Operating system: Linux (snap)

Follow the provided instructions to install and configure Certbot. The commands are executed directly in the terminal to automatically generate and apply the SSL certificate.

Once the process is complete, HTTP traffic is redirected to HTTPS to ensure secure communication.
To confirm that HTTPS has been successfully enabled, open the website in a web browser using:

https://yourdomain-name-goes-here.com

A padlock icon should appear in the address bar, indicating that the connection is secure. Clicking on the padlock allows verification of the certificate issuer and details.

#step 7: Install and Configure Database (MariaDB)
Install MariaDB, which will store all Moodle data like users, courses, and files.
sudo apt install mariadb-server mariadb-client -y

Start and enable service:
sudo systemctl start mariadb
sudo systemctl enable mariadb

Secure MariaDB
sudo mysql_secure_installation

Create a dedicated database and user for Moodle to improve security.
sudo mysql -u root -p
CREATE DATABASE moodle DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'moodleuser'@'localhost' IDENTIFIED BY 'StrongPassword';
GRANT ALL PRIVILEGES ON moodle.* TO 'moodleuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;

#Step 8: Install PHP and required extensions so Moodle can run properly.
sudo apt install php-fpm php-mysql php-xml php-curl php-zip php-gd php-mbstring php-intl php-soap php-cli unzip git -y

sudo systemctl restart php8.1-fpm

#Step 9: Install Moodle
Download Moodle source code from GitHub and use a stable version
cd /var/www/
sudo git clone https://github.com/moodle/moodle.git
cd moodle
sudo git branch --track MOODLE_401_STABLE origin/MOODLE_401_STABLE
sudo git checkout MOODLE_401_STABLE

Create Moodle Data Directory
sudo mkdir /var/moodledata
sudo chown -R www-data:www-data /var/moodledata
sudo chmod -R 770 /var/moodledata

Set Permissions
Give the web server permission to access Moodle files.
sudo chown -R www-data:www-data /var/www/moodle
sudo chmod -R 755 /var/www/moodle


#Step 10: Configure Nginx for Moodle
onfigure Nginx to serve Moodle and connect it with PHP and HTTPS.

sudo nano /etc/nginx/sites-available/yourdomain


sudo nginx -t
sudo systemctl reload nginx

#Step 11: Run Moodle Installer
Complete the setup using the web interface in your browser.
   https://vle-edu.org

Choose language
Enter database details
Create admin account






