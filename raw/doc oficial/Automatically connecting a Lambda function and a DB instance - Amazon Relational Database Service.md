---
title: "Automatically connecting a Lambda function and a DB instance - Amazon Relational Database Service"
source: "https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/lambda-rds-connect.html"
author:
published:
created: 2026-10-02
description: "Connect a Lambda function and a DB instance."
tags:
  - "clippings"
---
Automatically connecting a Lambda function and a DB instance - Amazon Relational Database Service

You can use the Amazon RDS console to simplify setting up a connection between a Lambda function and a DB instance. Often, your DB instance is in a private subnet within a VPC. The Lambda function can be used by applications to access your private DB instance.

For instructions on setting up a connection between a Lambda function and a Multi-AZ DB cluster, see [Automatically connecting a Lambda function and a Multi-AZ DB cluster](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/multiaz-lambda-rds-connect.html).

The following image shows a direct connection between your DB instance and your Lambda function.

![Automatically connect a DB instance with a Lambda function](https://docs.aws.amazon.com/images/AmazonRDS/latest/UserGuide/images/auto-connect-rds-lambda.png)

You can set up the connection between your Lambda function and your DB instance through RDS Proxy to improve your database performance and resiliency. Often, Lambda functions make frequent, short database connections that benefit from connection pooling that RDS Proxy offers. You can take advantage of any AWS Identity and Access Management (IAM) authentication that you already have for Lambda functions, instead of managing database credentials in your Lambda application code. For more information, see [Amazon RDS Proxy](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-proxy.html).

When you use the console to connect with an existing proxy, Amazon RDS updates the proxy security group to allow connections from your DB instance and Lambda function.

You can also create a new proxy from the same console page. When you create a proxy in the console, to access the DB instance, you must input your database credentials or select an AWS Secrets Manager secret.

![Automatically connect a DB instance with a Lambda function through RDS Proxy](https://docs.aws.amazon.com/images/AmazonRDS/latest/UserGuide/images/auto-connect-rds-lambda-Proxy.png)

###### Tip

To quickly connect a Lambda function to a DB instance, you can also use the in-console guided wizard. To open the wizard, do the following:

1. Open the [Functions page](https://console.aws.amazon.com/lambda/home#/functions) of the Lambda console.
2. Select the function you want to connect a database to.
3. On the **Configuration** tab, select **RDS databases**.
4. Choose **Connect to RDS database**.

After you've connected your function to a database, you can create a proxy by choosing **Add proxy**.

## Overview of automatic connectivity with a Lambda function

The following are requirements for connecting a Lambda function with an RDS DB instance:

- The Lambda function must exist in the same VPC as the DB instance.
- The user who sets up connectivity must have permissions to perform the following Amazon RDS, Amazon EC2, Lambda, Secrets Manager, and IAM operations:
	- Amazon RDS
		- `rds:CreateDBProxies`
		- `rds:DescribeDBInstances`
		- `rds:DescribeDBProxies`
		- `rds:ModifyDBInstance`
		- `rds:ModifyDBProxy`
		- `rds:RegisterProxyTargets`
	- Amazon EC2
		- `ec2:AuthorizeSecurityGroupEgress`
		- `ec2:AuthorizeSecurityGroupIngress`
		- `ec2:CreateSecurityGroup`
		- `ec2:DeleteSecurityGroup`
		- `ec2:DescribeSecurityGroups`
		- `ec2:RevokeSecurityGroupEgress`
		- `ec2:RevokeSecurityGroupIngress`
	- Lambda
		- `lambda:CreateFunctions`
		- `lambda:ListFunctions`
		- `lambda:UpdateFunctionConfiguration`
	- Secrets Manager
		- `secretsmanager:CreateSecret`
		- `secretsmanager:DescribeSecret`
	- IAM
		- `iam:AttachPolicy`
		- `iam:CreateRole`
		- `iam:CreatePolicy`
	- AWS KMS
		- `kms:describeKey`

###### Note

If the DB instance and Lambda function are in different Availability Zones, your account might incur cross-Availability Zone costs.

When you set up a connection between a Lambda function and an RDS database, Amazon RDS configures the VPC security group for your function and for your DB instance. If you use RDS Proxy, then Amazon RDS also configures the VPC security group for the proxy. Amazon RDS acts according to the current configuration of the security groups associated with the DB instance, Lambda function, and proxy, as described in the following table.

| Current RDS security group configuration | Current Lambda security group configuration | Current proxy security group configuration | RDS action |
| --- | --- | --- | --- |
| There are one or more security groups associated with the DB instance with a name that matches the pattern `` rds-lambda-`n` `` or if a proxy is already connected to your DB instance, RDS checks if the `TargetHealth` of an associated proxy is `AVAILABLE`.  A security group that matches the pattern hasn't been modified. This security group has only one inbound rule with the VPC security group of the Lambda function or proxy as the source. | There are one or more security groups associated with the Lambda function with a name that matches the pattern `` lambda-rds-`n` `` or `` lambda-rdsproxy-`n` `` (where `` `n` `` is a number).  A security group that matches the pattern hasn't been modified. This security group has only one outbound rule with either the VPC security group of the DB instance or the proxy as the destination. | There are one or more security groups associated with the proxy with a name that matches the pattern `` rdsproxy-lambda-`n` `` (where `` `n` `` is a number).  A security group that matches the pattern hasn't been modified. This security group has inbound and outbound rules with the VPC security groups of the Lambda function and the DB instance. | Amazon RDS takes no action.  A connection was already configured automatically between the Lambda function, the proxy (optional), and DB instance. Because a connection already exists between the function, proxy, and the database, the security groups aren't modified. |
| Either of the following conditions apply:  - There is no security group associated with the DB instance with a name that matches the pattern `` rds-lambda-`n` `` or if the `TargetHealth` of an associated proxy is `AVAILABLE`. - There are one or more security groups associated with the DB instance with a name that matches the pattern `` rds-lambda-`n` `` or if the `TargetHealth` of an associated proxy is `AVAILABLE`. However, none of these security groups can be used for the connection with the Lambda function.  Amazon RDS can't use a security group that doesn't have one inbound rule with the VPC security group of the Lambda function or proxy as the source. Amazon RDS also can't use a security group that has been modified. Examples of modifications include adding a rule or changing the port of an existing rule. | Either of the following conditions apply:  - There is no security group associated with the Lambda function with a name that matches the pattern `` lambda-rds-`n` `` or `` lambda-rdsproxy-`n` ``. - There are one or more security groups associated with the Lambda function with a name that matches the pattern `` lambda-rds-`n` `` or `` lambda-rdsproxy-`n` ``. However, Amazon RDS can't use any of these security groups for the connection with the DB instance.  Amazon RDS can't use a security group that doesn't have one outbound rule with the VPC security group of the DB instance or proxy as the destination. Amazon RDS also can't use a security group that has been modified. | Either of the following conditions apply:  - There is no security group associated with the proxy with a name that matches the pattern `` rdsproxy-lambda-`n` ``. - There are one or more security groups associated with the proxy with a name that matches `` rdsproxy-lambda-`n` ``. However, Amazon RDS can't use any of these security groups for the connection with the DB instance or Lambda function.  Amazon RDS can't use a security group that doesn't have inbound and outbound rules with the VPC security group of the DB instance and the Lambda function. Amazon RDS also can't use a security group that has been modified. | [RDS action: create new security groups](#rds-lam-action-create-new-security-groups) |
| There are one or more security groups associated with the DB instance with a name that matches the pattern `` rds-lambda-`n` `` or if the `TargetHealth` of an associated proxy is `AVAILABLE`.  A security group that matches the pattern hasn't been modified. This security group has only one inbound rule with the VPC security group of the Lambda function or proxy as the source. | There are one or more security groups associated with the Lambda function with a name that matches the pattern `` lambda-rds-`n` `` or `` lambda-rdsproxy-`n` ``.  However, Amazon RDS can't use any of these security groups for the connection with the DB instance. Amazon RDS can't use a security group that doesn't have one outbound rule with the VPC security group of the DB instance or proxy as the destination. Amazon RDS also can't use a security group that has been modified. | There are one or more security groups associated with the proxy with a name that matches the pattern `` rdsproxy-lambda-`n` ``.  However, Amazon RDS can't use any of these security groups for the connection with the DB instance or Lambda function. Amazon RDS can't use a security group that doesn't have inbound and outbound rules with the VPC security group of the DB instance and the Lambda function. Amazon RDS also can't use a security group that has been modified. | [RDS action: create new security groups](#rds-lam-action-create-new-security-groups) |
| There are one or more security groups associated with the DB instance with a name that matches the pattern `` rds-lambda-`n` `` or if the `TargetHealth` of an associated proxy is `AVAILABLE`.  A security group that matches the pattern hasn't been modified. This security group has only one inbound rule with the VPC security group of the Lambda function or proxy as the source. | A valid Lambda security group for the connection exists, but it isn't associated with the Lambda function. This security group has a name that matches the pattern `` lambda-rds-`n` `` or `` lambda-rdsproxy-`n` ``. It hasn't been modified. It has only one outbound rule with the VPC security group of the DB instance or proxy as the destination. | A valid proxy security group for the connection exists, but it isn't associated with the proxy. This security group has a name that matches the pattern `` rdsproxy-lambda-`n` ``. It hasn't been modified. It has inbound and outbound rules with the VPC security group of the DB instance and the Lambda function. | [RDS action: associate Lambda security group](#rds-lam-action-associate-lam-security-group) |
| Either of the following conditions apply:  - There is no security group associated with the DB instance with a name that matches the pattern `` rds-lambda-`n` `` or if the `TargetHealth` of an associated proxy is `AVAILABLE`. - There are one or more security groups associated with the DB instance with a name that matches the pattern `` rds-lambda-`n` `` or if the `TargetHealth` of an associated proxy is `AVAILABLE`. However, Amazon RDS can't use any of these security groups for the connection with the Lambda function or proxy.  Amazon RDS can't use a security group that doesn't have one inbound rule with the VPC security group of the Lambda function or proxy as the source. Amazon RDS also can't use a security group that has been modified. | There are one or more security groups associated with the Lambda function with a name that matches the pattern `` lambda-rds-`n` `` or `` lambda-rdsproxy-`n` ``.  A security group that matches the pattern hasn't been modified. This security group has only one outbound rule with the VPC security group of the DB instance or proxy as the destination. | There are one or more security groups associated with the proxy with a name that matches the pattern `` rdsproxy-lambda-`n` ``.  A security group that matches the pattern hasn't been modified. This security group has inbound and outbound rules with the VPC security group of the DB instance and the Lambda function. | [RDS action: create new security groups](#rds-lam-action-create-new-security-groups) |
| Either of the following conditions apply:  - There is no security group associated with the DB instance with a name that matches the pattern `` rds-lambda-`n` `` or if the `TargetHealth` of an associated proxy is `AVAILABLE`. - There are one or more security groups associated with the DB instance with a name that matches the pattern `` rds-lambda-`n` `` or if the `TargetHealth` of an associated proxy is `AVAILABLE`. However, Amazon RDS can't use any of these security groups for the connection with the Lambda function or proxy.  Amazon RDS can't use a security group that doesn't have one inbound rule with the VPC security group of the Lambda function or proxy as the source. Amazon RDS also can't use a security group that has been modified. | Either of the following conditions apply:  - There is no security group associated with the Lambda function with a name that matches the pattern `` lambda-rds-`n` `` or `` lambda-rdsproxy-`n` ``. - There are one or more security groups associated with the Lambda function with a name that matches the pattern `` lambda-rds-`n` `` or `` lambda-rdsproxy-`n` ``. However, Amazon RDS can't use any of these security groups for the connection with the DB instance.  Amazon RDS can't use a security group that doesn't have one outbound rule with the VPC security group of the DB instance or proxy as the source. Amazon RDS also can't use a security group that has been modified. | Either of the following conditions apply:  - There is no security group associated with the proxy with a name that matches the pattern `` rdsproxy-lambda-`n` ``. - There are one or more security groups associated with the proxy with a name that matches `` rdsproxy-lambda-`n` ``. However, Amazon RDS can't use any of these security groups for the connection with the DB instance or Lambda function.  Amazon RDS can't use a security group that doesn't have inbound and outbound rules with the VPC security group of the DB instance and the Lambda function. Amazon RDS also can't use a security group that has been modified. | [RDS action: create new security groups](#rds-lam-action-create-new-security-groups) |

###### RDS action: create new security groups

Amazon RDS takes the following actions:

- Creates a new security group that matches the pattern `` rds-lambda-`n` `` or `` rds-rdsproxy-`n` `` (if you choose to use RDS Proxy). This security group has an inbound rule with the VPC security group of the Lambda function or proxy as the source. This security group is associated with the DB instance and allows the function or proxy to access the DB instance.
- Creates a new security group that matches the pattern `` lambda-rds-`n` `` or `` lambda-rdsproxy-`n` ``. This security group has an outbound rule with the VPC security group of the DB instance or proxy as the destination. This security group is associated with the Lambda function and allows the function to send traffic to the DB instance or send traffic through a proxy.
- Creates a new security group that matches the pattern `` rdsproxy-lambda-`n` ``. This security group has inbound and outbound rules with the VPC security group of the DB instance and the Lambda function.

###### RDS action: associate Lambda security group

Amazon RDS associates the valid, existing Lambda security group with the Lambda function. This security group allows the function to send traffic to the DB instance or send traffic through a proxy.

## Automatically connecting a Lambda function and an RDS database

You can use the Amazon RDS console to automatically connect a Lambda function to your DB instance. This simplifies the process of setting up a connection between these resources.

You can also use RDS Proxy to include a proxy in your connection. Lambda functions make frequent short database connections that benefit from the connection pooling that RDS Proxy offers. You can also use any IAM authentication that you've already set up for your Lambda function, instead of managing database credentials in your Lambda application code.

You can connect an existing DB instance to new and existing Lambda functions using the **Set up Lambda connection** page. The setup process automatically sets up the required security groups for you.

Before setting up a connection between a Lambda function and a DB instance, make sure that:

- Your Lambda function and DB instance are in the same VPC.
- You have the right permissions for your user account. For more information about the requirements, see [Overview of automatic connectivity with a Lambda function](#lambda-rds-connect-overview).

If you change security groups after you configure connectivity, the changes might affect the connection between the Lambda function and the DB instance.

###### Note

You can automatically set up a connection between a DB instance and a Lambda function only in the AWS Management Console. To connect a Lambda function, the DB instance must be in the **Available** state.

###### To automatically connect a Lambda function and a DB instance

1. Sign in to the AWS Management Console and open the Amazon RDS console at [https://console.aws.amazon.com/rds/](https://console.aws.amazon.com/rds/).
2. In the navigation pane, choose **Databases**, and then choose the DB instance that you want to connect to a Lambda function.
3. For **Actions**, choose **Set up Lambda connection**.
4. On the **Set up Lambda connection** page, under **Select Lambda function**, do either of the following:
	- If you have an existing Lambda function in the same VPC as your DB instance, choose **Choose existing function**, and then choose the function.
		- If you don't have a Lambda function in the same VPC, choose **Create new function**, and then enter a **Function name**. The default runtime is set to Nodejs.18. You can modify the settings for your new Lambda function in the Lambda console after you complete the connection setup.
5. (Optional) Under **RDS Proxy**, select **Connect using RDS Proxy**, and then do any of the following:
	- If you have an existing proxy that you want to use, choose **Choose existing proxy**, and then choose the proxy.
		- If you don't have a proxy, and you want Amazon RDS to automatically create one for you, choose **Create new proxy**. Then, for **Database credentials**, do either of the following:
		1. Choose **Database username and password**, and then enter the **Username** and **Password** for your DB instance.
				2. Choose **Secrets Manager secret**. Then, for **Select secret**, choose an AWS Secrets Manager secret. If you don't have a Secrets Manager secret, choose **Create new Secrets Manager secret** to [create a new secret](https://docs.aws.amazon.com/secretsmanager/latest/userguide/create_secret.html). After you create the secret, for **Select secret**, choose the new secret.
		After you create the new proxy, choose **Choose existing proxy**, and then choose the proxy. Note that it might take some time for your proxy to be available for connection.
6. (Optional) Expand **Connection summary** and verify the highlighted updates for your resources.
7. Choose **Set up**.

## Viewing connected compute resources

You can use the AWS Management Console to view the Lambda functions that are connected to your DB instance. The resources shown include compute resource connections that Amazon RDS set up automatically.

The listed compute resources don't include those that are manually connected to the DB instance. For example, you can allow a compute resource to access your DB instance manually by adding a rule to your VPC security group associated with the database.

For the console to list a Lambda function, the following conditions must apply:

- The name of the security group associated with the compute resource matches the pattern `` lambda-rds-`n` `` or `` lambda-rdsproxy-`n` `` (where `` `n` `` is a number).
- The security group associated with the compute resource has an outbound rule with the port range set to the port of the DB instance or an associated proxy. The destination for the outbound rule must be set to a security group associated with the DB instance or an associated proxy.
- If the configuration includes a proxy, the name of the security group attached to the proxy associated with your database matches the pattern `` rdsproxy-lambda-`n` `` (where `` `n` `` is a number).
- The security group associated with the function has an outbound rule with the port set to the port that the DB instance or associated proxy uses. The destination must be set to a security group associated with the DB instance or associated proxy.

###### To view compute resources automatically connected to an DB instance

1. Sign in to the AWS Management Console and open the Amazon RDS console at [https://console.aws.amazon.com/rds/](https://console.aws.amazon.com/rds/).
2. In the navigation pane, choose **Databases**, and then choose the DB instance.
3. On the **Connectivity & security** tab, view the compute resources under **Connected compute resources**.