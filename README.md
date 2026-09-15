Plan-Letting Hospitality Platform: Cloud Infrastructure
This repository contains the automated infrastructure code for the "Plan-Letting Hospitality Platform" project, developed in accordance with the 505AZ_2627_FT_B1_N.pdf specification for the Software Defined Networks and Edge Services (505AZ) module.  

Project Overview
The final architecture will be a multi-cloud, multi-layered environment. The core application will run on AWS and must be managed through a fully automated CI/CD pipeline using GitHub Actions and AWS CloudFormation. The frontend will feature high-definition video demonstrations using HTTP Live Streaming (HLS), with assets stored in Azure Blob Storage and delivered via Azure Front Door. This Azure edge environment will be configured manually via the Azure CLI.  

Current Release: Version 1 (Trial Build)
The v1 code in this repository is an initial, functional prototype. It successfully provisions a single baseline VPC, public and private subnets, routing tables, and a standalone Apache web server deployed via an EC2 bootstrap script.

Known Limitations in v1:
Monolithic Architecture: All resources are currently defined within a single CloudFormation stack, which creates a large blast radius and is difficult to maintain.
Exposure: The web server is located in a public subnet, whereas the final production design requires it to be secured in a private subnet.  
Incomplete Scope: The environment lacks the required database tier, load balancing, and management resources.  

Development Roadmap for Final Submission
To meet the final architectural requirements, the monolithic stack will be refactored into modular components and expanded to include:
VPC Peering: A secondary Private Services network (VPC B) securely peered with the Public Services network (VPC A).  
Application Tier (VPC A): An internet-facing Application Load Balancer (ALB) routing traffic to an Auto Scaling group of web servers located in a private subnet.  
Data Tier (VPC B): An Amazon RDS database (with a Read Replica) that only accepts traffic from the web servers on the database port.  
Management Tier (VPC B): A corporate EC2 management instance allowing SSH and ICMP access to the web and database resources for diagnostics.  
