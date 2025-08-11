# Project Screenshots
This section visually documents the step-by-step deployment of my AWS Three-Tier Web Architecture.
From networking to the final running application, each screenshot highlights a specific part of the infrastructure.
---

## Networking Layer

The foundation of the project — configuring the VPC, subnets, and route tables to enable secure communication between tiers.

![VPC Dashboard](./VPC_Dashboard.png)
![Subnets](./subnet.png)
![Private Route Tables](./Private_RouteTable.png)
![Public Route Tables](./Public_RouteTable.png)

---

## Web Tier

The entry point for all user traffic — publicly accessible, secured, and load-balanced.


Web Tier EC2 Instances: Instances serving static and dynamic content.
![Web Tier EC2 Instances](./web-tier-ec2-list.png)

Web Tier Security Group: Firewall rules controlling inbound HTTP/HTTPS traffic.
![Web Tier Security Group](./web-tier-sg.png)

Internet-Facing Application Load Balancer: Distributes incoming requests to the web tier.
![Internet-Facing ALB](./alb-dashboard.png)

---

## App Tier

Handles the business logic, isolated from public access, and only reachable through the internal load balancer.

App Tier EC2 Instances: Backend instances processing application requests.
![App Tier EC2 Instances](./app-tier-ec2-list.png)

App Tier Security Group: Restricts inbound traffic to only the web tier.
![App Tier Security Group](./app-tier-sg.png)

Internal Load Balancer: Routes traffic securely between web tier and app tier.
![Internal Load Balancer](./internal-lb-dashboard.png)

---

## Database Tier

Secure and private — stores application data, only accessible by the app tier.

RDS Dashboard: Overview of the managed MySQL database instance.
![RDS Dashboard](./RDS_DB.png)

DB Subnet Group: Ensures database redundancy and availability.
![DB Subnet Group](./RDS_SubnetGroup.png)

DB Security Group: Limits inbound traffic to only the application tier.
![DB Security Group](./db-sg.png)

---

## Final Working Application
A visual wrap-up showing the completed architecture and the live application.

CloudCraft Architecture Diagram: 3D view of the deployed AWS infrastructure.
![CloudCraft Architecture](./Web App Reference Architecture.png)

Website in Action: Screenshots of the running application.
![Website Running](./Website_page_1.png)
![Website Running](./Website_page_2.png)
