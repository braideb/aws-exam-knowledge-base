---
title: "WKLD.03 Use ephemeral secrets or a secrets-management service - AWS Prescriptive Guidance"
source: "https://docs.aws.amazon.com/prescriptive-guidance/latest/aws-startup-security-baseline/wkld-03.html"
author:
published:
created: 2026-10-02
description: "Manage application secrets using AWS Systems Manager Parameter Store and AWS Secrets Manager to securely store credentials and access tokens."
tags:
  - "clippings"
---
WKLD.03 Use ephemeral secrets or a secrets-management service - AWS Prescriptive Guidance

[Cost considerations](#wkld-03-costs)

Application secrets include credentials, such as key pairs, access tokens, digital certificates, and sign-in credentials. The application uses these secrets to gain access to other services it depends upon, such as a database. To help protect these secrets, we recommend they are either *ephemeral* (generated at the time of request and short-lived, such as with IAM roles) or retrieved from a secrets management service. This reduces the risk of secrets being accidentally stored in static configuration files, environment variables, or source code. Centralizing secrets management also makes it straightforward to move application code between development and production environments without reconfiguring credentials.

For a secrets management, use a combination of Parameter Store (a capability of AWS Systems Manager) and AWS Secrets Manager:

- Use Parameter Store to manage secrets and other parameters that are individual key-value pairs, string-based, short in overall length, and accessed frequently. You use an AWS Key Management Service (AWS KMS) key to encrypt the secret. There is no charge to store parameters in the standard tier of Parameter Store. For more information about parameter tiers, see [Managing parameter tiers](https://docs.aws.amazon.com/systems-manager/latest/userguide/parameter-store-advanced-parameters.html) in the Systems Manager documentation.
- Use Secrets Manager to store secrets that are in document form (such as multiple, related key-value pairs), that are larger than 4 KB (such as digital certificates), or that would benefit from automated rotation.

You can use Parameter Store APIs to retrieve secrets stored in Secrets Manager. With this approach, you can standardize the code in your application when using a combination of both services.

**To manage secrets in Parameter Store**

1. Create a symmetric AWS KMS key. For more information, see [Create a symmetric encryption KMS key](https://docs.aws.amazon.com/kms/latest/developerguide/create-symmetric-cmk.html) in the AWS KMS documentation.
2. Create a `SecureString` parameter. For more information, see [Create a SecureString parameter](https://docs.aws.amazon.com/systems-manager/latest/userguide/param-create-cli.html#param-create-cli-securestring) in the Systems Manager documentation. Secrets in Parameter Store use the `SecureString` data type.
3. In your application, retrieve a parameter from Parameter Store by using the AWS SDK for your programming language. For code examples, see [GetParameter](https://docs.aws.amazon.com/systems-manager/latest/APIReference/API_GetParameter.html) in the Systems Manager documentation.

**To manage secrets in Secrets Manager**

1. Create a secret. For more information, see [Create a secret](https://docs.aws.amazon.com/secretsmanager/latest/userguide/create_secret.html) in the Secrets Manager documentation.
2. Retrieve secrets from Secrets Manager in code. For more information, see [Get secrets from AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/retrieving-secrets.html) in the Secrets Manager documentation.

For information about improving the availability and latency of secret retrieval, see [Use AWS Secrets Manager client-side caching libraries to improve the availability and latency of using your secrets](https://aws.amazon.com/blogs/security/use-aws-secrets-manager-client-side-caching-libraries-to-improve-the-availability-and-latency-of-using-your-secrets/) on the AWS Security Blog. The client-side caching SDK reduces the number of API calls your application makes to Secrets Manager and can improve secret retrieval performance.

## Cost considerations

The cost of secrets management depends on which service you use and how your application accesses secrets:

- For Parameter Store, standard tier parameters are available at no additional charge for values up to 4 KB. The advanced tier applies additional charges for larger parameters or higher throughput.
- AWS Secrets Manager charges for each secret stored on a monthly basis and charges for each API call made to retrieve secrets. Using the Secrets Manager client-side caching SDK reduces the number of API calls your application makes to Secrets Manager, which can reduce costs.
- Encrypting secrets with an AWS managed KMS key is available at no additional charge. Customer-managed keys incur a monthly charge for each key and a charge for each API call.

For most early-stage startups, a cost effective starting point is to use Parameter Store with an AWS managed KMS key for frequently accessed secrets and use Secrets Manager for secrets that benefit from automated rotation.

For current pricing, see [AWS Systems Manager pricing](https://aws.amazon.com/systems-manager/pricing/) and [AWS Key Management Service pricing](https://aws.amazon.com/kms/pricing/).