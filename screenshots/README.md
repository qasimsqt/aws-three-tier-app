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
![Web Tier EC2 Instances](./Web-tier-ec2.png)


---

## App Tier

Handles the business logic, isolated from public access, and only reachable through the internal load balancer.

App Tier EC2 Instances: Backend instances processing application requests.
![App Tier EC2 Instances](./App-Tier.png)

---

## Database Tier

Secure and private — stores application data, only accessible by the app tier.

RDS Dashboard: Overview of the managed MySQL database instance.
![RDS Dashboard](./RDS_DB.png)

DB Subnet Group: Ensures database redundancy and availability.
![DB Subnet Group](./RDS_SubnetGroup.png)

---

All Subnets (Can't show them Tier wise)
![Subnet Groups](./subnet.png)

All NAT-Gateways
![NAT Gateway](./NGW.png)

All IGW-Gateways
![IGW Gateway](./IGW.png)

ALL Load Balancer
![Load Balancer](./All_LB.png)

ALL Route Tables
![Public Route Tables](./Public_RouteTable.png)
![Private Route Tables](./Private_RouteTable.png)

---

## Final Working Application
A visual wrap-up showing the completed architecture and the live application.

CloudCraft Architecture Diagram: 3D view of the deployed AWS infrastructure.
![CloudCraft Architecture](./Web_App_Reference_Architecture.png)

Website in Action: Screenshots of the running application.
![Website Running](./Website_page_1.png)
![Website Running](./Website_page_2.png)
