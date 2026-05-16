# Production Deployment Using AWS Elastic Beanstalk with Secure RDS Integration

## Overview

This project demonstrates deploying a Python Flask Task Manager application on AWS using Elastic Beanstalk, with a secure MySQL RDS database integrated inside a custom VPC. Database credentials are stored securely using AWS Secrets Manager.

---

## Table of Contents

1. [Application Development](#step-1-application-development-local-setup)
2. [AWS Network Setup](#step-2-aws-network-setup-vpc-architecture)
3. [Security Groups](#step-3-create-security-groups)
4. [RDS Database](#step-4-create-rds-database)
5. [Secrets Manager](#step-5-store-credentials-in-secrets-manager)
6. [IAM Role](#step-6-create-iam-role-for-elastic-beanstalk)
7. [Deploy with Elastic Beanstalk](#step-7-deploy-application-using-elastic-beanstalk)
8. [Access Application](#step-8-access-application)
9. [Outcome](#outcome)

---

## Architecture

- Application runs inside Elastic Beanstalk (EC2)
- RDS MySQL deployed within the same VPC
- Database access restricted via security groups
- EC2 instance connects securely to RDS
- Credentials stored securely in AWS Secrets Manager

---

## Step 1: Application Development (Local Setup)

The web application was developed and tested locally before deploying to the cloud.

**Technologies Used:** Python, Flask, MySQL, HTML

**Project Folder Structure:**

```
task-manager-app/
    application.py
    requirements.txt
    procfile
    templates/
        index.html
```

**Commands for Local Run:**

```bash
# Install dependencies
pip install flask pymysql

# Run the application
python application.py

# Open in browser
http://127.0.0.1:5000
```

> The application was fully tested locally before any cloud deployment.



**Screenshot - Local Application Running:**

<img width="941" height="354" alt="image" src="https://github.com/user-attachments/assets/f9dbca6f-e849-4b8e-bac8-ced3935ad02e" />


<!-- Add screenshot here: local_app.png -->

---

## Step 2: AWS Network Setup (VPC Architecture)

A custom network architecture was created using VPC.

**VPC Created:** `tech-vpc` — CIDR: `10.0.0.0/16`


**Subnets Created:**

| Subnet Name | Type    | CIDR Block   |
|-------------|---------|--------------|
| Web Subnet  | Public  | 10.0.1.0/24  |
| App Subnet  | Private | 10.0.2.0/24  |
| DB Subnet 1 | Private | 10.0.3.0/24  |
| DB Subnet 2 | Private | 10.0.4.0/24  |


**Internet Gateway:** Created and attached to `tech-vpc`


**Route Table:** Public route table created and associated with the Web Subnet.


---

## Step 3: Create Security Groups

Navigate to AWS EC2 > Security Groups.

**1. Application Security Group: `task-app-sg`**

Inbound Rules:

| Type  | Port | Source    |
|-------|------|-----------|
| HTTP  | 80   | 0.0.0.0/0 |
| SSH   | 22   | My IP     |

Purpose: Allow users to access the application.

**Screenshot - App Security Group:**

<img width="940" height="205" alt="image" src="https://github.com/user-attachments/assets/de8c1339-00b0-4b1e-9439-c64a9889b032" />


<!-- Add screenshot here: app_sg.png -->

**2. Database Security Group: `task-db-sg`**

Inbound Rules:

| Type  | Port | Source       |
|-------|------|--------------|
| MySQL | 3306 | task-app-sg  |

Purpose: Allow only the application server to access the database.

**Screenshot - DB Security Group:**

<img width="940" height="147" alt="image" src="https://github.com/user-attachments/assets/53d4c49e-f639-4c4f-ac2c-6aefafdbc7c4" />


---

## Step 4: Create RDS Database

**Database Configuration:**

| Setting        | Value            |
|----------------|------------------|
| Engine         | MySQL            |
| Database Name  | taskdb           |
| Instance Class | Free Tier        |
| VPC            | tech-vpc         |
| Subnet Group   | db-subnet-group  |
| Security Group | task-db-sg       |
| Public Access  | No               |



**DB Subnet Group Created:**



---

## Step 5: Store Credentials in Secrets Manager

Database credentials were stored securely in AWS Secrets Manager instead of hardcoding them in the application.

Steps:
1. Go to AWS Secrets Manager
2. Choose "Store a new secret"
3. Select "Credentials for RDS database"
4. Enter the database username and password
5. Select the RDS instance
6. Name the secret (e.g., `prod/taskdb/credentials`)
7. Save

---

## Step 6: Create IAM Role for Elastic Beanstalk

1. Go to IAM > Roles > Create Role
2. Select EC2 as the trusted entity
3. Attach the following policy: `SecretsManagerReadWrite`
4. Name the role: `aws-elasticbeanstalk-ec2-role`

Purpose: Allows the application running on EC2 to retrieve database credentials from Secrets Manager securely.


---

## Step 7: Deploy Application Using Elastic Beanstalk

1. Create application: `task-manager-app`
2. Create environment: `task-manager-env`
3. Platform: Python
4. Upload the application package: `task-manager-app.zip`
5. Configure networking:
   - VPC: `tech-vpc`
   - Subnet: Web Subnet (public)
   - Security Group: `task-app-sg`
6. Assign IAM instance profile: `aws-elasticbeanstalk-ec2-role`


<img width="940" height="469" alt="image" src="https://github.com/user-attachments/assets/b2bd9a15-aedc-48bf-af28-10cf667fc8ac" />




**Screenshot - Beanstalk Configuration:**

<!-- Add screenshot here: eb_config.png -->

---

## Step 8: Access Application

Elastic Beanstalk automatically generates a domain URL after deployment.

**Application URL:**
```
http://task-manager-env.eba-zvctkgip.ap-south-1.elasticbeanstalk.com/
```

**Screenshot - Application Live:**

<img width="940" height="267" alt="image" src="https://github.com/user-attachments/assets/ad283027-6912-4ffc-9df4-8c27f136c37d" />

<img width="940" height="343" alt="image" src="https://github.com/user-attachments/assets/f793cb81-1ffa-49df-890c-db583ad84848" />



An EC2 instance is automatically created by Elastic Beanstalk. The application can also be accessed via the EC2 instance's public IP.


**Screenshot - Access via Public IP:**

<img width="940" height="309" alt="image" src="https://github.com/user-attachments/assets/45cd575c-6a69-43a7-a569-886eda8e69f7" />


---

## Outcome

The Python Task Manager application was successfully deployed on AWS with:

- Secure networking using a custom VPC with public and private subnets
- Managed deployment using Elastic Beanstalk
- Secure database access restricted by security groups
- Credentials managed securely using AWS Secrets Manager
- Scalable and production-ready architecture using AWS services

---

