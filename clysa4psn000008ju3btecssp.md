---
title: "Building a Production-Grade AWS VPC: High Availability, Security Groups, and Scalability with EC2 and Auto Scaling"
datePublished: Fri Jul 19 2024 05:48:34 GMT+0000 (Coordinated Universal Time)
cuid: clysa4psn000008ju3btecssp
slug: building-a-production-grade-aws-vpc-high-availability-security-groups-and-scalability-with-ec2-and-auto-scaling
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1721368031665/115e8b2f-b40e-44dd-a7c6-3e60089295bb.png
tags: cloud, aws, devops, vpc, aws-devops

---

### Introduction:

In today's cloud-driven world, having a solid and secure setup is key to keeping your apps running smoothly and scaling easily. Amazon Web Services (AWS) offers a bunch of tools to build a top-notch Virtual Private Cloud (VPC) that can handle these needs. In this blog, we'll guide you through setting up a resilient AWS VPC with multiple availability zones, auto-scaling EC2 instances in private subnets, a public-facing load balancer, and secure access through a bastion host. Plus, we'll set up security groups to protect our EC2 instances, making sure your app stays secure and highly available.

### Overview:

* **Creating a VPC with 2 Availability Zones:** We're setting up the basic network to keep things redundant and highly available.
    
* **Configuring Auto Scaling Groups for EC2 Instances:** We'll make sure our instances can scale up or down based on demand, all within private subnets for extra security.
    
* **Setting Up a Load Balancer in the Public Subnet:** This will help spread incoming traffic across multiple EC2 instances for better performance and reliability.
    
* **NAT Gateway**: Deployed in each AZ to enable outbound internet access for instances in private subnets.
    
* **Establishing a Bastion Host:** We'll set up secure SSH access to EC2 instances within private subnets.
    
* **Applying Security Groups:** We'll configure security groups to control traffic to and from your EC2 instances, adding an extra layer of security.
    
* **Deploying a Sample Python Application:** We'll install and run a Python app on one of the EC2 instances and access it through the load balancer to show how everything works.
    

### **What is a VPC?**

A Virtual Private Cloud (VPC) is a virtual network that you can create in the cloud. It allows you to have your own private sections of the network, similar to having your own network within a larger network. Within a VPC, you can manage various resources such as servers, databases, and storage, among others.

### VPC Components:

AWS VPC consists of several key components that work together to provide a secure and scalable virtual network environment. Let's explore these components:

1. **Virtual Private Clouds (VPC):** The foundational component, a VPC is a virtual network where you launch your AWS resources.
    
2. **Subnets:** A subnet is a range of IP addresses within your VPC. Each subnet must reside entirely within a single Availability Zone (AZ), allowing you to distribute your resources across multiple AZs for high availability.
    
3. **IP Addressing:** You can assign both IPv4 and IPv6 addresses to your VPCs and subnets. Additionally, you can bring your public IP addresses and allocate them to resources within your VPC.
    
4. **Network Access Control Lists (NACLs):** NACLs act as stateless firewalls, controlling inbound and outbound traffic at the subnet level based on the rules you define. They operate at the IP address level and provide an additional layer of network security for your VPC.
    
5. **Security Groups:** Security groups are virtual firewalls that control inbound and outbound traffic at the instance level (e.g., EC2 instances). You can define rules to permit or restrict traffic based on protocols, ports, and IP addresses.
    
6. **Routing:** Route tables determine where network traffic from your subnets or gateways is directed, allowing you to control the flow of traffic within and outside your VPC.
    
7. **Gateways and Endpoints:** Gateways, such as Internet Gateways, connect your VPC to other networks, while VPC Endpoints enable private connections to AWS services without an Internet gateway or NAT device.
    
8. **Peering Connections:** VPC peering connections allow you to route traffic between resources in two different VPCs, enabling secure communication between your virtual networks.
    
9. **Traffic Mirroring:** With Traffic Mirroring, you can copy network traffic from network interfaces and send it to security and monitoring appliances for deep packet inspection.
    
