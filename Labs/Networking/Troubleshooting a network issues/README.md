# Lab: Troubleshooting a Network Issue – AWS


**Role:** Cloud Support Engineer  

---

## Lab Overview

In this lab, I acted as a cloud support engineer tasked with troubleshooting a networking issue in a customer’s AWS environment. The goal was to resolve connectivity issues for an **Apache web server** running on an EC2 instance.  

The lab allowed me to demonstrate practical skills in **VPC configuration, security group management, EC2 administration, and cloud troubleshooting**.

---

## Scenario & Problem

The customer reported:

> “When I create an Apache server through the command line, I cannot ping it. I also get an error when I enter the IP address in the browser.”

Although the EC2 instance was running and Apache was installed, the server was **inaccessible externally**, preventing HTTP requests from reaching it.  

After investigating the environment, I identified the **root cause**:  

- The **security group attached to the EC2 instance did not allow inbound traffic** for HTTP (port 80) or ICMP (ping).  
- This misconfiguration blocked all external access, even though the server and VPC routing were correct.

---

## Solution & Outcome

I updated the security group to allow **public access** for the necessary ports:

After this change, the Apache server became **fully accessible externally**.  

---

## Key Takeaways

- **Security groups are critical:** Even if the server is running and routing is correct, incorrect inbound rules can block access.  
- **AWS networking best practices:** Always verify VPC, subnet, route tables, and security group settings when troubleshooting connectivity issues.  
- **Troubleshooting mindset:** Isolate the problem logically by checking service status, routing, and network permissions.  

**Outcome:**  
I successfully resolved the connectivity issue, reinforced my understanding of AWS **VPC networking**, **security groups**, and developed a **step-by-step troubleshooting approach** for real-world cloud issues.
