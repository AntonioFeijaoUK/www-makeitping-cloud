---
layout: post
title: Ping from an EC2 instance on private subnet and on different VPC via TGW
author.name: Antonio Feijao UK
author: AntonioFeijao UK
toc: true
categories:
  - ec2
  - networking
tags:
  - tgw
  - vpc
  - networking
  - ec2
---

In this exercise we will `ping` from [EC2 instance](https://aws.amazon.com/ec2/) in a [private subnet](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario2.html), going across a [Transit Gateway (TGW)](https://aws.amazon.com/transit-gateway/) until we reach another [EC2 instance](https://aws.amazon.com/ec2/) in another [private subnet](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario2.html).

---

## Table of contents

* auto-gen TOC:
{:toc}

---

## Useful links

* [sli.io](https://sli.do/)

* [whiteboard](https://awwapp.com/b/u6hxhr9cvgwgw/)

* feedback (coming soon)

---

## Simplified architect diagram

* image ![ec2-vpc-tgw-vpc-ec2-make-it-ping](/assets/images/ec2-vpc-tgw-vpc-ec2-make-it-ping.png)

---

## Hands-on creating VPCs using VPC wizard

* Sign-In into you AWS Dev/Test account

* Check your region - `N. Virginia` - US East (N. Virginia) us-east-1

* Services search and select `VPC` "Isolated Cloud Resources"

* image ![aws-services-search-vpc](/assets/images/aws-services-search-vpc.png)

---

### Allocating an Elastic IP address

* VPC

* Elastic IPs

* Click `Allocate Elastic IP`

* Leave defaul - `Amazon's pool of IPv4 addresses`, click `Allocate`

* image ![vpc-allocate-elastic-ip](/assets/images/vpc-allocate-elastic-ip.png)

---

### Creating the VPC-10-0-0

* Click `VPC Dashboard`

* Click `Launch VPC Wizard`

* image ![launch-vpc-wizard](/assets/images/launch-vpc-wizard.png)

* Select second option - `VPC with Public and Private Subnets` and click `Select`

* image ![vpc-with-public-and-private-subnets](/assets/images/vpc-with-public-and-private-subnets.png)

* IPv4 CIDR block: `10.0.0.0/16`

  > meaning IP range from 10.0.0.0 up to 10.0.255.255
  > "The allowed block size is between a /16 netmask (65,536 IP addresses) and /28 netmask (16 IP addresses). [details here](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html#VPC_Sizing)"
  > IPs ending in `.0`, `.1`, `.2`, `.3` , `.255` are AWS reserved IPs - [details here](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html)
  > [more details here on VPC IP Addressing](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-ip-addressing.html))
  > [more details here on Private Address Space](https://tools.ietf.org/html/rfc1918#section-3)

* VPC Name: `VPC-10-0-0`

* Public subnet's IPv4 CIDR: `10.0.1.0/24`

* Availability Zone: `us-east-1a`

* Public subnet name: `Public subnet-10-0-1`

* Private subnet's IPv4 CIDR: `10.0.4.0/24`

* Availability Zone: `us-east-1a`

* Private subnet name: `Private subnet-10-0-4`

* `Use a NAT instance instead).` - `t2.nano` - `No key pair`

* leave remaning options as default, click `Create VPC`

* image ![vpc-10-0-0](/assets/images/vpc-10-0-0.png)

* Click `OK` when completed

---

### Creating the VPC-192-168

* DO NOT click on `Create VPC`

* Get back to the `VPC Wizard` by clicling on `VPC Dasboard`, top-left hand side

* Click on `Launch VPC Wizard`

* Select second option - `VPC with Public and Private Subnets` and click `Select`

* IPv4 CIDR block: `192.168.0.0/16`

* VPC Name: `VPC-192-168`

* Public subnet's IPv4 CIDR: `192.168.1.0/24`

* Availability Zone: `us-east-1a`

* Public subnet name: `Public subnet-192-168-1`

* Private subnet's IPv4 CIDR: `192.168.4.0/24`

* Availability Zone: `us-east-1a`

* Private subnet name: `Private subnet-192-168-4`

* Click on the right-hand side `Use a NAT instance instead).` - `t2.nano` - `No key pair`

* leave remaning options as default, click `Create VPC`

* image ![vpc-192-168.png](/assets/images/vpc-192-168.png)

You should now have 2 new VPCs - `VPC-10-0-0` and `VPC-192-168`

Take 2 minutes to review your new subnets and route tables.

Tip - Use `Filter by VPC`

- Review `Subnets`

  - Do you know the difference between `Availability Zone` and `Availability Zone ID`?

- Review `Route Tables`

  - Give a name to the route tables - click on tab `Routes` tabs
  - Check for Destination `0.0.0.0/0`, Target `ine-xx (or nat-xxx)` - This is your `Private-Route-Table`
  - Check for Destination `0.0.0.0/0`, Target `igw-xx (or nat-xxx)` - This is your `Public-Route-Table`
  - Notice the `Main` = `Yes` on route table and the message:  *"The following subnets have not been explicitly associated with any route tables and are therefore associated with the main route table"

- Review `Internet Gateways`

- Review `NAT Gateways`

  - Noticed that VPC-192-168 as no `NAT Gateways`, instead we used the `nat instance` which is an EC2 managed by you.
  - Link on AWS [Comparison of NAT instances and NAT gatewaysn](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-comparison.html)


---

## Creating the TGW

* Still in `VPC service`

* Scroll down to `Transit Gateway`

* Click `Create Transit Gateway`

* Leave all default, click `Create Transit Gateway`

* That's all for now

---

## Creating the SSM role for EC2

* Change to service to `Identity and Access Management (IAM)`

* Click `Role`, then click `Create role`

* Leave default of `AWS Service`, select `EC2`

* Click `Next: Permissions`

* In the search box type `SSM` and from the list select `AmazonEC2RoleforSSM`

* Click `Next: Tags`

* Key - `Name`, Value - `ROLE-AmazonEC2RoleforSSM`, and them click `Next: Review`

* Role name `ROLE-AmazonEC2RoleforSSM`, click `Create role`

We done here.

---

## Launching EC2 instance-10-0-4-first in VPC-10-0 and private subnet

* Change to service to `EC2 - Virtual Servers in the Cloud`

* Click on `Launch instance`

* Step 1: Choose an Amazon Machine Image (AMI)
  
  * Select first `Amazon Linux 2 AMI (HVM), SSD Volume Type`, click `Select`

* Step 2: Choose an Instance Type

  * Leave the `Free tier eligible` option, click `Next: Configure Instance Details`

* Step 3: Configure Instance Details
  
  * In `Network` select `VPC-10-0`
  
  * In `Subnet` select `Private subnet-10-0-4`
  
  * In `IAM role` select `ROLE-AmazonEC2RoleforSSM`

  * Suggestion, in `Advanced Details` you can add the following scrip to the  `User data`
  * Always review any script before running.
  * This below script will install httpd service to displays some [instance-metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html).

```bash

#!/bin/bash

yum update -y

curl https://gist.githubusercontent.com/AntonioFeijaoUK/d8533a71e5ecff2971f6859a7be426da/raw/3d0930004b937f6dd7f273021218327b7129d609/aws-ec2-userdata-landing-webpage.sh | bash

```

  * Click `Next: Add Storage`

* Step 4: Add Storage

  * Leave Default, click `Next: Add Tags`


* Step 5: Add Tags

  * Key - `Name`, Value - `instance-10-0-4-first`
  
  * Click `Next: Configure Security Group`
  
* Step 6: Configure Security Group

  * Security group name: `SG-inbound-from-192-168`
  
  * Security group name: `SG-inbound-from-192-168`

  * Change first Type to `HTTP`, source `192.168.0.0/16`
  
  * Click `Add Rule`, Type `All ICMP - IPv4`, source `192.168.0.0/16`
  
  * Do you need more rules? or less?
  
  * Click `Review and Launch`
  
* Step 7: Review Instance Launch

  * Review, when happy click `Launch`
  
  * Change to `Proceed without a key pair`, `I acknowledge...`, click `Launch Instances`

---

Now let's do the same for the instance in the other vpc. Remember to change order of IPs.

---

## Launching EC2 instance-192-168-4-first in VPC-192-168 and private subnet

* Change to service to `EC2 - Virtual Servers in the Cloud`

* Click on `Launch instance`

* Step 1: Choose an Amazon Machine Image (AMI)
  
  * Select first `Amazon Linux 2 AMI (HVM), SSD Volume Type`, click `Select`

* Step 2: Choose an Instance Type

  * Leave the `Free tier eligible` option, click `Next: Configure Instance Details`

* Step 3: Configure Instance Details
  
  * In `Network` select `VPC-192-168`
  
  * In `Subnet` select `Private subnet-192-168-4`
  
  * In `IAM role` select `ROLE-AmazonEC2RoleforSSM`

  * Suggestion, in `Advanced Details` you can add the following scrip to the  `User data`
  * Always review any script before running.
  * This below script will install httpd service to displays some [instance-metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html).

```bash

#!/bin/bash

yum update -y

curl https://gist.githubusercontent.com/AntonioFeijaoUK/d8533a71e5ecff2971f6859a7be426da/raw/3d0930004b937f6dd7f273021218327b7129d609/aws-ec2-userdata-landing-webpage.sh | bash

```

  * Click `Next: Add Storage`

* Step 4: Add Storage

  * Leave Default, click `Next: Add Tags`


* Step 5: Add Tags

  * Key - `Name`, Value - `instance-192-168-4-first`
  
  * Click `Next: Configure Security Group`
  
* Step 6: Configure Security Group

  * Security group name: `SG-inbound-from-10-0`
  
  * Security group name: `SG-inbound-from-10-0`

  * Change first Type to `HTTP`, source `10.0.0.0/16`
  
  * Click `Add Rule`, Type `All ICMP - IPv4`, source `10.0.0.0/16`
  
  * Do you need more rules? or less?
  
  * Click `Review and Launch`
  
* Step 7: Review Instance Launch

  * Review, when happy click `Launch`
  
  * Change to `Proceed without a key pair`, `I acknowledge...`, click `Launch Instances`

---




## Testing and Advanced commands

* Can you ping on both directions? - `ping xxx instance-ip xxx`

* Can you curl on both directions? (with `python3` package you can start a simple webserver `python3 -m http.server 80`
 * to install python3, type in the instance command line - `sudo yum install python3`

* **Advanced** - Check the throughput and bandwidth with `iperf` ?

* **Advanced** - `iperf` via VPC Peering, can you change the routing to use VPC Peering? Is it the same if you do a VPC peering?
  * Detail about [VPC Peering](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html)

* Get you public ip `curl ipinfo.io/json`

* Check the local ip addresses - `ifconfig`

* Check TCP connections - `sudo netstat -pant`

* **Advanced** - If you enable SSM Endpoint, can you see a difference on the connections?

---

## Alternative creating VPCs with AWS CLI

```bash

aws ec2 create-vpc --cidr-block 10.0.0.0/16

VPC_ID="Copy-VPC-ID-value-from-previous-command"

```

### Create subnets with that VPC

```bash

aws ec2 create-subnet --vpc-id ${VPC_ID} --cidr-block 10.0.1.0/24
aws ec2 create-subnet --vpc-id ${VPC_ID} --cidr-block 10.0.2.0/24
aws ec2 create-subnet --vpc-id ${VPC_ID} --cidr-block 10.0.3.0/24

aws ec2 create-subnet --vpc-id ${VPC_ID} --cidr-block 10.0.4.0/24
aws ec2 create-subnet --vpc-id ${VPC_ID} --cidr-block 10.0.5.0/24
aws ec2 create-subnet --vpc-id ${VPC_ID} --cidr-block 10.0.6.0/24

```

### Create a Internet Gateway

```bash

aws ec2 create-internet-gateway

```

### Create NAT Gateway

* First allocate and Elastic IP

* Create NAT Gateway

### Creating Route tables and add the default routes IGW and NGW

* Creating `Public-route-table-192-168` and assigned the `Internet Gateway (IGW)`

0.0.0.0/0 -> IGW-xxxx

...

* Creating `Private-route-table-10-0` and assigned the `NAT Gateway`

0.0.0.0 -> NGW-xxx


... work in progress


### Associating the Subnets with Route tables

... work in progress


---

## Altervative creating VPCs with Cloud Formation

.... working in progress, any voluntair

---

## AWS Resources

* [VPC subnets commands example](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-subnets-commands-example.html)

* [EC2 instance](https://aws.amazon.com/ec2/)
* [VPC with Private and Public Subnet](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario2.html)
* [Transit Gateway (TGW)](https://aws.amazon.com/transit-gateway/)
