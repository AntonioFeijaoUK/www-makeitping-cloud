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

Workshop 003, hands-on learning on AWS. This workshop covers AWS network infrastructure fundamentals, AWS Security fundamentals, logging, monitoring and incident reponse.

Link to [AWS Products page](https://aws.amazon.com/products/)

---

## Table of contents

* auto-gen TOC:
{:toc}

---

## Useful links

* ...links to be shared here

* [dasbboard](https://dashboard.eventengine.run/login)

* Feedback day 2...

---

## Confirm your Region

* Region **US East (N. Virginia)** `us-east-1`

---

## Enable GuardDuty

![Amazon GuardDuty](/assets/images/product-page-diagram-Amazon-GuardDuty_how-it-works.png)

Services --> `GuardDuty`

* Enable GuardDuty

* Settings
  * Sample findings, `Generated sample findings`

* Review sample findings in `Findings`

---

## Create IAM Read Only Group and User

Service --> `IAM`

Create Group 

* Group, Create New Group: `NAME-RO-GROUP`

* Attach Policy, Filter `AWS Managed`, search and select `ReadOnlyAccess`

* Create Group


Create Users

* Add users

* User name: `NAME-RO`, select only `AWS Management Console access`, take note of password and login link

* Add user: Add to `NAME-RO-GROUP`

* Next, Create user

---

## Practice Cross Account Role permissions

* Create Role, `Another AWS account`, `Specify accounts that can use this role`, 

* Practice Switch roles

* Aditional info about [Roles and Session Duration](https://docs.aws.amazon.com/singlesignon/latest/userguide/howtosessionduration.html)

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

* `Your VPCs`

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
  * Create a new security group
  * Name and Description: `http-world`
  * Type: `HTTP`, Protocol: `TCP`, Port Range: `80`, Source Custom: `0.0.0.0/0`, Description

* 7 - Review
  * Launch instance without key par - `Proceed without a key pair`, acknowledge and click `Launch Instances`

---

## Connect to the instance with AWS Systems Manager

Services --> `AWS Systems Manager`

* Managed Instances, select your instance, `Actions`, `Start Session`

Sample suggestive test commands

```bash
sudo su

curl ipinfo.io

## check the instance public IP and open a browser on that ip

cd /var/log/httpd/

## check the files `access_log` and `error_log`

cat access_log

cat error_log

## ...

```

---

## Part 1 workshop review

* Enabled [Amazon GuardDuty](https://aws.amazon.com/guardduty/)
    * Supported security [findings](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-active.html).

* Practiced IAM cross account Roles

* Created an EC2 Service Role to access the webserver instance without using key par

* Named the Default VPC - *more about* [VPC Security](https://docs.aws.amazon.com/vpc/latest/userguide/security.html)

* Created instance with a script in the `user data` field to install a web service service at launch

* The web service instance only protection is the `inbound` [Security Group](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html) with `http to the world`, which allow ports 80 from anywhere `0.0.0.0/0`

Currently this instance is exposed to the world (`0.0.0.0/0`), we will start adding protection and apply best pratices.

---

## Next we will look at

* [Amazon EC2](https://aws.amazon.com/ec2/)
> Secure and resizable compute capacity in the cloud. Launch applications when needed without upfront commitments.

* [Amazon EC2 Auto Scaling](https://aws.amazon.com/ec2/autoscaling/)
> Add or remove compute capacity to meet changes in demand

* [Amazon Inspector](https://aws.amazon.com/inspector/)
> Automated security assessment service to help improve the security and compliance of applications deployed on AWS

* [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/)
> Observability of your AWS resources and applications on AWS and on-premises

* [AWS Auto Scaling](https://aws.amazon.com/autoscaling/)
> Application scaling to optimize performance and costs

* [AWS CloudTrail](https://aws.amazon.com/cloudtrail/)
> Track user activity and API usage

* [AWS Config](https://aws.amazon.com/config/)
> Record and evaluate configurations of your AWS resources

* [AWS Well-Architected Tool](https://aws.amazon.com/well-architected-tool/)
> Review your architecture and adopt best practices

* [AWS Trusted Advisor](https://aws.amazon.com/premiumsupport/technology/trusted-advisor/)
> Reduce costs, increase performance, and improve security

---

## Create a multi-layer secure and high available webserver app architecture

Lets create a web app well architected, secure and high available

Sample diagram ![sample-3-tier-web-app-v2](/assets/images/sample-3-tier-web-app-v2.png)


---

## Create a new VPC with Private and Public subnets

![AWS sample network AZ 1](/assets/images/aws-sample-network-az1.png)

![AWS sample network multi AZ private subnets public subnet](/assets/images/aws-sample-network-multiple-az-public-and-private-subnet.png)

Services --> VPC

Create VPC

* Name: `VPC-10-0-0-0`
* IP Range: `10.0.0.0/16`

Select VPC, `Actions`, `Edit DNS hostname`

* enable `DNS hostnames` for the VPC


Create Public Subnets

* `Public-Subnet-a-10-0-1`, `us-east-1a`, `10.0.1.0/24`
* `Public-Subnet-a-10-0-2`, `us-east-1b`, `10.0.2.0/24`
* `Public-Subnet-a-10-0-3`, `us-east-1c`, `10.0.3.0/24`


Create Private Subnets

* `Private-Subnet-a-10-0-4`, `us-east-1a`, `10.0.4.0/24`
* `Private-Subnet-a-10-0-5`, `us-east-1b`, `10.0.5.0/24`
* `Private-Subnet-a-10-0-6`, `us-east-1c`, `10.0.6.0/24`


Create Public and Private Route tables

* `Public-Route-Table`

* `Private-Route-Table-a`
* `Private-Route-Table-b`
* `Private-Route-Table-c`


Do the `Subnet Associations`

* `Public-Route-Table` --> `Public-Subnet-a-10-0-1`
* `Public-Route-Table` --> `Public-Subnet-a-10-0-2`
* `Public-Route-Table` --> `Public-Subnet-a-10-0-3`

* `Public-Route-Table-a` --> `Public-Subnet-a-10-0-4`
* `Public-Route-Table-b` --> `Public-Subnet-a-10-0-5`
* `Public-Route-Table-c` --> `Public-Subnet-a-10-0-6`


Create 1x `Internet gateways`

* Name `IGW-VPC-10-0-0-0`

* Select and `Attach to VPC`, `VPC-10-0-0-0`


Create 3x `NAT Gateway`, associate with the currespondent Public Subnet

* `Name`, `NGW-Public-subnet-10-0-1`

* `Name`, `NGW-Public-subnet-10-0-2`

* `Name`, `NGW-Public-subnet-10-0-3`


Go back to route tables

* For `Public-Route-Table`, add `0.0.0.0/0` to the single `IGW-VPC-10-0-0-0` (Internet Gateway)

* For each `Private Route table`, add `0.0.0.0/0` route to the correspondent `NGW-Public-subnet-10-0-x`





---

## Enable VPC flow logs to CloudWatch Logs

Service --> `CloudWatch`

* Go to `Log groups`

* Create log group, Name: `Flow-logs-VPC-10-0-0-0`


Service --> `VPC`

Select your VPC

* Down the page, tab `Flow Logs`

* Create flow logs, Filter `All`

* Destination `Send to CloudWatch Logs`, select log group previously created

* IAM role, click on `Set Up Permissions`, refresh, select IAM role just created

* Keep `AWS Default format`

`${version} ${account-id} ${interface-id} ${srcaddr} ${dstaddr} ${srcport} ${dstport} ${protocol} ${packets} ${bytes} ${start} ${end} ${action} ${log-status}`

---

## Enable VPC flow logs to S3 bucket

Service --> `S3`

* Create new bucket

* Confirm Region

* Name `s3-vpc-flow-logs-vpc-10-0-0-0-xxxxx`, Next, Next, Next

* Copy bucket ARN, sample `arn:aws:s3:::s3-vpc-flow-logs-vpc-10-0-0-0-xxxxx`


Service --> `VPC`

Select your VPC

* Down the page, tab `Flow Logs`

* Create flow logs, Filter `All`

* Destination `Send to an S3 bucket`, past the S3 ARN, sample `arn:aws:s3:::s3-vpc-flow-logs-vpc-10-0-0-0-xxxxx`

* Keep `AWS Default format`

`${version} ${account-id} ${interface-id} ${srcaddr} ${dstaddr} ${srcport} ${dstport} ${protocol} ${packets} ${bytes} ${start} ${end} ${action} ${log-status}`

---

## Launch an Ubuntu instance in a Private Subnet

Similar previously created `Amazon Linux 2 AMI (HVM), SSD Volume Type`, no launch a version of Ubuntu and select the new VPC and a private subnet

Services --> VPC

* Launch instance

  * `Ubuntu Server 18.04 LTS (HVM), SSD Volume Type`

  * VPC-10-0-0-0

  * Private Subnet

  * ROLE-AmazonEC2RoleforSSM
  
  * Next, Add Tags `Name` - `Ubuntu-instance-001-no-updates`

  * Select an existing security group, `default VPC security group`

  * Launch without a key par


--

## Launch a Windows instance in a Private Subnet

* same steps above but choose Windows instance instead


---

## Amazon Inspector

[Amazon Inspector](https://aws.amazon.com/inspector/)

Services --> `Inspector`

* Get started

* Advanced setup

* Name: `Assessment-Target-All-Instances`, Next

* Name: `Assessment-Template-Default`

* untick [ ] Assessment Schedule (this is to prevent Inspector to run on creation), Next

* Create


Create your own `Assessment targets` for all instances

* Create, Name `Assessment-all-instances`

* Verify that you could filter by tags,example `Prod`, `Dev` or `Test`

* For this workshop tick `All Instances`, and `Install Agents`

* Save


Create your own `Assessment templates` for CVEs

* Create, Name: `Assessment-for-CVEs-only`

* Target name: `Assessment-all-instances`

* Rules packages: `Common Vulnerabilities and Exposures-1.1`
  * Review the other possible packages

* Duration: `15 minutes`


Select your ``Assessment-for-CVEs-only``

* Review `Preview Target`

* Click `Run`


Confirm assessment is running under `Assessment runs`

---

## Investigate with CloudWatch Logs S3 Logs groups and AWS CloudTrail

* Review `Logs Insights` in CloudWatch Logs

* Review VPC flow logs, sample filter

* `[version, account_id, interface_id, srcaddr, dstaddr, srcport, dstport, protocol, packets, bytes, start, end, action != "ACCEPT", log_status ]`

* `[version, account_id, interface_id, srcaddr, dstaddr, srcport, dstport, protocol, packets, bytes, start, end, action != "-", log_status ]`

* `[version, account_id, interface_id, srcaddr, dstaddr, srcport, dstport = "53", protocol, packets, bytes, start, end, action, log_status ]`

* <https://docs.aws.amazon.com/athena/latest/ug/vpc-flow-logs.html>

* <https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/FilterAndPatternSyntax.html>

* Protocol numbers <http://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml>


With [AWS Glue](https://aws.amazon.com/glue/), you can create crawlers to check the S3 metadata

then, with [Amazon Athena](https://aws.amazon.com/athena/), you can run standard SQL. Athena is serverless, so there is no infrastructure to manage, and you pay only for the queries that you run.


---

## Workshop with CloudTrail API and CloudWatch


![cloudtrail-api-review](/assets/images/cloudtrail-api-review.png)

Let do this! - [How can I use CloudTrail to review what API calls and actions have occurred in my AWS account?](https://youtu.be/4ztTv5rIRv8)

* [AWS CloudTrail](https://aws.amazon.com/cloudtrail/)

```SQL

select count (*) as TotalEvents, useridentity.username, eventname
from cloudtrail_logs_sample-REVIEW-YOUR-LOG-GROUP-NAME
where useridentity.type = 'IAMUser'
group by useridentity.username, eventname
order by TotalEvents desc;

```

---

## Day 2 

* [ALB AutoScaling and Certificate Manager](http://alb.demo.root.pt/) - from <http://alb.demo.root.pt/>

* [Guarduty automatically blocks suspicious hosts](https://aws.amazon.com/blogs/security/how-to-use-amazon-guardduty-and-aws-web-application-firewall-to-automatically-block-suspicious-hosts/)
 
* [Demo 1 Security Hub](https://youtu.be/o0NDi01YPXs?t=1793) from "Introduction to AWS Security Hub - AWS Online Tech Talks"

* [Demo 2 Security Hub](https://youtu.be/h3yAxwXv5bE?t=847) from "Continuous Monitoring for Compliance with AWS Security Hub - AWS Online Tech Talks"

* [Demo 3 Security Hub](https://youtu.be/HsWtPG_rTak?t=859) from "AWS re:Inforce 2019: AWS Security Hub: Manage Security Alerts & Automate Compliance (DEM15)"

* [Demo hands-on coding with Guarduty and Security Hub](https://youtu.be/nyh4imv8zuk) - "Remediating Amazon GuardDuty and AWS Security Hub Findings - AWS Online Tech Talks"

* [Demo Amazon Detective](https://youtu.be/MPQe-4NvesM?t=1477) from "AWS re:Invent 2019: [NEW LAUNCH!] Introducing Amazon Detective (SEC312)"

---

## Links to resources

* <https://aws.amazon.com/organizations/getting-started/best-practices/>

* <https://aws.amazon.com/blogs/security/top-10-security-items-to-improve-in-your-aws-account/>

* <https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_example-scps.html>
    * AWS Organizations SCPs

* <https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/compliance-validation.html>

* <https://docs.aws.amazon.com/secretsmanager/latest/userguide/best-practices.html>

* <https://aws.amazon.com/security/partner-solutions/>

* <https://aws.amazon.com/products/security/>

* <https://d0.awsstatic.com/whitepapers/compliance/AWS_CIS_Foundations_Benchmark.pdf>

* <https://aws.amazon.com/whitepapers/?whitepapers-main.sort-by=item.additionalFields.sortDate&whitepapers-main.sort-order=desc#security>

Examples for config rules and SCP

<https://docs.aws.amazon.com/controltower/latest/userguide/guardrails-reference.html>

Security Hub workshop

* <https://github.com/aws-samples/aws-security-hub-workshop>

* Security Workshops series

<https://awssecworkshops.com>


Security Logging, monitoring.

<https://aws.amazon.com/guardduty/>

<https://aws.amazon.com/security-hub/>

Videos

* AWS re:Invent 2017: The AWS Philosophy of Security (SID322) - <https://youtu.be/KJiCfPXOW-U>
* AWS re:Invent 2018: The Tension Between Absolutes & Ambiguity in Security (SEC310) - <https://youtu.be/GXTvlQXVCOs>
* AWS re:Inforce 2019: Leadership Session: Aspirational Security (SEP318-L) - <https://youtu.be/ad9180b4Xew>
* AWS re:Invent 2018: Leadership Session: AWS Security (SEC305-L) - <https://youtu.be/MdlFuFhetug>



<https://aws.amazon.com/blogs/publicsector/how-to-think-about-zero-trust-architectures-on-aws/>

<https://github.com/aws-samples/aws-refarch-wordpress>






---

## Thank you

Done by Antonio Feijao UK

* Linkedin - [AntonioFeijaoUK](https://www.linkedin.com/in/antoniofeijaouk/)
* Twitter - [@AntonioFeijaoUK](https://twitter.com/AntonioFeijaoUK)
* Personal Webpage - [AntonioCloud.Com](https://www.antoniocloud.com)
