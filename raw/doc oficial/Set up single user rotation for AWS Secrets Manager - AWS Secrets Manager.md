---
title: "Set up single user rotation for AWS Secrets Manager - AWS Secrets Manager"
source: "https://docs.aws.amazon.com/secretsmanager/latest/userguide/tutorials_rotation-single.html"
author:
published:
created: 2026-10-02
description: "Learn how to set up automatic rotation for database credentials."
tags:
  - "clippings"
---
Set up single user rotation for AWS Secrets Manager - AWS Secrets Manager

In this tutorial, you learn how to set up single user rotation for a secret that contains database credentials. *Single user rotation* is a rotation strategy where Secrets Manager updates a user's credentials in both the secret and the database. For more information, see [Rotation strategy: single user](https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotation-strategy.html#rotating-secrets-one-user-one-password).

After you finish the tutorial, we recommend that you clean up the resources from the tutorial. Don't use them in a production setting.

Secrets Manager rotation uses an AWS Lambda function to update the secret and the database. For information about the costs of using a Lambda function, see [Pricing](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html#asm_pricing).

## Permissions

For the tutorial prerequisites, you need administrative permissions to your AWS account. In a production setting, it is a best practice to use different roles for each of the steps. For example, a role with database admin permissions would create the Amazon RDS database, and a role with network admin permissions would set up the VPC and security groups. For the tutorial steps, we recommend you continue using the same identity.

For information about how to set up permissions in a production environment, see [Authentication and access control for AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/auth-and-access.html).

## Prerequisites

The prerequisite for this tutorial is [Set up alternating users rotation for AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/tutorials_rotation-alternating.html). Don't clean up the resources at the end of the first tutorial. After that tutorial, you have a realistic environment with an Amazon RDS database and a Secrets Manager secret that contains admin credentials for the database. You also have a second secret that contains credentials for a database user, but you don't use that secret in this tutorial.

You also have a connection configured in MySQL Workbench to connect to the database with the admin credentials.

## Step 1: Create an Amazon RDS database user

First, you need a user whose credentials will be stored in the secret. To create the user, log into the Amazon RDS database with admin credentials that are stored in a secret. For simplicity, in the tutorial, you create a user with full permission to a database. In a production setting, this is not typical, and we recommend that you follow the principle of least privilege.

###### To retrieve the admin password

1. In the Amazon RDS console, navigate to your database.
2. On the **Configuration** tab, under **Master Credentials ARN**, choose **Manage in Secrets Manager**.
	The Secrets Manager console opens.
3. In the secret details page, choose **Retrieve secret value**.
4. The password appears in the **Secret value** section.

###### To create a database user

1. In MySQL Workbench, right-click the connection **SecretsManagerTutorial** and then choose **Edit Connection**.
2. In the **Manage Server Connections** dialog box, for **Username**, enter `admin`, and then choose **Close**.
3. Back in MySQL Workbench, choose the connection **SecretsManagerTutorial**.
4. Enter the admin password you retrieved from the secret.
5. In MySQL Workbench, in the **Query** window, enter the following commands (including a strong password) and then choose **Execute**. The rotation function tests the updated secret by using SELECT, so the `dbuser` must have that privilege at minimum.
	```
	CREATE USER 'dbuser'@'%' IDENTIFIED BY 'EXAMPLE-PASSWORD';
	GRANT SELECT ON myDB . * TO 'dbuser'@'%';
	```
	In the **Output** window, you see the commands are successful.

## Step 2: Create a secret for the database user credentials

Next, you create a secret to store the credentials of the user you just created, and you turn on automatic rotation, including an immediate rotation. Secrets Manager rotates the secret, which means the password is programmatically generated – no human has seen this new password. Having the rotation begin immediately can also help you determine if rotation is set up correctly.

1. Choose **Store a new secret**.
2. On the **Choose secret type** page, do the following:
	1. For **Secret type**, choose **Credentials for Amazon RDS database**.
		2. For **Credentials**, enter the username `dbuser` and the password you entered for the database user you created using MySQL Workbench.
		3. For **Database**, choose **secretsmanagertutorialdb**.
		4. Choose **Next**.
3. On the **Configure secret** page, for **Secret name**, enter `SecretsManagerTutorialDbuser` and then choose **Next**.
4. On the **Configure rotation** page, do the following:
	1. Turn on **Automatic rotation**.
		2. For **Rotation schedule**, set a schedule of **Days**: `2` Days with **Duration**: `2h`. Keep **Rotate immediately** selected.
		3. For **Rotation function**, choose **Create a rotation function**, and then for the function name, enter `tutorial-single-user-rotation`.
		4. For **Rotation strategy**, choose **Single user**.
		5. Choose **Next**.
5. On the **Review** page, choose **Store**.
	Secrets Manager returns to the secret details page. At the top of the page, you can see the rotation configuration status. Secrets Manager uses CloudFormation to create resources such as the Lambda rotation function and an execution role that runs the Lambda function. When CloudFormation finishes, the banner changes to **Secret scheduled for rotation**. The first rotation is complete.

## Step 3: Test the rotated password

After the first secret rotation, which might take a few seconds, you can check that the secret still contains valid credentials. The password in the secret has changed from the original credentials.

###### To retrieve the new password from the secret

1. Choose **Secrets**, and then choose the secret `SecretsManagerTutorialDbuser`.
2. On the **Secret details** page, scroll down and choose **Retrieve secret value**.
3. In the **Key/value** table, copy the **Secret value** for `password`.

###### To test the credentials

1. In MySQL Workbench, right-click the connection **SecretsManagerTutorial** and then choose **Edit Connection**.
2. In the **Manage Server Connections** dialog box, for **Username**, enter `dbuser`, and then choose **Close**.
3. Back in MySQL Workbench, choose the connection **SecretsManagerTutorial**.
4. In the **Open SSH Connection** dialog box, for **Password**, paste the password you retrieved from the secret, and then choose **OK**.
	If the credentials are valid, then MySQL Workbench opens to the design page for the database.

## Step 4: Clean up resources

To avoid potential charges, delete the secret you created in this tutorial. For instructions, see [Delete an AWS Secrets Manager secret](https://docs.aws.amazon.com/secretsmanager/latest/userguide/manage_delete-secret.html).

To clean up resources created in the previous tutorial, see [Step 4: Clean up resources](https://docs.aws.amazon.com/secretsmanager/latest/userguide/tutorials_rotation-alternating.html#tutorials_rotation-alternating_step-cleanup).

## Next steps

- Learn how to retrieve secrets in your applications. See [Get secrets from AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/retrieving-secrets.html).
- Learn about other rotation schedules. See [Rotation schedules](https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotate-secrets_schedule.html).