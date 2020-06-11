---
layout: post
date: 2020-05-18 00:15:00 +0000
last_modified_at: 2020-06-11 00:55:00 +0000
title: "Transit Gateway with 2 route tables connecting shared-services VPC and isolated Dev Test and Prod VPCs and all connected back to on-prem via Customer Gateway VPN"
categories:
  - AWS
  - Networking
  - TGW
  - VPC
  - EC2
tags:
  - tgw
  - vpc
  - networking
  - ec2
  - aws
---

**Coming soon...** - **[Makeitping.cloud](https://www.makeitping.cloud) Workshop 002** - In this hands-on workshop we will use a Transit Gateway with 2 route-tables to isolate Dev-VPC, Test-VPC and Prod-VPC from the Shared-Services VPC. We will also add a connection to On-prem-VPC.

**Requirements** : All VPCs can connect to with On-prem-VPC and Shared-services-VPC. Dev-VPC, Test-VPC, Prod-VPC cannot interact with each other.

We will use the 2 Route tables from the Transit Gateway to achive these requirements.

In this workshop we will have:

* 1 Transit Gateway (TGW)
  - with 2 Route-tables (routing domains)

* 5 VPCs
  - Dev-VPC
  - Test-VPC
  - Prod-VPC
  - Shared-services-VPC
  - On-prem (another VPC to simulate on-prem network)

---

## Useful links

* [sli.io](https://sli.do/)

* whiteboard(*coming soon..*)

* Feedback(*feedback is now closed, thank you*)


---

## Table of contents

* auto-gen TOC:
{:toc}

`work in progres....`

* getting ideas from:
  * <https://d1.awsstatic.com/whitepapers/building-a-scalable-and-secure-multi-vpc-aws-network-infrastructure.pdf>
  * [AWS re:Invent 2019: [REPEAT 1] AWS Transit Gateway reference architectures for many VPCs (NET406-R1)](https://youtu.be/9Nikqn_02Oc)


![Reference Network Architecture NET406-R1](/assets/images/Reference-Network-Architecture-NET406-R1.png)
