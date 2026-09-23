---
title: "Delete a security group - Amazon Virtual Private Cloud"
source: "https://docs.aws.amazon.com/vpc/latest/userguide/deleting-security-groups.html"
author:
published:
created: 2026-09-19
description: "Learn how to delete a security group for your VPC."
tags:
  - "clippings"
---
Delete a security group - Amazon Virtual Private Cloud

When you are finished with a security group that you created, you can delete it.

###### Requirements

- The security group can't be associated with any resources.
- The security group can't be referenced by a rule in another security group.
- The security group can't be the default security group for a VPC.

###### To delete a security group using the console

1. In the navigation pane, choose **Security groups**.
2. Select the security group and choose **Actions**, **Delete security groups**.
3. If you selected more than one security group, you are prompted for confirmation. If some of the security groups can't be deleted, we display the status of each security group, which indicates whether it will be deleted. To confirm deletion, enter **Delete**.
4. Choose **Delete**.

###### To delete a security group using the AWS CLI

Use the [delete-security-group](https://docs.aws.amazon.com/cli/latest/reference/ec2/delete-security-group.html) command.