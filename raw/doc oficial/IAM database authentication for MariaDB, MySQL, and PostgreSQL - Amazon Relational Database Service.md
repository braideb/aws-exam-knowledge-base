---
title: "IAM database authentication for MariaDB, MySQL, and PostgreSQL - Amazon Relational Database Service"
source: "https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.html"
author:
published:
created: 2026-10-02
description: "Authenticate to your DB instance or cluster using AWS Identity and Access Management (IAM) database authentication."
tags:
  - "clippings"
---
IAM database authentication for MariaDB, MySQL, and PostgreSQL - Amazon Relational Database Service

You can authenticate to your DB instance using AWS Identity and Access Management (IAM) database authentication. IAM database authentication works with MariaDB, MySQL, and PostgreSQL. With this authentication method, you don't need to use a password when you connect to a DB instance. Instead, you use an authentication token.

An *authentication token* is a unique string of characters that Amazon RDS generates on request. Authentication tokens are generated using AWS Signature Version 4. Each token has a lifetime of 15 minutes. You don't need to store user credentials in the database, because authentication is managed externally using IAM. You can also still use standard database authentication. The token is only used for authentication and doesn't affect the session after it is established.

IAM database authentication provides the following benefits:

- Network traffic to and from the database is encrypted using Secure Socket Layer (SSL) or Transport Layer Security (TLS). For more information about using SSL/TLS with Amazon RDS, see [Using SSL/TLS to encrypt a connection to a DB instance or cluster](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.SSL.html) .
- You can use IAM to centrally manage access to your database resources, instead of managing access individually on each DB instance.
- For applications running on Amazon EC2, you can use profile credentials specific to your EC2 instance to access your database instead of a password, for greater security.

In general, consider using IAM database authentication when you don't want to manage usernames and passwords directly in your application code.

The Amazon Web Services (AWS) JDBC Driver supports IAM database authentication. For more information, see [AWS IAM Authentication Plugin](https://github.com/aws/aws-advanced-jdbc-wrapper/blob/main/docs/using-the-jdbc-driver/using-plugins/UsingTheIamAuthenticationPlugin.md) in the [Amazon Web Services (AWS) JDBC Driver GitHub repository](https://github.com/aws/aws-advanced-jdbc-wrapper).

The Amazon Web Services (AWS) Python Driver supports IAM database authentication. For more information, see [AWS IAM Authentication Plugin](https://github.com/aws/aws-advanced-python-wrapper/blob/main/docs/using-the-python-driver/using-plugins/UsingTheIamAuthenticationPlugin.md) in the [Amazon Web Services (AWS) Python Driver GitHub repository](https://github.com/aws/aws-advanced-python-wrapper).

Navigate through the following topics to learn the process to set IAM for DB authentication:

- [Enabling and disabling IAM database authentication](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.Enabling.html)
- [Creating and using an IAM policy for IAM database access](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.IAMPolicy.html)
- [Creating a database account using IAM authentication](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.DBAccounts.html)
- [Connecting to your DB instance using IAM authentication](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.Connecting.html)

## Region and version availability

Feature availability and support varies across specific versions of each database engine. For more information on engine, version, and Region availability with Amazon RDS and IAM database authentication, see [Supported Regions and DB engines for IAM database authentication in Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RDS_Fea_Regions_DB-eng.Feature.IamDatabaseAuthentication.html).

## CLI and SDK support

IAM database authentication is available for the [AWS CLI](https://docs.aws.amazon.com/cli/latest/reference/rds/generate-db-auth-token.html) and for the following language-specific AWS SDKs:

- [AWS SDK for.NET](https://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/RDS/TRDSAuthTokenGenerator.html)
- [AWS SDK for C++](https://docs.aws.amazon.com/sdk-for-cpp/latest/api/class_aws_1_1_r_d_s_1_1_r_d_s_client.html#ae134ffffed5d7672f6156d324e7bd392)
- [AWS SDK for Go](https://docs.aws.amazon.com/sdk-for-go/api/service/rds/#pkg-overview)
- [AWS SDK for Java](https://docs.aws.amazon.com/sdk-for-java/latest/reference/software/amazon/awssdk/services/rds/RdsUtilities.html)
- [AWS SDK for JavaScript](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/modules/_aws_sdk_rds_signer.html)
- [AWS SDK for PHP](https://docs.aws.amazon.com/aws-sdk-php/v3/api/class-Aws.Rds.AuthTokenGenerator.html)
- [AWS SDK for Python (Boto3)](https://docs.aws.amazon.com/boto3/latest/reference/services/rds.html#RDS.Client.generate_db_auth_token)
- [AWS SDK for Ruby](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/RDS/AuthTokenGenerator.html)

## Limitations for IAM database authentication

When using IAM database authentication, the following limitations apply:

- Currently, IAM database authentication doesn't support all global condition context keys.
	For more information about global condition context keys, see [AWS global condition context keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html) in the *IAM User Guide*.
- For PostgreSQL, if the IAM role (`rds_iam`) is added to a user (including the RDS master user), IAM authentication takes precedence over password authentication, so the user must log in as an IAM user.
- For PostgreSQL, Amazon RDS does not support enabling both IAM and Kerberos authentication methods at the same time.
- You cannot use a custom Route 53 DNS record instead of the DB instance endpoint to generate the authentication token.
- CloudWatch and CloudTrail don't log IAM authentication. These services do not track `generate-db-auth-token` API calls that authorize the IAM role to enable database connection.
- IAM DB authentication requires compute resources on the database instance. You must have between 300 and 1000 MiB extra memory on your database for reliable connectivity. To see the memory needed for your workload, compare the RES column for RDS processes in the Enhanced Monitoring processlist before and after enabling IAM DB authentication. See [Viewing OS metrics in the RDS console](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_Monitoring.OS.Viewing.html).
	If you are using a burstable class instance, avoid running out of memory by reducing the memory used by other parameters like buffers and cache by the same amount.
- IAM DB authentication is not supported for RDS on Outposts for any engine.

## Recommendations for IAM database authentication

We recommend the following when using IAM database authentication:

- The size of an IAM database authentication token depends on many things including the number of IAM tags, IAM service policies, ARN lengths, as well as other IAM and database properties. The minimum size of this token is generally about 1 KB but can be larger. Since this token is used as the password in the connection string to the database using IAM authentication, you should ensure that your database driver (for example, ODBC) and/or any tools do not limit or otherwise truncate this token due to its size. A truncated token will cause the authentication validation done by the database and IAM to fail.
- If you are using temporary credentials when creating an IAM database authentication token, the temporary credentials must still be valid when using the IAM database authentication token to make a connection request.

## Unsupported AWS global condition context keys

IAM database authentication does not support the following subset of AWS global condition context keys.

- `aws:Referer`
- `aws:SourceIp`
- `aws:SourceVpc`
- `aws:SourceVpce`
- `aws:UserAgent`
- `aws:VpcSourceIp`

For more information, see [AWS global condition context keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html) in the *IAM User Guide*.