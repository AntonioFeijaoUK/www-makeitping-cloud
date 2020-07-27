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

## Check your region

* Region **US East (N. Virginia)** `us-east-1`

---

## Enable GuardDuty

Services --> GuardDuty

* Enable GuardDuty

---

## Name the Default VPC

Services --> VPC
* Default name `Default-VPC`on existent VPC

Services --> EC2 --> Launch Instance

* 1 - Choose AMI - `Amazon Linux 2 AMI (HVM), SSD Volume Type`

* 2 - Choose Instance Type - `t2.micro, Free tier eligible`

* 3 - Configure Instance - leave default, add in the "Advanced Details, User data":
```bash
#!/bin/bash
yum  update -y
# end script
```

* 4 - Add Storage
  * Click `Next`

* 5 - Add Tags
  * `Name` - `instance-001-default-vpc`

* 6 - Configure Security Group
  * `Select an existing security group`
  * Select existent security group - `default VPC security group`

* 7 - Review
  * Launch instance without key par - `Proceed without a key pair`, acknowledge and click `Launch Instances`



