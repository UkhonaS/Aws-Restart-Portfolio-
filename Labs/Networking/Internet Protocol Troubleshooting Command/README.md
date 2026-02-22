# Internet Protocol Troubleshooting Lab | AWS

## 📌 Lab Summary
Performed hands-on network troubleshooting within an AWS cloud environment using an Amazon Linux EC2 instance.  
Applied OSI-layer methodology to diagnose connectivity, routing, port, and application-level issues in simulated real-world customer scenarios.


---

## ☁️ AWS Services & Technologies Used
- Amazon EC2 (Amazon Linux)
- Amazon VPC
- Security Groups
- Network ACLs
- SSH
- Linux CLI

---

## 🔍 Technical Implementation

### Layer 3 – Network Diagnostics
- Used `ping` to verify ICMP reachability and detect packet loss
- Used `traceroute` to analyze routing paths and identify latency bottlenecks
- Validated AWS Security Group and NACL configurations affecting traffic flow

### Layer 4 – Transport Layer Analysis
- Used `netstat` to identify active TCP connections and listening services
- Detected open/unauthorized ports during simulated security scans
- Used `telnet` to verify port accessibility and differentiate between:
  - Connection refused (service/firewall issue)
  - Connection timed out (routing/network issue)

### Layer 7 – Application Layer Testing
- Used `curl` to validate HTTP/HTTPS connectivity
- Confirmed server responses (e.g., 200 OK)
- Verified SSL/TLS communication with external services

---

## 🧠 Key Skills Demonstrated
- Structured troubleshooting using OSI model methodology
- Cloud networking fundamentals in AWS
- TCP/IP protocol analysis
- Security group and firewall validation
- CLI-based diagnostic tools
- Root cause isolation across networking layers

---

## 🎯 Outcome
Successfully identified and differentiated between network-level, transport-level, and application-level issues within a cloud-based infrastructure environment.
