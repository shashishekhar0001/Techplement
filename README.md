# 🖥️ WordPress Deployment on AWS EC2 (Monolithic & Microservices)

## 📌 Objective

This project demonstrates how to deploy a WordPress website using:
1. **Monolithic Architecture** – WordPress and MySQL installed on a single EC2 instance.
2. **Microservices Architecture** – WordPress and MySQL deployed on two separate EC2 instances.

The final result includes a working WordPress homepage with a custom **Welcome Page**, and properly secured inter-instance communication using AWS Security Groups.

---

## ⚙️ Infrastructure & Configuration

- **Cloud Provider**: AWS
- **Operating System**: Ubuntu Server 22.04 LTS
- **Instance Type**: t2.micro
- **Key Pair**: PEM file (converted to PPK for Windows/PuTTY)
- **Connection Tool**: PuTTY (Windows)
- **Security**: Configured Security Groups for HTTP, SSH, and MySQL (internal)

---

## 🧱 Part 1: Monolithic Architecture (Single EC2 Instance)

### ✅ Steps

#### 1. Launch EC2 Instance
- Go to [AWS EC2 Console](https://console.aws.amazon.com/ec2)
- Launch a new instance using **Ubuntu Server 22.04 LTS**
- Type: `t2.micro`
- Configure Security Group:
  - SSH (22) from your IP
  - HTTP (80) from Anywhere

#### 2. Connect via PuTTY (Windows)
- Use **PuTTYgen** to convert `.pem` to `.ppk`
- Connect using public IP:
- Host: ubuntu@<PUBLIC_IP>
- 
#### 3. Install Apache, PHP, and MySQL

sudo apt update
sudo apt install apache2 mysql-server php php-mysql libapache2-mod-php php-cli -y



#### 4. Download and Set Up WordPress

cd /tmp
curl -O https://wordpress.org/latest.tar.gz
tar -xvzf latest.tar.gz
sudo rsync -av wordpress/ /var/www/html/
sudo chown -R www-data:www-data /var/www/html/


#### 5.Configure MySQL Database
sudo mysql

Inside MySQL shell:

CREATE DATABASE wordpress;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;


#### 6. Configure WordPress

cd /var/www/html
sudo mv wp-config-sample.php wp-config.php
sudo nano wp-config.php


Update:

define('DB_NAME', 'wordpress');
define('DB_USER', 'wpuser');
define('DB_PASSWORD', 'password');
define('DB_HOST', 'localhost');


#### 7. Restart Apache

sudo systemctl restart apache2


#### 8. Finish Installation
Open a browser → http://<EC2_PUBLIC_IP>

Complete the WordPress setup

Create a page called Welcome, set it as Homepage under Settings > Reading




## 🧱 Part 2: Microservices Architecture (Two EC2 Instances)

✅ Overview
EC2 #1 – MySQL Server

EC2 #2 – WordPress Server


#### Step 1: Launch Two EC2 Instances

Instance 1: mysql-server
Ubuntu 22.04, t2.micro

Security Group:

SSH (22): Your IP

MySQL (3306): Custom — allow from WordPress EC2 (use its private IP or SG)

Instance 2: wordpress-server
Ubuntu 22.04, t2.micro

Security Group:

SSH (22): Your IP

HTTP (80): Anywhere

#### Step 2: Configure MySQL Server

Connect via PuTTY:

ssh -i yourkey.pem ubuntu@<MYSQL_EC2_PUBLIC_IP>

Install MySQL:

sudo apt update
sudo apt install mysql-server -y

Allow Remote Access:

sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
Change:

ini

bind-address = 0.0.0.0


Restart MySQL:
sudo systemctl restart mysql

Create Database and User:

sudo mysql

CREATE DATABASE wordpress;
CREATE USER 'wpuser'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'%';
FLUSH PRIVILEGES;
EXIT;


#### Step 3: Configure WordPress Server
Connect via PuTTY:
ssh -i yourkey.pem ubuntu@<WORDPRESS_EC2_PUBLIC_IP>

Install Apache, PHP, and modules:
sudo apt update
sudo apt install apache2 php php-mysql libapache2-mod-php -y

Download and Install WordPress:

cd /tmp
curl -O https://wordpress.org/latest.tar.gz
tar -xvzf latest.tar.gz
sudo rsync -av wordpress/ /var/www/html/
sudo chown -R www-data:www-data /var/www/html/
Configure wp-config.php:

cd /var/www/html
sudo mv wp-config-sample.php wp-config.php
sudo nano wp-config.php

Set DB_HOST to private IP of MySQL EC2:


define('DB_NAME', 'wordpress');
define('DB_USER', 'wpuser');
define('DB_PASSWORD', 'password');
define('DB_HOST', 'PRIVATE_IP_OF_MYSQL_SERVER');


Restart Apache:
sudo systemctl restart apache2


Complete WordPress Setup:
Open browser → http://<WORDPRESS_EC2_PUBLIC_IP>

Complete installation

Create a Welcome page and set as homepage under Settings > Reading



