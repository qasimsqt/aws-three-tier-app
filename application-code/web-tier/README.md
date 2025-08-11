# Web Tier

The **Web Tier** is the entry point of the AWS Three-Tier Web Architecture.  
It serves static content, routes traffic, and forwards application requests to the App Tier.

## Key Features
- **Public EC2 Instances** – Deployed in public subnets with internet access.
- **Internet-Facing Load Balancer (ALB)** – Distributes incoming requests across multiple web servers.
- **Secure Networking** – Security groups allow HTTP/HTTPS traffic from the internet and limit backend access to the App Tier only.
- **High Availability** – Instances spread across multiple Availability Zones for redundancy.

This tier ensures users can access the application reliably and securely from anywhere.
