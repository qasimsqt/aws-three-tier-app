# AWS Three-Tier Web Architecture Deployment

[![AWS](https://img.shields.io/badge/Cloud-AWS-orange?logo=amazon-aws&logoColor=white)](https://aws.amazon.com/)
[![Markdown](https://img.shields.io/badge/Format-Markdown-blue?logo=markdown)](https://www.markdownguide.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![GitHub Repo stars](https://img.shields.io/github/stars/qasimsqt/aws-three-tier-app?style=social)](https://github.com/qasimsqt/aws-three-tier-app)


A fully deployed, secure, and scalable three-tier application on AWS, built from scratch.

---

## 📌 Overview
This project demonstrates the deployment of a **three-tier architecture** consisting of:

- **Web Tier** – Internet-facing EC2 instances behind a public Application Load Balancer (ALB).
- **App Tier** – Private EC2 instances behind an internal Load Balancer.
- **Database Tier** – Amazon RDS MySQL instance in private subnets.

---

## 🛠 Services Used

- **Amazon EC2** – Web and App tiers
- **Elastic Load Balancing (ALB)** – Public & internal load balancers
- **Amazon RDS (MySQL)** – Managed relational database
- **Amazon VPC** – Custom networking with 5 subnets (public & private)
- **Security Groups & NACLs** – Network security
- **IAM** – Role-based access control
- **Cloudcraft** – Architecture diagram design

---

## 🖼 Architecture Diagram

**3-Tier Application Architecture**
![Architecture](./screenshots/3TierArch.png)

**CloudCraft 3D Architecture View**
![CloudCraft Architecture](./screenshots/Web_App_Reference_Architecture.png)

---

## 🚀 Deployment Steps

### 1. Networking
- Created a VPC with **5 subnets** (public & private).
- Attached an **Internet Gateway** and configured **route tables**.

### 2. Web Tier
- Launched EC2 instances in **public subnets**.
- Configured an **Internet-facing ALB**.

### 3. App Tier
- Launched EC2 instances in **private subnets**.
- Connected via an **internal ALB** from the web tier.

### 4. Database Tier
- Created an **RDS MySQL** instance in **private subnets**.
- Restricted access to **App tier only**.

### 5. Security
- Configured **Security Groups** to allow only necessary traffic (HTTP, HTTPS, MySQL).
- Used **IAM roles** for EC2 and Cloudcraft integration.

---

## 📂 Screenshots
See the [`/screenshots`](./screenshots) folder for:
- Running application views
- AWS Console screenshots
- Architecture diagrams

---

## 📚 Lessons Learned
- Difference between **internal** and **internet-facing** load balancers.
- How **private subnets** secure backend services.
- Troubleshooting **Security Group** & **networking issues** effectively.

---

## 📝 Additional Resources
- [AWS VPC Documentation](https://docs.aws.amazon.com/vpc/)
- [AWS Load Balancer Guide](https://docs.aws.amazon.com/elasticloadbalancing/)
- [Amazon RDS MySQL Overview](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_MySQL.html)
