---
title: "Grants in AWS KMS - AWS Key Management Service"
source: "https://docs.aws.amazon.com/kms/latest/developerguide/grants.html"
author:
published:
created: 2026-07-19
description: "Use grants to allow access to AWS KMS keys in AWS Key Management Service (AWS KMS)."
tags:
  - "clippings"
---
Grants in AWS KMS - AWS Key Management Service

[Grant concepts](#grant-concepts)

A *grant* is a policy instrument that allows [AWS principals](https://docs.aws.amazon.com/IAM/latest/UserGuide/intro-structure.html#intro-structure-principal) or AWS [service principals](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#principal-services) to use KMS keys in cryptographic operations. It also can let them view a KMS key (`DescribeKey`) and create and manage grants. When authorizing access to a KMS key, grants are considered along with [key policies](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html) and [IAM policies](https://docs.aws.amazon.com/kms/latest/developerguide/iam-policies.html). Grants are often used for temporary permissions because you can create one, use its permissions, and delete it without changing your key policies or IAM policies.

Grants are commonly used by AWS services that integrate with AWS KMS to encrypt your data at rest. The service creates a grant on behalf of a user in the account, uses its permissions, and retires the grant as soon as its task is complete. For details about how AWS services, use grants, see the *Encryption at rest* topic in the service's user guide or developer guide.

Grants are a very flexible and useful access control mechanism. When you create a grant for a KMS key, the grant allows the grantee principal to call the specified grant operations on the KMS key provided that all conditions specified in the grant are met.

- Each grant allows access to exactly one KMS key. You can create a grant for a KMS key in a different AWS account.
- A grant can allow access to a KMS key, but not deny access.
- Each grant has one grantee, which can be either a [grantee principal](#terms-grantee-principal) (an AWS IAM identity) or a [grantee service principal](#terms-grantee-service-principal) (an AWS service principal). You must specify either `GranteePrincipal` or `GranteeServicePrincipal`, but not both.
- A grant can only allow [grant operations](#terms-grant-operations). The grant operations must be supported by the KMS key in the grant. If you specify an unsupported operation, the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) request fails with a `ValidationError` exception.
- The grantee principal can use the permissions that the grant gives them without specifying the grant, just as they would if the permissions came from a key policy or IAM policy. However, because the AWS KMS API follows an [eventual consistency](#terms-eventual-consistency) model, when you create, retire, or revoke a grant, there might be a brief delay, before the change is available throughout AWS KMS. To use the permissions in a grant immediately, [use a grant token](https://docs.aws.amazon.com/kms/latest/developerguide/using-grant-token.html).
- An authorized principal can delete the grant ([retire](#terms-retire-grant) or [revoke](#terms-revoke-grant) it). Deleting a grant eliminates all permissions that the grant allowed. You do not have to figure out which policies to add or remove to undo the grant.
- AWS KMS limits the number of grants on each KMS key. For details, see [Grants per KMS key: 50,000](https://docs.aws.amazon.com/kms/latest/developerguide/resource-limits.html#grants-per-key).

Be cautious when creating grants and when giving others permission to create grants. Permission to create grants has security implications, much like allowing the [kms:PutKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_PutKeyPolicy.html) permission to set policies.

- Users with permission to create grants for a KMS key (`kms:CreateGrant`) can use a grant to allow users and roles, including AWS services, to use the KMS key. The principals can be identities in your own AWS account or identities in a different account or organization.
- Grants can allow only a subset of AWS KMS operations. You can use grants to allow principals to view the KMS key, use it in cryptographic operations, and create and retire grants. For details, see [Grant operations](#terms-grant-operations). You can also use [grant constraints](https://docs.aws.amazon.com/kms/latest/developerguide/create-grant-overview.html#grant-constraints) to limit the permissions in a grant.
- Principals can get permission to create grants from a key policy or IAM policy. Principals who get `kms:CreateGrant` permission from a policy can create grants for any [grant operation](#terms-grant-operations) on the KMS key. These principals are not required to have the permission that they are granting on the key. When you allow `kms:CreateGrant` permission in a policy, you can use [policy conditions](https://docs.aws.amazon.com/kms/latest/developerguide/grant-authorization.html) to limit this permission.
- Principals can also get permission to create grants from a grant. These principals can only delegate the permissions that they were granted, even if they have other permissions from a policy. For details, see [Granting CreateGrant permission](https://docs.aws.amazon.com/kms/latest/developerguide/create-grant-overview.html#grant-creategrant).

## Grant concepts

To use grants effectively, you'll need to understand the terms and concepts that AWS KMS uses.

**Grant constraint**

A condition that limits the permissions in a grant. AWS KMS supports two types of grant constraints:

- **Encryption context constraint** — Restricts permissions based on the [encryption context](https://docs.aws.amazon.com/kms/latest/developerguide/encrypt_context.html) included in the request for a cryptographic operation. This constraint works only with symmetric encryption KMS keys.
- **SourceArn constraint** — Restricts grant permissions to requests made on behalf of a specific AWS resource. This is effectively putting an [aws:SourceArn](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourcearn) global condition key into the grant. This constraint works with *all types* of KMS keys.

For details, see [Using grant constraints](https://docs.aws.amazon.com/kms/latest/developerguide/create-grant-overview.html#grant-constraints).

**Grant ID**

The unique identifier of a grant for a KMS key. You can use a grant ID, along with a [key identifier](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id), to identify a grant in a [RetireGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RetireGrant.html) or [RevokeGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RevokeGrant.html) request.

**Grant operations**

The AWS KMS operations that you can allow in a grant. If you specify other operations, the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) request fails with a `ValidationError` exception. These are also the operations that accept a [grant token](#grant_token). For detailed information about these permissions, see the [AWS KMS permissions](https://docs.aws.amazon.com/kms/latest/developerguide/kms-api-permissions-reference.html).

These grant operations actually represent permission to use the operation. Therefore, for the `ReEncrypt` operation, you can specify `ReEncryptFrom`, `ReEncryptTo`, or both `ReEncrypt*`.

The grant operations are:

The grant operations that you allow must be supported by the KMS key in the grant. If you specify an unsupported operation, the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) request fails with a `ValidationError` exception. For example, grants for symmetric encryption KMS keys cannot allow the [Sign](https://docs.aws.amazon.com/kms/latest/APIReference/API_Sign.html), [Verify](https://docs.aws.amazon.com/kms/latest/APIReference/API_Verify.html), [`GenerateMac`](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateMac.html) or [`VerifyMac`](https://docs.aws.amazon.com/kms/latest/APIReference/API_VerifyMac.html) operations. Grants for asymmetric KMS keys cannot allow any operations that generate data keys or data key pairs.

**Grant token**

The AWS KMS API follows an [eventual consistency](#terms-eventual-consistency) model. When you create a grant, there might be a brief delay before the change is available throughout AWS KMS. It typically takes less than a few seconds for the change to propagate throughout the system, but in some cases it can take several minutes. If you try to use a grant before it fully propagates through the system, you might get an access denied error. A grant token lets you refer to the grant and use the grant permissions immediately.

A *grant token* is a unique, nonsecret, variable-length, base64-encoded string that represents a grant. You can use the grant token to identify the grant in any [grant operation](#terms-grant-operations). However, because the token value is a hash digest, it doesn't reveal any details about the grant.

A grant token is designed to be used only until the grant has fully propagated throughout AWS KMS. After that, the [grantee principal](#terms-grantee-principal) can use the permission in the grant without providing a grant token or any other evidence of the grant. You can use a grant token at any time, but once the grant is eventually consistent, AWS KMS uses the grant to determine permissions, not the grant token.

For example, the following command calls the [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) operation. It uses a grant token to represent the grant that gives the caller (the grantee principal) permission to call `GenerateDataKey` on the specified KMS key.

```bash
$ aws kms generate-data-key \
        --key-id 1234abcd-12ab-34cd-56ef-1234567890ab \
        --key-spec AES_256 \
        --grant-token $token
```

You can also use a grant token to identify a grant in operations that manage grants. For example, the [retiring principal](#terms-retiring-principal) can use a grant token in a call to the [RetireGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RetireGrant.html) operation.

```
$ aws kms retire-grant \
        --grant-token $token
```

`CreateGrant` is the only operation that returns a grant token. You cannot get a grant token from any other AWS KMS operation or from the [CloudTrail log event](https://docs.aws.amazon.com/kms/latest/developerguide/ct-creategrant.html) for the CreateGrant operation. The [ListGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html) and [ListRetirableGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListRetirableGrants.html) operations return the [grant ID](#terms-grant-id), but not a grant token.

For details, see [Using a grant token](https://docs.aws.amazon.com/kms/latest/developerguide/using-grant-token.html).

**Grantee principal**

An AWS principal (IAM identity) that gets the permissions specified in the grant.

The grantee principal can be any AWS principal, including an AWS account (root), an [IAM user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html), an [IAM role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html), a [federated role or user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers.html), or an assumed role user. The grantee principal can be in the same account as the KMS key or a different account. However, the grantee principal cannot be a [service principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#principal-services), an [IAM group](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_groups.html), or an [AWS organization](https://docs.aws.amazon.com/organizations/latest/userguide/).

To specify the grantee principal, use the `GranteePrincipal` parameter in the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) request.

###### Note

IAM best practices discourage the use of IAM users with long-term credentials. Whenever possible, use IAM roles, which provide temporary credentials. For details, see [Security best practices in IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) in the *IAM User Guide*.

**Grantee service principal**

An AWS [service principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#principal-services) that gets the permissions specified in the grant. To specify the grantee service principal, use the `GranteeServicePrincipal` parameter in the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) request.

When you create a grant with a `GranteeServicePrincipal`, you must also include a [SourceArn grant constraint](https://docs.aws.amazon.com/kms/latest/developerguide/create-grant-overview.html#terms-source-arn-constraint). The `SourceArn` constraint ensures that the service principal can use the KMS key only when the request is made on behalf of the specified AWS resource.

When you specify a `GranteeServicePrincipal`, you must also specify either a [RetiringPrincipal](#terms-retiring-principal) or a [RetiringServicePrincipal](#terms-retiring-service-principal).

###### Note

You must specify either [GranteePrincipal](#terms-grantee-principal) or [GranteeServicePrincipal](#terms-grantee-service-principal), but not both.

**Retire (a grant)**

Terminates a grant. You retire a grant when you finish using the permissions.

Revoking and retiring a grant both delete the grant. But retiring is done by a principal specified in the grant. Revoking is typically done by a key administrator. For details, see [Retiring and revoking grants](https://docs.aws.amazon.com/kms/latest/developerguide/grant-delete.html).

**Retiring principal**

A principal who can [retire a grant](#terms-retire-grant). You can specify a retiring principal in a grant, but it is not required. The retiring principal can be any AWS principal, including AWS accounts, IAM users, IAM roles, federated users, and assumed role users. The retiring principal can be in the same account as the KMS key or a different account.

To specify the retiring principal, use the `RetiringPrincipal` parameter in the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) request.

###### Note

IAM best practices discourage the use of IAM users with long-term credentials. Whenever possible, use IAM roles, which provide temporary credentials. For details, see [Security best practices in IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) in the *IAM User Guide*.

**Retiring service principal**

An AWS [service principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#principal-services) that has permission to [retire a grant](#terms-retire-grant). To specify the retiring service principal, use the `RetiringServicePrincipal` parameter in the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) request.

###### Note

You can specify either [RetiringPrincipal](#terms-retiring-principal) or [RetiringServicePrincipal](#terms-retiring-service-principal), but not both.

For more information about who can retire a grant, see [Retiring and revoking grants](https://docs.aws.amazon.com/kms/latest/developerguide/grant-delete.html).

**Revoke (a grant)**

Terminates a grant. You revoke a grant to actively deny the permissions that the grant allows.

Revoking and retiring a grant both delete the grant. But retiring is done by a principal specified in the grant. Revoking is typically done by a key administrator. For details, see [Retiring and revoking grants](https://docs.aws.amazon.com/kms/latest/developerguide/grant-delete.html).

**Eventual consistency (for grants)**

The AWS KMS API follows an [eventual consistency](https://en.wikipedia.org/wiki/Eventual_consistency) model. When you create, retire, or revoke a grant, there might be a brief delay before the change is available throughout AWS KMS. It typically takes less than a few seconds for the change to propagate throughout the system, but in some cases it can take several minutes.

You might become aware of this brief delay if you get unexpected errors. For example, If you try to manage a new grant or use the permissions in a new grant before the grant is known throughout AWS KMS, you might get an access denied error. If you retire or revoke a grant, the grantee principal might still be able to use its permissions for a brief period until the grant is fully deleted. The typical strategy is to retry the request, and some AWS SDKs include automatic backoff and retry logic.

AWS KMS has features to mitigate this brief delay.

- To use the permissions in a new grant immediately, use a [grant token](https://docs.aws.amazon.com/kms/latest/developerguide/using-grant-token.html). You can use a grant token to refer to a grant in any [grant operation](#terms-grant-operations). For instructions, see [Using a grant token](https://docs.aws.amazon.com/kms/latest/developerguide/using-grant-token.html).
- The [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) operation has a `Name` parameter that prevents retry operations from creating duplicate grants.

###### Note

Grant tokens supersede the validity of the grant until all endpoints in the service have been updated with the new grant state. In most cases, eventual consistency will be achieved within five minutes.

For more information, see [AWS KMS eventual consistency](https://docs.aws.amazon.com/kms/latest/developerguide/accessing-kms.html#programming-eventual-consistency).