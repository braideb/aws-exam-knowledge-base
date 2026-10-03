---
title: "Password management with Amazon RDS and AWS Secrets Manager - Amazon Relational Database Service"
source: "https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-secrets-manager.html"
author:
published:
created: 2026-10-02
description: "Learn about managing Amazon RDS passwords with AWS Secrets Manager."
tags:
  - "clippings"
---
Password management with Amazon RDS and AWS Secrets Manager - Amazon Relational Database Service

Amazon RDS integrates with Secrets Manager to manage master user passwords for your DB instances and Multi-AZ DB clusters.

## Limitations for Secrets Manager integration with Amazon RDS

Managing master user passwords with Secrets Manager isn't supported for the following features:

- Creating a read replica when the source DB or DB cluster manages credentials with Secrets Manager. This applies to all DB engines except RDS for SQL Server and RDS for Db2.
- Amazon RDS Blue/Green Deployments
- Amazon RDS Custom
- Oracle Data Guard switchover

## Overview of managing master user passwords with AWS Secrets Manager

With AWS Secrets Manager, you can replace hard-coded credentials in your code, including database passwords, with an API call to Secrets Manager to retrieve the secret programmatically. For more information about Secrets Manager, see the [AWS Secrets Manager User Guide](https://docs.aws.amazon.com/secretsmanager/latest/userguide/).

When you store database secrets in Secrets Manager, your AWS account incurs charges. For information about pricing, see [AWS Secrets Manager Pricing](https://aws.amazon.com/secrets-manager/pricing).

You can specify that RDS manages the master user password in Secrets Manager for an Amazon RDS DB instance or Multi-AZ DB cluster when you perform one of the following operations:

- Create a DB instance
- Create a Multi-AZ DB cluster
- Create a tenant database in an RDS for Oracle CDB
- Modify a DB instance
- Modify a Multi-AZ DB cluster
- Modify a tenant database (RDS for Oracle only)
- Restore a DB instance from Amazon S3
- Restore a DB instance from a snapshot or to a point in time (RDS for Oracle only)

When you specify that RDS manages the master user password in Secrets Manager, RDS generates the password and stores it in Secrets Manager. You can interact directly with the secret to retrieve the credentials for the master user. You can also specify a customer managed key to encrypt the secret, or use the KMS key that is provided by Secrets Manager.

RDS manages the settings for the secret and rotates the secret every seven days by default. You can modify some of the settings, such as the rotation schedule. If you delete a DB instance that manages a secret in Secrets Manager, the secret and its associated metadata are also deleted.

To connect to a DB instance or Multi-AZ DB cluster with the credentials in a secret, you can retrieve the secret from Secrets Manager. For more information, see [Retrieve secrets from AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/retrieving-secrets.html) and [Connect to a SQL database with credentials in an AWS Secrets Manager secret](https://docs.aws.amazon.com/secretsmanager/latest/userguide/retrieving-secrets_jdbc.html) in the *AWS Secrets Manager User Guide*.

## Benefits of managing master user passwords with Secrets Manager

Managing RDS master user passwords with Secrets Manager provides the following benefits:

- RDS automatically generates database credentials.
- RDS automatically stores and manages database credentials in AWS Secrets Manager.
- RDS rotates database credentials regularly, without requiring application changes.
- Secrets Manager secures database credentials from human access and plain text view.
- Secrets Manager allows retrieval of database credentials in secrets for database connections.
- Secrets Manager allows fine-grained control of access to database credentials in secrets using IAM.
- You can optionally separate database encryption from credentials encryption with different KMS keys.
- You can eliminate manual management and rotation of database credentials.
- You can monitor database credentials easily with AWS CloudTrail and Amazon CloudWatch.

For more information about the benefits of Secrets Manager, see the [AWS Secrets Manager User Guide](https://docs.aws.amazon.com/secretsmanager/latest/userguide/).

## Permissions required for Secrets Manager integration

Users must have the required permissions to perform operations related to Secrets Manager integration. You can create IAM policies that grant permissions to perform specific API operations on the specified resources they need. You can then attach those policies to the IAM permission sets or roles that require those permissions. For more information, see [Identity and access management for Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAM.html).

For create, modify, or restore operations, the user who specifies that Amazon RDS manages the master user password in Secrets Manager must have permissions to perform the following operations:

- `kms:DescribeKey`
- `secretsmanager:CreateSecret`
- `secretsmanager:TagResource`

The `kms:DescribeKey` permission is required to access your customer-managed key for the `MasterUserSecretKmsKeyId` and to describe `aws/secretsmanager`.

For create, modify, or restore operations, the user who specifies the customer managed key to encrypt the secret in Secrets Manager must have permissions to perform the following operations:

- `kms:Decrypt`
- `kms:GenerateDataKey`
- `kms:CreateGrant`

For modify operations, the user who rotates the master user password in Secrets Manager must have permissions to perform the following operation:

- `secretsmanager:RotateSecret`

## Enforcing RDS management of the master user password in AWS Secrets Manager

You can use IAM condition keys to enforce RDS management of the master user password in AWS Secrets Manager. The following policy doesn't allow users to create or restore DB instances or DB clusters or create or modify tenant databases unless the master user password is managed by RDS in Secrets Manager.

anchor

```json
{
    "Version":"2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": ["rds:CreateDBInstance", "rds:CreateDBCluster", "rds:RestoreDBInstanceFromS3", "rds:RestoreDBClusterFromS3",
                       "rds:RestoreDBInstanceFromDBSnapshot", "rds:RestoreDBInstanceToPointInTime", "rds:CreateTenantDatabase",
                       "rds:ModifyTenantDatabase"],
            "Resource": "*",
            "Condition": {
                "Bool": {
                    "rds:ManageMasterUserPassword": false
                }
            }
        }
    ]
}
```

###### Note

This policy enforces password management in AWS Secrets Manager at creation. However, you can still disable Secrets Manager integration and manually set a master password by modifying the instance.

To prevent this, include `rds:ModifyDBInstance`, `rds:ModifyDBCluster` in the action block of the policy. Be aware, this prevents the user from applying any further modifications to existing instances that don't have Secrets Manager integration enabled.

For more information about using condition keys in IAM policies, see [Policy condition keys for Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/security_iam_service-with-iam.html#UsingWithRDS.IAM.Conditions) and [Example policies: Using condition keys](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAM.Conditions.Examples.html).

## Managing the master user password for a DB instance with Secrets Manager

You can configure RDS management of the master user password in Secrets Manager when you perform the following actions:

- [Creating an Amazon RDS DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_CreateDBInstance.html)
- [Modifying an Amazon RDS DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.DBInstance.Modifying.html)
- [Restoring a backup into an Amazon RDS for MySQL DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/MySQL.Procedural.Importing.html)
- [Restoring to a DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_RestoreFromSnapshot.html) (RDS for Oracle only)
- [Restoring a DB instance to a specified time for Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PIT.html) (RDS for Oracle only)

You can perform the preceding operations using the RDS console, the AWS CLI, or the RDS API.

Follow the instructions for creating or modifying a DB instance with the RDS console:

- [Creating a DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_CreateDBInstance.html#USER_CreateDBInstance.Creating)
- [Modifying an Amazon RDS DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.DBInstance.Modifying.html)
- [Importing data from Amazon S3 to a new MySQL DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/MySQL.Procedural.Importing.html#MySQL.Procedural.Importing.PerformingImport)

When you use the RDS console to perform one of these operations, you can specify that the master user password is managed by RDS in Secrets Manager. When you're creating or restoring a DB instance, select **Manage master credentials in AWS Secrets Manager** in **Credential settings**. When you're modifying a DB instance, select **Manage master credentials in AWS Secrets Manager** in **Settings**.

The following image is an example of the **Manage master credentials in AWS Secrets Manager** setting when you are creating or restoring a DB instance.

![Manage master credentials in AWS Secrets Manager.](https://docs.aws.amazon.com/images/AmazonRDS/latest/UserGuide/images/secrets-manager-credential-settings-db-instance.png)

When you select this option, RDS generates the master user password and manages it throughout its lifecycle in Secrets Manager.

![Manage master credentials in AWS Secrets Manager selected.](https://docs.aws.amazon.com/images/AmazonRDS/latest/UserGuide/images/secrets-manager-integration-create-db-instance.png)

You can choose to encrypt the secret with a KMS key that Secrets Manager provides or with a customer managed key that you create. After RDS is managing the database credentials for a DB instance, you can't change the KMS key used to encrypt the secret.

You can choose other settings to meet your requirements. For more information about the available settings when you're creating a DB instance, see [Settings for DB instances](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_CreateDBInstance.Settings.html). For more information about the available settings when you're modifying a DB instance, see [Settings for DB instances](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ModifyInstance.Settings.html).

## Console

Follow the instructions for creating or modifying a DB instance with the RDS console:

- [Creating a DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_CreateDBInstance.html#USER_CreateDBInstance.Creating)
- [Modifying an Amazon RDS DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.DBInstance.Modifying.html)
- [Importing data from Amazon S3 to a new MySQL DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/MySQL.Procedural.Importing.html#MySQL.Procedural.Importing.PerformingImport)

When you use the RDS console to perform one of these operations, you can specify that the master user password is managed by RDS in Secrets Manager. When you're creating or restoring a DB instance, select **Manage master credentials in AWS Secrets Manager** in **Credential settings**. When you're modifying a DB instance, select **Manage master credentials in AWS Secrets Manager** in **Settings**.

The following image is an example of the **Manage master credentials in AWS Secrets Manager** setting when you are creating or restoring a DB instance.

![Manage master credentials in AWS Secrets Manager.](https://docs.aws.amazon.com/images/AmazonRDS/latest/UserGuide/images/secrets-manager-credential-settings-db-instance.png)

When you select this option, RDS generates the master user password and manages it throughout its lifecycle in Secrets Manager.

![Manage master credentials in AWS Secrets Manager selected.](https://docs.aws.amazon.com/images/AmazonRDS/latest/UserGuide/images/secrets-manager-integration-create-db-instance.png)

You can choose to encrypt the secret with a KMS key that Secrets Manager provides or with a customer managed key that you create. After RDS is managing the database credentials for a DB instance, you can't change the KMS key used to encrypt the secret.

You can choose other settings to meet your requirements. For more information about the available settings when you're creating a DB instance, see [Settings for DB instances](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_CreateDBInstance.Settings.html). For more information about the available settings when you're modifying a DB instance, see [Settings for DB instances](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ModifyInstance.Settings.html).

To manage the master user password with RDS in Secrets Manager, specify the `--manage-master-user-password` option in one of the following AWS CLI commands:

When you specify the `--manage-master-user-password` option in these commands, RDS generates the master user password and manages it throughout its lifecycle in Secrets Manager.

To encrypt the secret, you can specify a customer managed key or use the default KMS key that is provided by Secrets Manager. Use the `--master-user-secret-kms-key-id` option to specify a customer managed key. The AWS KMS key identifier is the key ARN, key ID, alias ARN, or alias name for the KMS key. To use a KMS key in a different AWS account, specify the key ARN or alias ARN. After RDS is managing the database credentials for a DB instance, you can't change the KMS key that is used to encrypt the secret.

You can choose other settings to meet your requirements. For more information about the available settings when you are creating a DB instance, see [Settings for DB instances](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_CreateDBInstance.Settings.html). For more information about the available settings when you are modifying a DB instance, see [Settings for DB instances](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ModifyInstance.Settings.html).

The following example creates a DB instance and specifies that RDS manages the master user password in Secrets Manager. The secret is encrypted using the KMS key that is provided by Secrets Manager.

###### Example

For Linux, macOS, or Unix:

```
aws rds create-db-instance \
    --db-instance-identifier mydbinstance \
    --engine mysql \
    --engine-version 8.0.39 \
    --db-instance-class db.r5b.large \
    --allocated-storage 200 \
    --master-username testUser \
    --manage-master-user-password
```

For Windows:

```
aws rds create-db-instance ^
    --db-instance-identifier mydbinstance ^
    --engine mysql ^
    --engine-version 8.0.39 ^
    --db-instance-class db.r5b.large ^
    --allocated-storage 200 ^
    --master-username testUser ^
    --manage-master-user-password
```

## AWS CLI

To manage the master user password with RDS in Secrets Manager, specify the `--manage-master-user-password` option in one of the following AWS CLI commands:

When you specify the `--manage-master-user-password` option in these commands, RDS generates the master user password and manages it throughout its lifecycle in Secrets Manager.

To encrypt the secret, you can specify a customer managed key or use the default KMS key that is provided by Secrets Manager. Use the `--master-user-secret-kms-key-id` option to specify a customer managed key. The AWS KMS key identifier is the key ARN, key ID, alias ARN, or alias name for the KMS key. To use a KMS key in a different AWS account, specify the key ARN or alias ARN. After RDS is managing the database credentials for a DB instance, you can't change the KMS key that is used to encrypt the secret.

You can choose other settings to meet your requirements. For more information about the available settings when you are creating a DB instance, see [Settings for DB instances](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_CreateDBInstance.Settings.html). For more information about the available settings when you are modifying a DB instance, see [Settings for DB instances](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ModifyInstance.Settings.html).

The following example creates a DB instance and specifies that RDS manages the master user password in Secrets Manager. The secret is encrypted using the KMS key that is provided by Secrets Manager.

###### Example

For Linux, macOS, or Unix:

```
aws rds create-db-instance \
    --db-instance-identifier mydbinstance \
    --engine mysql \
    --engine-version 8.0.39 \
    --db-instance-class db.r5b.large \
    --allocated-storage 200 \
    --master-username testUser \
    --manage-master-user-password
```

For Windows:

```
aws rds create-db-instance ^
    --db-instance-identifier mydbinstance ^
    --engine mysql ^
    --engine-version 8.0.39 ^
    --db-instance-class db.r5b.large ^
    --allocated-storage 200 ^
    --master-username testUser ^
    --manage-master-user-password
```

To specify that RDS manages the master user password in Secrets Manager, set the `ManageMasterUserPassword` parameter to `true` in one of the following RDS API operations:

When you set the `ManageMasterUserPassword` parameter to `true` in one of these operations, RDS generates the master user password and manages it throughout its lifecycle in Secrets Manager.

To encrypt the secret, you can specify a customer managed key or use the default KMS key that is provided by Secrets Manager. Use the `MasterUserSecretKmsKeyId` parameter to specify a customer managed key. The AWS KMS key identifier is the key ARN, key ID, alias ARN, or alias name for the KMS key. To use a KMS key in a different AWS account, specify the key ARN or alias ARN. After RDS is managing the database credentials for a DB instance, you can't change the KMS key that is used to encrypt the secret.

## RDS API

To specify that RDS manages the master user password in Secrets Manager, set the `ManageMasterUserPassword` parameter to `true` in one of the following RDS API operations:

When you set the `ManageMasterUserPassword` parameter to `true` in one of these operations, RDS generates the master user password and manages it throughout its lifecycle in Secrets Manager.

To encrypt the secret, you can specify a customer managed key or use the default KMS key that is provided by Secrets Manager. Use the `MasterUserSecretKmsKeyId` parameter to specify a customer managed key. The AWS KMS key identifier is the key ARN, key ID, alias ARN, or alias name for the KMS key. To use a KMS key in a different AWS account, specify the key ARN or alias ARN. After RDS is managing the database credentials for a DB instance, you can't change the KMS key that is used to encrypt the secret.

## Managing the master user password for an RDS for Oracle tenant database with Secrets Manager

You can configure RDS management of the master user password in Secrets Manager when you perform the following actions:

- [Adding an RDS for Oracle tenant database to your CDB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/oracle-cdb-configuring.adding.pdb.html)
- [Modifying an RDS for Oracle tenant database](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/oracle-cdb-configuring.modifying.pdb.html)

You can use the RDS console, the AWS CLI, or the RDS API to perform the preceding actions.

Follow the instructions for creating or modifying an RDS for Oracle tenant database with the RDS console:

- [Adding an RDS for Oracle tenant database to your CDB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/oracle-cdb-configuring.adding.pdb.html)
- [Modifying an RDS for Oracle tenant database](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/oracle-cdb-configuring.modifying.pdb.html)

When you use the RDS console to perform one of the preceding operations, you can specify that RDS manage the master password in Secrets Manager. When you create a tenant database, select **Manage master credentials in AWS Secrets Manager** in **Credential settings**. When you modify a tenant database, select **Manage master credentials in AWS Secrets Manager** in **Settings**.

The following image is an example of the **Manage master credentials in AWS Secrets Manager** setting when you are creating a tenant database.

![Manage master credentials in AWS Secrets Manager.](https://docs.aws.amazon.com/images/AmazonRDS/latest/UserGuide/images/secrets-manager-credential-settings-db-instance.png)

When you select this option, RDS generates the master user password and manages it throughout its lifecycle in Secrets Manager.

![Manage master credentials in AWS Secrets Manager selected.](https://docs.aws.amazon.com/images/AmazonRDS/latest/UserGuide/images/secrets-manager-integration-create-db-instance.png)

You can choose to encrypt the secret with a KMS key that Secrets Manager provides or with a customer managed key that you create. After RDS is managing the database credentials for a tenant database, you can't change the KMS key that is used to encrypt the secret.

You can choose other settings to meet your requirements. For more information about the available settings when you are creating a tenant database, see [Settings for DB instances](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_CreateDBInstance.Settings.html). For more information about the available settings when you are modifying a tenant database, see [Settings for DB instances](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ModifyInstance.Settings.html).

## Console

Follow the instructions for creating or modifying an RDS for Oracle tenant database with the RDS console:

- [Adding an RDS for Oracle tenant database to your CDB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/oracle-cdb-configuring.adding.pdb.html)
- [Modifying an RDS for Oracle tenant database](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/oracle-cdb-configuring.modifying.pdb.html)

When you use the RDS console to perform one of the preceding operations, you can specify that RDS manage the master password in Secrets Manager. When you create a tenant database, select **Manage master credentials in AWS Secrets Manager** in **Credential settings**. When you modify a tenant database, select **Manage master credentials in AWS Secrets Manager** in **Settings**.

The following image is an example of the **Manage master credentials in AWS Secrets Manager** setting when you are creating a tenant database.

![Manage master credentials in AWS Secrets Manager.](https://docs.aws.amazon.com/images/AmazonRDS/latest/UserGuide/images/secrets-manager-credential-settings-db-instance.png)

When you select this option, RDS generates the master user password and manages it throughout its lifecycle in Secrets Manager.

![Manage master credentials in AWS Secrets Manager selected.](https://docs.aws.amazon.com/images/AmazonRDS/latest/UserGuide/images/secrets-manager-integration-create-db-instance.png)

You can choose to encrypt the secret with a KMS key that Secrets Manager provides or with a customer managed key that you create. After RDS is managing the database credentials for a tenant database, you can't change the KMS key that is used to encrypt the secret.

You can choose other settings to meet your requirements. For more information about the available settings when you are creating a tenant database, see [Settings for DB instances](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_CreateDBInstance.Settings.html). For more information about the available settings when you are modifying a tenant database, see [Settings for DB instances](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ModifyInstance.Settings.html).

To manage the master user password with RDS in Secrets Manager, specify the `--manage-master-user-password` option in one of the following AWS CLI commands:

- [create-tenant-database](https://docs.aws.amazon.com/cli/latest/reference/rds/create-tenant-database.html)
- [modify-tenant-database](https://docs.aws.amazon.com/cli/latest/reference/rds/modify-tenant-database.html)

When you specify the `--manage-master-user-password` option in the preceding commands, RDS generates the master user password and manages it throughout its lifecycle in Secrets Manager.

To encrypt the secret, you can specify a customer managed key or use the default KMS key that is provided by Secrets Manager. Use the `--master-user-secret-kms-key-id` option to specify a customer managed key. The AWS KMS key identifier is the key ARN, key ID, alias ARN, or alias name for the KMS key. To use a KMS key in a different AWS account, specify the key ARN or alias ARN. After RDS is managing the database credentials for a tenant database, you can't change the KMS key that is used to encrypt the secret.

You can choose other settings to meet your requirements. For more information about the available settings when you are creating a tenant database, see [create-tenant-database](https://docs.aws.amazon.com/cli/latest/reference/rds/create-tenant-database.html). For more information about the available settings when you are modifying a tenant database, see [modify-tenant-database](https://docs.aws.amazon.com/cli/latest/reference/rds/modify-tenant-database.html).

The following example creates an RDS for Oracle tenant database and specifies that RDS manages the master user password in Secrets Manager. The secret is encrypted using the KMS key that is provided by Secrets Manager.

###### Example

For Linux, macOS, or Unix:

```
aws rds create-tenant-database --region us-east-1 \
    --db-instance-identifier my-cdb-inst \
    --tenant-db-name mypdb2 \
    --master-username mypdb2-admin \
    --character-set-name UTF-16 \
    --manage-master-user-password
```

For Windows:

```
aws rds create-tenant-database --region us-east-1 ^
    --db-instance-identifier my-cdb-inst ^
    --tenant-db-name mypdb2 ^
    --master-username mypdb2-admin ^
    --character-set-name UTF-16 ^
    --manage-master-user-password
```

## AWS CLI

To manage the master user password with RDS in Secrets Manager, specify the `--manage-master-user-password` option in one of the following AWS CLI commands:

- [create-tenant-database](https://docs.aws.amazon.com/cli/latest/reference/rds/create-tenant-database.html)
- [modify-tenant-database](https://docs.aws.amazon.com/cli/latest/reference/rds/modify-tenant-database.html)

When you specify the `--manage-master-user-password` option in the preceding commands, RDS generates the master user password and manages it throughout its lifecycle in Secrets Manager.

To encrypt the secret, you can specify a customer managed key or use the default KMS key that is provided by Secrets Manager. Use the `--master-user-secret-kms-key-id` option to specify a customer managed key. The AWS KMS key identifier is the key ARN, key ID, alias ARN, or alias name for the KMS key. To use a KMS key in a different AWS account, specify the key ARN or alias ARN. After RDS is managing the database credentials for a tenant database, you can't change the KMS key that is used to encrypt the secret.

You can choose other settings to meet your requirements. For more information about the available settings when you are creating a tenant database, see [create-tenant-database](https://docs.aws.amazon.com/cli/latest/reference/rds/create-tenant-database.html). For more information about the available settings when you are modifying a tenant database, see [modify-tenant-database](https://docs.aws.amazon.com/cli/latest/reference/rds/modify-tenant-database.html).

The following example creates an RDS for Oracle tenant database and specifies that RDS manages the master user password in Secrets Manager. The secret is encrypted using the KMS key that is provided by Secrets Manager.

###### Example

For Linux, macOS, or Unix:

```
aws rds create-tenant-database --region us-east-1 \
    --db-instance-identifier my-cdb-inst \
    --tenant-db-name mypdb2 \
    --master-username mypdb2-admin \
    --character-set-name UTF-16 \
    --manage-master-user-password
```

For Windows:

```
aws rds create-tenant-database --region us-east-1 ^
    --db-instance-identifier my-cdb-inst ^
    --tenant-db-name mypdb2 ^
    --master-username mypdb2-admin ^
    --character-set-name UTF-16 ^
    --manage-master-user-password
```

To specify that RDS manages the master user password in Secrets Manager, set the `ManageMasterUserPassword` parameter to `true` in one of the following RDS API operations:

- [CreateTenantDatabase](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_CreateTenantDatabase.html)
- [ModifyTenantDatabase](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_ModifyTenantDatabase.html)

When you set the `ManageMasterUserPassword` parameter to `true` in one of these operations, RDS generates the master user password and manages it throughout its lifecycle in Secrets Manager.

To encrypt the secret, you can specify a customer managed key or use the default KMS key that is provided by Secrets Manager. Use the `MasterUserSecretKmsKeyId` parameter to specify a customer managed key. The AWS KMS key identifier is the key ARN, key ID, alias ARN, or alias name for the KMS key. To use a KMS key in a different AWS account, specify the key ARN or alias ARN. After RDS is managing the database credentials for a tenant database, you can't change the KMS key that is used to encrypt the secret.

## RDS API

To specify that RDS manages the master user password in Secrets Manager, set the `ManageMasterUserPassword` parameter to `true` in one of the following RDS API operations:

- [CreateTenantDatabase](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_CreateTenantDatabase.html)
- [ModifyTenantDatabase](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_ModifyTenantDatabase.html)

When you set the `ManageMasterUserPassword` parameter to `true` in one of these operations, RDS generates the master user password and manages it throughout its lifecycle in Secrets Manager.

To encrypt the secret, you can specify a customer managed key or use the default KMS key that is provided by Secrets Manager. Use the `MasterUserSecretKmsKeyId` parameter to specify a customer managed key. The AWS KMS key identifier is the key ARN, key ID, alias ARN, or alias name for the KMS key. To use a KMS key in a different AWS account, specify the key ARN or alias ARN. After RDS is managing the database credentials for a tenant database, you can't change the KMS key that is used to encrypt the secret.

## Managing the master user password for a Multi-AZDB cluster with Secrets Manager

You can configure RDS management of the master user password in Secrets Manager when you perform the following actions:

- [Creating a Multi-AZ DB cluster for Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/create-multi-az-db-cluster.html)
- [Modifying a Multi-AZ DB cluster for Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/modify-multi-az-db-cluster.html)

You can use the RDS console, the AWS CLI, or the RDS API to perform these actions.

Follow the instructions for creating or modifying a Multi-AZ DB cluster with the RDS console:

- [Creating a DB cluster](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/create-multi-az-db-cluster.html#create-multi-az-db-cluster-creating)
- [Modifying a Multi-AZ DB cluster for Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/modify-multi-az-db-cluster.html)

When you use the RDS console to perform one of these operations, you can specify that the master user password is managed by RDS in Secrets Manager. To do so when you are creating a DB cluster, select **Manage master credentials in AWS Secrets Manager** in **Credential settings**. When you are modifying a DB cluster, select **Manage master credentials in AWS Secrets Manager** in **Settings**.

The following image is an example of the **Manage master credentials in AWS Secrets Manager** setting when you are creating a DB cluster.

![Manage master credentials in AWS Secrets Manager.](https://docs.aws.amazon.com/images/AmazonRDS/latest/UserGuide/images/secrets-manager-credential-settings.png)

When you select this option, RDS generates the master user password and manages it throughout its lifecycle in Secrets Manager.

![Manage master credentials in AWS Secrets Manager selected.](https://docs.aws.amazon.com/images/AmazonRDS/latest/UserGuide/images/secrets-manager-integration-create.png)

You can choose to encrypt the secret with a KMS key that Secrets Manager provides or with a customer managed key that you create. After RDS is managing the database credentials for a DB cluster, you can't change the KMS key that is used to encrypt the secret.

You can choose other settings to meet your requirements.

For more information about the available settings when you are creating a Multi-AZ DB cluster, see [Settings for creating Multi-AZ DB clusters](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/create-multi-az-db-cluster.html#create-multi-az-db-cluster-settings). For more information about the available settings when you are modifying a Multi-AZ DB cluster, see [Settings for modifying Multi-AZ DB clusters](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/modify-multi-az-db-cluster.html#modify-multi-az-db-cluster-settings).

## Console

Follow the instructions for creating or modifying a Multi-AZ DB cluster with the RDS console:

- [Creating a DB cluster](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/create-multi-az-db-cluster.html#create-multi-az-db-cluster-creating)
- [Modifying a Multi-AZ DB cluster for Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/modify-multi-az-db-cluster.html)

When you use the RDS console to perform one of these operations, you can specify that the master user password is managed by RDS in Secrets Manager. To do so when you are creating a DB cluster, select **Manage master credentials in AWS Secrets Manager** in **Credential settings**. When you are modifying a DB cluster, select **Manage master credentials in AWS Secrets Manager** in **Settings**.

The following image is an example of the **Manage master credentials in AWS Secrets Manager** setting when you are creating a DB cluster.

![Manage master credentials in AWS Secrets Manager.](https://docs.aws.amazon.com/images/AmazonRDS/latest/UserGuide/images/secrets-manager-credential-settings.png)

When you select this option, RDS generates the master user password and manages it throughout its lifecycle in Secrets Manager.

![Manage master credentials in AWS Secrets Manager selected.](https://docs.aws.amazon.com/images/AmazonRDS/latest/UserGuide/images/secrets-manager-integration-create.png)

You can choose to encrypt the secret with a KMS key that Secrets Manager provides or with a customer managed key that you create. After RDS is managing the database credentials for a DB cluster, you can't change the KMS key that is used to encrypt the secret.

You can choose other settings to meet your requirements.

For more information about the available settings when you are creating a Multi-AZ DB cluster, see [Settings for creating Multi-AZ DB clusters](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/create-multi-az-db-cluster.html#create-multi-az-db-cluster-settings). For more information about the available settings when you are modifying a Multi-AZ DB cluster, see [Settings for modifying Multi-AZ DB clusters](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/modify-multi-az-db-cluster.html#modify-multi-az-db-cluster-settings).

To specify that RDS manages the master user password in Secrets Manager, specify the `--manage-master-user-password` option in one of the following commands:

- [create-db-cluster](https://docs.aws.amazon.com/cli/latest/reference/rds/create-db-cluster.html)
- [modify-db-cluster](https://docs.aws.amazon.com/cli/latest/reference/rds/modify-db-cluster.html)

When you specify the `--manage-master-user-password` option in these commands, RDS generates the master user password and manages it throughout its lifecycle in Secrets Manager.

To encrypt the secret, you can specify a customer managed key or use the default KMS key that is provided by Secrets Manager. Use the `--master-user-secret-kms-key-id` option to specify a customer managed key. The AWS KMS key identifier is the key ARN, key ID, alias ARN, or alias name for the KMS key. To use a KMS key in a different AWS account, specify the key ARN or alias ARN. After RDS is managing the database credentials for a DB cluster, you can't change the KMS key that is used to encrypt the secret.

You can choose other settings to meet your requirements.

For more information about the available settings when you are creating a Multi-AZ DB cluster, see [Settings for creating Multi-AZ DB clusters](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/create-multi-az-db-cluster.html#create-multi-az-db-cluster-settings). For more information about the available settings when you are modifying a Multi-AZ DB cluster, see [Settings for modifying Multi-AZ DB clusters](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/modify-multi-az-db-cluster.html#modify-multi-az-db-cluster-settings).

This example creates a Multi-AZ DB cluster and specifies that RDS manages the password in Secrets Manager. The secret is encrypted using the KMS key that is provided by Secrets Manager.

###### Example

For Linux, macOS, or Unix:

```
aws rds create-db-cluster \
   --db-cluster-identifier mysql-multi-az-db-cluster \
   --engine mysql \
   --engine-version 8.0.39  \
   --backup-retention-period 1  \
   --allocated-storage 4000 \
   --storage-type io1 \
   --iops 10000 \
   --db-cluster-instance-class db.r6gd.xlarge \
   --master-username testUser \
   --manage-master-user-password
```

For Windows:

```
aws rds create-db-cluster ^
   --db-cluster-identifier mysql-multi-az-db-cluster ^
   --engine mysql ^
   --engine-version 8.0.39 ^
   --backup-retention-period 1 ^
   --allocated-storage 4000 ^
   --storage-type io1 ^
   --iops 10000 ^
   --db-cluster-instance-class db.r6gd.xlarge ^
   --master-username testUser ^
   --manage-master-user-password
```

## AWS CLI

To specify that RDS manages the master user password in Secrets Manager, specify the `--manage-master-user-password` option in one of the following commands:

- [create-db-cluster](https://docs.aws.amazon.com/cli/latest/reference/rds/create-db-cluster.html)
- [modify-db-cluster](https://docs.aws.amazon.com/cli/latest/reference/rds/modify-db-cluster.html)

When you specify the `--manage-master-user-password` option in these commands, RDS generates the master user password and manages it throughout its lifecycle in Secrets Manager.

To encrypt the secret, you can specify a customer managed key or use the default KMS key that is provided by Secrets Manager. Use the `--master-user-secret-kms-key-id` option to specify a customer managed key. The AWS KMS key identifier is the key ARN, key ID, alias ARN, or alias name for the KMS key. To use a KMS key in a different AWS account, specify the key ARN or alias ARN. After RDS is managing the database credentials for a DB cluster, you can't change the KMS key that is used to encrypt the secret.

You can choose other settings to meet your requirements.

For more information about the available settings when you are creating a Multi-AZ DB cluster, see [Settings for creating Multi-AZ DB clusters](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/create-multi-az-db-cluster.html#create-multi-az-db-cluster-settings). For more information about the available settings when you are modifying a Multi-AZ DB cluster, see [Settings for modifying Multi-AZ DB clusters](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/modify-multi-az-db-cluster.html#modify-multi-az-db-cluster-settings).

This example creates a Multi-AZ DB cluster and specifies that RDS manages the password in Secrets Manager. The secret is encrypted using the KMS key that is provided by Secrets Manager.

###### Example

For Linux, macOS, or Unix:

```
aws rds create-db-cluster \
   --db-cluster-identifier mysql-multi-az-db-cluster \
   --engine mysql \
   --engine-version 8.0.39  \
   --backup-retention-period 1  \
   --allocated-storage 4000 \
   --storage-type io1 \
   --iops 10000 \
   --db-cluster-instance-class db.r6gd.xlarge \
   --master-username testUser \
   --manage-master-user-password
```

For Windows:

```
aws rds create-db-cluster ^
   --db-cluster-identifier mysql-multi-az-db-cluster ^
   --engine mysql ^
   --engine-version 8.0.39 ^
   --backup-retention-period 1 ^
   --allocated-storage 4000 ^
   --storage-type io1 ^
   --iops 10000 ^
   --db-cluster-instance-class db.r6gd.xlarge ^
   --master-username testUser ^
   --manage-master-user-password
```

To specify that RDS manages the master user password in Secrets Manager, set the `ManageMasterUserPassword` parameter to `true` in one of the following operations:

- [CreateDBCluster](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_CreateDBCluster.html)
- [ModifyDBCluster](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_ModifyDBCluster.html)

When you set the `ManageMasterUserPassword` parameter to `true` in one of these operations, RDS generates the master user password and manages it throughout its lifecycle in Secrets Manager.

To encrypt the secret, you can specify a customer managed key or use the default KMS key that is provided by Secrets Manager. Use the `MasterUserSecretKmsKeyId` parameter to specify a customer managed key. The AWS KMS key identifier is the key ARN, key ID, alias ARN, or alias name for the KMS key. To use a KMS key in a different AWS account, specify the key ARN or alias ARN. After RDS is managing the database credentials for a DB cluster, you can't change the KMS key that is used to encrypt the secret.

## RDS API

To specify that RDS manages the master user password in Secrets Manager, set the `ManageMasterUserPassword` parameter to `true` in one of the following operations:

- [CreateDBCluster](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_CreateDBCluster.html)
- [ModifyDBCluster](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_ModifyDBCluster.html)

When you set the `ManageMasterUserPassword` parameter to `true` in one of these operations, RDS generates the master user password and manages it throughout its lifecycle in Secrets Manager.

To encrypt the secret, you can specify a customer managed key or use the default KMS key that is provided by Secrets Manager. Use the `MasterUserSecretKmsKeyId` parameter to specify a customer managed key. The AWS KMS key identifier is the key ARN, key ID, alias ARN, or alias name for the KMS key. To use a KMS key in a different AWS account, specify the key ARN or alias ARN. After RDS is managing the database credentials for a DB cluster, you can't change the KMS key that is used to encrypt the secret.

## Rotating the master user password secret for a DB instance

When RDS rotates a master user password secret, Secrets Manager generates a new secret version for the existing secret. The new version of the secret contains the new master user password. Amazon RDS changes the master user password for the DB instance to match the password for the new secret version.

You can rotate a secret immediately instead of waiting for a scheduled rotation. To rotate a master user password secret in Secrets Manager, modify the DB instance. For information about modifying a DB instance, see [Modifying an Amazon RDS DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.DBInstance.Modifying.html).

You can rotate a master user password secret immediately with the RDS console, the AWS CLI, or the RDS API. The new password is always 28 characters long and contains at least one upper and lowercase character, one number, and one punctuation.

To rotate a master user password secret using the RDS console, modify the DB instance and select **Rotate secret immediately** in **Settings**.

![Rotate a master user password secret immediately.](https://docs.aws.amazon.com/images/AmazonRDS/latest/UserGuide/images/secrets-manager-integration-rotate.png)

Follow the instructions for modifying a DB instance with the RDS console in [Modifying an Amazon RDS DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.DBInstance.Modifying.html). You must choose **Apply immediately** on the confirmation page.

## Console

To rotate a master user password secret using the RDS console, modify the DB instance and select **Rotate secret immediately** in **Settings**.

![Rotate a master user password secret immediately.](https://docs.aws.amazon.com/images/AmazonRDS/latest/UserGuide/images/secrets-manager-integration-rotate.png)

Follow the instructions for modifying a DB instance with the RDS console in [Modifying an Amazon RDS DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.DBInstance.Modifying.html). You must choose **Apply immediately** on the confirmation page.

To rotate a master user password secret using the AWS CLI, use the [modify-db-instance](https://docs.aws.amazon.com/cli/latest/reference/rds/modify-db-instance.html) command and specify the `--rotate-master-user-password` option. You must specify the `--apply-immediately` option when you rotate the master password.

This example rotates a master user password secret.

###### Example

For Linux, macOS, or Unix:

```
aws rds modify-db-instance \
    --db-instance-identifier mydbinstance \
    --rotate-master-user-password \
    --apply-immediately
```

For Windows:

```
aws rds modify-db-instance ^
    --db-instance-identifier mydbinstance ^
    --rotate-master-user-password ^
    --apply-immediately
```

## AWS CLI

To rotate a master user password secret using the AWS CLI, use the [modify-db-instance](https://docs.aws.amazon.com/cli/latest/reference/rds/modify-db-instance.html) command and specify the `--rotate-master-user-password` option. You must specify the `--apply-immediately` option when you rotate the master password.

This example rotates a master user password secret.

###### Example

For Linux, macOS, or Unix:

```
aws rds modify-db-instance \
    --db-instance-identifier mydbinstance \
    --rotate-master-user-password \
    --apply-immediately
```

For Windows:

```
aws rds modify-db-instance ^
    --db-instance-identifier mydbinstance ^
    --rotate-master-user-password ^
    --apply-immediately
```

You can rotate a master user password secret using the [ModifyDBInstance](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_ModifyDBInstance.html) operation and setting the `RotateMasterUserPassword` parameter to `true`. You must set the `ApplyImmediately` parameter to `true` when you rotate the master password.

## RDS API

You can rotate a master user password secret using the [ModifyDBInstance](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_ModifyDBInstance.html) operation and setting the `RotateMasterUserPassword` parameter to `true`. You must set the `ApplyImmediately` parameter to `true` when you rotate the master password.

## Rotating the master user password secret for a Multi-AZDB cluster

When RDS rotates a master user password secret, Secrets Manager generates a new secret version for the existing secret. The new version of the secret contains the new master user password. Amazon RDS changes the master user password for the Multi-AZ DB cluster to match the password for the new secret version.

You can rotate a secret immediately instead of waiting for a scheduled rotation. To rotate a master user password secret in Secrets Manager, modify the Multi-AZ DB cluster. For information about modifying a Multi-AZ DB cluster, see [Modifying a Multi-AZ DB cluster for Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/modify-multi-az-db-cluster.html).

You can rotate a master user password secret immediately with the RDS console, the AWS CLI, or the RDS API. The new password is always 28 characters long and contains atleast one upper and lowercase character, one number, and one punctuation.

To rotate a master user password secret using the RDS console, modify the Multi-AZ DB cluster and select **Rotate secret immediately** in **Settings**.

![Rotate a master user password secret immediately.](https://docs.aws.amazon.com/images/AmazonRDS/latest/UserGuide/images/secrets-manager-integration-rotate-taz-cluster.png)

Follow the instructions for modifying a Multi-AZ DB cluster with the RDS console in [Modifying a Multi-AZ DB cluster for Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/modify-multi-az-db-cluster.html). You must choose **Apply immediately** on the confirmation page.

## Console

To rotate a master user password secret using the RDS console, modify the Multi-AZ DB cluster and select **Rotate secret immediately** in **Settings**.

![Rotate a master user password secret immediately.](https://docs.aws.amazon.com/images/AmazonRDS/latest/UserGuide/images/secrets-manager-integration-rotate-taz-cluster.png)

Follow the instructions for modifying a Multi-AZ DB cluster with the RDS console in [Modifying a Multi-AZ DB cluster for Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/modify-multi-az-db-cluster.html). You must choose **Apply immediately** on the confirmation page.

To rotate a master user password secret using the AWS CLI, use the [modify-db-cluster](https://docs.aws.amazon.com/cli/latest/reference/rds/modify-db-cluster.html) command and specify the `--rotate-master-user-password` option. You must specify the `--apply-immediately` option when you rotate the master password.

This example rotates a master user password secret.

###### Example

For Linux, macOS, or Unix:

```
aws rds modify-db-cluster \
    --db-cluster-identifier mydbcluster \
    --rotate-master-user-password \
    --apply-immediately
```

For Windows:

```
aws rds modify-db-cluster ^
    --db-cluster-identifier mydbcluster ^
    --rotate-master-user-password ^
    --apply-immediately
```

## AWS CLI

To rotate a master user password secret using the AWS CLI, use the [modify-db-cluster](https://docs.aws.amazon.com/cli/latest/reference/rds/modify-db-cluster.html) command and specify the `--rotate-master-user-password` option. You must specify the `--apply-immediately` option when you rotate the master password.

This example rotates a master user password secret.

###### Example

For Linux, macOS, or Unix:

```
aws rds modify-db-cluster \
    --db-cluster-identifier mydbcluster \
    --rotate-master-user-password \
    --apply-immediately
```

For Windows:

```
aws rds modify-db-cluster ^
    --db-cluster-identifier mydbcluster ^
    --rotate-master-user-password ^
    --apply-immediately
```

You can rotate a master user password secret using the [ModifyDBCluster](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_ModifyDBCluster.html) operation and setting the `RotateMasterUserPassword` parameter to `true`. You must set the `ApplyImmediately` parameter to `true` when you rotate the master password.

## RDS API

You can rotate a master user password secret using the [ModifyDBCluster](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_ModifyDBCluster.html) operation and setting the `RotateMasterUserPassword` parameter to `true`. You must set the `ApplyImmediately` parameter to `true` when you rotate the master password.

## Viewing the details about a secret for a DB instance

You can retrieve your secrets using the console ([https://console.aws.amazon.com/secretsmanager/](https://console.aws.amazon.com/secretsmanager/)) or the AWS CLI ([get-secret-value](https://docs.aws.amazon.com/cli/latest/reference/secretsmanager/get-secret-value.html) Secrets Manager command).

You can find the Amazon Resource Name (ARN) of a secret managed by RDS in Secrets Manager with the RDS console, the AWS CLI, or the RDS API.

###### To view the details about a secret managed by RDS in Secrets Manager

1. Sign in to the AWS Management Console and open the Amazon RDS console at [https://console.aws.amazon.com/rds/](https://console.aws.amazon.com/rds/).
2. In the navigation pane, choose **Databases**.
3. Choose the name of the DB instance to show its details.
4. Choose the **Configuration** tab.
	In **Master Credentials ARN**, you can view the secret ARN.
	![View the details about a secret managed by RDS in Secrets Manager.](https://docs.aws.amazon.com/images/AmazonRDS/latest/UserGuide/images/secrets-manager-integration-view-instance.png)
	You can follow the **Manage in Secrets Manager** link to view and manage the secret in the Secrets Manager console.

## Console

###### To view the details about a secret managed by RDS in Secrets Manager

1. Sign in to the AWS Management Console and open the Amazon RDS console at [https://console.aws.amazon.com/rds/](https://console.aws.amazon.com/rds/).
2. In the navigation pane, choose **Databases**.
3. Choose the name of the DB instance to show its details.
4. Choose the **Configuration** tab.
	In **Master Credentials ARN**, you can view the secret ARN.
	![View the details about a secret managed by RDS in Secrets Manager.](https://docs.aws.amazon.com/images/AmazonRDS/latest/UserGuide/images/secrets-manager-integration-view-instance.png)
	You can follow the **Manage in Secrets Manager** link to view and manage the secret in the Secrets Manager console.

You can use the [describe-db-instances](https://docs.aws.amazon.com/cli/latest/reference/rds/describe-db-instances.html) RDS CLI command to find the following information about a secret managed by RDS in Secrets Manager:

- `SecretArn` – The ARN of the secret
- `SecretStatus` – The status of the secret
	The possible status values include the following:
	- `creating` – The secret is being created.
	- `active` – The secret is available for normal use and rotation.
	- `rotating` – The secret is being rotated.
	- `impaired` – The secret can be used to access database credentials, but it can't be rotated. A secret might have this status if, for example, permissions are changed so that RDS can no longer access the secret or the KMS key for the secret.
		When a secret has this status, you can correct the condition that caused the status. If you correct the condition that caused status, the status remains `impaired` until the next rotation. Alternatively, you can modify the DB instance to turn off automatic management of database credentials, and then modify the DB instance again to turn on automatic management of database credentials. To modify the DB instance, use the `--manage-master-user-password` option in the [modify-db-instance](https://docs.aws.amazon.com/cli/latest/reference/rds/modify-db-instance.html) command.
- `KmsKeyId` – The ARN of the KMS key that is used to encrypt the secret

Specify the `--db-instance-identifier` option to show output for a specific DB instance. This example shows the output for a secret that is used by a DB instance.

###### Example

```
aws rds describe-db-instances --db-instance-identifier mydbinstance
```

Following is sample output for a secret:

```bash
"MasterUserSecret": {
                "SecretArn": "arn:aws:secretsmanager:eu-west-1:123456789012:secret:rds!db-033d7456-2c96-450d-9d48-f5de3025e51c-xmJRDx",
                "SecretStatus": "active",
                "KmsKeyId": "arn:aws:kms:eu-west-1:123456789012:key/0987dcba-09fe-87dc-65ba-ab0987654321"
            }
```

When you have the secret ARN, you can view details about the secret using the [get-secret-value](https://docs.aws.amazon.com/cli/latest/reference/secretsmanager/get-secret-value.html) Secrets Manager CLI command.

This example shows the details for the secret in the previous sample output.

###### Example

For Linux, macOS, or Unix:

```
aws secretsmanager get-secret-value \
    --secret-id 'arn:aws:secretsmanager:eu-west-1:123456789012:secret:rds!db-033d7456-2c96-450d-9d48-f5de3025e51c-xmJRDx'
```

For Windows:

```
aws secretsmanager get-secret-value ^
    --secret-id 'arn:aws:secretsmanager:eu-west-1:123456789012:secret:rds!db-033d7456-2c96-450d-9d48-f5de3025e51c-xmJRDx'
```

## AWS CLI

You can use the [describe-db-instances](https://docs.aws.amazon.com/cli/latest/reference/rds/describe-db-instances.html) RDS CLI command to find the following information about a secret managed by RDS in Secrets Manager:

- `SecretArn` – The ARN of the secret
- `SecretStatus` – The status of the secret
	The possible status values include the following:
	- `creating` – The secret is being created.
	- `active` – The secret is available for normal use and rotation.
	- `rotating` – The secret is being rotated.
	- `impaired` – The secret can be used to access database credentials, but it can't be rotated. A secret might have this status if, for example, permissions are changed so that RDS can no longer access the secret or the KMS key for the secret.
		When a secret has this status, you can correct the condition that caused the status. If you correct the condition that caused status, the status remains `impaired` until the next rotation. Alternatively, you can modify the DB instance to turn off automatic management of database credentials, and then modify the DB instance again to turn on automatic management of database credentials. To modify the DB instance, use the `--manage-master-user-password` option in the [modify-db-instance](https://docs.aws.amazon.com/cli/latest/reference/rds/modify-db-instance.html) command.
- `KmsKeyId` – The ARN of the KMS key that is used to encrypt the secret

Specify the `--db-instance-identifier` option to show output for a specific DB instance. This example shows the output for a secret that is used by a DB instance.

###### Example

```
aws rds describe-db-instances --db-instance-identifier mydbinstance
```

Following is sample output for a secret:

```bash
"MasterUserSecret": {
                "SecretArn": "arn:aws:secretsmanager:eu-west-1:123456789012:secret:rds!db-033d7456-2c96-450d-9d48-f5de3025e51c-xmJRDx",
                "SecretStatus": "active",
                "KmsKeyId": "arn:aws:kms:eu-west-1:123456789012:key/0987dcba-09fe-87dc-65ba-ab0987654321"
            }
```

When you have the secret ARN, you can view details about the secret using the [get-secret-value](https://docs.aws.amazon.com/cli/latest/reference/secretsmanager/get-secret-value.html) Secrets Manager CLI command.

This example shows the details for the secret in the previous sample output.

###### Example

For Linux, macOS, or Unix:

```
aws secretsmanager get-secret-value \
    --secret-id 'arn:aws:secretsmanager:eu-west-1:123456789012:secret:rds!db-033d7456-2c96-450d-9d48-f5de3025e51c-xmJRDx'
```

For Windows:

```
aws secretsmanager get-secret-value ^
    --secret-id 'arn:aws:secretsmanager:eu-west-1:123456789012:secret:rds!db-033d7456-2c96-450d-9d48-f5de3025e51c-xmJRDx'
```

You can view the ARN, status, and KMS key for a secret managed by RDS in Secrets Manager by using the [DescribeDBInstances](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_DescribeDBInstances.html) operation and setting the `DBInstanceIdentifier` parameter to a DB instance identifier. Details about the secret are included in the output.

When you have the secret ARN, you can view details about the secret using the [GetSecretValue](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_GetSecretValue.html) Secrets Manager operation.

## RDS API

You can view the ARN, status, and KMS key for a secret managed by RDS in Secrets Manager by using the [DescribeDBInstances](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_DescribeDBInstances.html) operation and setting the `DBInstanceIdentifier` parameter to a DB instance identifier. Details about the secret are included in the output.

When you have the secret ARN, you can view details about the secret using the [GetSecretValue](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_GetSecretValue.html) Secrets Manager operation.

## Viewing the details about a secret for a Multi-AZDB cluster

You can retrieve your secrets using the console ([https://console.aws.amazon.com/secretsmanager/](https://console.aws.amazon.com/secretsmanager/)) or the AWS CLI ([get-secret-value](https://docs.aws.amazon.com/cli/latest/reference/secretsmanager/get-secret-value.html) Secrets Manager command).

You can find the Amazon Resource Name (ARN) of a secret managed by RDS in Secrets Manager with the RDS console, the AWS CLI, or the RDS API.

###### To view the details about a secret managed by RDS in Secrets Manager

1. Sign in to the AWS Management Console and open the Amazon RDS console at [https://console.aws.amazon.com/rds/](https://console.aws.amazon.com/rds/).
2. In the navigation pane, choose **Databases**.
3. Choose the name of the Multi-AZ DB cluster to show its details.
4. Choose the **Configuration** tab.
	In **Master Credentials ARN**, you can view the secret ARN.
	![View the details about a secret managed by RDS in Secrets Manager.](https://docs.aws.amazon.com/images/AmazonRDS/latest/UserGuide/images/secrets-manager-integration-view-taz-cluster.png)
	You can follow the **Manage in Secrets Manager** link to view and manage the secret in the Secrets Manager console.

## Console

###### To view the details about a secret managed by RDS in Secrets Manager

1. Sign in to the AWS Management Console and open the Amazon RDS console at [https://console.aws.amazon.com/rds/](https://console.aws.amazon.com/rds/).
2. In the navigation pane, choose **Databases**.
3. Choose the name of the Multi-AZ DB cluster to show its details.
4. Choose the **Configuration** tab.
	In **Master Credentials ARN**, you can view the secret ARN.
	![View the details about a secret managed by RDS in Secrets Manager.](https://docs.aws.amazon.com/images/AmazonRDS/latest/UserGuide/images/secrets-manager-integration-view-taz-cluster.png)
	You can follow the **Manage in Secrets Manager** link to view and manage the secret in the Secrets Manager console.

You can use the RDS AWS CLI [describe-db-clusters](https://docs.aws.amazon.com/cli/latest/reference/rds/describe-db-clusters.html) command to find the following information about a secret managed by RDS in Secrets Manager:

- `SecretArn` – The ARN of the secret
- `SecretStatus` – The status of the secret
	The possible status values include the following:
	- `creating` – The secret is being created.
	- `active` – The secret is available for normal use and rotation.
	- `rotating` – The secret is being rotated.
	- `impaired` – The secret can be used to access database credentials, but it can't be rotated. A secret might have this status if, for example, permissions are changed so that RDS can no longer access the secret or the KMS key for the secret.
		When a secret has this status, you can correct the condition that caused the status. If you correct the condition that caused status, the status remains `impaired` until the next rotation. Alternatively, you can modify the DB cluster to turn off automatic management of database credentials, and then modify the DB cluster again to turn on automatic management of database credentials. To modify the DB cluster, use the `--manage-master-user-password` option in the [modify-db-cluster](https://docs.aws.amazon.com/cli/latest/reference/rds/modify-db-cluster.html) command.
- `KmsKeyId` – The ARN of the KMS key that is used to encrypt the secret

Specify the `--db-cluster-identifier` option to show output for a specific DB cluster. This example shows the output for a secret that is used by a DB cluster.

###### Example

```
aws rds describe-db-clusters --db-cluster-identifier mydbcluster
```

The following sample shows the output for a secret:

```bash
"MasterUserSecret": {
                "SecretArn": "arn:aws:secretsmanager:eu-west-1:123456789012:secret:rds!cluster-033d7456-2c96-450d-9d48-f5de3025e51c-xmJRDx",
                "SecretStatus": "active",
                "KmsKeyId": "arn:aws:kms:eu-west-1:123456789012:key/0987dcba-09fe-87dc-65ba-ab0987654321"
            }
```

When you have the secret ARN, you can view details about the secret using the [get-secret-value](https://docs.aws.amazon.com/cli/latest/reference/secretsmanager/get-secret-value.html) Secrets Manager CLI command.

This example shows the details for the secret in the previous sample output.

###### Example

For Linux, macOS, or Unix:

```
aws secretsmanager get-secret-value \
    --secret-id 'arn:aws:secretsmanager:eu-west-1:123456789012:secret:rds!cluster-033d7456-2c96-450d-9d48-f5de3025e51c-xmJRDx'
```

For Windows:

```
aws secretsmanager get-secret-value ^
    --secret-id 'arn:aws:secretsmanager:eu-west-1:123456789012:secret:rds!cluster-033d7456-2c96-450d-9d48-f5de3025e51c-xmJRDx'
```

## AWS CLI

You can use the RDS AWS CLI [describe-db-clusters](https://docs.aws.amazon.com/cli/latest/reference/rds/describe-db-clusters.html) command to find the following information about a secret managed by RDS in Secrets Manager:

- `SecretArn` – The ARN of the secret
- `SecretStatus` – The status of the secret
	The possible status values include the following:
	- `creating` – The secret is being created.
	- `active` – The secret is available for normal use and rotation.
	- `rotating` – The secret is being rotated.
	- `impaired` – The secret can be used to access database credentials, but it can't be rotated. A secret might have this status if, for example, permissions are changed so that RDS can no longer access the secret or the KMS key for the secret.
		When a secret has this status, you can correct the condition that caused the status. If you correct the condition that caused status, the status remains `impaired` until the next rotation. Alternatively, you can modify the DB cluster to turn off automatic management of database credentials, and then modify the DB cluster again to turn on automatic management of database credentials. To modify the DB cluster, use the `--manage-master-user-password` option in the [modify-db-cluster](https://docs.aws.amazon.com/cli/latest/reference/rds/modify-db-cluster.html) command.
- `KmsKeyId` – The ARN of the KMS key that is used to encrypt the secret

Specify the `--db-cluster-identifier` option to show output for a specific DB cluster. This example shows the output for a secret that is used by a DB cluster.

###### Example

```
aws rds describe-db-clusters --db-cluster-identifier mydbcluster
```

The following sample shows the output for a secret:

```bash
"MasterUserSecret": {
                "SecretArn": "arn:aws:secretsmanager:eu-west-1:123456789012:secret:rds!cluster-033d7456-2c96-450d-9d48-f5de3025e51c-xmJRDx",
                "SecretStatus": "active",
                "KmsKeyId": "arn:aws:kms:eu-west-1:123456789012:key/0987dcba-09fe-87dc-65ba-ab0987654321"
            }
```

When you have the secret ARN, you can view details about the secret using the [get-secret-value](https://docs.aws.amazon.com/cli/latest/reference/secretsmanager/get-secret-value.html) Secrets Manager CLI command.

This example shows the details for the secret in the previous sample output.

###### Example

For Linux, macOS, or Unix:

```
aws secretsmanager get-secret-value \
    --secret-id 'arn:aws:secretsmanager:eu-west-1:123456789012:secret:rds!cluster-033d7456-2c96-450d-9d48-f5de3025e51c-xmJRDx'
```

For Windows:

```
aws secretsmanager get-secret-value ^
    --secret-id 'arn:aws:secretsmanager:eu-west-1:123456789012:secret:rds!cluster-033d7456-2c96-450d-9d48-f5de3025e51c-xmJRDx'
```

You can view the ARN, status, and KMS key for a secret managed by RDS in Secrets Manager using the [DescribeDBClusters](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_DescribeDBClusters.html) RDS operation and setting the `DBClusterIdentifier` parameter to a DB cluster identifier. Details about the secret are included in the output.

When you have the secret ARN, you can view details about the secret using the [GetSecretValue](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_GetSecretValue.html) Secrets Manager operation.

## RDS API

You can view the ARN, status, and KMS key for a secret managed by RDS in Secrets Manager using the [DescribeDBClusters](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_DescribeDBClusters.html) RDS operation and setting the `DBClusterIdentifier` parameter to a DB cluster identifier. Details about the secret are included in the output.

When you have the secret ARN, you can view details about the secret using the [GetSecretValue](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_GetSecretValue.html) Secrets Manager operation.

## Viewing the details about a secret for a tenant database

You can retrieve your secrets using the console ([https://console.aws.amazon.com/secretsmanager/](https://console.aws.amazon.com/secretsmanager/)) or the AWS CLI ([get-secret-value](https://docs.aws.amazon.com/cli/latest/reference/secretsmanager/get-secret-value.html) Secrets Manager command).

You can find the Amazon Resource Name (ARN) of a secret managed by Amazon RDS in AWS Secrets Manager with the Amazon RDS console, the AWS CLI, or the Amazon RDS API.

###### To view the details about a secret managed by Amazon RDS in AWS Secrets Manager for a tenant database

1. Sign in to the AWS Management Console and open the Amazon RDS console at [https://console.aws.amazon.com/rds/](https://console.aws.amazon.com/rds/).
2. In the navigation pane, choose **Databases**.
3. Choose the name of the DB instance that contains the tenant database to show its details.
4. Choose the **Configuration** tab.
	In the **Tenant databases** section, find the tenant database and view its **Master Credentials ARN**.
	You can follow the **Manage in Secrets Manager** link to view and manage the secret in the Secrets Manager console.

## Console

###### To view the details about a secret managed by Amazon RDS in AWS Secrets Manager for a tenant database

1. Sign in to the AWS Management Console and open the Amazon RDS console at [https://console.aws.amazon.com/rds/](https://console.aws.amazon.com/rds/).
2. In the navigation pane, choose **Databases**.
3. Choose the name of the DB instance that contains the tenant database to show its details.
4. Choose the **Configuration** tab.
	In the **Tenant databases** section, find the tenant database and view its **Master Credentials ARN**.
	You can follow the **Manage in Secrets Manager** link to view and manage the secret in the Secrets Manager console.

You can use the [describe-tenant-databases](https://docs.aws.amazon.com/cli/latest/reference/rds/describe-tenant-databases.html) Amazon RDS AWS CLI command to find the following information about a secret managed by Amazon RDS in AWS Secrets Manager for a tenant database:

- `SecretArn` – The ARN of the secret
- `SecretStatus` – The status of the secret
	The possible status values include the following:
	- `creating` – The secret is being created.
	- `active` – The secret is available for normal use and rotation.
	- `rotating` – The secret is being rotated.
	- `impaired` – The secret can be used to access database credentials, but it can't be rotated. A secret might have this status if, for example, permissions are changed so that Amazon RDS can no longer access the secret or the KMS key for the secret.
		When a secret has this status, you can correct the condition that caused the status. If you correct the condition that caused status, the status remains `impaired` until the next rotation. Alternatively, you can modify the tenant database to turn off automatic management of database credentials, and then modify the tenant database again to turn on automatic management of database credentials. To modify the tenant database, use the `--manage-master-user-password` option in the [modify-tenant-database](https://docs.aws.amazon.com/cli/latest/reference/rds/modify-tenant-database.html) command.
- `KmsKeyId` – The ARN of the KMS key that is used to encrypt the secret

Specify the `--db-instance-identifier` option to show output for tenant databases in a specific DB instance. You can also specify the `--tenant-db-name` option to show output for a specific tenant database. This example shows the output for a secret that is used by a tenant database.

###### Example

```sql
aws rds describe-tenant-databases \
    --db-instance-identifier database-3 \
    --query "TenantDatabases[0].MasterUserSecret"
```

Following is sample output for a secret:

```
{
    "SecretArn": "arn:aws:secretsmanager:us-east-2:123456789012:secret:rds!db-ABC123",
    "SecretStatus": "active",
    "KmsKeyId": "arn:aws:kms:us-east-2:123456789012:key/aa11bb22-####-####-####-fedcba123456"
}
```

When you have the secret ARN, you can view details about the secret using the [get-secret-value](https://docs.aws.amazon.com/cli/latest/reference/secretsmanager/get-secret-value.html) Secrets Manager AWS CLI command.

This example shows the details for the secret in the previous sample output.

###### Example

For Linux, macOS, or Unix:

```
aws secretsmanager get-secret-value \
    --secret-id 'arn:aws:secretsmanager:us-east-2:123456789012:secret:rds!db-ABC123'
```

For Windows:

```
aws secretsmanager get-secret-value ^
    --secret-id 'arn:aws:secretsmanager:us-east-2:123456789012:secret:rds!db-ABC123'
```

## AWS CLI

You can use the [describe-tenant-databases](https://docs.aws.amazon.com/cli/latest/reference/rds/describe-tenant-databases.html) Amazon RDS AWS CLI command to find the following information about a secret managed by Amazon RDS in AWS Secrets Manager for a tenant database:

- `SecretArn` – The ARN of the secret
- `SecretStatus` – The status of the secret
	The possible status values include the following:
	- `creating` – The secret is being created.
	- `active` – The secret is available for normal use and rotation.
	- `rotating` – The secret is being rotated.
	- `impaired` – The secret can be used to access database credentials, but it can't be rotated. A secret might have this status if, for example, permissions are changed so that Amazon RDS can no longer access the secret or the KMS key for the secret.
		When a secret has this status, you can correct the condition that caused the status. If you correct the condition that caused status, the status remains `impaired` until the next rotation. Alternatively, you can modify the tenant database to turn off automatic management of database credentials, and then modify the tenant database again to turn on automatic management of database credentials. To modify the tenant database, use the `--manage-master-user-password` option in the [modify-tenant-database](https://docs.aws.amazon.com/cli/latest/reference/rds/modify-tenant-database.html) command.
- `KmsKeyId` – The ARN of the KMS key that is used to encrypt the secret

Specify the `--db-instance-identifier` option to show output for tenant databases in a specific DB instance. You can also specify the `--tenant-db-name` option to show output for a specific tenant database. This example shows the output for a secret that is used by a tenant database.

###### Example

```sql
aws rds describe-tenant-databases \
    --db-instance-identifier database-3 \
    --query "TenantDatabases[0].MasterUserSecret"
```

Following is sample output for a secret:

```
{
    "SecretArn": "arn:aws:secretsmanager:us-east-2:123456789012:secret:rds!db-ABC123",
    "SecretStatus": "active",
    "KmsKeyId": "arn:aws:kms:us-east-2:123456789012:key/aa11bb22-####-####-####-fedcba123456"
}
```

When you have the secret ARN, you can view details about the secret using the [get-secret-value](https://docs.aws.amazon.com/cli/latest/reference/secretsmanager/get-secret-value.html) Secrets Manager AWS CLI command.

This example shows the details for the secret in the previous sample output.

###### Example

For Linux, macOS, or Unix:

```
aws secretsmanager get-secret-value \
    --secret-id 'arn:aws:secretsmanager:us-east-2:123456789012:secret:rds!db-ABC123'
```

For Windows:

```
aws secretsmanager get-secret-value ^
    --secret-id 'arn:aws:secretsmanager:us-east-2:123456789012:secret:rds!db-ABC123'
```

You can view the ARN, status, and KMS key for a secret managed by Amazon RDS in AWS Secrets Manager by using the [DescribeTenantDatabases](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_DescribeTenantDatabases.html) operation and setting the `DBInstanceIdentifier` parameter to a DB instance identifier. You can also set the `TenantDBName` parameter to a specific tenant database name. Details about the secret are included in the output.

When you have the secret ARN, you can view details about the secret using the [GetSecretValue](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_GetSecretValue.html) Secrets Manager operation.

## Amazon RDS API

You can view the ARN, status, and KMS key for a secret managed by Amazon RDS in AWS Secrets Manager by using the [DescribeTenantDatabases](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_DescribeTenantDatabases.html) operation and setting the `DBInstanceIdentifier` parameter to a DB instance identifier. You can also set the `TenantDBName` parameter to a specific tenant database name. Details about the secret are included in the output.

When you have the secret ARN, you can view details about the secret using the [GetSecretValue](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_GetSecretValue.html) Secrets Manager operation.

## Region and version availability

Feature availability and support varies across specific versions of each database engine and across AWS Regions. For more information about version and Region availability with Secrets Manager integration with Amazon RDS, see [Supported Regions and DB engines for the Secrets Manager integration with Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RDS_Fea_Regions_DB-eng.Feature.SecretsManager.html).