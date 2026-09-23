---
title: "Add internet access to a subnet - Amazon Virtual Private Cloud"
source: "https://docs.aws.amazon.com/vpc/latest/userguide/working-with-igw.html"
author:
published:
created: 2026-09-19
description: "Learn how to create an internet gateway and configure internet access for a subnet."
tags:
  - "clippings"
---
Add internet access to a subnet - Amazon Virtual Private Cloud

The following describes how to support internet access from a subnet in a nondefault VPC using an internet gateway. You must create the internet gateway, attach it to the VPC, and configure routing for the subnet.

After you configure internet access for your subnet, you must ensure that resources in the subnet can access the internet. For example, your EC2 instances must have a public IPv4 or IPv6 address, and the security groups for your instances must allow specific traffic to and from the internet.

Alternatively, to provide your instances with internet access without assigning them a public IP address, use a NAT device instead. For more information, see [NAT devices](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat.html).

To remove internet access, you can detach the internet gateway from your VPC and then delete it. For more information, see [Delete an internet gateway](https://docs.aws.amazon.com/vpc/latest/userguide/delete-igw.html).

## Step 1: Create an internet gateway

Use the following procedure to create an internet gateway.

###### To create an internet gateway using the console

1. In the navigation pane, choose **Internet gateways**.
2. Choose **Create internet gateway**.
3. (Optional) Enter a name for your internet gateway.
4. (Optional) To add a tag, choose **Add new tag** and enter the tag key and value.
5. Choose **Create internet gateway**.
6. (Optional) To attach the internet gateway to a VPC now, choose **Attach to a VPC** from the banner at the top of the screen, select an available VPC, and then choose **Attach internet gateway**. Otherwise, you can attach your internet gateway to a VPC at another time.

###### To create an internet gateway using the command line

- [create-internet-gateway](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-internet-gateway.html) (AWS CLI)
- [New-EC2InternetGateway](https://docs.aws.amazon.com/powershell/latest/reference/items/New-EC2InternetGateway.html) (AWS Tools for Windows PowerShell)

## Step 2: Attach the internet gateway to the VPC

To use an internet gateway, you must attach it to a VPC.

###### To attach an internet gateway to a VPC using the console

1. In the navigation pane, choose **Internet gateways**.
2. Select the check box for the internet gateway.
3. To attach it, choose **Actions**, Attach to VPC, select an available VPC, and choose **Attach internet gateway**.
4. To detach it, choose **Actions**, **Detach from VPC** and choose **Detach internet gateway**. When prompted for confirmation, choose **Detach internet gateway**.

###### To attach an internet gateway to a VPC using the command line

- [attach-internet-gateway](https://docs.aws.amazon.com/cli/latest/reference/ec2/attach-internet-gateway.html) (AWS CLI)
- [Add-EC2InternetGateway](https://docs.aws.amazon.com/powershell/latest/reference/items/Add-EC2InternetGateway.html) (AWS Tools for Windows PowerShell)

## Step 3: Add a route to the subnet route table

The route table for the subnet must have a route that sends internet traffic to the internet gateway.

###### To configure the subnet route table using the console

1. In the navigation pane, choose **Route tables**.
2. Select the route table for the subnet. By default, a subnet uses the main route table for the VPC. Alternatively, you can [create a route table](https://docs.aws.amazon.com/vpc/latest/userguide/create-vpc-route-table.html#CustomRouteTable) and then [associate the subnet with the new route table](https://docs.aws.amazon.com/vpc/latest/userguide/create-vpc-route-table.html#AssociateSubnet).
3. On the **Routes** tab, choose **Edit routes** and then choose **Add route**.
4. Enter 0.0.0.0/0 for **Destination** and select the internet gateway for **Target**.
5. Choose **Save changes**.

###### To configure the subnet route table using the command line

- [create-route](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-route.html) (AWS CLI)
- [New-EC2Route](https://docs.aws.amazon.com/powershell/latest/reference/items/New-EC2Route.html) (AWS Tools for Windows PowerShell)