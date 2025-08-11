# Deployment Guide – AWS Three-Tier Web Architecture

## 1. Architecture Overview
In this architecture, a public-facing Application Load Balancer forwards client traffic to our **Web Tier** EC2 instances.  
The web tier runs **Nginx** web servers that are configured to serve a **React.js** website and redirect API calls to the **Application Tier’s** internal-facing load balancer.

The internal load balancer forwards traffic to the application tier, which is built in **Node.js**. The application tier interacts with an **Aurora MySQL** Multi-AZ database and returns data to the web tier.

**Key features**:
- Load balancing, health checks, and Auto Scaling Groups at each layer for high availability.
- Secure access to EC2 instances via **AWS Systems Manager Session Manager** (no SSH keys needed).
- Code hosted in **S3** for easy retrieval by instances.

---

## 2. Learning Objectives
1. Create an **S3 Bucket** to store application code.
2. Create an **IAM EC2 Instance Role** with required permissions.
3. Download application code from **GitHub**.

---

## 3. Download Code from GitHub
Run the following command locally or in your instance to clone the repository:

```bash
git clone https://github.com/qasimsqt/aws-three-tier-app.git
```

## 4. S3 Bucket Creation
1. In the AWS Console, navigate to the **S3** service.
2. Click **Create Bucket**.
3. Enter a **unique name** for your bucket.
4. Select the **region** where you will deploy this lab.
5. Leave all other settings as default.
6. Click **Create Bucket**.

> This bucket will store your application code so your EC2 instances can access it.

---

## 5. IAM EC2 Instance Role Creation
1. Navigate to the **IAM** dashboard in the AWS Console.
2. Click **Roles** → **Create Role**.
3. Select **AWS Service** and choose **EC2** as the trusted entity.
4. Click **Next** and attach the following **AWS Managed Policies**:
   - `AmazonSSMManagedInstanceCore`
   - `AmazonS3ReadOnlyAccess`
5. Click **Next**, give your role a name (e.g., `EC2-SSM-S3-Access`), and click **Create Role**.

> This role allows EC2 instances to:
> - Access S3 objects (Read-Only)
> - Connect securely via Systems Manager Session Manager without SSH keys

## Part 1: Networking and Security

In this section, we will build out the **VPC networking components** as well as **security groups** that will add a layer of protection around our EC2 instances, Aurora databases, and Elastic Load Balancers.

---

### Learning Objectives
Create an isolated network with the following components:
1. VPC  
2. Subnets  
3. Route Tables  
4. Internet Gateway  
5. NAT Gateway  
6. Security Groups  

---

## 1. VPC Creation
1. Navigate to the **VPC** dashboard in the AWS Console.
2. Click **Your VPCs** on the left-hand side.
3. Make sure **VPC only** is selected.
4. Fill out the **VPC Settings**:
   - Add a **Name tag**.
   - Specify a **CIDR range** of your choice.
5. **Important Notes**:
   - Ensure you stay consistent with the **AWS region** for all resources in this workshop.
   - Choose a **CIDR range** that allows you to create at least **6 subnets**.

---

## 2. Subnet Creation
1. Navigate to **Subnets** on the left-hand side of the VPC dashboard.
2. Click **Create subnet**.
3. Select the **VPC** you created in Step 1.
4. Create **six subnets** across **two availability zones**:
   - **3 subnets** in **Availability Zone 1**
   - **3 subnets** in **Availability Zone 2**
5. Each subnet in one availability zone will correspond to a **layer of the three-tier architecture**:
   - Public-Web-Subnet
   - Private-App-Subnet
   - Private-DB-Subnet
6. For each subnet:
   - Choose a **name** (use a clear naming convention).
   - Select the **availability zone**.
   - Assign an **appropriate CIDR range** (subset of your VPC’s CIDR range).
7. **Naming Example**:
   - `Public-Web-Subnet-AZ-1`
   - `Private-App-Subnet-AZ-1`
   - `Private-DB-Subnet-AZ-1`
8. Verify your final subnet setup matches:
   - **3 subnets** in **AZ-1**
   - **3 subnets** in **AZ-2**

---



## 3. Internet Connectivity

To enable internet access for our VPC, we will configure both an **Internet Gateway** (for public subnets) and **NAT Gateways** (for private subnets).

---

