---
layout: post
date: 2020-07-27 16:30:00 +0000
last_modified_at: 
title: "Workshop 003 - AWS Security logging monitoring and incident reponse"
categories:
  - AWS
  - Security
  - Logging
  - Monitoring
  - Incident-Response
  - VPC
  - EC2
tags:
  - security
  - vpc
  - logging
  - monitoring
  - response
---

**Workshop 003 (comming soon...)** - AWS Security fundamentals, logging, monitoring and incident reponse.

---

## Useful links

* ...links to be shared

---

## Confirm your Region

* Region **US East (N. Virginia)** `us-east-1`

---

## Enable GuardDuty

Services --> `GuardDuty`

* Enable GuardDuty

---

## Create AWS Systems Manager role to access the webserver

Services --> `IAM`

* Roles
  * Create role, AWS service, `EC2`
  * Attach permissions policies: `AmazonEC2RoleforSSM`
  * Add tags: `Name` - `ROLE-AmazonEC2RoleforSSM`
  * Role name: `ROLE-AmazonEC2RoleforSSM`

---

## Name the Default VPC

Services --> `VPC`

* Default name `Default-VPC`on existent VPC

---

## Launch first webserver with Public IP

Services --> `EC2` --> `Launch Instance`

* 1 - Choose AMI - `Amazon Linux 2 AMI (HVM), SSD Volume Type`

* 2 - Choose Instance Type - `t2.micro, Free tier eligible`

* 3 - Configure Instance
  * IAM role: `ROLE-AmazonEC2RoleforSSM`
  * **Advanced Details**, `User data`:

```bash
#!/bin/bash
yum  update -y

curl https://gist.githubusercontent.com/AntonioFeijaoUK/d8533a71e5ecff2971f6859a7be426da/raw/3d0930004b937f6dd7f273021218327b7129d609/aws-ec2-userdata-landing-webpage.sh | bash

# end script
```

* 4 - Add Storage
  * Click `Next`

* 5 - Add Tags
  * `Name` - `webserver-001-default-vpc`

* 6 - Configure Security Group
  * `Create a new security group`
  * Name and Description: `http-world`
  * Type: `HTTP`, Protocol: `TCP`, Port Range: `80`, Source Custom: `0.0.0.0/0`, Description

* 7 - Review
  * Launch instance without key par - `Proceed without a key pair`, acknowledge and click `Launch Instances`

---


