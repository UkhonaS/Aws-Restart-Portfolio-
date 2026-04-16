<div align="center">

# 🔧 Troubleshooting a Network Issue
### AWS Cloud Support Engineer Lab

![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![VPC](https://img.shields.io/badge/Amazon%20VPC-8C4FFF?style=for-the-badge&logo=amazon-aws&logoColor=white)
![EC2](https://img.shields.io/badge/Amazon%20EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white)
![Apache](https://img.shields.io/badge/Apache-D22128?style=for-the-badge&logo=apache&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)

---

> *"This lab put me in the shoes of a real AWS Cloud Support Engineer — someone sends you a broken network and it's your job to figure out why. No hints, just digging."*

---

</div>

## 🧠 The Scenario

Imagine you just clocked in for your shift as a **Cloud Support Engineer at AWS**. You open your inbox and there's this email:

---

> 📧 **From:** Ana (Contractor)
>
> *"Hello, Cloud Support! When I create an Apache server through the command line, I cannot ping it. I also get an error when I enter the IP address in the browser. Can you please help figure out what is blocking my connection? Thanks! — Ana"*

---

Ana's got an Apache web server running inside an AWS VPC — but nothing can reach it. No ping, no browser response, nothing. Your job? Find what's broken and fix it.

Her setup looks like this:

```
🌐 Internet
      |
  [Internet Gateway]
      |
  [Public Subnet]
      |
  [EC2 Instance]  ← Apache running here, but unreachable
```

**VPC → Internet Gateway → Public Subnet → EC2 Instance**

Sounds simple. But somewhere in that chain, something's blocking the traffic. Let's find it. 🔍

---

## 📋 Table of Contents

- [Objectives](#-objectives)
- [Task 1: SSH into the EC2 Instance](#-task-1-ssh-into-the-ec2-instance)
- [Task 2: Install & Start Apache (httpd)](#-task-2-install--start-apache-httpd)
- [Task 3: Investigate the VPC Configuration](#-task-3-investigate-the-vpc-configuration)
  - [Check: Subnets & Route Tables](#check-subnets--route-tables)
  - [Check: Internet Gateway](#check-internet-gateway)
  - [Check: Security Groups & Network ACLs](#check-security-groups--network-acls)
- [The Fix](#-the-fix)
- [Key Takeaways](#-key-takeaways)

---

## 🎯 Objectives

| # | Objective |
|---|-----------|
| 1 | Analyze a real customer networking scenario |
| 2 | SSH into an EC2 instance via command line |
| 3 | Install and start an Apache HTTP server |
| 4 | Investigate VPC components to find the misconfiguration |
| 5 | Restore connectivity to the web server |

**⏱ Lab Duration:** ~1 hour

---

## 🖥️ Task 1: SSH into the EC2 Instance

### 💡 What's the point?

Before I could investigate anything, I needed to actually *get into* the EC2 instance. That means connecting via **SSH (Secure Shell)** — the standard way to remotely access Linux servers. Think of it as opening a terminal window that lives inside Ana's server.

---

### 🪜 Steps I Followed

**If you're on macOS/Linux:**

**Step 1 — Download the PEM key file**

From the lab credentials panel, I downloaded `labsuser.pem` — this is the private key that proves I'm allowed to connect. Without it, access denied.

**Step 2 — Navigate to the download location**

```bash
cd ~/Downloads
```

**Step 3 — Lock down the key file permissions**

```bash
chmod 400 labsuser.pem
```

> 💬 *This is important. SSH will actually refuse to connect if your key file has permissions that are too open. `chmod 400` makes it readable only by you — nobody else.*

**Step 4 — Connect to the instance**

```bash
ssh -i labsuser.pem ec2-user@<PUBLIC-IP-ADDRESS>
```

Replace `<PUBLIC-IP-ADDRESS>` with the actual public IP from the lab details panel.

Type `yes` when prompted about the fingerprint — you're confirming you trust this server for the first time.

**You're in.** ✅

---

**If you're on Windows:**

Download the `.ppk` file from the credentials panel and use **PuTTY** to connect. PuTTY is a free SSH client for Windows — [download it here](https://www.putty.org/).

---

> 💬 *SSH felt intimidating the first time I saw it. But once you understand it's just "open a terminal on a remote computer using a key instead of a password" — it clicks. The key file IS your password, just in a different form.*

---

## ⚙️ Task 2: Install & Start Apache (httpd)

### 💡 What's the point?

Before I could troubleshoot Ana's issue, I needed to replicate it in my own environment — an exact copy of her VPC. That meant getting Apache (called `httpd` on Linux) installed and running, then watching what happened when I tried to access it.

---

### 🪜 Steps I Followed

**Step 1 — Check if httpd is already installed**

```bash
sudo systemctl status httpd.service
```

Output showed:

```
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled)
   Active: inactive (dead)
```

Installed — but not running. Makes sense for a fresh instance.

**Step 2 — Start the Apache service**

```bash
sudo systemctl start httpd.service
```

**Step 3 — Verify it's now running**

```bash
sudo systemctl status httpd.service
```

Output now shows:

```
● httpd.service - The Apache HTTP Server
   Active: active (running) ✅
```

Apache is live on the server. 

**Step 4 — Try to access it in a browser**

Opened a new tab and entered:

```
http://<PUBLIC-IP-OF-INSTANCE>
```

**Result: Page won't load.** ❌

The server is running. The browser can't reach it. Something between the internet and the instance is blocking the traffic — and that something lives in the **VPC configuration**.

---

> 💬 *This was the moment the investigation really started. The app works fine. The network doesn't. Classic cloud support scenario.*

---

## 🔍 Task 3: Investigate the VPC Configuration

### 💡 What's the point?

AWS networking has multiple layers, and a misconfiguration at any one of them can silently drop all your traffic. I needed to go through each component methodically — like a checklist — until I found the culprit.

Here's the mental model I used:

```
Traffic flow:  Internet → IGW → Route Table → Subnet → Security Group → NACL → EC2
```

Any one of those steps could be broken. Time to check them all.

---

Navigated to the **VPC Console:**
`Services → Networking & Content Delivery → VPC`

---

### Check: Subnets & Route Tables

**What I checked:**
- Is the subnet marked as **public**?
- Is the correct **route table** associated with the subnet?
- Does the route table have a route to the internet?

A public subnet needs a route table with an entry like this:

| Destination | Target |
|-------------|--------|
| `0.0.0.0/0` | `igw-xxxxxxxx` (Internet Gateway) |
| `10.0.0.0/16` | `local` |

The `0.0.0.0/0 → igw` route is what says *"send all internet-bound traffic to the Internet Gateway."* Without it, your instance is on a private island with no bridge to the outside world.

> 💬 *Route tables are easy to overlook because they're not visually obvious — but they are the actual "road map" that tells traffic where to go. If there's no route to the internet gateway, nothing gets in or out, period.*

---

### Check: Internet Gateway

**What I checked:**
- Does an Internet Gateway exist?
- Is it **attached** to the VPC?

An IGW that exists but isn't attached to the VPC does absolutely nothing. It's like having a door that's not connected to any wall.

**Steps:**
`VPC Console → Internet Gateways → check the State column`

| State | Meaning |
|-------|---------|
| ✅ Attached | IGW is connected to the VPC — traffic can flow |
| ❌ Detached | IGW exists but does nothing |

---

### Check: Security Groups & Network ACLs

This is where I found the issue. 🎯

**Security Groups** are stateful firewalls attached to your EC2 instance. They control what traffic is allowed **in** (inbound rules) and **out** (outbound rules).

**The problem:** The security group had no inbound rule for **HTTP (port 80).**

Apache serves web traffic on **port 80** by default. If port 80 is blocked at the security group level, no browser request ever reaches the server — even if everything else is configured perfectly.

**The Fix — Adding the missing inbound rule:**

`VPC Console → Security Groups → select the instance's security group → Inbound Rules → Edit Inbound Rules → Add Rule`

| Type | Protocol | Port | Source |
|------|----------|------|--------|
| HTTP | TCP | 80 | 0.0.0.0/0 (Anywhere-IPv4) |

Clicked **Save Rules.** ✅

> 💬 *Security groups follow a "default deny" model — if there's no rule explicitly allowing something, it's blocked. This is great for security but means you need to consciously open every port you want to use. Port 80 for HTTP, port 443 for HTTPS, port 22 for SSH — each one needs its own rule.*

---

**Also checked — Network ACLs (NACLs):**

NACLs are stateless firewalls that operate at the **subnet level** — one layer above security groups. Unlike security groups, they evaluate both inbound AND outbound rules independently (stateless = no memory of the connection).

For this lab, the NACL was configured correctly with default allow rules — so the security group was the only issue.

| Feature | Security Group | Network ACL |
|---------|---------------|-------------|
| Level | Instance | Subnet |
| Stateful? | ✅ Yes | ❌ No |
| Default | Deny all inbound | Allow all |
| Rules | Allow only | Allow + Deny |

---

## ✅ The Fix

After opening port 80 in the security group, I went back to the browser and refreshed:

```
http://<PUBLIC-IP-OF-INSTANCE>
```

```
🟢 Apache HTTP Server Test Page — It works!
```

**Ana's server is reachable.** The issue was a missing inbound rule for HTTP traffic in the security group. One rule, one fix.

---

## 🧠 Key Takeaways

```
💡 "It's not working" almost always means "something is blocking the traffic" — not that the app is broken.
💡 Always check the full traffic path: IGW → Route Table → Subnet → Security Group → NACL → EC2.
💡 Security Groups are "default deny" — you must explicitly allow every port you need.
💡 Apache (httpd) runs on port 80. No port 80 rule = no web traffic. Full stop.
💡 A detached Internet Gateway is as useless as no Internet Gateway at all.
💡 NACLs are stateless — you need both inbound AND outbound rules (unlike security groups).
💡 SSH key permissions matter. chmod 400 is not optional.
```

---

### 🗺️ Root Cause Summary

```
❌ Problem:   Inbound HTTP (port 80) not allowed in Security Group
✅ Solution:  Added inbound rule → Type: HTTP | Source: 0.0.0.0/0
🟢 Result:    Apache web server accessible from browser
```

---

<div align="center">

---

*Another ticket closed. Another misconfiguration found. Cloud Support work never gets old.* 🔧☁️

