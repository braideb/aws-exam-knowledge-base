---
title: "Connect to an RDS PostgreSQL instance using IAM authentication"
source: "https://repost.aws/knowledge-center/rds-postgresql-connect-using-iam"
author:
  - "[[AWS Official]]"
published: 2020-08-24
created: 2026-10-02
description: "I have an Amazon Relational Database Service (Amazon RDS) for PostgreSQL or Amazon Aurora PostgreSQL-Compatible Edition DB instance. I want to use AWS Identity and Access Management (IAM) authentic..."
tags:
  - "clippings"
---
1

I have an Amazon Relational Database Service (Amazon RDS) for PostgreSQL or Amazon Aurora PostgreSQL-Compatible Edition DB instance. I want to use AWS Identity and Access Management (IAM) authentication to connect to my instance.

## Short description

IAM database authentication is more secure than native authentication methods. For more information, see [IAM database authentication for MariaDB, MySQL, and PostgreSQL](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.html).

To use IAM roles to set up IAM database authentication, complete the following steps:

1. Turn on IAM DB authentication on the RDS DB instance.
2. Create an IAM user, and then attach an IAM policy that maps the database user to the IAM role.
3. [Attach the IAM role to the Amazon Elastic Compute Cloud (Amazon EC2) instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/attach-iam-role.html).
4. To identify the IAM role, generate an AWS authentication token.
5. Download the SSL root certificate file or certificate bundle file.
6. To connect to the RDS DB instance, use your IAM role credentials and the authentication token or an SSL certificate.

