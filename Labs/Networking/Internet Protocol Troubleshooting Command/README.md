<div align="center">

# 🌐 Internet Protocol Troubleshooting Commands
### AWS Re/Start Portfolio — Networking Lab

![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Networking](https://img.shields.io/badge/Networking-0066CC?style=for-the-badge&logo=cisco&logoColor=white)
![OSI Model](https://img.shields.io/badge/OSI%20Model-Layer%203%2F4%2F7-6DB33F?style=for-the-badge)
![Shell](https://img.shields.io/badge/Shell-Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white)

---

> *"Troubleshooting a network without the right commands is like trying to fix a car in the dark. These tools are your flashlight."*

---

</div>

## 📌 Overview

In this lab, I stepped into the role of a **Network Administrator** responsible for diagnosing and resolving customer-reported connectivity issues on AWS infrastructure. The focus was on building fluency with essential **IP troubleshooting commands** and understanding how each one maps to a specific layer of the **OSI (Open Systems Interconnection) model**.

Rather than memorising commands in isolation, this lab reinforced the importance of knowing *which tool to reach for* depending on where in the network stack the issue is occurring.

---

## 📋 Table of Contents

- [The OSI Model & Troubleshooting Context](#-the-osi-model--troubleshooting-context)
- [Task 1: SSH into the EC2 Instance](#-task-1-ssh-into-the-ec2-instance)
- [Task 2: Troubleshooting Commands by OSI Layer](#-task-2-troubleshooting-commands-by-osi-layer)
  - [Layer 3 — Network: ping & traceroute](#layer-3--network-ping--traceroute)
  - [Layer 4 — Transport: netstat & telnet](#layer-4--transport-netstat--telnet)
  - [Layer 7 — Application: curl](#layer-7--application-curl)
- [Command Reference Cheat Sheet](#-command-reference-cheat-sheet)
- [Key Takeaways](#-key-takeaways)

---

## 🗂️ The OSI Model & Troubleshooting Context

The **OSI model** is a conceptual framework that divides network communication into seven distinct layers. Each layer has a specific responsibility, and when network issues arise, understanding which layer is affected dramatically narrows down the troubleshooting path.

This lab focused on three layers where most network issues manifest:

| OSI Layer | Name | Responsibility | Commands Covered |
|-----------|------|----------------|-----------------|
| Layer 3 | Network | IP addressing, routing, packet delivery | `ping`, `traceroute` |
| Layer 4 | Transport | Port management, TCP/UDP connections | `netstat`, `telnet` |
| Layer 7 | Application | End-user protocols (HTTP, HTTPS, DNS) | `curl` |

The general principle when troubleshooting: **start at the lowest layer and work upward.** If Layer 3 connectivity is broken, there is no point investigating Layer 7 behaviour — the issue is further down the stack.

---

## 🖥️ Task 1: SSH into the EC2 Instance

### Overview

Before executing any diagnostic commands, I established a secure connection to the Amazon Linux EC2 instance via **SSH (Secure Shell)**. SSH provides encrypted remote access to Linux-based systems and is the standard method for interacting with EC2 instances from the command line.

---

### Steps

**macOS / Linux:**

**1. Navigate to the directory containing the key file:**
```bash
cd ~/Downloads
```

**2. Restrict key file permissions (required by SSH):**
```bash
chmod 400 labsuser.pem
```

> 📝 *SSH enforces strict key file permission requirements. If the private key is accessible by other users on the system, SSH will reject the connection with a "permissions are too open" error. `chmod 400` restricts access to the file owner only.*

**3. Initiate the SSH connection:**
```bash
ssh -i labsuser.pem ec2-user@<PUBLIC-IP-ADDRESS>
```

Type `yes` when prompted to add the host to your list of known hosts. This is a one-time step for new connections.

---

**Windows:**

Download `labsuser.ppk` from the credentials panel and connect using **PuTTY**. Detailed setup instructions are available at [putty.org](https://www.putty.org/).

---

## 🔧 Task 2: Troubleshooting Commands by OSI Layer

---

### Layer 3 — Network: `ping` & `traceroute`

#### `ping` — Testing Basic IP Connectivity

**Applicable Scenario:**
A customer has launched a new EC2 instance and wants to verify that it has outbound internet connectivity, and that ICMP traffic is permitted through their security groups and network ACLs.

**Command:**
```bash
ping 8.8.8.8 -c 5
```

**Flag Breakdown:**

| Flag | Meaning |
|------|---------|
| `8.8.8.8` | Google's public DNS server — a reliable target for connectivity tests |
| `-c 5` | Limits the request to 5 ICMP echo packets (without this, ping runs indefinitely) |

**What to look for in the output:**

```
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=118 time=1.23 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=118 time=1.19 ms
...
5 packets transmitted, 5 received, 0% packet loss
```

| Result | Interpretation |
|--------|----------------|
| Replies received, 0% loss | ✅ Layer 3 connectivity is healthy |
| Request timeout | ❌ ICMP may be blocked by a security group or NACL |
| 100% packet loss | ❌ No route to destination — check IGW and route tables |

> 📝 *`ping` operates using the **ICMP (Internet Control Message Protocol)**. It is worth noting that some servers intentionally block ICMP requests for security reasons — a lack of ping response does not always mean the server is down. Always corroborate with other tests.*

---

#### `traceroute` — Mapping the Network Path & Diagnosing Latency

**Applicable Scenario:**
A customer reports significant latency and intermittent packet loss. They are uncertain whether the degradation originates within their AWS environment or with their external Internet Service Provider (ISP). `traceroute` allows you to map each hop along the network path and isolate where the delay or loss begins.

**Command:**
```bash
traceroute 8.8.8.8
```

**Sample Output:**
```
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max
 1  10.0.0.1      1.234 ms   1.102 ms   1.089 ms
 2  203.0.113.1   5.321 ms   5.298 ms   5.300 ms
 3  * * *
 4  8.8.8.8       12.45 ms  12.32 ms   12.28 ms
```

**How to interpret the output:**

| Indicator | Meaning |
|-----------|---------|
| Normal response times | ✅ That hop is functioning correctly |
| Significantly higher latency at a specific hop | ⚠️ Potential bottleneck — investigate that network segment |
| `* * *` (three asterisks) | ❌ The hop did not respond — may be a failed hop or a router configured to block ICMP |
| Consistent `* * *` from a specific point onward | ❌ Likely a routing failure beyond that node |

**Diagnosing the source of packet loss:**

```
Loss occurring in early hops  →  Likely a local network or AWS configuration issue
Loss occurring in later hops  →  Likely an ISP or destination server issue
```

> 📝 *A `* * *` response does not always indicate a failure. Many network devices are configured to silently drop traceroute probes for security reasons while still forwarding traffic normally. Context and pattern matter.*

---

### Layer 4 — Transport: `netstat` & `telnet`

#### `netstat` — Auditing Active Connections & Listening Ports

**Applicable Scenario:**
A routine security audit has flagged a potentially compromised port on a subnet. As the network administrator, you need to verify whether that port is actively listening on a local host when it should not be.

**Commands:**

```bash
# View established TCP connections
netstat -tp

# View all services currently listening for incoming connections
netstat -tlp

# View listening services without resolving port names to numbers
netstat -ntlp
```

**Flag Breakdown:**

| Flag | Description |
|------|-------------|
| `-t` | Filter for TCP connections only |
| `-p` | Display the process ID (PID) and program name associated with each connection |
| `-l` | Show only sockets that are actively listening |
| `-n` | Display numerical addresses and port numbers (no DNS resolution) |

**Sample Output (`netstat -tp`):**
```
Proto  Local Address        Foreign Address      State        PID/Program
tcp    0.0.0.0:22           0.0.0.0:*            LISTEN       1234/sshd
tcp    10.0.1.5:22          203.0.113.10:54321   ESTABLISHED  5678/sshd
```

> 📝 *`netstat` provides a point-in-time snapshot of Layer 4 activity. When investigating a networking issue, running this command first on the host machine helps determine whether the problem is local (a service not listening, or listening on the wrong port) before looking outward at routing or firewall rules.*

---

#### `telnet` — Testing TCP Port Connectivity

**Applicable Scenario:**
A customer has configured custom security group and network ACL rules on a secure web server and wants to verify that port 80 (HTTP) is genuinely blocked — not just reported as blocked. `telnet` can be used to attempt a direct TCP connection to that port and confirm the actual firewall behaviour.

**Install telnet (if not already present):**
```bash
sudo yum install telnet -y
```

**Command:**
```bash
telnet www.google.com 80
```

**Interpreting the response:**

| Response | Meaning |
|----------|---------|
| `Connected to www.google.com` | ✅ TCP connection established — port is open and reachable |
| `Connection refused` | ❌ The destination is actively rejecting connections on that port — a firewall or security group rule is blocking it |
| `Connection timed out` | ❌ No response at all — likely no network route to the destination, or the host is unreachable |

> 📝 *`telnet` is particularly useful for verifying security group behaviour. While the AWS console may show a rule as active, `telnet` provides empirical confirmation that the rule is actually being enforced at the network level.*

---

### Layer 7 — Application: `curl`

#### `curl` — Testing Application-Layer HTTP/S Requests

**Applicable Scenario:**
A customer has an Apache web server deployed and wants to confirm it is returning a successful `200 OK` HTTP response — the standard indicator that the application is operating correctly and serving content.

**Command:**
```bash
curl -vLo /dev/null https://aws.com
```

**Flag Breakdown:**

| Flag | Description |
|------|-------------|
| `-v` | Verbose mode — outputs detailed request/response headers and connection metadata |
| `-L` | Follow HTTP redirects automatically (e.g., HTTP → HTTPS) |
| `-o /dev/null` | Discards the response body (HTML/CSS) — we only care about the headers and status code |
| `-I` | Retrieve headers only (HEAD request) |
| `-k` | Bypass SSL certificate verification (useful when testing with self-signed certificates) |

**What a successful response looks like:**
```
* Connected to aws.com (x.x.x.x) port 443
* SSL connection established
> GET / HTTP/2
< HTTP/2 200
< content-type: text/html
```

**Common HTTP status codes to watch for:**

| Status Code | Meaning |
|-------------|---------|
| `200 OK` | ✅ Request successful — server is responding correctly |
| `301 / 302` | ↪️ Redirect — the resource has moved (usually handled by `-L`) |
| `403 Forbidden` | ⚠️ Server received the request but is denying access |
| `404 Not Found` | ⚠️ Server is reachable but the requested resource doesn't exist |
| `500 Internal Server Error` | ❌ Server is reachable but the application itself has an error |
| `Connection refused / timed out` | ❌ Did not reach the application layer — investigate lower OSI layers first |

> 📝 *`curl` is the go-to command for confirming that all lower layers (3, 4) are functioning correctly and that the application itself is responding as expected. A `200 OK` response provides end-to-end validation of the entire network path.*

---

## 📖 Command Reference Cheat Sheet

| Command | OSI Layer | Primary Use Case |
|---------|-----------|-----------------|
| `ping <IP> -c 5` | Layer 3 | Test basic IP connectivity and ICMP reachability |
| `traceroute <IP>` | Layer 3 | Map the network path; identify latency and failed hops |
| `netstat -tp` | Layer 4 | View active/established TCP connections |
| `netstat -tlp` | Layer 4 | View all services currently listening on open ports |
| `netstat -ntlp` | Layer 4 | Same as above, with numerical port display |
| `telnet <host> <port>` | Layer 4 | Test TCP connectivity to a specific port |
| `curl -vLo /dev/null <URL>` | Layer 7 | Validate full HTTP/S request and response cycle |

---

## 🧠 Key Takeaways

```
💡 Always troubleshoot from the lowest OSI layer upward — confirm Layer 3 before investigating Layer 7.
💡 ping tests ICMP reachability, not full application connectivity. Use it as a first-pass check only.
💡 traceroute pinpoints exactly where latency or packet loss occurs in the network path.
💡 netstat gives you a real-time view of what your host is actually listening on — invaluable for security audits.
💡 telnet provides empirical confirmation of whether a specific TCP port is open, blocked, or unreachable.
💡 curl validates the complete request lifecycle — from DNS resolution to HTTP response code.
💡 * * * in traceroute output does not always mean failure — some routers drop ICMP probes intentionally.
💡 "Connection refused" and "Connection timed out" point to very different problems — know the distinction.
```

---

<div align="center">

---

*Systematic. Methodical. Layer by layer.* 🌐🔧

![AWS](https://img.shields.io/badge/Made%20with-AWS-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)
![AWS ReStart](https://img.shields.io/badge/AWS-Re%2FStart%20Graduate-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=FF9900)
![Networking](https://img.shields.io/badge/Skill-IP%20Troubleshooting-0066CC?style=for-the-badge&logo=amazon-aws&logoColor=white)

</div>
