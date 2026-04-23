# 🗄️ Build a DB Server and Interact With It Using a Web App

In this lab, I set up a fully managed relational database on AWS using Amazon RDS and connected it to a live web application. The goal was to get hands-on experience with how cloud databases are provisioned, secured, and connected to real applications — no manual server management required.

---

## ☁️ Services Used

- **Amazon RDS (MySQL)** — managed relational database
- **Amazon VPC** — networking and subnet configuration
- **EC2 Security Groups** — controlling inbound/outbound traffic
- **AWS Management Console** — provisioning and configuration

---

## 🎯 What I Did

### Task 1: Created a Security Group for the RDS Instance

I navigated to VPC under Networking & Content Delivery and created a dedicated security group called **DB Security Group**. I then added an inbound rule on port **3306 (MySQL/Aurora)** that only allows traffic originating from the **Web Security Group** — locking down the database so only the web server can reach it.

![Security Group Created](screenshots/task1-security-group.png)

---

### Task 2: Created a DB Subnet Group

I set up a **DB Subnet Group** that spans two private subnets across two Availability Zones. This is a requirement for Multi-AZ deployments and ensures the database has the network coverage it needs for automatic failover.

![DB Subnet Group Created](screenshots/task2-subnet-group.png)

---

### Task 3: Launched a Multi-AZ RDS MySQL Instance

I configured and launched an Amazon RDS MySQL instance with the following setup:

| Setting | Value |
|---|---|
| DB Identifier | lab-db |
| Engine | MySQL |
| Template | Dev/Test |
| Availability | Multi-AZ DB Instance |
| Instance Class | db.t3.medium |
| Storage | General Purpose SSD |
| VPC | Lab VPC |
| Security Group | DB Security Group |
| Initial DB Name | lab |

Multi-AZ means AWS automatically creates a **primary instance and a standby replica** in a separate Availability Zone — if one goes down, the other takes over with no manual intervention needed.

Once the instance status changed to **Available**, I copied the RDS endpoint:
```
lab-db.c6t87nhuelzb.us-west-2.rds.amazonaws.com
```

![RDS Instance Available](screenshots/task3-rds-available.png)

---

### Task 4: Connected a Web App to the Database

Using the RDS endpoint, I configured a web application running on an EC2 instance to connect to the database. I entered the endpoint, database name, username, and password — and the app connected successfully.

I tested it end-to-end by **adding, editing, and removing contacts** in a live Address Book application. All data was being written to the RDS database and automatically replicated to the second Availability Zone in real time.

![Web App Connected to RDS](screenshots/task4-webapp-connected.png)
![Address Book Working](screenshots/task4-address-book.png)

---

## 💡 What I Learned

- How to provision a fully managed relational database on AWS without touching a single server
- Why **Multi-AZ deployments** matter — they provide automatic failover and high availability, which is critical for production workloads
- How **security groups** act as a firewall layer, and why restricting database access to only the web server is best practice
- How **DB subnet groups** work and why spanning multiple Availability Zones is required for resilient architectures
- How a real web application connects to a backend RDS database using an endpoint, credentials, and a database name

---

## ⚙️ Environment

- **Platform:** AWS Management Console
- **Database Engine:** MySQL on Amazon RDS
- **Deployment:** Multi-AZ for high availability
- **Duration:** ~45 minutes
