<div align="center">

# ☁️ AWS Re/Start Portfolio
### by J

![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![EC2](https://img.shields.io/badge/Amazon%20EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white)
![EBS](https://img.shields.io/badge/Amazon%20EBS-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)
![CloudWatch](https://img.shields.io/badge/Amazon%20CloudWatch-FF4F8B?style=for-the-badge&logo=amazoncloudwatch&logoColor=white)
![VPC](https://img.shields.io/badge/Amazon%20VPC-8C4FFF?style=for-the-badge&logo=amazon-aws&logoColor=white)

---

## ☁️ Lab: Introduction to Amazon EC2


## 📋 Table of Contents

- [Lab: Introduction to Amazon EC2](#-lab-introduction-to-amazon-ec2)
  - [Task 1: Launching My EC2 Instance](#-task-1-launching-my-ec2-instance)
  - [Task 2: Monitoring My Instance](#-task-2-monitoring-my-instance)
  - [Task 3: Security Groups & Accessing the Web Server](#-task-3-security-groups--accessing-the-web-server)
  - [Task 4: Resizing the Instance & EBS Volume](#-task-4-resizing-the-instance--ebs-volume)
  - [Task 5: Testing Termination Protection](#-task-5-testing-termination-protection)
- [AWS Services Used](#-aws-services-used)
- [Key Takeaways](#-key-takeaways)

---



### 🎯 Objectives

| # | Objective |
|---|-----------|
| 1 | Launch an EC2 instance with termination protection enabled |
| 2 | Deploy a web server using User Data scripts |
| 3 | Monitor EC2 instance health and performance |
| 4 | Configure security group rules to allow HTTP access |
| 5 | Resize an EC2 instance and modify EBS storage |
| 6 | Test and disable termination protection |
| 7 | Terminate an EC2 instance safely |

---

## ✅ Task 1: Launching My EC2 Instance

### 💡 What's the point?

I launched my first Amazon EC2 instance — basically a virtual computer living in AWS's cloud infrastructure. The big thing here was enabling **Termination Protection**, which stops anyone (including past-me who clicks too fast) from accidentally deleting the instance. I also attached a **User Data script** that automatically sets up a web server the moment the instance boots. Pretty cool that you can automate setup from day one.

---

### 🪜 Steps I Followed

**Step 1 — Log into the AWS Management Console**

I signed in using my **IAM Role** credentials with my existing email and password. IAM (Identity and Access Management) is AWS's way of controlling *who* can access *what* — instead of using the root account for everything (which is a big security no-no), IAM lets you create specific roles and users with limited permissions.

> 💬 *First time setting up AWS? Create your account at [aws.amazon.com](https://aws.amazon.com) and follow the IAM setup guide before anything else.*

---

**Step 2 — Navigate to EC2**

From the **Services** menu → searched **EC2** → landed on the **EC2 Dashboard**. This is your command centre. Everything you need to manage virtual servers lives here.

---

**Step 3 — Name the Instance**

Clicked **Launch Instance** → named it `Web Server`.

Under the hood, AWS stores this as a **key-value tag** (`Name: Web Server`). Tagging your resources properly is a habit worth building early — when you have 20 instances running, you'll thank yourself for naming them clearly.

---

**Step 4 — Choose an AMI (Amazon Machine Image)**

> 🖼️ *An AMI is basically a snapshot/blueprint for your server — it includes the operating system, pre-installed software, and configuration settings.*

I kept the default: **Amazon Linux 2023**. It's stable, AWS-native, and well-supported. No reason to change it for this lab.

---

**Step 5 — Choose Instance Type**

Selected **t3.micro** — 2 virtual CPUs, 1 GiB RAM.

| Instance Type | vCPU | Memory | Use Case |
|---------------|------|--------|----------|
| t3.micro | 2 | 1 GiB | ✅ Learning / light workloads |
| t3.small | 2 | 2 GiB | Small apps |
| m5.medium | 2 | 4 GiB | Production workloads |

For this lab, t3.micro is perfect. Small, cheap, and gets the job done.

---

**Step 6 — Key Pair**

Since I wasn't SSH-ing into this instance, I chose **"Proceed without a key pair."**

> ⚠️ *In real-world scenarios you'd ALWAYS create a key pair. It's your only way to securely connect to Linux instances. For this lab though, we don't need it.*

---

**Step 7 — Network Settings**

- **VPC:** Selected `Lab VPC`
- **Security Group Name:** `Web Server security group`
- **Description:** `Security group for my web server`
- **Removed** the default SSH inbound rule

A **Security Group** works like a virtual firewall — it controls what traffic can flow in and out of your instance. Removing SSH access here was intentional: if we're not using SSH, there's no point leaving that door open. Less exposure = better security. 🔒

---

**Step 8 — Storage**

Kept the default **8 GiB EBS (Elastic Block Store) volume**. EBS is the virtual hard drive attached to your EC2 instance — it stores your OS, files, and data even after a reboot.

---

**Step 9 — Advanced Details: Termination Protection + User Data** ⭐

This is where things got interesting.

Under **Advanced Details:**
- Set **Termination Protection** → `Enable` ✅

This is a safeguard. Once enabled, any attempt to terminate the instance will be **blocked** until you explicitly disable it first. Great habit for anything you care about keeping alive.

Then I pasted in this **User Data script:**

```bash
#!/bin/bash
yum -y install httpd
systemctl enable httpd
systemctl start httpd
echo '<html><h1>Hello From Your Web Server!</h1></html>' > /var/www/html/index.html
```

**What this script does, line by line:**

| Line | What It Does |
|------|-------------|
| `yum -y install httpd` | Installs Apache web server |
| `systemctl enable httpd` | Makes Apache start automatically on every reboot |
| `systemctl start httpd` | Starts Apache right now |
| `echo '...' > index.html` | Creates a simple HTML webpage |

This script runs *automatically* the first time the instance boots — zero manual setup needed. That's the beauty of User Data.

---

**Step 10 — Launch!**

Hit **Launch Instance** → **View All Instances**.

Watched the status go:

```
Pending ⏳ → Running ✅ → 2/2 checks passed 🟢
```

Once I saw **2/2 checks passed**, the instance was fully healthy and ready to go.

---

## 📊 Task 2: Monitoring My Instance

### 💡 What's the point?

You can't manage what you can't see. AWS gives you built-in monitoring tools through **Amazon CloudWatch** — no setup required. This task was about learning where to look when something goes wrong (or right!).

---

### 🪜 Steps I Followed

**Step 1 — Status Checks Tab**

Selected my instance → clicked the **Status Checks** tab.

Two checks run automatically on every EC2 instance:

| Check | What It Tests |
|-------|--------------|
| ✅ System reachability | Is the underlying AWS hardware healthy? |
| ✅ Instance reachability | Is the OS on MY instance responding? |

Both passed. 

---

**Step 2 — Monitoring Tab (CloudWatch)**

Clicked the **Monitoring** tab to see live CloudWatch metrics — CPU usage, network traffic, disk reads, etc.

> 💬 *Since I'd just launched the instance, there wasn't much data yet. But this is where you'd spot a CPU spike, memory issue, or unusual network traffic in a real scenario.*

By default, EC2 sends metrics to CloudWatch every **5 minutes** (Basic Monitoring). You can upgrade to **1-minute intervals** with Detailed Monitoring if you need faster visibility.

---

**Step 3 — Instance Screenshot**

Went to **Actions → Monitor and Troubleshoot → Get Instance Screenshot.**

This captures what the instance's screen would look like *if it had a monitor attached*. Super useful when you can't SSH in and need to see if the OS even booted correctly.

> 💬 *Think of it as the AWS equivalent of walking over to a server and looking at its screen — except you're doing it from anywhere in the world.*

---

## 🔐 Task 3: Security Groups & Accessing the Web Server

### 💡 What's the point?

My web server was running — but nobody could reach it. That's because security groups work on a **default deny** principle: all inbound traffic is blocked unless you explicitly allow it. This task was about opening up port 80 (HTTP) so browsers could actually load the webpage.

---

### 🪜 Steps I Followed

**Step 1 — Try to Access the Web Server (It Fails)**

From the **Details** tab, I copied the **Public IPv4 address** → pasted it into a new browser tab → pressed Enter.

**Result: Nothing loaded.** ❌

This is expected! The security group had no inbound rules, so all traffic was being silently dropped. The server was running perfectly — the firewall just wasn't letting anyone through.

---

**Step 2 — Update the Security Group**

Navigated to **Security Groups** in the left sidebar → selected `Web Server security group` → **Inbound Rules** tab → **Edit Inbound Rules** → **Add Rule:**

| Setting | Value |
|---------|-------|
| Type | HTTP |
| Source | Anywhere-IPv4 (0.0.0.0/0) |

Clicked **Save Rules**. ✅

**Port 80 (HTTP)** is now open. Any browser in the world can now reach my web server.

---

**Step 3 — Refresh the Browser Tab**

Went back to the browser tab with my IP address → refreshed the page.

```
Hello From Your Web Server!
```

**It works!** 🎉 That HTML page from the User Data script — live on the internet.

---

> 💬 *This was a lightbulb moment for me. The server was running the whole time — it was the security group acting as the gatekeeper. Understanding that separation between "is the app running?" and "can traffic reach it?" is huge in cloud networking.*

---

## 📦 Task 4: Resizing the Instance & EBS Volume

### 💡 What's the point?

One of the biggest advantages of cloud computing is **elasticity** — you can scale resources up or down based on your actual needs. This task showed me how to upgrade both the compute power (instance type) and storage (EBS volume) of a running instance.

---

### 🪜 Steps I Followed

**Step 1 — Stop the Instance**

You can't resize a running instance, so first I had to stop it:

**Actions → Instance State → Stop Instance** → confirmed → waited for status: `Stopped` ⏹️

> 💬 *Stopping ≠ Terminating. Stopped instances don't get charged for compute, but your EBS storage charges still apply. The data is all still there — it's just "off."*

---

**Step 2 — Change the Instance Type**

**Actions → Instance Settings → Change Instance Type**

| Before | After |
|--------|-------|
| t3.micro (1 GiB RAM) | t3.small (2 GiB RAM) |

Selected `t3.small` → **Change Instance Type** ✅

Double the memory for when the workload demands more.

---

**Step 3 — Resize the EBS Volume**

Left nav → **Elastic Block Store → Volumes** → selected my volume → **Actions → Modify Volume**

| Before | After |
|--------|-------|
| 8 GiB | 10 GiB |

Changed size to `10` → **Modify** → confirmed. ✅

> 💬 *Important: EBS volumes can be expanded while the instance is stopped (or even running!) but you can only increase the size — you can't shrink it. Plan ahead!*

---

**Step 4 — Start the Instance Again**

Left nav → **Instances** → selected `Web Server` → **Instance State → Start Instance** ▶️

The instance came back online with upgraded specs:
- **Instance type:** t3.small ✅
- **Storage:** 10 GiB ✅

---

## 🛡️ Task 5: Testing Termination Protection

### 💡 What's the point?

Earlier I enabled termination protection — this task was about *testing* that it actually works, then properly disabling it before a clean shutdown.

---

### 🪜 Steps I Followed

**Step 1 — Try to Terminate (Watch It Fail)**

Selected `Web Server` instance → **Instance State → Terminate (delete) instance** → confirmed.

**Result:**

```
❌ Failed to terminate an instance: The instance 'i-xxxxxxxxxx' may not be terminated. 
Modify its 'disableApiTermination' instance attribute and try again.
```

Termination protection did exactly what it was supposed to do. The instance refused to die. 💪

---

**Step 2 — Disable Termination Protection**

**Actions → Instance Settings → Change Termination Protection** → unchecked `Enable` → **Save** ✅

---

**Step 3 — Terminate the Instance**

**Actions → Instance State → Terminate Instance** → **Terminate** ✅

The instance status moved to `Shutting down` → `Terminated`. Clean exit. 

---

> 💬 *This felt like a full circle moment. I launched it, watched it run, resized it, and then properly shut it down — going through the correct process instead of just yanking the plug. That discipline matters in production environments.*

---

## 🛠️ AWS Services Used

| Service | What I Used It For |
|---------|-------------------|
| ![EC2](https://img.shields.io/badge/Amazon%20EC2-FF9900?style=flat&logo=amazonec2&logoColor=white) | Launched and managed my virtual server |
| ![EBS](https://img.shields.io/badge/Amazon%20EBS-FF9900?style=flat&logo=amazon-aws&logoColor=white) | Block storage attached to my EC2 instance |
| ![CloudWatch](https://img.shields.io/badge/CloudWatch-FF4F8B?style=flat&logo=amazoncloudwatch&logoColor=white) | Monitored instance health and performance metrics |
| ![VPC](https://img.shields.io/badge/Amazon%20VPC-8C4FFF?style=flat&logo=amazon-aws&logoColor=white) | Isolated network environment for my instance |
| ![Security Groups](https://img.shields.io/badge/Security%20Groups-232F3E?style=flat&logo=amazon-aws&logoColor=white) | Controlled inbound/outbound traffic (virtual firewall) |

---

## 🧠 Key Takeaways

```
💡 EC2 is just a virtual machine in the cloud — but the power is in everything around it.
💡 Security Groups = virtual firewall. Default deny. Always open only what you need.
💡 Termination Protection is a small setting with a massive consequence if ignored.
💡 User Data scripts let you automate server setup from the very first boot.
💡 The cloud is elastic — resize up, resize down, pay only for what you use.
💡 Stopping ≠ Terminating. Know the difference before you click anything.
```

---

<div align="center">

---

*Built with curiosity, caffeine, and a lot of AWS console tabs open simultaneously* ☕

![Made with AWS](https://img.shields.io/badge/Made%20with-AWS-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)
![AWS ReStart](https://img.shields.io/badge/AWS-Re%2FStart%20Graduate-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=FF9900)

</div>
