---
title: "Amazon Relational Database Service - AWS Prescriptive Guidance"
source: "https://docs.aws.amazon.com/prescriptive-guidance/latest/encryption-best-practices/rds.html"
author:
published:
created: 2026-10-02
description: "Review best practices and recommendations for encrypting data in Amazon Relational Database Service (Amazon RDS)."
tags:
  - "clippings"
---
Amazon Relational Database Service - AWS Prescriptive Guidance

[Amazon Relational Database Service (Amazon RDS)](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html) helps you set up, operate, and scale a relational database (DB) in the AWS Cloud. Data that is encrypted at rest includes the underlying storage for the DB instances, its automated backups, read replicas, and snapshots.

The following are the approaches you can use to encrypt data at rest in RDS DB instances:

- You can encrypt Amazon RDS DB instances with AWS KMS keys, either an AWS managed key or a customer managed key. For more information, see [AWS Key Management Service](https://docs.aws.amazon.com/prescriptive-guidance/latest/encryption-best-practices/kms.html) in this guide.
- Amazon RDS for Oracle and Amazon RDS for SQL Server support encrypting DB instances with Transparent Data Encryption (TDE). For more information, see [Oracle Transparent Data Encryption](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.Oracle.Options.AdvSecurity.html) or [Support for Transparent Data Encryption in SQL Server](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.SQLServer.Options.TDE.html).
	You can use both TDE and KMS keys to encrypt DB instances. However, this can slightly affect the performance of your database, and you must manage these keys separately.

The following are the approaches you can use to encrypt data in transit to or from RDS DB instances:

- For an Amazon RDS DB instance running MariaDB, Microsoft SQL Server, MySQL, Oracle, or PostgreSQL, you can use SSL to encrypt the connection. For more information, see [Using SSL/TLS to encrypt a connection to a DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.SSL.html).
- Amazon RDS for Oracle also supports Oracle native network encryption (NNE), which encrypts data as it moves to and from a DB instance. NNE and SSL encryption cannot be used simultaneously. For more information, see [Oracle native network encryption](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.Oracle.Options.NetworkEncryption.html).

Consider the following encryption best practices for this service:

- When connecting to Amazon RDS for SQL Server or Amazon RDS for PostgreSQL DB instances in order to process, store, or transmit data that requires encryption, use the RDS Transport Encryption feature to encrypt the connection. You can implement this by setting the `rds.force_ssl` parameter to `1` in the parameter group. For more information, see [Working with parameter groups](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithParamGroups.html). Amazon RDS for Oracle uses Oracle database native network encryption.
- Customer managed keys for RDS DB instance encryption should be used solely for that purpose and not used with any other AWS services.
- Before encrypting an RDS DB instance, establish KMS key requirements. The key used by the instance cannot be changed later. For example, in your encryption policy, define use and management standards for AWS managed keys or customer managed keys, based on your business requirements.
- When authorizing access to a customer managed KMS key, follow the principle of least-privilege by using condition keys in IAM policies. For example, to allow a customer managed key to be used only for requests that originate in Amazon RDS, use the [kms:ViaService condition key](https://docs.aws.amazon.com/kms/latest/developerguide/policy-conditions.html#conditions-kms-via-service) with the `rds.<region>.amazonaws.com` value. Additionally, you can use keys or values in the [Amazon RDS encryption context](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.Encryption.Keys.html#Overview.Encryption.Keys.encryptioncontext) as a condition for using the customer managed key.
- It is strongly recommended that you enable backups for encrypted RDS DB instances. Amazon RDS can lose access to the KMS key for a DB instance, such as when the KMS key isn't enabled or when RDS access to a KMS key is revoked. If this occurs, the encrypted DB instance goes into a recoverable state for seven days. If the DB instance does not regain access to the key after seven days, the database becomes terminally inaccessible and must be restored from a backup. For more information, see [Encrypting a DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.Encryption.html#Overview.Encryption.Enabling).
- If a read replica and its encrypted DB instance are in the same AWS Region, you must use the same KMS key to encrypt both.
- In AWS Config, implement the [rds-storage-encrypted](https://docs.aws.amazon.com/config/latest/developerguide/rds-storage-encrypted.html) AWS managed rule to validate and enforce encryption for RDS DB instances and the [rds-snapshots-encrypted](https://docs.aws.amazon.com/config/latest/developerguide/rds-snapshot-encrypted.html) rule to validate and enforce encryption for RDS database snapshots.
- Use AWS Security Hub CSPM to evaluate whether your Amazon RDS resources follow security best practices. For more information, see [Security Hub CSPM controls for Amazon RDS](https://docs.aws.amazon.com/securityhub/latest/userguide/rds-controls.html).