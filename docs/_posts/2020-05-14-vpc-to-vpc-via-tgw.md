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

## Table of content

* auto-gen TOC:
{:toc}

---

## Useful links

* [sli.io](https://sli.do/)

* [whiteboard](https://awwapp.com/b/u6hxhr9cvgwgw/)

* feedback (coming soon)

---

## Simplified architect diagram

![ec2-vpc-tgw-vpc-ec2-make-it-ping](/assets/images/ec2-vpc-tgw-vpc-ec2-make-it-ping.png)

---

## Hands-on creating VPCs using VPC wizard

* Sign-In into you AWS Dev/Test account

* Check your region - `N. Virginia` - US East (N. Virginia) us-east-1

*-* Services search and select `VPC` "Isolated Cloud Resources"

![aws-services-search-vpc](/assets/images/aws-services-search-vpc.jpeg)

### Creating the VPC-10-0-0

* Click on `Launch VPC Wizard`

![launch-vpc-wizard](/assets/images/launch-vpc-wizard.jpeg)

* Select second option - `VPC with Public and Private Subnets` and click `Select`

![vpc-with-public-and-private-subnets](/assets/images/vpc-with-public-and-private-subnets.jpeg)

* IPv4 CIDR block: `10.0.0.0/16`
  * meaning IP range from 10.0.0.0 up to 10.0.255.255
  * "The allowed block size is between a /16 netmask (65,536 IP addresses) and /28 netmask (16 IP addresses). [details here](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html#VPC_Sizing)"
  * IPs ending in `.0`, `.1`, `.2`, `.3` , `.255` are AWS reserved IPs - [details here](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html)
  * [more details here on VPC IP Addressing](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-ip-addressing.html))
  * [more details here on Private Address Space](https://tools.ietf.org/html/rfc1918#section-3)

* VPC Name: `VPC-10-0-0`

* Public subnet's IPv4 CIDR: `10.0.1.0/24`

* Availability Zone: `us-east-1a`

* Public subnet name: `Public subnet-10-0-1`

* Private subnet's IPv4 CIDR: `10.0.4.0/24`

* Availability Zone: `us-east-1a`

* Private subnet name: `Private subnet-10-0-4`

* `Use a NAT instance instead).` - `t2.nano` - `No key pair`

* leave remaning options as default, click `Create VPC`

![vpc-10-0-0](/assets/images/vpc-10-0-0.jpeg)

* Click `OK` when completed

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

* `Use a NAT instance instead).` - `t2.nano` - `No key pair`

* leave remaning options as default, click `Create VPC`

---

## Creating the TGW

## Creating SSM role for EC2

## Launching 2 EC2 instance in different VPCs and private subnets

* suggestion for `user data` for EC2

```bash

#!/bin/bash

yum update -y

curl https://gist.githubusercontent.com/AntonioFeijaoUK/d8533a71e5ecff2971f6859a7be426da/raw/3d0930004b937f6dd7f273021218327b7129d609/aws-ec2-userdata-landing-webpage.sh | bash

```


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
