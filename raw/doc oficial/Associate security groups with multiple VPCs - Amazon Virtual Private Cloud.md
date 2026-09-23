---
title: "Associate security groups with multiple VPCs - Amazon Virtual Private Cloud"
source: "https://docs.aws.amazon.com/vpc/latest/userguide/security-group-assoc.html"
author:
published:
created: 2026-09-19
description: "Learn how to associate security group with multiple VPCs"
tags:
  - "clippings"
---
Associate security groups with multiple VPCs - Amazon Virtual Private Cloud

If you have workloads running in multiple VPCs that share network security requirements, you can use the Security Group VPC Associations feature to associate a security group with multiple VPCs in the same Region. This enables you to manage and maintain security groups in one place for multiple VPCs in your account.

![A diagram of security group associated with two VPCs.](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/sec-group-vpc-assoc.png)

The diagram above shows AWS account A with two VPCs in it. Each of the VPCs has workloads running in a private subnet. In this case, workloads in VPC A and B subnets share the same network traffic requirements, so Account A can use the Security Group VPC associations feature to associate the security group in VPC A with VPC B. Any updates made to the associated security group are automatically applied to the traffic to workloads in the VPC B subnet.

###### Requirements of the Security Group VPC Associations feature

- You must own the VPC or have one of the VPC subnets shared with you to associate a security group with the VPC.
- The VPC and security group must be in the same AWS Region.
- You cannot associate a default security group with another VPC or associate a security group with a default VPC.
- Both the security group owner and the VPC owner can view the security group VPC associations.

**Services that support this feature**

- Amazon API Gateway (REST APIs only)
- AWS Auto Scaling
- CloudFormation
- Amazon EC2
- Amazon EFS
- Amazon EKS
- Amazon FSx
- AWS PrivateLink
- Amazon Route 53
- Elastic Load Balancing
	- Application Load Balancer
	- Network Load Balancer

## Associate a security group with another VPC

This section explains how to use the AWS Management Console and the AWS CLI to associate a security group with VPCs.

anchor anchor

The VPC is now associated with the security group.

Once you've associated the VPC with the security group, you can, for example, [launch an instance into the VPC and choose this new security group](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/LaunchingAndUsingInstances.html) or [reference this security group in an existing security group rule](https://docs.aws.amazon.com/vpc/latest/userguide/security-group-rules.html#security-group-referencing).

## Disassociate a security group from another VPC

This section explains how to use the AWS Management Console and the AWS CLI to disassociate a security group from VPCs. You may want to do this if your goal is to delete the security group. Security groups cannot be deleted if they are associated. You can only diassociate a security group if there are no network interfaces in the associated VPC using that security group.

anchor anchor

The VPC is now disassociated with the security group.