### 3.1 Internet Gateway
1. Navigate to **Internet Gateway** in the left-hand menu of the **VPC dashboard**.
2. Click **Create internet gateway**.
3. Enter a **name** for your internet gateway.
4. Click **Create internet gateway**.
5. Attach the Internet Gateway to your **VPC** created in the **VPC and Subnet Creation** step:
   - Option 1: Use the **creation success message** to attach it directly.
   - Option 2: Use the **Actions** drop-down → **Attach to VPC**.
6. Select the correct VPC and click **Attach internet gateway**.

---

### 3.2 NAT Gateway
1. Navigate to **NAT Gateways** in the left-hand menu of the **VPC dashboard**.
2. Click **Create NAT Gateway**.
3. For each NAT Gateway:
   - Enter a **name**.
   - Select **one of the public subnets** you created earlier.
   - Allocate an **Elastic IP**.
   - Click **Create NAT Gateway**.
4. Repeat the process for the **second public subnet** to ensure **high availability**.

---


## 4. Route Tables

We will now configure route tables to manage network traffic between our subnets, the internet gateway, and the NAT gateways.

---

### 4.1 Create Public Route Table
1. Navigate to **Route Tables** in the **VPC dashboard**.
2. Click **Create route table**.
3. Enter a **name** for the public route table (e.g., `Public-Route-Table`).
4. Select the **VPC** created earlier.
5. Click **Create route table**.

---

### 4.2 Configure Public Route Table
1. After creation, open the **details page** for the new route table.
2. Go to the **Routes** tab → Click **Edit routes**.
3. Add a new route:
   - **Destination:** `0.0.0.0/0` (all traffic outside the VPC CIDR range)
   - **Target:** Select your **Internet Gateway**
4. Save the changes.
5. Open the **Subnet Associations** tab → Click **Edit subnet associations**.
6. Select the **two public web layer subnets** created earlier.
7. Save associations.

---

### 4.3 Create Private Route Tables for App Layer
1. Create **two more route tables**—one for each **app layer private subnet** in each availability zone.
2. For each private route table:
   - Open **Routes** → Click **Edit routes**.
   - Add a route:
     - **Destination:** `0.0.0.0/0`
     - **Target:** Select the **NAT Gateway** in the same availability zone.
   - Save the changes.
3. Go to **Subnet Associations** → Click **Edit subnet associations**.
4. Select the **corresponding app layer private subnet**.
5. Save associations.

---


## 5. Security Groups

Security Groups control inbound and outbound traffic to your AWS resources. We will configure five security groups to protect the different layers of our architecture.

---

### 5.1 Public Internet-Facing Load Balancer (External LB SG)
1. Navigate to **Security Groups** in the **VPC dashboard** under **Security**.
2. Click **Create security group**.
3. Enter:
   - **Name:** `External-LB-SG`
   - **Description:** Security group for the public internet-facing load balancer.
   - **VPC:** Select the one created earlier.
4. Add an **Inbound Rule**:
   - **Type:** HTTP
   - **Source:** Your IP
5. Save the security group.

---

### 5.2 Web Tier Instances (Web Tier SG)
1. Create a new security group:
   - **Name:** `Web-Tier-SG`
   - **Description:** Security group for web tier instances.
   - **VPC:** Select the same VPC.
2. Add **Inbound Rules**:
   - **Rule 1:**
     - **Type:** HTTP
     - **Source:** `External-LB-SG` (the security group from step 5.1)
   - **Rule 2:**
     - **Type:** HTTP
     - **Source:** Your IP
3. Save the security group.

---

### 5.3 Internal Load Balancer (Internal LB SG)
1. Create a new security group:
   - **Name:** `Internal-LB-SG`
   - **Description:** Security group for internal load balancer.
   - **VPC:** Select the same VPC.
2. Add an **Inbound Rule**:
   - **Type:** HTTP
   - **Source:** `Web-Tier-SG` (security group from step 5.2)
3. Save the security group.

---

### 5.4 Private App Tier Instances (Private Instance SG)
1. Create a new security group:
   - **Name:** `Private-Instance-SG`
   - **Description:** Security group for private app tier instances.
   - **VPC:** Select the same VPC.
2. Add **Inbound Rules**:
   - **Rule 1:**
     - **Type:** TCP
     - **Port Range:** 4000
     - **Source:** `Internal-LB-SG` (security group from step 5.3)
   - **Rule 2:**
     - **Type:** TCP
     - **Port Range:** 4000
     - **Source:** Your IP
