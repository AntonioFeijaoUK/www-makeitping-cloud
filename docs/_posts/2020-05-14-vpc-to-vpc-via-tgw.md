---
layout: post
title: Ping from an EC2 instance on private subnet and on different VPC via TGW
author: Antonio Feijao UK
toc: true
last_modified_at: 2020-05-19
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

{% avatar AntonioFeijaoUK %}


## Table of contents

* auto-gen TOC:
{:toc}


---

## Useful links

* [sli.io](https://sli.do/)

* [whiteboard](https://awwapp.com/b/u6hxhr9cvgwgw/)

* Feedback(*feedback is now closed, thank you*)


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

* Leave default - `Amazon's pool of IPv4 addresses`, click `Allocate`

* image ![vpc-allocate-elastic-ip](/assets/images/vpc-allocate-elastic-ip.png)

* image ![vpc-allocate-elastic-ip-address-from-amazon-pool](/assets/images/vpc-allocate-elastic-ip-address-from-amazon-pool.png)

---

### Creating the VPC-10-0

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

* VPC Name: `VPC-10-0`

* Public subnet's IPv4 CIDR: `10.0.1.0/24`

* Availability Zone: `us-east-1a`

* Public subnet name: `Public subnet-10-0-1`

* Private subnet's IPv4 CIDR: `10.0.4.0/24`

* Availability Zone: `us-east-1a`

* Private subnet name: `Private subnet-10-0-4`

* Allocate the Elastic IP created previously

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

You should now have 2 new VPCs - `VPC-10-0` and `VPC-192-168`

Take 2 minutes to review your new subnets and route tables.

Tip - Use `Filter by VPC`

* Review `Subnets`

  * Do you know the difference between `Availability Zone` and `Availability Zone ID`?

* Review `Route Tables`

  * Give a name to the route tables - click on tab `Routes` tabs
  
  * Check for Destination `0.0.0.0/0`, Target `ine-xx (or nat-xxx)` - This is your `Private-Route-Table`
  
  * Check for Destination `0.0.0.0/0`, Target `igw-xx` - This is your `Public-Route-Table`
  
  * Notice the `Main` = `Yes` on route table and the message:  *"The following subnets have not been explicitly associated with any route tables and are therefore associated with the main route table"

* Review `Internet Gateways`

* Review `NAT Gateways`

  * Notice that VPC-192-168 as no `NAT Gateways`, instead we used the `nat instance` which is an EC2 managed by you.
  
  * Link on AWS [Comparison of NAT instances and NAT gateways](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-comparison.html)


---

## Creating the TGW

* Still in `VPC service`

* Scroll down to `Transit Gateway`

* Click `Create Transit Gateway`

* Leave all default, click `Create Transit Gateway`

* Add a name - `TGW-networking-101`

* That is all for now.


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

## Access your instance through AWS Systems Manager

After a few minutes (can take up to 10 minutes), you should see your instance on service `AWS Systems Manager` section `Managed Instances`

* Change to service `AWS Systems Manager`

* Scroll down on the left until `Managed Instances`

* Select one of the available instance, for example `instance-10-0-4-first`

* Click `Actions` top-right hand side, select `Start session`

You are now connected to the console of the instance in the private subnet.

> Did you notice we did not have to manage `ssh key` to access this instance command line


---

## There is no bug instance-192-168-4-first

There is no bug!

If you follow the steps above, the instance `instance-192-168-4-first` will NOT show up on `AWS Systems Manager` section `Managed Instances`.

The instance will also not install the `user data` script.

Can you figure out why?


---

## Testing using ping and other command line tools

While connected to the instance...

* Can you reach the internet? - `ping aws.amazon.com`

* What is your private ip address? - `ifconfig` or `ip address`

* What is your public ip address? - `curl ipinfo.io/json` or `curl ifconfig.io`

* What route are in this instance? - `route -n` or `ip route`

* Is the `httpd` service running? - `curl localhost`

  * If you did not install the `httpd` package, you run run a simple webserver with `python3`

```bash
## installs python3
sudo yum install -y python3

## creates index.html files with message "Hello from instance XXX"
cd /tmp
echo "Hello from instance XXX" > index.html

## start the python3 module http.server
sudo python3 -m http.server 80

```
   
  * `Control+C` stops the command from running
  
  * Port 80 already in use? No problem, you can change it to another port, remember if you need to adjust the security groups.

  * To leave the command running in the background, use the `&` at the end of the command - `sudo python3 -m http.server 8080 &` and then test with `curl localhost:8080`

* What services or IPs are connected to this instance? `sudo netstat -pant`

* Can we reach 192.168.0.0/16 subnet? - `traceroute -n 192.168.4.10`  - `Control+C` to stop the traceroute command


---

## Checkpoint current situation

* Current situation

* image ![ec2-vpc-tgw-vpc-ec2](/assets/images/ec2-vpc-tgw-vpc-ec2.png)

Great, we have 2 VPCs with 1 ec2 instance in each private subnet.

We also have 1 Transit Gateway, but the instances still cannot reach each other.

Next we will be looking at Transit Gateway Routes


---

## Transit Gateway Attachments and Route Tables

* Service `VPC`, scroll down to `Transit Gateway` sections

* Check your `Transit Gateway` that you created, did you git it a named? - `TGW-networking-101`

---

### Transit Gateway Attachment to VPC-10-0

* Click the `Create Transit Gateway Attachment` button to create your first Transit Gateway Attachment

* Transit Gateway ID - `TGW-networking-101`

* Attachment type - `VPC`

* Attachment name tag - `TGW-att-to-VPC-10-0`

* VPC ID - `VPC-10-0`

  * You can only chose one subnet per Availability Zone - choose `Private Subnet-10-0-4`
  
---

### Transit Gateway Attachment to VPC-192-168

Repeat for VPC-192-168

* Click the `Create Transit Gateway Attachment` 

* Transit Gateway ID - `TGW-networking-101`

* Attachment type - `VPC`

* Attachment name tag - `TGW-att-to-VPC-192-168`

* VPC ID - `VPC-192-168`

  * You can only chose one subnet per Availability Zone - choose `Private Subnet-192-168-4`

---

### Transit Gateway Route Tables

Now is time to check theTGW route tables

* Click on `Transit Gateway Route Table`

* Add a name - `TGW-default-route-table`
 
* Review the tabs:

  * `Associations` -  should have 2 VPCs
  
  * `Propagations` - should have 2 VPCs
  
  * `Routes` - should have the 2 subnets `10.0.0.0/16` and `192.168.0.0/16`

And that is it! All the work with `Transit Gateway` is done.

---

### Review Transit Gateway configurations

Note that when we create the `Transit Gateway`, we left all the default options, including

* Reminder of the TGW options

  * **Amazon side ASN** - *"Autonomous System Number (ASN) of your Transit Gateway. You can use an existing ASN assigned to your network. If you don't have one, you can use a private ASN in the 64512-65534 or 4200000000-4294967294 range."*

  * **DNS support** - *"Enable Domain Name System resolution for VPCs attached to this Transit Gateway."*
  
  * **VPN ECMP support** - *"Equal-cost multi-path routing for VPN Connections that are attached to this Transit Gateway."*

  * **Default route table association** - *"Automatically associate Transit Gateway attachments with this Transit Gateway's default route table."*
  
  * **Default route table propagation** - *"Automatically propagate Transit Gateway attachments with this Transit Gateway's default route table."*
  
  * **Multicast support** - *"Enables the ability to create multicast domains in this Transit Gateway."*
  
  * **Auto accept shared attachments** - "*Automatically accept cross account attachments that are attached to this Transit Gateway.*"

* AWS documentation [how-transit-gateways-work](https://docs.aws.amazon.com/vpc/latest/tgw/how-transit-gateways-work.html)

> "By default, this route table is the default association route table and the default propagation route table."

* image ![create-transit-gateway](/assets/iamges/create-transit-gateway.png)


---

## VPCs route tables

Now that the Transit Gateway knows how to reach both VPCs, we will update the `VPCs route tables` to use the `Transit Gateway Attachment` to reach the other VPC.

---

### VPC-10-0 update route table to use TGW to reach VPC-192-168

* Service `VPC`

* Filter by VPC - Select `VPC-10-0`

* Go to `Route Tables`

* Check the tab `Routes`

* Select your `Private-Route-Table`, is the route table that has the default route of 0.0.0.0 to the nat gateway 

* Click `Edit routes` add a new route Destination `192.168.0.0/16` - Target `tgw-xxxx`

---

### VPC-192-168 update route table to use TGW to reach VPC-10-0

* Change Filter by VPC - Select `VPC-192-168`

* Go to `Route Tables`

* Check the tab `Routes`

* Select your `Private-Route-Table`, is the route table that has the default route of 0.0.0.0 to the nat instance (eni-xxx) 

* Click `Edit routes` add a new route Destination `10.0.0.0/16` - Target `tgw-xxxx`

* image 1 ![route-table-vpc-192-168-step1](/assets/images/route-table-vpc-192-168-step1.png)

* image 2 ![route-table-vpc-192-168-step2](/assets/images/route-table-vpc-192-168-step2.png)

* image 3 ![route-table-vpc-192-168-step3](/assets/images/route-table-vpc-192-168-step3.png)

* Important to understand

  * `VPC-192-168` will forward packets with `Destination` to `10.0.0.0/16` via the `Transit Gateway Attachment`
  
  * Packet "arrives" at the `Transit Gateway Attachment` within the `Associated route table`, from here check where to forward the packet with `Destination` to `10.0.0.0/16`
  
  * Transit Gateway "knows" how to reach the subnet `10.0.0.0/16` via the attachment to the `VPC-10-0`
  
---


## Make it ping

* Now that all the route tables are updated, is time for testing

* Going back to `AWS Systems Manager`, section `Managed Instance`

* Select one of the instance, click `Actions`, `Start Session`

  * Did you figure why the `instance-192-168-4-first` was not showing on `Managed Instance`

* You can go back to the session above on [#testing-using-ping-and-other-command-line-tools](#testing-using-ping-and-other-command-line-tools)

* image ![aws-systems-manager-start-session](/assets/images/aws-systems-manager-start-session.png)


---

## Advanced challenges

* While connected to the instances, the command `sudo netstat -pant` shows  `ESTABLISHED` for the service `ssm-session-w`, can you change that to only use the subnet private ips? (tip: SSM Endpoint)

* image ![netstat-pant-ssm-public-ips.png](/assets/images/netstat-pant-ssm-public-ips.png)

* Check the throughput and bandwidth with `iperf` via TGW ?

* `iperf` via VPC Peering, can you change the routing to use VPC Peering? Is it the same if you do a VPC peering?

  * Detail about [VPC Peering](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html)


---

## Happy with the exercise remember to clear your account

Delete at your own risk! So be careful with what you delete :)


* Go to `Transit Gateway Route Table`

  * Remove Route tables associations - `Delete association`
  
* Delete `Transit Gateway attachment`

* Delete `Transit Gateway`

* Delete instances created with this exercise

* Delete `VPCs`


---


## The End



---

Sections below is WORK IN PROGRESS - Waiting for feedback

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

- [ ] testt area..
- [x] testt area..
- [ ] testt area..

:grinning:

:innocent:

:gb:


---

## Altervative creating VPCs with Cloud Formation

.... working in progress, any volunteer?

---

## AWS Resources

* [VPC subnets commands example](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-subnets-commands-example.html)

* [EC2 instance](https://aws.amazon.com/ec2/)

* [VPC with Private and Public Subnet](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario2.html)

* [Transit Gateway (TGW)](https://aws.amazon.com/transit-gateway/)

* Youtube videos
  * <https://youtu.be/awrdICiS6ug> - “Advanced Architectures with AWS Transit Gateway - AWS Online Tech Talks”

  * <https://youtu.be/9Nikqn_02Oc> - “AWS re:Invent 2019: [REPEAT 1] AWS Transit Gateway reference architectures for many VPCs (NET406-R1)”