10. **Transit Gateways:** Transit Gateways act as central hubs, routing traffic between your VPCs, VPN connections, and AWS Direct Connect connections.
    
11. **VPC Flow Logs:** Flow Logs capture information about the IP traffic going to and from network interfaces in your VPC, providing valuable insights for security analysis and compliance auditing.
    

We will implement this architecture in this blog.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721366731628/7e139c84-a107-48ab-9edd-67d0202e1f89.png align="center")

### Step-by-Step Guide:

* **Go to your AWS Management Console**
    
* **Navigate to VPC then click on create vpc.**
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721326998823/b800011a-2e20-4828-8cdf-0b6324352cd7.jpeg align="center")
    
* Next click on VPC and more and set the project name as "aws-vpc-project"
    
* Select the autogenerated IPv4 CIDR block.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721327056962/2febcdf8-2aa2-4d1f-adf2-4013779f06d9.jpeg align="center")
    
* Next, choose 2 Availability Zones
    
* Choose 2 public and private subnets as per our architecture
    
* Choose 1 NAT gateway per Availability zone and select none for vpc endpoints
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721327148735/6e8fed3f-c0bb-4deb-a047-1eb5dbea773a.jpeg align="center")
    
* Next click on create vpc
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721327289149/374fcbc2-2df4-4cf0-8ac7-73725ae5efac.jpeg align="center")
    

### Step -2 Creating EC2 instance using auto scaling groups

Navigate to EC2 instances then click on Auto Scaling groups.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721327394640/0a70a0c8-334d-4edb-88dd-77daf384224d.jpeg align="center")

Give a desired name for your EC2 instance, then select Ubuntu as the OS.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721327458845/028cb8e1-bba8-4231-9a4a-84cb43318283.jpeg align="center")

Next, select t2.micro as the EC2 instance type, and choose the appropriate key pair. Then, click to create a security group.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721327515373/58fff920-2d65-47fe-8311-c7bdb06b31c2.jpeg align="center")

Now, give a desired name and description for your security group. Under the VPC section, select the VPC you created.

Next, under Inbound security group rules, add a rule that allows traffic from anywhere in the world and another rule that allows traffic on port 8000.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721327569287/f8abdf20-272e-4831-a895-708d992175cc.jpeg align="center")

Next click on launch template to create the template.

**AutoScaling Group:**

Head over to the EC2 dashboard and on the left panel click on Auto Scaling Groups. Create an Auto Scaling Group using the Launch Template.

Under the Launch Template section, select the template you created..

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721327771465/fffabffc-c633-4d4e-ab58-1f7e73000653.jpeg align="center")

Next, under the network section, select the VPC you created and choose "PRIVATE SUBNETS". Click on next.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721328017587/9a6a12ce-d88c-40ae-9f12-e9d3413f0d3f.jpeg align="center")

Specify the desired, minimum, and maximum capacity. Click on Next, then click on Create Auto Scaling Group.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721328134381/8c400c23-a508-4fd6-8adc-41e28f19feda.jpeg align="center")

Let's verify that our desired EC2 instances are created. Go to Instances and check to confirm.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721328251821/d55e522d-2e5b-4012-8467-393c276133d5.jpeg align="center")

### Step-3 Creating a bastian Host:

what is bastian host??

**Launch Bastion Host:**

Navigate to EC2 instances and create an instance with the following specifications:

* **Name**: bastion-host
    
* **AMI**: Ubuntu
    
* **Key Pair**: Select your own
    
* **VPC**: Select the VPC you created
    
* Enable auto-assign IP address
    
* Allow SSH from My IP on port 22
    
* Click on create to create the instance.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721328479280/de86d998-c804-4996-8e43-c9efa5ecc184.jpeg align="center")

### Step-4 : Deploying the application.

Before deploying our application on our ec2 instance first we have to connect to the ec2 instance,since our ec2 instance is in private subnet we will use bastian host to connect to our ec2 instance to do that follow these steps:

