---
layout: post
title: Ping from an EC2 instance on private subnet and on different VPC via TGW
categories:
  - ec2
  - networking
tags:
  - tgw
  - vpc
  - networking
  - ec2
---

In this exercise we will `ping` from [EC2 instance](https://aws.amazon.com/ec2/) in a [private subnet](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario2.html), going across a [Transit Gateway (TGW)](https://aws.amazon.com/transit-gateway/) until we reach another [EC2 instance](https://aws.amazon.com/ec2/) in another [private subnet].

---

## Whiteboard for discussion

- whiteboard [link here](https://awwapp.com/b/u6hxhr9cvgwgw/)

---

## Simplified architect diagram

![ec2-vpc-tgw-vpc-ec2-make-it-ping](/assets/images/ec2-vpc-tgw-vpc-ec2-make-it-ping.png)

---

## Hands-on using VPC wizard

- Login into you AWS Dev/Test account

- Check your region - `N. Virginia` - US East (N. Virginia) us-east-1

- Services `VPC` "Isolated Cloud Resources"

- Click on `Launch VPC Wizard`

- Select second option - `VPC with Public and Private Subnets` and click `Select`

### Creating the VPC-10-0-0

- IPv4 CIDR block: `10.0.0.0/16`
   + (meaning IP range from 10.0.0.0 up to 10.0.255.255, [more details here on VPC IP Addressing](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-ip-addressing.html))

- VPC Name: `VPC-10-0-0`

- Public subnet's IPv4 CIDR: `10.0.1.0/24`

- Availability Zone: `us-east-1a`

- Public subnet name: `Public subnet-10-0-1`

- Private subnet's IPv4 CIDR: `10.0.4.0/24`

- Availability Zone: `us-east-1a`

- Private subnet name: `Private subnet-10-0-4`

- `Use a NAT instance instead).` - `t2.nana` - `No key pair`

- leave remaning options as default, click `Create VPC`


### Creating the VPC-192-168

- IPv4 CIDR block: `192.168.0.0/16`
   + meaning IP range from 10.0.0.0 up to 10.0.255.255
   + "The allowed block size is between a /16 netmask (65,536 IP addresses) and /28 netmask (16 IP addresses). [details here](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html#VPC_Sizing)"
   + IPs ending in `.0`, `.1`, `.2`, `.3` , `.255` are AWS reserved IPs - [details here](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html) )
   + [more details here on VPC IP Addressing](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-ip-addressing.html))
   + [more details here on Private Address Space](https://tools.ietf.org/html/rfc1918#section-3)

- VPC Name: `VPC-10-0-0`

- Public subnet's IPv4 CIDR: `10.0.1.0/24`

- Availability Zone: `us-east-1a`

- Public subnet name: `Public subnet-10-0-1`

- Private subnet's IPv4 CIDR: `10.0.4.0/24`

- Availability Zone: `us-east-1a`

- Private subnet name: `Private subnet-10-0-4`

- `Use a NAT instance instead).` - `t2.nana` - `No key pair`

- leave remaning options as default, click `Create VPC`













## Create a VPC-10-0

```shell

aws ec2 create-vpc --cidr-block 10.0.0.0/16

#VPC_ID=.....
```

## Create subnets with that VPC

```shell

aws ec2 create-subnet --vpc-id ${VPC_ID} --cidr-block 10.0.1.0/24
aws ec2 create-subnet --vpc-id ${VPC_ID} --cidr-block 10.0.2.0/24
aws ec2 create-subnet --vpc-id ${VPC_ID} --cidr-block 10.0.3.0/24

aws ec2 create-subnet --vpc-id ${VPC_ID} --cidr-block 10.0.4.0/24
aws ec2 create-subnet --vpc-id ${VPC_ID} --cidr-block 10.0.5.0/24
aws ec2 create-subnet --vpc-id ${VPC_ID} --cidr-block 10.0.6.0/24
```


# Create a Internet Gateway

```shell

aws ec2 create-internet-gateway

```

.... working in progress




---

## AWS Resources

- <https://docs.aws.amazon.com/vpc/latest/userguide/vpc-subnets-commands-example.html>

- [EC2 instance](https://aws.amazon.com/ec2/)
- [VPC with Private and Public Subnet](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario2.html)
- [Transit Gateway (TGW)](https://aws.amazon.com/transit-gateway/)