If you run MySQL, then see [How do I allow users to authenticate to an Amazon RDS for MySQL DB instance using their IAM credentials?](https://repost.aws/knowledge-center/users-connect-rds-iam)

## Resolution

### Prerequisites

Complete the following prerequisites:

- Launch an Amazon RDS for PostgreSQL DB instance or Aurora PostgreSQL-Compatible cluster that supports IAM database authentication
- Launch an EC2 instance to connect to the database

For more information, see [IAM database authentication](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/UsingWithRDS.IAMDBAuth.html) for Aurora and [IAM database authentication](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.html) for Amazon RDS.

To verify the configuration required for IAM authentication with an Amazon RDS instance or Aurora Cluster or troubleshoot IAM authentication, use the **AWSSupport-TroubleshootRDSIAMAuthentication** runbook. For more information, see [AWSSupport-TroubleshootRDSIAMAuthentication](https://docs.aws.amazon.com/systems-manager-automation-runbooks/latest/userguide/automation-aws-troubleshoot-rds-iam-authentication.html).

### Use IAM database authentication

To connect to an RDS DB instance or Aurora PostgreSQL-Compatible DB cluster, use [IAM database authentication for PostgreSQL](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.html):

1. Turn on IAM authentication on your [RDS DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.Enabling.html) or your [Aurora cluster](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/UsingWithRDS.IAMDBAuth.Enabling.html).
2. [Create an IAM user](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/UsingWithRDS.IAMDBAuth.IAMPolicy.html), and then attach the following policy:
	```
	{  "Version": "2012-10-17",
	  "Statement": [
	    {
	      "Effect": "Allow",
	      "Action": [
	        "rds-db:connect"
	      ],
	      "Resource": [
	        "arn:aws:rds-db:REGION:ACCOUNT:dbuser:RESOURCE_ID/iamuser"
	      ]
	    }
	  ]
	}
	```
	**Note:** Replace **iamuser** with the IAM user information.
3. Log in to your Amazon RDS for PostgreSQL DB instance or Aurora PostgreSQL-Compatible cluster as the primary user:  
	**psql**
	```
	psql -h {database or cluster endpoint} -U {Master username} -d {database name}
	```
4. **pgAdmin** Choose **Servers** from the pgAdmin navigation pane. Then, choose the server name and enter the primary user password.
5. Create a child user that has the same name as the IAM user:
	```
	CREATE USER iamuser WITH LOGIN; GRANT rds_iam TO iamuser;
	```
6. Run [generate-db-auth-token](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/rds/generate-db-auth-token.html) with the username that you created. This action creates a temporary password that you use in a later step:
	```
	[ec2-user@ip-172-31-24-237 ~]$ export RDSHOST="aurorapg-ssl.cluster-XXXXXXXXXXX.us-west-2.rds.amazonaws.com"[ec2-user@ip-172-31-24-237 ~]$ export PGPASSWORD="$(aws rds generate-db-auth-token --hostname $RDSHOST --port 5432 --region us-west-2 --username iamuser)"
	[ec2-user@ip-172-31-24-237 ~]$ echo $PGPASSWORD
	aurorapg-ssl.cluster-XXXXXXX.us-west-2.rds.amazonaws.com:5432/?Action=connect&DBUser=iamuser&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Expires=900&X-Amz-Credential=AKIAIOSFODNN7EXAMPLE%2F20190909%2Fus-west-2%2Frds-db%2Faws4_request&X-Amz-SignedHeaders=host&X-Amz-Date=20190909T171907Z&X-Amz-Signature=ead28003477c3082e5a17529ac8316db4f4bdf2fa8f79d3aaea806e9bafa2673
	[ec2-user@ip-172-31-24-237 ~]$
	```
7. To create a new server connection in **pgAdmin**, choose the **General** tab, and then clear the **Connect now** box. From the **Connection** tab, enter the hostname, port, and username, but don't enter the password yet. From the SSL tab, set the **SSL mode** to **Require**, and then save the server connection.
8. (Optional) You can change the **SSL mode** to **verify-full** from the SSL tab. For the selected SSL mode, enter the certificate's path. To download the path, run the following command:
	```
	wget https://s3.amazonaws.com/rds-downloads/rds-ca-2019-root.pem
	```
9. If your application doesn't accept certificate chains, then download the certificate bundle that includes both the old and new root certificates:
	```
	$ wget https://truststore.pki.rds.amazonaws.com/global/global-bundle.pem
	```
10. Connect to the server. When prompted, enter the temporary token that **generate-db-auth-token** generated for the user **iamuser**.For **psql**, run the following command to connect. This example uses the environment variable **$PGPASSWORD** that you set when you generated the token. This variable initiates the connection:
	```
	psql -h aurorapg-ssl.cfkx5hi8csxj.us-west-2.rds.amazonaws.com -p 5432 "sslmode=verify-full sslrootcert=rds-ca-2019-root.pem dbname=aurora_pg_ssl user=iamuser"
	```
	**Note:** Each token expires 15 minutes after you generate it. If you try to re-establish the connection with the same token, the connection fails. You must generate a new token.

If you still receive an error that's similar to **PAM authentication failed for your user**, then check whether the AWS account is part of an AWS Organizations organization. If the account is part of an organization, then add **rds-db:\*** to the [service control policy (SCP) of the organization](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_create.html) that the account belongs to. Also, check whether there's a hierarchy of the IAM user or role that doesn't have the **rds-db** permission.

For more information, see [How to use service control policies to set permission guardrails across accounts in your AWS Organization](https://aws.amazon.com/blogs/security/how-to-use-service-control-policies-to-set-permission-guardrails-across-accounts-in-your-aws-organization/).

## Related information

[Using IAM authentication with Aurora PostgreSQL-Compatible](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/UsingWithRDS.IAMDBAuth.DBAccounts.html#UsingWithRDS.IAMDBAuth.DBAccounts.PostgreSQL)

[Rotating your SSL/TLS certificate](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.SSL-certificate-rotation.html).

5 Comments

Are the instructions for the certificate still correct? For example [https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.SSL.html#UsingWithRDS.SSL.CertificatesAllRegions](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.SSL.html#UsingWithRDS.SSL.CertificatesAllRegions) lists different URLs to download the certificate bundle from.

Is the database then still aware of the user that assumed the IAM role? Otherwise one could not trace executed queries back to certain users.

top