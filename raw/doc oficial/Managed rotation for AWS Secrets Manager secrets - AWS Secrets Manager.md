---
title: "Managed rotation for AWS Secrets Manager secrets - AWS Secrets Manager"
source: "https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotate-secrets_managed.html"
author:
published:
created: 2026-10-02
description: "Learn how to set up automatic rotation for secrets that are managed by other services."
tags:
  - "clippings"
---
Managed rotation for AWS Secrets Manager secrets - AWS Secrets Manager

Some services offer *managed rotation*, where the service configures and manages rotation for you. With managed rotation, you don't use an AWS Lambda function to update the secret and the credentials in the database.

The following services offer managed rotation:

- **Amazon Aurora** offers managed rotation for master user credentials. For more information, see [Password management with Amazon Aurora and AWS Secrets Manager](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/rds-secrets-manager.html) in the *Amazon Aurora User Guide*.
- **Amazon ECS** Service Connect offers managed rotation for AWS Private Certificate Authority TLS certificates. For more information, see [TLS with Service Connect](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-connect-tls.html) in the *Amazon Elastic Container Service Developer Guide*.
- **Amazon RDS** offers managed rotation for master user credentials. For more information, see [Password management with Amazon RDS and AWS Secrets Manager](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-secrets-manager.html) in the *Amazon RDS User Guide*.
- **Amazon DocumentDB** offers managed rotation for master user credentials. For more information, see [Password management with Amazon DocumentDB and AWS Secrets Manager](https://docs.aws.amazon.com/documentdb/latest/developerguide/docdb-secrets-manager.html) in the *Amazon DocumentDB User Guide*.
- **Amazon Redshift** offers managed rotation for admin passwords. For more information, see [Managing Amazon Redshift admin passwords using AWS Secrets Manager](https://docs.aws.amazon.com/redshift/latest/mgmt/redshift-secrets-manager-integration.html) in the *Amazon Redshift Management Guide*.
- **managed external secrets** offers managed rotation for secrets held by Secrets Manager partners. For more information, see [Using AWS Secrets Manager managed external secrets to manage Third Party secrets](https://docs.aws.amazon.com/secretsmanager/latest/userguide/managed-external-secrets.html).

###### Tip

For all other types of secrets, see [Rotation by Lambda function](https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotate-secrets_lambda.html).

Rotation for managed secrets typically completes within one minute. During rotation, new connections that retrieve the secret may get the previous version of the credentials. In applications, we strongly recommend that you follow the best practice of using a database user created with the minimal privileges required for your application, rather than using the master user. For application users, for highest availability, you can use the [Alternating users rotation strategy](https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotation-strategy.html).

For secrets held by Secrets Manager partners,

###### To change the schedule for managed rotation

1. Open the managed secret in the Secrets Manager console. You can follow a link from the managing service, or [search for the secret](https://docs.aws.amazon.com/secretsmanager/latest/userguide/service-linked-secrets.html) in the Secrets Manager console.
2. Under **Rotation schedule**, enter your schedule in UTC time zone in either the **Schedule expression builder** or as a **Schedule expression**. Secrets Manager stores your schedule as a `rate()` or `cron()` expression. The rotation window automatically starts at midnight unless you specify a **Start time**. You can rotate a secret as often as every four hours. For more information, see [Rotation schedules](https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotate-secrets_schedule.html).
3. (Optional) For **Window duration**, choose the length of the window during which you want Secrets Manager to rotate your secret, for example `3h` for a three hour window. The window must not extend into the next rotation window. If you don't specify **Window duration**, for a rotation schedule in hours, the window automatically closes after one hour. For a rotation schedule in days, the window automatically closes at the end of the day.
4. Choose **Save**.

###### To change the schedule for managed rotation (AWS CLI)

- Call [`rotate-secret`](https://docs.aws.amazon.com/cli/latest/reference/secretsmanager/rotate-secret.html). The following example rotates the secret between 16:00 and 18:00 UTC on the 1st and 15th day of the month. For more information, see [Rotation schedules](https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotate-secrets_schedule.html).
	```
	aws secretsmanager rotate-secret \
	    --secret-id MySecret \
	    --rotation-rules \
	        "{\"ScheduleExpression\": \"cron(0 16 1,15 * ? *)\", \"Duration\": \"2h\"}"
	```