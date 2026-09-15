# 🏨 Plan-Letting Hospitality Platform — Cloud Infrastructure

> **Software Defined Networks & Edge Services (505AZ)**
> Automated cloud infrastructure for the **Plan-Letting Hospitality Platform**, developed in accordance with the `505AZ_2627_FT_B1_N.pdf` module specification.

---

## 📖 Project Overview

The **Plan-Letting Hospitality Platform** is designed as a **multi-cloud, multi-layered cloud architecture**, combining AWS and Microsoft Azure services.

The core application infrastructure will run on **AWS** and will be provisioned and managed through a fully automated **Infrastructure as Code (IaC)** and **CI/CD** workflow using:

* ☁️ **Amazon Web Services (AWS)**
* 🏗️ **AWS CloudFormation**
* 🔄 **GitHub Actions**
* 🌐 **Microsoft Azure**
* 📦 **Azure Blob Storage**
* 🚪 **Azure Front Door**
* 🎥 **HTTP Live Streaming (HLS)**

The frontend will support **high-definition video demonstrations** using HLS. Video assets will be stored in **Azure Blob Storage** and delivered globally through **Azure Front Door**.

The Azure edge environment will be configured manually using the **Azure CLI**.

---

## 🚀 Current Release — Version 1 (Trial Build)

The current **v1 release** is an initial functional prototype focused on establishing the foundational AWS networking infrastructure.

The deployment successfully provisions:

| Component                     | Status            |
| ----------------------------- | ----------------- |
| 🌐 VPC                        | ✅ Implemented     |
| 🔓 Public Subnet              | ✅ Implemented     |
| 🔒 Private Subnet             | ✅ Implemented     |
| 🛣️ Route Tables              | ✅ Implemented     |
| 💻 EC2 Web Server             | ✅ Implemented     |
| ⚙️ EC2 Bootstrap Script       | ✅ Implemented     |
| 🏗️ CloudFormation Deployment | ✅ Implemented     |
| 🔄 CI/CD Pipeline             | 🚧 In Development |
| ⚖️ Application Load Balancer  | 🚧 Planned        |
| 📈 Auto Scaling               | 🚧 Planned        |
| 🗄️ RDS Database              | 🚧 Planned        |
| 🔗 VPC Peering                | 🚧 Planned        |
| 🛠️ Management Instance       | 🚧 Planned        |
| ☁️ Azure Edge Environment     | 🚧 Planned        |

---

## ⚠️ Known Limitations in v1

The current implementation is intentionally a **trial build** and does not yet represent the final production architecture.

### 🧱 1. Monolithic Architecture

All infrastructure resources are currently defined within a **single CloudFormation stack**.

This creates:

* 🔴 A larger blast radius
* 🔴 Difficult maintenance
* 🔴 Reduced modularity
* 🔴 Less flexibility when deploying individual components

The final implementation will separate the infrastructure into **modular CloudFormation stacks**.

---

### 🌍 2. Public Web Server Exposure

The current Apache web server is deployed within a **public subnet**.

While this is suitable for the initial prototype, the final architecture requires application servers to be deployed within a **private subnet**.

The production traffic flow will instead be:

```text
🌐 Internet
     │
     ▼
⚖️ Application Load Balancer
     │
     ▼
🔒 Private Subnet
     │
     ▼
🖥️ Auto Scaling Web Servers
```

This will significantly improve the security posture of the application tier.

---

### 🧩 3. Incomplete Infrastructure Scope

The v1 prototype does not yet include several components required by the final architecture:

* ⚖️ Application Load Balancer
* 📈 Auto Scaling Group
* 🗄️ Amazon RDS
* 🔁 RDS Read Replica
* 🔗 VPC Peering
* 🛠️ Management EC2 Instance
* 🔐 Production security rules
* ☁️ Azure Blob Storage
* 🚪 Azure Front Door
* 🎥 HLS video delivery infrastructure

These components will be introduced during the subsequent development phases.

---

# 🗺️ Development Roadmap

The final submission will transform the current prototype into a **modular, secure, multi-tier, multi-cloud architecture**.

## 1️⃣ Modular CloudFormation Architecture

The current monolithic CloudFormation stack will be refactored into smaller, independently manageable components.

### Planned structure

```text
☁️ AWS Environment
│
├── 🌐 VPC / Networking
├── ⚖️ Load Balancing
├── 🖥️ Application Tier
├── 🗄️ Database Tier
└── 🛠️ Management Tier
```

This approach will reduce the blast radius of infrastructure changes and improve maintainability.

---

## 2️⃣ 🔗 VPC Peering

A secondary **Private Services VPC (VPC B)** will be introduced and securely peered with the **Public Services VPC (VPC A)**.

```text
                 ☁️ AWS
                  │
        ┌─────────┴─────────┐
        │                   │
   🅰️ VPC A             🅱️ VPC B
 Public Services       Private Services
        │                   │
        └────── 🔗 ─────────┘
            VPC Peering
```