3. Save the security group.

---

### 5.5 Private Database Instances (DB SG)
1. Create a new security group:
   - **Name:** `DB-SG`
   - **Description:** Security group for private database instances.
   - **VPC:** Select the same VPC.
2. Add an **Inbound Rule**:
   - **Type:** MySQL/Aurora
   - **Port Range:** 3306
   - **Source:** `Private-Instance-SG` (security group from step 5.4)
3. Save the security group.

---

## 6. Database Layer Deployment

This section covers deploying the database layer of our three-tier architecture using Amazon RDS with Amazon Aurora.

---

### Learning Objectives
- Deploy the Database Layer
- Configure Subnet Groups
- Enable Multi-AZ Database Deployment

---

### 6.1 Create the DB Subnet Group
1. Navigate to the **RDS dashboard** in the AWS Console.
2. On the left-hand side, click **Subnet groups**.
3. Click **Create DB subnet group**.
4. Fill in:
   - **Name:** `db-subnet-group`
   - **Description:** Subnet group for database layer
   - **VPC:** Select the VPC created earlier
5. Under **Add subnets**, select the **database subnets** in **each Availability Zone**.
   - Tip: Verify subnet IDs in the VPC dashboard to ensure you select the correct ones.
6. Save the subnet group.

---

### 6.2 Create the Database
1. Navigate to **Databases** in the RDS dashboard.
2. Click **Create database**.
3. **Engine options**:
   - **Engine type:** Amazon Aurora
   - **Edition:** MySQL-Compatible
   - **Creation method:** Standard create
4. **Templates**:
   - Select **Dev/Test**
5. **Settings**:
   - Enter a **DB cluster identifier** of your choice
   - Set a **Master username** and **password** (note these for later use)
6. **Availability & Durability**:
   - Choose **Create an Aurora Replica in a different AZ** (Multi-AZ deployment)
7. **Connectivity**:
   - **VPC:** Select your project VPC
   - **Subnet group:** Select the DB subnet group created in step 6.1
   - **Public access:** No
   - **VPC security group:** Select the `DB-SG` security group created earlier
8. **Authentication**:
   - Select **Password authentication**
9. Review the settings and click **Create database**.

---

### 6.3 Verify the Deployment
1. Once provisioning is complete, confirm:
   - One **Writer instance**
   - One **Reader instance** in different Availability Zones
2. Note down the **Writer endpoint** — this will be used later for connecting the application to the database.

---


# App Tier Deployment Guide

## Overview
In this section of the workshop, we will:
- Create an EC2 instance for the App Tier.
- Install and configure the required software stack.
- Set up the database schema and insert sample data.
- Test database connectivity and application endpoints.

The App Tier runs a **Node.js** application on port **4000** and connects to an Aurora RDS database.

---

## Learning Objectives
1. Create App Tier EC2 Instance.
2. Configure the software stack.
3. Set up database schema and sample data.
4. Test database and application connectivity.

---

## 1. Launch App Tier Instance

1. Go to **EC2 Dashboard → Instances → Launch Instances**.
2. **Name and Tags**: Set a suitable name (e.g., `App-Tier`).
3. **OS Image (AMI)**:  
   - Amazon Linux 2 (HVM) - Kernel 5.10.
4. **Instance Type**:  
   - `t2.micro` (Free Tier eligible).
5. **Key Pair**:  
   - Select **Proceed without a key pair**.
6. **Network Settings**:  
   - **VPC**: Select your project VPC.  
   - **Subnet**: `Private-App-Subnet`.  
   - **Security Group**: `PrivateInstanceSG`.  
   - **Auto-assign Public IP**: Disable.
7. **Advanced Details**:  
   - **IAM Instance Profile**: Select the role you created earlier (e.g., `EC2andS3Role`).
8. **Launch Instance**.

---

## 2. Connect to Instance

1. Go to **EC2 → Instances** and wait for **Running** state.
2. Select your instance → Click **Connect** → **Session Manager** → **Connect**.
3. Switch to the `ec2-user`:
   ```bash
   sudo -su ec2-user


```

# App Tier Deployment Guide

## 1. Test Internet Connectivity
Run the following command to check internet connectivity:
```bash
ping 8.8.8.8

```

### Then Configure Database

(Will add the commands later)


# App Tier AMI, Autoscaling, and Internal Load Balancer Setup

