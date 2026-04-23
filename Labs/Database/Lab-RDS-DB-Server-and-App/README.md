# 🗄️ Build a DB Server and Interact With It Using a Web App

In this lab, I set up a fully managed relational database on AWS using Amazon RDS and connected it to a live web application. The goal was to get hands-on experience with how cloud databases are provisioned, secured, and used in a real app environment — no manual database administration needed.

---

## ☁️ Services Used

- **Amazon RDS (MySQL)** — managed relational database
- **Amazon VPC** — networking and subnet configuration
- **EC2 Security Groups** — controlling inbound/outbound traffic
- **AWS Management Console** — provisioning and configuration

---

## 🎯 What I Did

### 1. Created a Security Group for the RDS Instance
I started by setting up a dedicated security group called **DB Security Group** inside the Lab VPC. I configured an inbound rule on port **3306 (MySQL/Aurora)** that only allows traffic from the **Web Security Group** — meaning only the web server can talk to the database, nothing else.

### 2. Created a DB Subnet Group
I set up a **DB Subnet Group** spanning two private subnets across two Availability Zones:
- `10.0.1.0/24` — Private Subnet 1
- `10.0.3.0/24` — Private Subnet 2

This is a requirement for Multi-AZ deployments and ensures the database has failover coverage.

### 3. Launched a Multi-AZ RDS MySQL Instance
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

Multi-AZ means AWS automatically creates a **primary instance and a standby replica** in a separate Availability Zone — so if one goes down, the other takes over automatically.

### 4. Connected a Web App to the Database
Once the instance was available, I grabbed the **RDS endpoint** and used it to configure a web application running on an EC2 instance. I entered the endpoint, database name, username, and password — and the app connected successfully.

I then tested it by **adding, editing, and removing contacts** in an Address Book app. All data was being written to the RDS database and automatically replicated to the second Availability Zone in real time.

---

## 💡 What I Learned

- How to provision a managed relational database on AWS without touching a single server
- Why **Multi-AZ deployments** matter — they provide automatic failover and high availability, which is critical for production workloads
- How **security groups** act as a firewall layer, and why restricting database access to only the web server (instead of the open internet) is best practice
- How **DB subnet groups** work and why spanning multiple Availability Zones is a requirement for resilient database architectures
- How a real web application communicates with a backend RDS database using an endpoint, credentials, and a database name

---

## ⚙️ Environment

- **Platform:** AWS Management Console
- **Database Engine:** MySQL on Amazon RDS
- **Deployment:** Multi-AZ for high availability