VPC B will host sensitive internal resources such as the database and management infrastructure.

---

## 3️⃣ ⚖️ Application Tier — VPC A

The application tier will be redesigned to remove direct internet exposure from the web servers.

### Architecture

```text
🌐 Internet
     │
     ▼
⚖️ Internet-Facing ALB
     │
     ▼
🔒 Private Subnets
     │
     ▼
📈 Auto Scaling Group
     │
 ┌───┴───┐
 ▼       ▼
🖥️ EC2  🖥️ EC2
Web      Web
Server   Server
```

The **Application Load Balancer (ALB)** will:

* 🌐 Accept internet traffic
* ⚖️ Distribute requests between web servers
* ❤️ Perform health checks
* 🔒 Prevent direct public access to individual EC2 instances

The web servers will run inside **private subnets** and will be managed by an **Auto Scaling Group**.

---

## 4️⃣ 🗄️ Data Tier — VPC B

The database infrastructure will be isolated within the **Private Services VPC**.

### Planned architecture

```text
🖥️ Web Servers
      │
      │ 🔐 Database Port
      ▼
🗄️ Amazon RDS
      │
      ▼
🔁 Read Replica
```

The RDS instance will only accept database traffic originating from the application/web tier.

Security controls will ensure that:

* 🔒 The database is not publicly accessible
* 🛡️ Only authorised application resources can connect
* 🚫 Unnecessary inbound traffic is blocked
* 🔁 A Read Replica is available for read scalability

---

## 5️⃣ 🛠️ Management Tier — VPC B

A dedicated corporate management EC2 instance will provide controlled administrative access to the private infrastructure.

```text
             🛠️ Management EC2
                    │
          ┌─────────┴─────────┐
          │                   │
        🔐 SSH             📡 ICMP
          │                   │
          ▼                   ▼
    🖥️ Web Servers       🗄️ Database
```

The management instance will be used for:

* 🔐 SSH administration
* 📡 ICMP diagnostics
* 🔎 Troubleshooting
* 🛠️ Infrastructure management

Access will be restricted through appropriate **security groups and network controls**.

---

# ☁️ Multi-Cloud Architecture

The final platform will combine AWS application infrastructure with Azure-based video delivery.

```text
                         🌐 Users
                            │
              ┌─────────────┴─────────────┐
              │                           │
              ▼                           ▼
       ☁️ AWS Application          ☁️ Azure Video
              │                       Delivery
              │                           │
              ▼                           ▼
       ⚖️ AWS ALB              🚪 Azure Front Door
              │                           │
              ▼                           ▼
       🔒 Private EC2              📦 Blob Storage
              │                           │
              ▼                           ▼
         🗄️ Amazon RDS              🎥 HLS Video
```

### AWS ☁️

Responsible for:

* 🌐 Networking
* ⚖️ Load balancing
* 🖥️ Application hosting
* 📈 Auto Scaling
* 🗄️ Database services
* 🛠️ Management infrastructure
* 🔗 VPC connectivity

### Azure 🔷

Responsible for:

* 📦 Video asset storage
* 🎥 HLS content
* 🚪 Global edge delivery
* 🌍 Frontend video distribution

---

# 🔄 Automation & Deployment

The final infrastructure will follow an **Infrastructure as Code** approach.

```text
👨‍💻 Developer
     │
     ▼
🐙 GitHub Repository
     │
     ▼
🔄 GitHub Actions
     │
     ▼
🏗️ CloudFormation
     │
     ▼
☁️ AWS Infrastructure
```

This approach will provide:

* ♻️ Repeatable deployments
* 🔍 Version-controlled infrastructure
* 🚀 Automated deployment workflows
* 🧪 Consistent environments
* 🔄 Easier infrastructure updates
* 🛡️ Reduced manual configuration

---

# 🎯 Final Architecture Goals

The final submission aims to provide a:

* ☁️ **Multi-cloud** architecture
* 🧩 **Modular** infrastructure design
* 🔒 **Secure** network architecture
* 📈 **Scalable** application tier
* 🗄️ **Highly isolated** database tier
* ⚖️ **Load-balanced** web application
* 🔄 **Automated** CI/CD deployment process
* 🏗️ **Infrastructure-as-Code** implementation
* 🌍 **Globally distributed** video delivery platform

---

## 📌 Project Status

> 🚧 **Current Status: Version 1 — Trial Build**

The current repository establishes the initial AWS networking and web-server foundation. Development will continue by progressively introducing the modular architecture, private application tier, database infrastructure, management layer, VPC peering, CI/CD automation, and Azure video delivery environment.

---

## 📚 Specification

This project is developed in accordance with the requirements outlined in:

**`505AZ_2627_FT_B1_N.pdf`**

**Module:** Software Defined Networks and Edge Services (**505AZ**)
