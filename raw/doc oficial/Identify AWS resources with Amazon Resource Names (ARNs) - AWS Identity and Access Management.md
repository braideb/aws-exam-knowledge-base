---
title: "Identify AWS resources with Amazon Resource Names (ARNs) - AWS Identity and Access Management"
source: "https://docs.aws.amazon.com/IAM/latest/UserGuide/reference-arns.html"
author:
published:
created: 2026-07-18
description: "Learn how Amazon Resource Names (ARNs) uniquely identify AWS resources for use in IAM policies, database tags, and API calls, including their syntax, partitions, services, regions, account IDs, resource types, paths, and wildcard usage."
tags:
  - "clippings"
---
Identify AWS resources with Amazon Resource Names (ARNs) - AWS Identity and Access Management

Amazon Resource Names (ARNs) uniquely identify AWS resources. We require an ARN when you need to specify a resource unambiguously across all of AWS, such as in IAM policies, Amazon Relational Database Service (Amazon RDS) tags, and API calls. While ARNs, like any identifying information, should be used and shared carefully, they are not considered secret, sensitive, or confidential information.

## ARN format

The following are the general formats for ARNs. The specific formats depend on the resource. To use an ARN, replace the `italicized` text with the resource-specific information. Be aware that the ARNs for some resources omit the Region, the account ID, or both the Region and the account ID.

```
arn:partition:service:region:account-id:resource-id
arn:partition:service:region:account-id:resource-type/resource-id
arn:partition:service:region:account-id:resource-type:resource-id
```

`partition`

The partition in which the resource is located. A *partition* is a group of AWS Regions. Each AWS account is scoped to one partition.

The following are the supported partitions:

- `aws` - AWS Regions
- `aws-cn` - China Regions
- `aws-us-gov` - AWS GovCloud (US) Regions

`service`

The service namespace that identifies the AWS product.

`region`

The Region code. For example, `us-east-2` for US East (Ohio). For the list of Region codes, see [Regional endpoints](https://docs.aws.amazon.com/general/latest/gr/rande.html#regional-endpoints) in the *AWS General Reference*.

`account-id`

The ID of the AWS account that owns the resource, without the hyphens. For example, `123456789012`.

`resource-type`

The resource type. For example, `vpc` for a virtual private cloud (VPC).

`resource-id`

The resource identifier. This is the name of the resource, the ID of the resource, or a [resource path](#arns-paths). Some resource identifiers include a parent resource (sub-resource-type/parent-resource/sub-resource) or a qualifier such as a version (resource-type:resource-name:qualifier).

###### Examples

IAM user

arn:aws:iam::`123456789012`:user/ `john`

SNS topic

arn:aws:sns:`us-east-1`:`123456789012`:`example-sns-topic-name`

VPC

arn:aws:ec2:`us-east-1`:`123456789012`:vpc/ `vpc-0e9801d129EXAMPLE`

## Look up the ARN format for a resource

The exact format of an ARN depends on the service and resource type. Some resource ARNs can include a path, a variable, or a wildcard. To look up the ARN format for a specific AWS resource, open the [Service Authorization Reference](https://docs.aws.amazon.com/service-authorization/latest/reference/), open the page for the service, and navigate to the resource types table.

## Paths in ARNs

Resource ARNs can include a path. For example, in Amazon S3, the resource identifier is an object name that can include forward slashes (`/`) to form a path. Similarly, IAM user names and group names can include paths. Only alphanumeric characters and the following characters are allowed in IAM paths: forward slash (`/`), plus (`+`), equals (`=`), comma (`,`), period (`.`), at (`@`), underscore (`_`), and hyphen (`-`).

### Using wildcards in paths

Paths can include a wildcard character, namely an asterisk (`*`). Some policy elements allow wildcards while others don't. You can use wildcards for the [Resource](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_resource.html) or [NotResource](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_notresource.html) elements but not for the [Principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html) or [NotPrincipal](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_notprincipal.html) elements. For more information, see [IAM JSON policy reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html).

You can specify `role/*` to mean all roles in the account 123456789012 as in the following example:

```
arn:aws:iam::123456789012:role/*
```

You can also end a resource name with a wildcard. For example, you can specify `service-*` to mean all roles starting with `service` and ending with different characters like `service-role1` or `service-test`:

```
arn:aws:iam::123456789012:role/service-*
```

The following example shows ARNs for objects in an Amazon S3 bucket in which the resource name includes a path. The ARN `arn:aws:s3:::amzn-s3-demo-bucket/*` is for all objects within that bucket, regardless of prefix. The ARN ``arn:aws:s3:::amzn-s3-demo-bucket/`Development`/*`` is for all objects created within the `/Development/` prefix.

You can also use the `?` wildcard character to specify one character in an ARN. For example, you could use the following ARN for all folders starting with four characters and ending in `-test` in the S3 bucket named amzn-s3-demo-bucket. Some folders that would match this include `1234-test`, `2024-test`, or `a100-test`.

```
arn:aws:s3:::amzn-s3-demo-bucket/????-test
```

You can also use wildcards in the different sections of an ARN, delimited by a colon “`:`”. In the following example, two wildcards are used to match all Amazon Q applications and resources within the applications in all regions for account 123456789012:

```
arn:aws:qbusiness:*:123456789012:*
```

Similarly, the following example matches all Amazon VPCs in all regions for account 123456789012:

```
arn:aws:ec2:*:123456789012:vpc/*
```

The following example matches all Amazon EBS volumes in all regions for account 123456789012:

```
arn:aws:ec2:*:123456789012:volume/*
```

###### Limitations on wildcard usage within ARNs

You cannot use a wildcard in the portion of the ARN that specifics the resource type. The following example ARN with a wildcard within the resource type is not valid:

```
arn:aws:lambda:us-east-2:123456789012:functi*:my-function <== not allowed
```

###### Note

When you specify an incomplete ARN (one with fewer than the standard six fields) in an identity-based policy, AWS automatically completes the ARN by adding wildcard characters (\*) to all missing fields. For example, specifying `arn:aws:sqs` is equivalent to `arn:aws:sqs:*:*:*`, which grants access to all Amazon Amazon SQS resources across all regions and accounts. However, session policies passed to AWS STS AssumeRole, AssumeRoleWithWebIdentity, and AssumeRoleWithSAML requests, do not support incomplete ARNs. Using an incomplete ARN in a session policy will result in a `MalformedPolicyDocumentException` error.

You also cannot use a wildcard in the prefix ARN, or have a wildcard in the partition section of an ARN.

```
arn:aws:redshift:us-east-1:123456789012:? <== not allowed
```