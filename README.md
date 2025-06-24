# AWS---VPC---Creation
This project demonstrates a production-ready AWS VPC setup with public and private subnets, an Auto Scaling Group, a bastion host, and an Application Load Balancer (ALB) routing traffic to instances in private subnets.


![image](https://github.com/user-attachments/assets/2546b3d7-e57d-41dd-8bcf-2e634f377834)



# 1. Create VPC
Select: VPC and more
Auto-generate: aws-prod-example

Pv4 CIDR block:cidr

IPv6 CIDR block: No IPv^ CIDR Block

Tenancy : default

Number of Availability Zones (AZs): 2

Number of public subnets:2

Number of private subnets:2

NAT gateways ($): 1 per AZ

VPC endpoints: None

Create VPC


# 2. Go to Auto Scaling group 
-Create Auto Scaling group

-Create Launch Template
Launch template name and description

Template version description

Launch template name - required
: aws-prod-example

Description: Proof of concept for app deployment

Recently launched: free tier eligible

Instance Type: (e.g., t2.micro)

Key Pair: aws-login

Security Group: aws-prod-example

All traffic allowed (⚠️ for demo only — restrict in production)

ssh one

Custom TCP: Port 8000

VPC: aws-prod-example(custom created)--> # Create launch template



# 3. Auto Scaling Group
Name: aws-prod-example

Launch Template: previously created

Availability Zones: private-subnet-1 & private-subnet-2 in us-east-1a and us-east-1b
Next -> No load  balancer -->   next

Desired Capacity: 2

Min Size: 1

Max Size: 4

Scaling Policies: None   then Next--> next -->Creat Auto Scaling group

Two EC2 instances have been successfully created in the private subnets of the VPC.
Both instances have private IP addresses and are not directly accessible from the internet.
They are intended to be accessed via a bastion host.

# 4. Create EC2 instance 
Name: bastion-host
slect: ubuntu

Key Pair: aws-login
VPC: created one

Security Group: allows SSH (port 22)

Network Placement: Public subnet of same VPC

public id: enable
Launch instance

Private EC2 instances that don’t have a key pair attached can't be accessed directly via SSH. However, there's a secure workaround: copy the bastion host’s .pem key into the private instance through an SCP command after SSH’ing into the bastion

Connect using: Commandline 
```
scp -i xxx.pem C:/Users/xxxx/Downloads/xxx.pem ubuntu@publicIP:/home/ubuntu
```
# To verify the copied file
```
ssh -i aws-login.pem ubuntu@<Bastion-Public-IP>
ls
```
```
scp -i xxx.pem ubuntu@private-ip

```
Now I am able to login private instance as well


# 5. Deploy Static Web Page on Private EC2
Connect via Bastion → Private IP
vim index.html

Example content:
<!DOCTYPE html>
<html>
<body>
  <h1>First AWS PROJECT to Demonstrate apps in private subnet</h1>
</body>
</html>
# Start simple Python web server:
```
python3 -m http.server 8000
```


# 6. Create load balancer
Application Load Balancer (ALB)
Name: aws-prod-example

Type: Internet-facing ALB
Ipv4

VPC: aws-prod-example

AZ/Subnets: Public subnet 1 and Public subnet2

Security Group: aws-prod-example

Listener: HTTP on port 8000

# Target Group:
Create target group

Target Type: Instance

Name: aws-prod-example

Port: 80

Instances: Private EC2s (exclude bastion)

Health Check: HTTP on /
Next
Select: two private instances which is created
Created target group


# 6. Create load balancer
Application Load Balancer (ALB)
Name: aws-prod-example

Type: Internet-facing ALB
Ipv4

VPC: aws-prod-example

AZ/Subnets: Public subnet 1 and Public subnet2

Security Group: aws-prod-example

Listener: HTTP 80
 port 8000
 Default action: selected Target group 
 
created load balancer

Then to verify using DNs name
```

```
# Ineterview FAQ

# 🛠️ VPC Architecture for a 2-Tier Highly Available and Scalable Application
# 📌 Design Overview
In this scenario, we need to build a 2-tier architecture (Web + Application) that is highly available and scalable on AWS. Here’s how we would design the VPC architecture:

✅ Key Components
VPC with CIDR block (e.g., 10.0.0.0/16)

Public Subnets (in 2 AZs)

For Load Balancer

Internet Gateway attached

Private Subnets (in 2 AZs)

For EC2 instances (App layer)

Route through NAT Gateway in each AZ(The NAT Gateway allows outbound internet access from private instances without exposing them directly to inbound internet traffic)

Auto Scaling Group in private subnets

Target Group for ALB pointing to private EC2 instances

Security Groups for secure access (HTTP, SSH via Bastion if needed)

🖼️ Architecture Diagram
⚙️ How It Works
Users access the app via ALB in the public subnet.

The ALB routes traffic to EC2 instances in private subnets via Target Groups.

Auto Scaling ensures the EC2 instances scale based on load.

Each AZ has its own NAT Gateway to allow private EC2s to access the internet (for updates, etc.).

Resources are distributed across multiple Availability Zones for fault tolerance and high availability.