In this section of the workshop, we will create an **Amazon Machine Image (AMI)** of the app tier instance we just created, and use that to set up **autoscaling** with a **load balancer** to make this tier highly available.

---

## Learning Objectives
- Create an AMI of our App Tier  
- Create a Launch Template  
- Configure Autoscaling  
- Deploy Internal Load Balancer  

---

## 1. Create App Tier AMI

1. Navigate to **Instances** on the left-hand side of the EC2 dashboard.  
2. Select the **app tier instance** you created.  
3. Under **Actions** → **Image and templates** → **Create Image**.  
4. Give the image a **name** and **description**, then click **Create image**.  

You can monitor the image creation progress under **AMIs** (left-hand navigation, under **Images**).

---

## 2. Create Target Group

1. In the EC2 dashboard, go to **Target Groups** (under **Load Balancing**) → **Create Target Group**.  
2. **Target type**: Select **Instances**.  
3. Give the target group a **name**.  
4. **Protocol**: `HTTP`  
5. **Port**: `4000` (this is the port our Node.js app is running on)  
6. Select the correct **VPC**.  
7. Change **Health Check Path** to `/health`.  
8. Click **Next**.  

> **Note:** Do **NOT** register targets yet. Just skip that step and create the target group.

---

## 3. Create Internal Load Balancer

1. Go to **Load Balancers** (under **Load Balancing**) → **Create Load Balancer**.  
2. Choose **Application Load Balancer** → Click **Create**.  
3. Give it a **name**.  
4. Select **Internal** (not public).  
5. Choose the **VPC** and the **private subnets**.  
6. Select the **security group** for the internal ALB.  
7. Configure **Listener**:
   - Protocol: `HTTP`
   - Port: `80`
   - Forward traffic to the **Target Group** created earlier.
8. Click **Create Load Balancer**.

---

## 4. Create Launch Template

1. In EC2 dashboard → **Launch Templates** → **Create Launch Template**.  
2. **Name** the Launch Template.  
3. Under **Application and OS Images**, select the **app tier AMI** you created earlier.  
4. **Instance Type**: `t2.micro`  
5. Do **NOT** include Key Pair and Network Settings.  
6. Set the **security group** for the app tier.  
7. Under **Advanced Details**, select the same **IAM instance profile** used for previous EC2 instances.  
8. Click **Create Launch Template**.

---

## 5. Create Auto Scaling Group

1. Go to **Auto Scaling Groups** → **Create Auto Scaling group**.  
2. Name the group and select the **Launch Template** you just created.  
3. **VPC**: Select your VPC.  
4. **Subnets**: Choose the **private subnets** for the app tier.  
5. Attach the **Target Group** from the Load Balancer you created earlier.  
6. Set group size:
   - Desired: `2`
   - Minimum: `2`
   - Maximum: `2`
7. Skip to review, then click **Create Auto Scaling Group**.

---

## 6. Verify Setup

- You should now have:
  - Internal Load Balancer configured  
  - Auto Scaling Group running **2 app tier instances**  

- Test the scaling:
  1. Manually delete one of the new instances.
  2. Wait to see if the ASG automatically creates a new one.  

> **Note:** The original app tier instance is **excluded** from the ASG. You will see **3 instances** in total. You can delete the original instance after confirming setup, but it’s recommended to keep it for troubleshooting.

---

## Part 5: Web Tier Instance Deployment

This section covers deploying an EC2 instance for the **web tier** and configuring the **NGINX web server** along with the **React.js website**.

### Learning Objectives
- Update NGINX configuration files
- Create and configure the web tier EC2 instance
- Deploy the required software stack

---

### 1. Update Config File

Before creating and configuring the web instances:

1. Open the `application-code/nginx.conf` file from the repo you downloaded.
2. Scroll to **line 58** and replace:

[INTERNAL-LOADBALANCER-DNS]

with your **internal load balancer’s DNS entry**.

You can find this value in the **Internal Load Balancer’s details page** in the AWS console.

3. Upload the updated `nginx.conf` file **and** the `application-code/web-tier` folder to the **S3 bucket** you created for this lab.

---

### 2. Web Instance Deployment

