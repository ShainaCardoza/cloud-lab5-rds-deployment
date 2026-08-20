Lab 5: AWS RDS Deployment and Connection for Cloud Akaunting Application

EC2 URL / API base URL 
http://13.50.107.182

---

1. Project Overview & Architecture
This project demonstrates the migration of the Akaunting web application database layer from a local MariaDB instance to a managed Amazon RDS MariaDB/MySQL instance. 

Application Host: AWS EC2 (Ubuntu 24.04 LTS, Nginx, PHP 8.3-FPM)
Database Service: Amazon RDS (MariaDB Engine)
Live Application URL: http://13.50.107.182

---

2. Amazon RDS Deployment Settings
DB Instance Identifier: `akaunting-rds-db`
Engine: MariaDB (Free Tier `db.t3.micro`)
Endpoint: `akaunting-rds-db.chuu006sofv4.eu-north-1.rds.amazonaws.com`
Port:** `3306`
Database Name: `akaunting`
Database Master User: `akaunting_user`

---

3. Security Configuration
Principle of Least Privilege: Amazon RDS is isolated inside the default VPC.
Security Group Rules: The RDS Security Group (`rds-db-sg`) accepts inbound traffic on Port 3306 exclusively from the EC2 Security Group ID.
Zero Public Access: Inbound connections from `0.0.0.0/0` on port 3306 are disabled.

---

4. Database Migration & Connection Steps

Step 1: Export Local Schema & Data
sudo mariadb-dump -u root akaunting > /tmp/akaunting_backup.sql

Step 2: Stream Data to Amazon RDS
mariadb -h akaunting-rds-db.chuu006sofv4.eu-north-1.rds.amazonaws.com \ -u akaunting_user -p akaunting < /tmp/akaunting_backup.sql

Step 3: Configure SSL Certificate Bundle
sudo wget [https://truststore.pki.rds.amazonaws.com/global/global-bundle.pem](https://truststore.pki.rds.amazonaws.com/global/global-bundle.pem) \
  -O /var/www/akaunting/rds-ca-bundle.pem

Step 4: Update Application Configuration (.env)
DB_CONNECTION=mysql
DB_HOST=akaunting-rds-db.chuu006sofv4.eu-north-1.rds.amazonaws.com
DB_PORT=3306
DB_DATABASE=akaunting
DB_USERNAME=akaunting_user
DB_PASSWORD="<RDS_MASTER_PASSWORD>"
DB_PREFIX=cja_
MYSQL_ATTR_SSL_CA=/var/www/akaunting/rds-ca-bundle.pem

Step 5: Flush Caches and Restart Web Server
cd /var/www/akaunting
sudo php artisan config:clear
sudo php artisan cache:clear
sudo systemctl restart php8.3-fpm nginx

---

5. CRUD Demonstration Summary
Create (C): Successfully added new invoices and customer entries.

Read (R): Dashboard loads receivables and balance sheets dynamically from RDS.

Update (U): Modified invoice pricing and item lines directly reflected in RDS tables.

Delete (D): Successfully removed test accounting records via the web interface.
