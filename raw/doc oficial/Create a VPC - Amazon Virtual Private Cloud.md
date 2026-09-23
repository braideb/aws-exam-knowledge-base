---
title: "Create a VPC - Amazon Virtual Private Cloud"
source: "https://docs.aws.amazon.com/vpc/latest/userguide/create-vpc.html"
author:
published:
created: 2026-09-19
description: "Create a VPC - configure options like IP CIDR blocks, subnets, route tables, and gateways. Procedures cover VPC creation through the console or CLI."
tags:
  - "clippings"
---
Create a VPC - Amazon Virtual Private Cloud

Use the following procedures to create a virtual private cloud (VPC). A VPC must have additional resources, such as subnets, route tables, and gateways, before you can create AWS resources in the VPC.

For information about modifying a VPC, see [Add or remove a CIDR block from your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/add-ipv4-cidr.html).

## Create a VPC plus other VPC resources

Use the following procedure to create a VPC plus the additional VPC resources that you need to run your application, such as subnets, route tables, internet gateways, and NAT gateways. For example VPC configurations, see [VPC examples](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-examples-intro.html).

###### To create a VPC, subnets, and other VPC resources using the console

1. On the VPC dashboard, choose **Create VPC**.
2. For **Resources to create**, choose **VPC and more**.
3. Keep **Name tag auto-generation** selected to create Name tags for the VPC resources or clear it to provide your own Name tags for the VPC resources.
4. For **IPv4 CIDR block**, enter an IPv4 address range for the VPC. A VPC must have an IPv4 address range.
5. (Optional) To support IPv6 traffic, choose **IPv6 CIDR block**, **Amazon-provided IPv6 CIDR block**.
6. Choose a **Tenancy** option. This option defines if EC2 instances that you launch into the VPC will run on hardware that's shared with other AWS accounts or on hardware that's dedicated for your use only. If you choose the tenancy of the VPC to be `Default`, EC2 instances launched into this VPC will use the tenancy attribute specified when you launch the instance. For more information, see [Launch an instance using defined parameters](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-instance-wizard.html) in the *Amazon EC2 User Guide*. If you choose the tenancy of the VPC to be `Dedicated`, the instances will always run as [Dedicated Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/dedicated-instance.html) on hardware that's dedicated for your use. If you're using AWS Outposts, your Outpost requires private connectivity; you must use `Default` tenancy.
7. For **Number of Availability Zones (AZs)**, we recommend that you provision subnets in at least two Availability Zones for a production environment. To choose the AZs for your subnets, expand **Customize AZs**. Otherwise, let AWS choose them for you.
8. To configure your subnets, choose values for **Number of public subnets** and **Number of private subnets**. To choose the IP address ranges for your subnets, expand **Customize subnets CIDR blocks**. Otherwise, let AWS choose them for you.
9. (Optional) If resources in a private subnet need access to the public internet over IPv4, for **NAT gateways**, choose the number of AZs in which to create NAT gateways. In production, we recommend that you deploy a NAT gateway in each AZ with resources that need access to the public internet. Note that there is a cost associated with NAT gateways. For more information, see [Pricing for NAT gateways](https://docs.aws.amazon.com/vpc/latest/userguide/nat-gateway-pricing.html).
10. (Optional) If resources in a private subnet need access to the public internet over IPv6, for **Egress only internet gateway**, choose **Yes**.
11. (Optional) If you need to access Amazon S3 directly from your VPC, choose **VPC endpoints**, **S3 Gateway**. This creates a gateway VPC endpoint for Amazon S3. For more information, see [Gateway endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/gateway-endpoints.html) in the *AWS PrivateLink Guide*.
12. (Optional) For **DNS options**, both options for domain name resolution are enabled by default. If the default doesn't meet your needs, you can disable these options.
13. (Optional) To add a tag to your VPC, expand **Additional tags**, choose **Add new tag**, and enter a tag key and a tag value.
14. In the **Preview** pane, you can visualize the relationships between the VPC resources that you've configured. Solid lines represent relationships between resources. Dotted lines represent network traffic to NAT gateways, internet gateways, and gateway endpoints. After you create the VPC, you can visualize the resources in your VPC in this format at any time using the **Resource map** tab. For more information, see [Visualize the resources in your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/view-vpc-resource-map.html).
15. When you are finished configuring your VPC, choose **Create VPC**.

## Create a VPC only

Use the following procedure to create a VPC with no additional VPC resources using the Amazon VPC console.

###### To create a VPC with no additional VPC resources using the console

1. On the VPC dashboard, choose **Create VPC**.
2. For **Resources to create**, choose **VPC only**.
3. (Optional) For **Name tag**, enter a name for your VPC. Doing so creates a tag with a key of `Name` and the value that you specify.
4. For **IPv4 CIDR block**, do one of the following:
	- Choose **IPv4 CIDR manual input** and enter an IPv4 address range for your VPC.
		- Choose **IPAM-allocated IPv4 CIDR block**, select your Amazon VPC IP Address Manager (IPAM) IPv4 address pool and a netmask. The size of the CIDR block is limited by the allocation rules on the IPAM pool. IPAM is a VPC feature that makes it easier for you to plan, track, and monitor IP addresses for your AWS workloads. For more information, see the [Amazon VPC IPAM User Guide](https://docs.aws.amazon.com/vpc/latest/ipam/what-it-is-ipam.html).
		If you are using IPAM to manage your IP addresses, we recommend that you choose this option. Otherwise, the CIDR block that you specify for your VPC might overlap with an IPAM CIDR allocation.
5. (Optional) To create a dual stack VPC, specify an IPv6 address range for your VPC. For **IPv6 CIDR block**, do one of the following:
	- Choose **IPAM-allocated IPv6 CIDR block** if you are using Amazon VPC IP Address Manager and you want to provision a IPv6 CIDR from an IPAM pool. If you use the IPAM-allocated IPv6 CIDR block to provision IPv6 CIDRs to VPCs, you get the benefit of contiguous IPv6 CIDRs for VPC creation. Contiguously-allocated CIDRs are CIDRs that are allocated sequentially. They enable you to simplify your security and networking rules; the IPv6 CIDRs can be aggregated in a single entry across networking and security constructs like access control lists, route tables, security groups, and firewalls.
		You have two options for provisioning an IP address range to the VPC under **CIDR block**:
		- **Netmask length**: Choose this option to select a netmask length for the CIDR. Do one of the following:
			- If there is a default netmask length selected for the IPAM pool, you can choose **Default to IPAM netmask length** to use the default netmask length set for the IPAM pool by the IPAM administrator. For more information about the optional default netmask length allocation rule, see [Create a Regional IPv6 pool](https://docs.aws.amazon.com/vpc/latest/ipam/create-ipv6-reg-pool.html) in the *Amazon VPC IPAM User Guide*.
			- If there is no default netmask length selected for the IPAM pool, choose a netmask length that's more specific than the netmask length of the IPAM pool CIDR. For example, if the IPAM pool CIDR is /50, you can choose a netmask length between **/52** to **/60** for the VPC. Possible netmask lengths are between **/44** and **/60** in increments of /4.
		- **Select a CIDR**: Choose this option to manually enter an IPv6 address. You can only choose a netmask length that's more specific than the netmask length of the IPAM pool CIDR. For example, if the IPAM pool CIDR is /50, you can choose a netmask length between **/52** to **/60** for the VPC. Possible IPv6 netmask lengths are between **/44** and **/60** in increments of /4.
	- Choose **Amazon-provided IPv6 CIDR block** to request an IPv6 CIDR block from an Amazon pool of IPv6 addresses. For **Network Border Group**, select the group from which AWS advertises IP addresses. Amazon provides a fixed IPv6 CIDR block size of **/56**.
	- Choose **IPv6 CIDR owned by me** to provision an IPv6 CIDR that you have already brought to AWS. For more information about bringing your own IP address ranges to AWS, see [Bring your own IP addresses (BYOIP)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-byoip.html) in the *Amazon EC2 User Guide*. You can provision an IP address range for the VPC using the following options for **CIDR block**:
		- **No preference**: Choose this option to use netmask length of **/56**.
		- **Select a CIDR**: Choose this option to manually enter an IPv6 address and choose a netmask length that's more specific than the size of BYOIP CIDR. For example, if the BYOIP pool CIDR is /50, you can choose a netmask length between **/52** to **/60** for the VPC. Possible IPv6 netmask lengths are between **/44** and **/60** in increments of /4.
6. (Optional) Choose a **Tenancy** option. This option defines if EC2 instances that you launch into the VPC will run on hardware that's shared with other AWS accounts or on hardware that's dedicated for your use only. If you choose the tenancy of the VPC to be `Default`, EC2 instances launched into this VPC will use the tenancy attribute specified when you launch the instance -- For more information, see [Launch an instance using defined parameters](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-instance-wizard.html) in the *Amazon EC2 User Guide*. If you choose the tenancy of the VPC to be `Dedicated`, the instances will always run as [Dedicated Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/dedicated-instance.html) on hardware that's dedicated for your use. If you're using AWS Outposts, your Outpost requires private connectivity; you must use `Default` tenancy.
7. (Optional) To add a tag to your VPC, choose **Add new tag** and enter a tag key and a tag value.
8. Choose **Create VPC**.
9. After you create a VPC, you can add subnets. For more information, see [Create a subnet](https://docs.aws.amazon.com/vpc/latest/userguide/create-subnets.html).

## Create a VPC using the AWS CLI

The following procedure contains example AWS CLI commands to create a VPC plus the additional VPC resources needed to run an application. If you run all of the commands in this procedure, you'll create a VPC, a public subnet, a private subnet, a route table for each subnet, an internet gateway, an egress-only internet gateway, and a public NAT gateway. If you do not need all of these resources, you can use only the example commands that you need.

###### Prerequisites

Before you begin, install and configure the AWS CLI. When you configure the AWS CLI, you are prompted for AWS credentials. The examples in this procedure assume that you also configured a default Region. Otherwise, add the `--region` option to each command. For more information, see [Installing or updating the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) and [Configuring the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html).

###### Tagging

You can add tags to a resource after you create it by using the [create-tags](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-tags.html) command. Alternatively, you can add the `--tag-specification` option to the creation command for the resource as follows.

```
--tag-specifications ResourceType=vpc,Tags=[{Key=Name,Value=my-project}]
```

###### To create a VPC plus VPC resources by using the AWS CLI

1. Use the following [create-vpc](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-vpc.html) command to create a VPC with the specified IPv4 CIDR block.
	```
	aws ec2 create-vpc --cidr-block 10.0.0.0/24 --query Vpc.VpcId --output text
	```
	Alternatively, to create a dual stack VPC, add the `--amazon-provided-ipv6-cidr-block` option to add an Amazon-provided IPv6 CIDR block, as shown in the following example.
	```
	aws ec2 create-vpc --cidr-block 10.0.0.0/24 --amazon-provided-ipv6-cidr-block --query Vpc.VpcId --output text
	```
	These commands return the ID of the new VPC. The following is an example.
	```
	vpc-1a2b3c4d5e6f1a2b3
	```
2. \[Dual stack VPC\] Get the IPv6 CIDR block that's associated with your VPC by using the following [describe-vpcs](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-vpcs.html) command.
	```
	aws ec2 describe-vpcs --vpc-id vpc-1a2b3c4d5e6f1a2b3 --query Vpcs[].Ipv6CidrBlockAssociationSet[].Ipv6CidrBlock --output text
	```
	The following is example output.
	```
	2600:1f13:cfe:3600::/56
	```
3. Create one or more subnets, depending on your use case. In production, we recommend that you launch resources in at least two Availability Zones. Use one of the following commands to create each subnet.
	- IPv4-only subnet – To create a subnet with a specific IPv4 CIDR block, use the following [create-subnet](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-subnet.html) command.
		```
		aws ec2 create-subnet --vpc-id vpc-1a2b3c4d5e6f1a2b3 --cidr-block 10.0.1.0/20 --availability-zone us-east-2a --query Subnet.SubnetId --output text
		```
		- Dual stack subnet – If you created a dual stack VPC, you can use the `--ipv6-cidr-block` option to create a dual stack subnet, as shown in the following command.
		```
		aws ec2 create-subnet --vpc-id vpc-1a2b3c4d5e6f1a2b3 --cidr-block 10.0.1.0/20 --ipv6-cidr-block 2600:1f13:cfe:3600::/64 --availability-zone us-east-2a --query Subnet.SubnetId --output text
		```
		- IPv6-only subnet – If you created a dual stack VPC, you can use the `--ipv6-native` option to create an IPv6-only subnet, as shown in the following command.
		```
		aws ec2 create-subnet --vpc-id vpc-1a2b3c4d5e6f1a2b3 --ipv6-native --ipv6-cidr-block 2600:1f13:cfe:3600::/64 --availability-zone us-east-2a --query Subnet.SubnetId --output text
		```
	These commands return the ID of the new subnet. The following is an example.
	```
	subnet-1a2b3c4d5e6f1a2b3
	```
4. If you need a public subnet for your web servers, or for a NAT gateway, do the following:
	1. Create an internet gateway by using the following [create-internet-gateway](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-internet-gateway.html) command. The command returns the ID of the new internet gateway.
		```
		aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text
		```
		2. Attach the internet gateway to your VPC by using the following [attach-internet-gateway](https://docs.aws.amazon.com/cli/latest/reference/ec2/attach-internet-gateway.html) command. Use the internet gateway ID returned from the previous step.
		```
		aws ec2 attach-internet-gateway --vpc-id vpc-1a2b3c4d5e6f1a2b3 --internet-gateway-id igw-id
		```
		3. Create a custom route table for your public subnet by using the following [create-route-table](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-route-table.html) command. The command returns the ID of the new route table.
		```
		aws ec2 create-route-table --vpc-id vpc-1a2b3c4d5e6f1a2b3 --query RouteTable.RouteTableId --output text
		```
		4. Create a route in the route table that sends all IPv4 traffic to the internet gateway by using the following [create-route](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-route.html) command. Use the ID of the route table for the public subnet.
		```
		aws ec2 create-route --route-table-id rtb-id-public --destination-cidr-block 0.0.0.0/0 --gateway-id igw-id
		```
		5. Associate the route table with the public subnet by using the following [associate-route-table](https://docs.aws.amazon.com/cli/latest/reference/ec2/associate-route-table.html) command. Use the ID of the route table for the public subnet and the ID of the public subnet.
		```
		aws ec2 associate-route-table --route-table-id rtb-id-public --subnet-id subnet-id-public-subnet
		```
5. \[IPv6\] You can add an egress-only internet gateway so that instances in a private subnet can access the internet over IPv6 (for example, to get software updates), but hosts on the internet can't access your instances.
	1. Create an egress-only internet gateway by using the following [create-egress-only-internet-gateway](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-egress-only-internet-gateway.html) command. The command returns the ID of the new internet gateway.
		```
		aws ec2 create-egress-only-internet-gateway --vpc-id vpc-1a2b3c4d5e6f1a2b3 --query EgressOnlyInternetGateway.EgressOnlyInternetGatewayId --output text
		```
		2. Create a custom route table for your private subnet by using the following [create-route-table](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-route-table.html) command. The command returns the ID of the new route table.
		```
		aws ec2 create-route-table --vpc-id vpc-1a2b3c4d5e6f1a2b3 --query RouteTable.RouteTableId --output text
		```
		3. Create a route in the route table for the private subnet that sends all IPv6 traffic to the egress-only internet gateway by using the following [create-route](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-route.html) command. Use the ID of the route table returned in the previous step.
		```
		aws ec2 create-route --route-table-id rtb-id-private --destination-cidr-block ::/0 --egress-only-internet-gateway eigw-id
		```
		4. Associate the route table with the private subnet by using the following [associate-route-table](https://docs.aws.amazon.com/cli/latest/reference/ec2/associate-route-table.html) command.
		```
		aws ec2 associate-route-table --route-table-id rtb-id-private --subnet-id subnet-id-private-subnet
		```
6. If you need a NAT gateway for your resources in a private subnet, do the following:
	1. Create an elastic IP address for the NAT gateway by using the following [allocate-address](https://docs.aws.amazon.com/cli/latest/reference/ec2/allocate-address.html) command.
		```
		aws ec2 allocate-address --domain vpc --query AllocationId --output text
		```
		2. Create the NAT gateway in the public subnet by using the following [create-nat-gateway](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-nat-gateway.html) command. Use the allocation ID returned from the previous step.
		```
		aws ec2 create-nat-gateway --subnet-id subnet-id-public-subnet --allocation-id eipalloc-id
		```
		3. (Optional) If you already created a route table for the private subnet in step 5, skip this step. Otherwise, use the following [create-route-table](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-route-table.html) command to create a route table for your private subnet. The command returns the ID of the new route table.
		```
		aws ec2 create-route-table --vpc-id vpc-1a2b3c4d5e6f1a2b3 --query RouteTable.RouteTableId --output text
		```
		4. Create a route in the route table for the private subnet that sends all IPv4 traffic to the NAT gateway by using the following [create-route](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-route.html) command. Use the ID of the route table for the private subnet, which you created either in this step or in step 5.
		```
		aws ec2 create-route --route-table-id rtb-id-private --destination-cidr-block 0.0.0.0/0 --gateway-id nat-id
		```
		5. (Optional) If you already associated a route table with the private subnet in step 5, skip this step. Otherwise, use the following [associate-route-table](https://docs.aws.amazon.com/cli/latest/reference/ec2/associate-route-table.html) command to associate the route table with the private subnet. Use the ID of the route table for the private subnet, which you created either in this step or in step 5.
		```
		aws ec2 associate-route-table --route-table-id rtb-id-private --subnet-id subnet-id-private-subnet
		```