First we need to copy our keyvalue pair into bastain-host.

Open your command line interface

```bash
 scp -i /path/to/your/local/key-pair-file.pem /path/to/your/local-file-to-copy username@bastionhost-public-ip:/path/to/destination-directory/
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721328690696/d3a1377d-ff61-4873-ac39-73ea947acbeb.jpeg align="center")

Next, SSH into the bastion host

```bash
 ssh -i "key-value-pair" ubuntu@"Ipaddress of your instance"
ls
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721328782053/07a51de2-91b4-400d-b37c-7e47fdf669e7.jpeg align="center")

Next, SSH into our EC2 instance.

```bash
ssh -i "key-value-pair" ubuntu@"Ipaddress of your instance"
ls
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721328844138/2cda5147-da49-4d1a-91b2-7d94ca2c965f.jpeg align="center")

As you can see, we have successfully logged into our EC2 instance using the bastion host.

Now, let's create a simple HTML file for our demo. To do this, use the following command:

```bash
vim index.html
```

```bash
<!DOCTYPE html> 
<html> 
<head> 
<title>aws-vpc-project</title> 
</head> 
<body> 
<h1>AWS Virtual Private Cloud demo/h1> 
</body> 
</html>
```

Save and quit from the file and execute this command to start an HTTP server for this application at Port 8000.

```bash
python3 –m http.server 8000
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721329058563/fa93a6bb-6020-4b8b-800d-e7241f91e847.jpeg align="center")

### Step-5: Configuring Load Balancer:

Now, we will create an Application Load Balancer at the public subnet level. We have installed a Python application on one instance to effectively demonstrate the functionality of the Load Balancer.

We will also observe the health checks in action. Despite having multiple instances available for load balancing, the Load Balancer will only route traffic to the healthy instance, i.e., the one with the application running.

Create application Load Balancer:

Navigate to the EC2 instance dashboard and select Load Balancer.

Click on "Create Load Balancer" and provide a desired name for the load balancer.

Select "Internet-facing" under the scheme section.

Choose IPv4 as the load balancer IP address type.

Under network mapping, select your VPC.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721329268515/f04d0430-a6a2-4c98-8814-5ba6e605322d.jpeg align="center")

Select two public subnets, as the load balancer resides in public subnets.

Next, choose the security group you created earlier.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721329393737/56d53f7c-56d1-4549-9646-9ce99a375937.jpeg align="center")

**Creating Target Group:**

We need to create a Target Group that will have both our instances. Click on Create Target Group -&gt; Open Link in the new tab.

* Under the port section, select 8000 (since our application is running on port 8000).
    
* Select the VPC you created, then click on Next.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721329467827/797cef1f-7a4e-48e4-9e12-5ac800c203d5.jpeg align="center")

Now, under "Register targets," select the two EC2 instances that are in the private subnet.

Click on Create Target Group

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721329674480/0331035d-29cd-448f-8771-a166c2c689e2.jpeg align="center")

Now go back and add this Target Group to the Load Balancer.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721329738066/10e97d67-4a9c-4d78-91bb-2fcdae132f4d.jpeg align="center")

* Click on Create Load Balancer.
    
* The load balancer has been successfully created.
    

Now, navigate to the security group and add a rule to the inbound rules. Set the type to "HTTP" and the source to "Anywhere IPv4". This rule allows HTTP traffic from any IPv4 address.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721329833299/0ee1de26-d16f-4791-9e77-1f5e286ca1f4.jpeg align="center")

### Step-6:Accessing the application:

Now come back to the Load Balancer dashboard. Copy the DNS name and paste it into the browser.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721329946050/27bfa22d-4eae-4663-ab7a-177ea9d9d7f8.jpeg align="center")

**Conclusion**

Congrats! You've set up a top-notch AWS VPC that's ready for production. It's got high availability, scalability, and beefed-up security. This means your app is now resilient, cost-efficient, and secure, perfect for real-time deployment. Great job!