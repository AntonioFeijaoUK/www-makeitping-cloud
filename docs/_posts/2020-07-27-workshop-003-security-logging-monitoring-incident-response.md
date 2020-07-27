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



---

## Warming up name default VPC

* Region **US East (N. Virginia)** `us-east-1`

* Services --> VPC
  * Default name `Default-VPC`on existent VPC

* Services --> EC2
  * `Launch Instance`
  1.) Choose AMI - `Amazon Linux 2 AMI (HVM), SSD Volume Type`
  2.) Choose Instance Type - `t2.micro, Free tier eligible`
  3.) Configure Instance - leave default, add in the "Advanced Details, User data":
  ```bash
  #!/bin/bash
  yum update -y
  ```






