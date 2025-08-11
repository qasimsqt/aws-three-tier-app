# Application Tier

The **Application Tier** serves as the middle layer of the AWS Three-Tier Web Architecture.  
It processes business logic, handles requests from the Web Tier, and interacts with the Database Tier.

## Key Features
- **Private EC2 Instances** – Deployed in private subnets with no public IPs.
- **Internal Load Balancer** – Ensures secure communication between Web Tier and App Tier.
- **Secure Networking** – Security groups restrict access to only necessary tiers.
- **Scalable** – Can be scaled horizontally based on demand.

This tier ensures backend logic remains secure, efficient, and isolated from direct internet access.
