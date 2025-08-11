# Cleanup Guide

To avoid incurring additional AWS charges, delete resources in the following order. Start with the services deployed inside the VPC, then proceed to delete the VPC components themselves.

---

## 1. Auto Scaling Groups
Delete the Auto Scaling Groups first since they will continue to deploy EC2 instances otherwise (minimum instance count is set to 2 at every layer).

1. Navigate to the **EC2 Console** → **Auto Scaling Groups**.
2. Select your **App** and **Web Tier** Auto Scaling Groups.
3. Click **Delete**.

---

## 2. Application Load Balancers
1. Navigate to **EC2 Console** → **Load Balancers**.
2. For each ALB:
   - Delete the **Listeners**.
   - Select the ALB and choose **Delete** from the **Actions** dropdown.

---

## 3. Target Groups
1. Navigate to **EC2 Console** → **Target Groups**.
2. Select your target groups.
3. Click **Delete** from the **Actions** dropdown.

---

## 4. Launch Templates
1. Navigate to **EC2 Console** → **Launch Templates**.
2. Select a launch template.
3. From the **Actions** dropdown, choose **Delete template**.
4. Type `Delete` in the confirmation popup.

---

## 5. AMIs
1. Navigate to **EC2 Console** → **AMIs** (under **Images**).
2. Select both AMIs.
3. From the **Actions** dropdown, choose **Deregister AMI**.

---

## 6. Remaining EC2 Instances
If you already deleted the instances used to create the AMIs, skip this step.

1. Navigate to **EC2 Console** → **Instances**.
2. Select all instances.
3. From the **Instance State** dropdown, choose **Terminate Instance**.

---

## 7. Aurora Database
1. Navigate to **RDS Console**.
2. Select the **Regional Cluster** and click **Modify**.
3. Scroll down, uncheck **Deletion Protection**, and select **Apply Immediately**.
4. Click **Modify Cluster**.

**Delete Database Instances:**
1. Select a database instance (reader or writer).
2. From the **Actions** dropdown, choose **Delete**.
3. Type `delete me` when prompted.
4. Repeat for the other instance.
5. For the last instance:
   - Do **not** create a snapshot.
   - Acknowledge and confirm deletion.

---

## 8. NAT Gateways
1. Navigate to **VPC Console** → **NAT Gateways**.
2. Select both NAT Gateways.
3. Click **Delete NAT Gateway**.
4. Type `delete` to confirm.

---

## 9. Elastic IPs
1. Navigate to **VPC Console** → **Elastic IPs**.
2. Select all Elastic IPs.
3. From the **Actions** dropdown, choose **Release Elastic IP addresses**.
4. Confirm release.

---

## 10. Route Tables
1. Navigate to **VPC Console** → **Route Tables**.
2. For each route table:
   - Select it.
   - Go to the **Subnet Associations** tab.
   - Click **Edit subnet associations**.
   - Uncheck all subnets and save.

3. Select all three route tables created for this lab.
4. Click **Delete** from the **Actions** dropdown.
5. Type `delete` to confirm.

---

## 11. Internet Gateway
1. Navigate to **VPC Console** → **Internet Gateways**.
2. Select the IGW.
3. Click **Detach from VPC**.
4. Confirm detachment.
5. Select it again and choose **Delete Internet Gateway**.
6. Type `delete` to confirm.

---

## 12. VPC
1. Navigate to **VPC Console** → **Your VPCs**.
2. Select the lab VPC.
3. From the **Actions** dropdown, choose **Delete VPC**.
4. This will also delete subnets and security groups.

---