1. In the **EC2 Dashboard**, click **Launch Instance**.
2. **Name and Tags**:
- Set an identifiable name for the web tier instance.
3. **OS and AMI**:
- Select **Amazon Linux** as the OS.
- Choose **Amazon Linux 2 AMI (HVM) - Kernel 5.10**.
4. **Instance Type**:
- Select `t2.micro`.
5. **Key Pair**:
- Select **Proceed without a key pair**.
6. **Network Settings** (Click **Edit**):
- **VPC**: Select your existing VPC.
- **Subnet**: Choose **Public-Web-Subnet**.
- **Security Group**: Select `WebTierSG` (created earlier).
- **Auto-assign Public IP**: Enable.
7. **Advanced Details**:
- In the **IAM instance profile** field, select the IAM Role created earlier (e.g., `EC2andS3Role`).
8. Click **Launch Instance**.

---

### 3. Connect to the Instance

Once the instance is running:

1. Connect to it using SSM Session Manager or SSH.
2. Switch to the `ec2-user` account:
```bash
sudo -su ec2-user
```

(Will add commands later)

# Part 6: External Load Balancer and Auto Scaling

In this section of the workshop, we will create an Amazon Machine Image (AMI) of the web tier instance we just created, and use that to set up Auto Scaling with an external-facing load balancer in order to make this tier highly available.

---

## Learning Objectives
1. Create an AMI of our Web Tier
2. Create a Launch Template
3. Configure Auto Scaling
4. Deploy External Load Balancer

---

## 1. Web Tier AMI
1. Navigate to **Instances** on the left-hand side of the EC2 dashboard.  
2. Select the web tier instance we created and under **Actions** select **Image and templates** → **Create Image**.  
3. Give the image a name and description, then click **Create image**.  
4. This will take a few minutes — to monitor status, go to **AMIs** under **Images** in the EC2 dashboard.

---

## 2. Target Group
1. While the AMI is being created, navigate to **Target Groups** under **Load Balancing** in the EC2 dashboard.  
2. Click **Create Target Group**.  
3. Select **Instances** as the target type and give it a name.  
4. Set:
   - **Protocol:** HTTP  
   - **Port:** 80 (NGINX listens here)  
   - **VPC:** Select your current VPC  
   - **Health Check Path:** `/health`  
5. Click **Next**.  
6. Skip registering targets and click **Create Target Group**.

---

## 3. Internet Facing Load Balancer
1. Navigate to **Load Balancers** under **Load Balancing** and click **Create Load Balancer**.  
2. Choose **Application Load Balancer** and click **Create**.  
3. Configure:
   - **Name:** Your choice  
   - **Scheme:** Internet-facing  
   - **VPC:** Select current VPC  
   - **Subnets:** Select **public** subnets  
4. Assign the **security group** created for the internal ALB.  
5. Set:
   - **Listener:** HTTP (Port 80)  
   - **Forward to:** The target group created earlier  
6. Click **Create Load Balancer**.

---

## 4. Launch Template
1. Navigate to **Launch Templates** under **Instances** and click **Create Launch Template**.  
2. Set:
   - **Name:** Your choice  
   - **AMI:** Select the web tier AMI created earlier  
   - **Instance Type:** t2.micro  
3. Leave **Key pair** and **Network settings** blank (will configure in ASG).  
4. Select the **Web Tier Security Group**.  
5. Under **Advanced details**, choose the IAM Instance Profile used for EC2 instances.  
6. Click **Create Launch Template**.

---

## 5. Auto Scaling
1. Navigate to **Auto Scaling Groups** and click **Create Auto Scaling group**.  
2. Set:
   - **Name:** Your choice  
   - **Launch Template:** Select the one created above  
3. Select:
   - **VPC:** Current VPC  
   - **Subnets:** Public subnets for the web tier  
4. Attach the Auto Scaling Group to the **Load Balancer’s Target Group** created earlier.  
5. Configure:
   - **Desired Capacity:** 2  
   - **Minimum:** 2  
   - **Maximum:** 2  
6. Click **Create Auto Scaling Group**.

---

## 6. Verification
1. You should now see the Auto Scaling Group launching **2 new web tier instances**.  
2. Test by deleting one instance and verifying that a replacement is launched automatically.  
3. Navigate to your **external load balancer** and open its **DNS name** in a browser to test the architecture.  

**Note:** The original web tier instance is excluded from the ASG, so you will see 3 instances. You may delete the original one or keep it for troubleshooting.

---

✅ **Congrats! You’ve implemented a 3-Tier Web Architecture!**

