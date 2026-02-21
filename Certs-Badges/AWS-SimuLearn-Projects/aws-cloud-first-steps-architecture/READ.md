☁️ AWS High Availability Architecture Simulation (EC2 Multi-AZ Deployment)
📌 Project Summary
Designed and implemented a high-availability cloud architecture on AWS as part of a simulated business engagement with a fictional island stabilization team.

The objective was to eliminate single points of failure and increase system reliability by leveraging AWS Global Infrastructure best practices.

This project was completed in a live AWS Management Console environment.

🏗️ Architecture Solution

Deployed Amazon EC2 instances across multiple Availability Zones

Applied AWS high-availability design principles

Reduced infrastructure risk through Multi-AZ deployment

Validated solution performance and resilience in a live lab

🛠️ AWS Services Used

Amazon EC2

AWS Regions

AWS Availability Zones

AWS Global Infrastructure concepts

🎯 Business Impact

Improved system reliability

Increased fault tolerance

Reduced downtime risk

Implemented scalable infrastructure design

💡 Technical Competencies Demonstrated

Cloud architecture design

High availability implementation

Multi-AZ deployment strategy

Infrastructure resilience planning

AWS console-based deployment

Requirement analysis and solution mapping

📊 Key Concepts Applied

Fault tolerance

Redundancy

Distributed infrastructure

Region vs Availability Zone strategy

Production-ready cloud deployment principles



                    AWS Region
        -----------------------------------
        |                                 |
        |     Availability Zone A         |
        |     --------------------        |
        |     |     EC2 Instance A  |     |
        |     --------------------        |
        |                                 |
        |     Availability Zone B         |
        |     --------------------        |
        |     |     EC2 Instance B  |     |
        |     --------------------        |
        |                                 |
        -----------------------------------
                     |
                 End Users
                 This clearly shows:
This diagram clearly shows 
One AWS Region

Two Availability Zones

EC2 deployed in both for high availability
