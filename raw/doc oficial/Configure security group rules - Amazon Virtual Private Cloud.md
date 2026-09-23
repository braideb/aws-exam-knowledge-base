---
title: "Configure security group rules - Amazon Virtual Private Cloud"
source: "https://docs.aws.amazon.com/vpc/latest/userguide/working-with-security-group-rules.html"
author:
published:
created: 2026-09-19
description: "Learn how to add, edit, and delete security group rules."
tags:
  - "clippings"
---
Configure security group rules - Amazon Virtual Private Cloud

After you create a security group, you can add, update, and delete its security group rules. When you add, update, or delete a rule, the change is automatically applied to the resources that are associated with the security group.

###### Required permissions

Before you begin, ensure that you have the required permissions. For more information, see [Manage security group rules](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-policy-examples.html#vpc-security-group-rules-iam).

###### Protocols and ports

- With the console, when you select a predefined type, **Protocol** and **Port range** are specified for you. To enter a port range, you must select one of the following custom types: **Custom TCP** or **Custom UDP**.
- With the AWS CLI, you can add a single rule with a single port using the `--protocol` and `--port` options. To add multiple rules, or a rule with a port range, use the `--ip-permissions` option instead.

###### Sources and destinations

- With the console, you can specify the following as sources for inbound rules or destinations for outbound rules:
	- **Custom** – An IPv4 CIDR block, an IPv6 CIDR block, a security group, or a prefix list.
	- **Anywhere-IPv4** – The 0.0.0.0/0 IPv4 CIDR block.
	- **Anywhere-IPv6** – The::/0 IPv6 CIDR block.
	- **My IP** – The public IPv4 address of your local computer.
- With the AWS CLI, you can specify an IPv4 CIDR block using the `--cidr` option or a security group using the `--source-group` option. To specify a prefix list or an IPv6 CIDR block, use the `--ip-permissions` option.

###### Warning

If you choose **Anywhere-IPv4**, you allow traffic from all IPv4 addresses. If you choose **Anywhere-IPv6**, you allow traffic from all IPv6 addresses. It is a best practice to authorize only the specific IP address ranges that need access to your resources.

###### To configure security group rules using the console

1. In the navigation pane, choose **Security groups**.
2. Select the security group.
3. To edit the inbound rules, choose **Edit inbound rules** from **Actions** or the **Inbound rules** tab.
	1. To add a rule, choose **Add rule** and enter the type, protocol, port, and source for the rule.
		If the type is TCP or UDP, you must enter the port range to allow. For custom ICMP, you must choose the ICMP type name from **Protocol**, and, if applicable, the code name from **Port range**. For any other type, the protocol and port range are configured for you.
		2. To update a rule, change its protocol, description, and source as needed. However, you can't change the source type. For example, if the source is an IPv4 CIDR block, you can't specify an IPv6 CIDR block, a prefix list, or a security group.
		3. To delete a rule, choose its **Delete** button.
4. To edit the outbound rules, choose **Edit outbound rules** from **Actions** or the **Outbound rules** tab.
	1. To add a rule, choose **Add rule** and enter the type, protocol, port, and destination for the rule. You can also enter an optional description.
		If the type is TCP or UDP, you must enter the port range to allow. For custom ICMP, you must choose the ICMP type name from **Protocol**, and, if applicable, the code name from **Port range**. For any other type, the protocol and port range are configured for you.
		2. To update a rule, change its protocol, description, and source as needed. However, you can't change the source type. For example, if the source is an IPv4 CIDR block, you can't specify an IPv6 CIDR block, a prefix list, or a security group.
		3. To delete a rule, choose its **Delete** button.
5. Choose **Save rules**.

###### To configure security group rules using the AWS CLI

- Add – Use the [authorize-security-group-ingress](https://docs.aws.amazon.com/cli/latest/reference/ec2/authorize-security-group-ingress.html) and [authorize-security-group-egress](https://docs.aws.amazon.com/cli/latest/reference/ec2/authorize-security-group-egress.html) commands.
- Remove – Use the [revoke-security-group-ingress](https://docs.aws.amazon.com/cli/latest/reference/ec2/revoke-security-group-ingress.html) and [revoke-security-group-egress](https://docs.aws.amazon.com/cli/latest/reference/ec2/revoke-security-group-egress.html) commands.
- Modify – Use the [modify-security-group-rules](https://docs.aws.amazon.com/cli/latest/reference/ec2/modify-security-group-rules.html), [update-security-group-rule-descriptions-ingress](https://docs.aws.amazon.com/cli/latest/reference/ec2/update-security-group-rule-descriptions-ingress.html), and [update-security-group-rule-descriptions-egress](https://docs.aws.amazon.com/cli/latest/reference/ec2/update-security-group-rule-descriptions-egress.html) commands.