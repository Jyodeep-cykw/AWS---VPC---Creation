# AWS---VPC---Creation
This project demonstrates a production-ready AWS VPC setup with public and private subnets, an Auto Scaling Group, a bastion host, and an Application Load Balancer (ALB) routing traffic to instances in private subnets.


![image](https://github.com/user-attachments/assets/2546b3d7-e57d-41dd-8bcf-2e634f377834)



# 1. Create VPC

Select: VPC and more

  Auto-generate: aws-prod-example (any name that you want)
  
  Pv4 CIDR block: Leave it the same (10.0.0.0/16)
  
  IPv6 CIDR block: No IPv6 CIDR Block
  
  Tenancy: default
  
  Number of Availability Zones (AZs): 2
  
  Number of public subnets: 2
  
  Number of private subnets: 2
  
  NAT gateways ($): 1 per AZ
  
  VPC endpoints: None
  
  Finally, Create a VPC


# 2. Go to the Auto Scaling group in the EC2 section

-Create an Auto Scaling group

  Choose a name for the group (usually keep it the same as the VPC name to make it easier to delete after your done)

-Create Launch Template

  Since we don't have a template, choose to create a template

  Launch template name and description (Same as the VPC)
  
  Template version description 

  Check the Auto Scaling guidelines

  Application and OS Images: Recently launched (Ubuntu version)

  Instance Type: t2.micro (free version)

  Key pair name: Spring Boot Application

  Subnet: don't include in template

  Firewall: Create a new security group

  Security group name: vpc name

  Description

  VPC: Select the VPC you created earlier

  Security Rules:

    Type                              Port Range                     Source Type
    
    Custom TCP                          8000                           Anywhere

    All Traffic                                                        Anywhere

      ssh                                                              Anywhere

    Custom TCP                          80                             Anywhere

  
  Finally, Click on Create Launch Template

  
- Go back to the Create Auto Scaling group

  Name: VPC name (any name that you want it to be)
  
  Launch Template: previously created

  Version: Default 1
  
  Next

  Network ---> VPC: Pick the vpc that we created --->  Availability Zones: private-subnet-1 & private-subnet-2

  Availability Zones Distribution: Balanced best effort

  Then click next
  
  Integrate with other services (Skip): Click next

  Configure Group size and scaling ---> Desired Capacity: 2 --->  Min Size: 1 and Max Size: 4 ----> Click Next

  Add notifications: Click Skip to review
 
  Scroll down and click on Create auto scaling group
  
  
  Two EC2 instances have been successfully created in the private subnets of the VPC.
  Both instances have private IP addresses and are not directly accessible from the internet.
  They are intended to be accessed via a bastion host.
  


# 3. Create EC2 instance 

  GO back to EC2 instance dashboard
  
  Click on instances 
  
  Click Create New Instance 
  
  Name: anyname
  
  Quick start: Ubuntu ---> AMI: Ubuntu (Free versions)
  
  Key Pair: SpringbootApplication
  
  VPC: created one
  
  Subnet: Any of the public subnets (1 or 2)
  
  Auto-assign public ID: enable
  
  Firewall ---> Select existing security group --> pick the security group that you created earlier
  
  Then launch the instance
  
  Private EC2 instances that don’t have a key pair attached can't be accessed directly via SSH. However, there's a secure workaround: copy the bastion host’s .pem key into the private instance through an SCP command after SSH’ing into the bastion
  
  Connect to it
  
  Connect using: Commandline from you pc
  ```
  scp -i xxx.pem C:/Users/xxxx/Downloads/xxx.pem ubuntu@publicIP:/home/ubuntu (windows)
  ```
  ```
  scp -i xxx.pem ~/Downloads/xxx.pem ubuntu@publicIP:/home/ubuntu (Mac)
  ```
  
  Then it will ask you to confirm it
  
  Now I am able to login private instance as well

  To check if the file is in the server type in "ls"

  This should show the .pem file


# 4. Deploy a Static Web Page on Private EC2

  We need to connect via Private IP

  ```
  ssh -i xxx.pem ubuntu@private-ip
  ```
  
  Then confirm it

  then type 
  ```
  sudo apt update
  ```

  This should download python which will allow us to import / create new files

  For this example we will create a html file

  First type vim index.html

  Then paste this code or your code
  ```
  <!DOCTYPE html>
  <html>
  <body>
    <h1>First AWS PROJECT to Demonstrate apps in private subnet</h1>
  </body>
  </html>
  ```

  Then, to run the server type this 
  
  ```
  python3 -m http.server 8000
  ```


# 5. Create load balancer and target groups

  Go to the sidebar and selectthe  target group
  
  Create a target group
  
  Target Type: Instances
  
  Name: any name
  
  Protocol: HTTP    Port: 8000

  IP address: IPv4
  
  VPC: the instance we created earlier

  
  
  Health Check: HTTP on /
  Next
  Select: two private instances which is created
  Created target group

  
  Application Load Balancer (ALB)
  Name: aws-prod-example
  
  Type: Internet-facing ALB
  Ipv4
  
  VPC: aws-prod-example
  
  AZ/Subnets: Public subnet 1 and Public subnet2
  
  Security Group: aws-prod-example
  
  Listener: HTTP on port 8000

  

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






