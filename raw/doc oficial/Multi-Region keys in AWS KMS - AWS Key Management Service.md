---
title: "Multi-Region keys in AWS KMS - AWS Key Management Service"
source: "https://docs.aws.amazon.com/kms/latest/developerguide/multi-region-keys-overview.html"
author:
published:
created: 2026-07-19
description: "Learn how to create and use multi-Region keys"
tags:
  - "clippings"
---
Multi-Region keys in AWS KMS - AWS Key Management Service

[Terminology and concepts](#multi-region-concepts)

AWS KMS supports *multi-Region keys*, which are AWS KMS keys in different AWS Regions that can be used interchangeably – as though you had the same key in multiple Regions. Each set of *related* multi-Region keys has the same key material and [key ID](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-key-id), so you can encrypt data in one AWS Region and decrypt it in a different AWS Region without re-encrypting or making a cross-Region call to AWS KMS.

Like all KMS keys, multi-Region keys never leave AWS KMS unencrypted. You can create symmetric or asymmetric multi-Region keys for encryption or signing, create HMAC multi-Region keys for generating and verifying HMAC tags, and create multi-Region keys with [imported key material](https://docs.aws.amazon.com/kms/latest/developerguide/importing-keys.html) or key material that AWS KMS generates. You must manage each multi-Region key independently, including creating aliases and tags, setting their key policies and grants, and enabling and disabling them selectively. You can use multi-Region keys in all cryptographic operations that you can do with single-Region keys.

Multi-Region keys are a flexible and powerful solution for many common data security scenarios.

**Disaster recovery**

In a backup and recovery architecture, multi-Region keys let you process encrypted data without interruption even in the event of an AWS Region outage. Data maintained in backup Regions can be decrypted in the backup Region, and data newly encrypted in the backup Region can be decrypted in the primary Region when that Region is restored.

**Global data management**

Businesses that operate globally need globally distributed data that is available consistently across AWS Regions. You can create multi-Region keys in all Regions where your data resides, then use the keys as though they were a single-Region key without the latency of a cross-Region call or the cost of re-encrypting data under a different key in each Region.

**Distributed signing applications**

Applications that require cross-Region signature capabilities can use multi-Region asymmetric signing keys to generate identical digital signatures consistently and repeatedly in different AWS Regions.

If you use certificate chaining with a single global trust store (for a single root certificate authority (CA), and Regional intermediate CAs signed by the root CA, you don't need multi-Region keys. However, if your system doesn't support intermediate CAs, such as application signing, you can use multi-Region keys to bring consistency to Regional certifications.

**Active-active applications that span multiple Regions**

Some workloads and applications can span multiple Regions in active-active architectures. For these applications, multi-Region keys can reduce complexity by providing the same key material for concurrent encrypt and decrypt operations on data that might be moving across Region boundaries.

You can use multi-Region keys with client-side encryption libraries, such as the [AWS Encryption SDK](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/), the [AWS Database Encryption SDK](https://docs.aws.amazon.com/dynamodb-encryption-client/latest/devguide/), and [Amazon S3 client-side encryption](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingClientSideEncryption.html).

Most [AWS services that integrate with AWS KMS](https://aws.amazon.com/kms/features/) for encryption at rest or digital signatures currently treat multi-Region keys as though they were single-Region keys. For example, Amazon S3 cross-Region replication decrypts and re-encrypts the data keys used to encrypt object data under the KMS key in the destination Region, even when the KMS key in both Regions is a related multi-Region key. Refer to service-specific documentation to understand how a service replicates encrypted data and if it treats multi-Region keys differently.

Multi-Region keys are not global. You create a multi-Region primary key and then replicate it into Regions that you select within an [AWS partition](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html). Then you manage the multi-Region key in each Region independently. Neither AWS nor AWS KMS ever automatically creates or replicates multi-Region keys into any Region on your behalf. [AWS managed keys](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-managed-key), the KMS keys that AWS services create in your account for you, are always single-Region keys.

In China Regions, you can use the multi-Region key feature to replicate KMS keys within the China Regions partition (`aws-cn`). For example, you can replicate a key from the China (Beijing) Region to the China (Ningxia) Region, or the reverse. By replicating a key from one China region to another, you agree to use the AWS Key Management Service of the destination region and comply with all applicable terms of agreement for the destination region. You cannot replicate a key from the Beijing and Ningxia Regions into an AWS Region outside of the China Regions partition. Similarly, you cannot replicate a key from a region outside of the China Regions partition into the Beijing and Ningxia Regions.

You cannot convert an existing single-Region key to a multi-Region key. This design ensures that all data protected with existing single-Region keys maintain the same data residency and data sovereignty properties.

For most data security needs, the Regional isolation and fault tolerance of Regional resources make standard AWS KMS single-Region keys a best-fit solution. However, when you need to encrypt or sign data in client-side applications across multiple Regions, multi-Region keys might be the solution.

**Regions**

Multi-Region keys are supported in all AWS Regions that AWS KMS supports.

**Pricing and quotas**

Every key in a set of related multi-Region keys counts as one KMS key for pricing and quotas. [AWS KMS quotas](https://docs.aws.amazon.com/kms/latest/developerguide/limits.html) are calculated separately for each Region of an account. Use and management of the multi-Region keys in each Region count toward the quotas for that Region.

**Supported KMS key types**

You can create the following types of multi-Region KMS keys:

- Symmetric encryption KMS keys
- Asymmetric KMS keys
- HMAC KMS keys
- KMS keys with imported key material

You cannot create multi-Region keys in a custom key store.

**Learn more**

- To learn how to control access to multi-Region KMS keys, see [Control access to multi-Region keys](https://docs.aws.amazon.com/kms/latest/developerguide/multi-region-keys-auth.html).
- To create multi-Region primary KMS keys of any type, see [Create multi-Region primary keys](https://docs.aws.amazon.com/kms/latest/developerguide/create-primary-keys.html).
- To create multi-Region replica KMS keys, see [Create multi-Region replica keys](https://docs.aws.amazon.com/kms/latest/developerguide/multi-region-keys-replicate.html).
- To update the primary Region, see [Change the primary key in a set of multi-Region keys](https://docs.aws.amazon.com/kms/latest/developerguide/multi-region-update.html).
- To identify and view multi-Region KMS keys, see [Identify HMAC KMS keys](https://docs.aws.amazon.com/kms/latest/developerguide/identify-key-types.html#hmac-view).
- To learn about special considerations for deleting multi-Region KMS keys, see [Deleting multi-Region keys](https://docs.aws.amazon.com/kms/latest/developerguide/deleting-keys.html#deleting-mrks).

## Terminology and concepts

The following terms and concepts are used with multi-Region keys.

### Multi-Region key

A *multi-Region key* is one of a set of KMS keys with the same key ID and key material (and other [shared properties](#mrk-replica-key)) in different AWS Regions. Each multi-Region key is a fully functioning KMS key that can be used entirely independently of its related multi-Region keys. Because all *related* multi-Region keys have the same key ID and key material, they are *interoperable*, that is, any related multi-Region key in any AWS Region can decrypt ciphertext encrypted by any other related multi-Region key.

You set the multi-Region property of a KMS key when you create it. You cannot change the multi-Region property on an existing key. You cannot convert a single-Region key to multi-Region key or a convert a multi-Region key to a single-Region key. To move existing workloads into multi-Region scenarios, you must re-encrypt your data or create new signatures with new multi-Region keys.

A multi-Region key can be [symmetric or asymmetric](https://docs.aws.amazon.com/kms/latest/developerguide/symmetric-asymmetric.html) and it can use AWS KMS key material or [imported key material](https://docs.aws.amazon.com/kms/latest/developerguide/importing-keys.html). You cannot create multi-Region keys in a [custom key store](https://docs.aws.amazon.com/kms/latest/developerguide/key-store-overview.html#custom-key-store-overview).

In a set of related multi-Region keys, there is exactly one [primary key](#mrk-primary-key) at any time. You can create [replica keys](#mrk-replica-key) of that primary key in other AWS Regions. You can also [update the primary region](https://docs.aws.amazon.com/kms/latest/developerguide/multi-region-update.html#update-primary-console), which changes the primary key to a replica key and changes a specified replica key to the primary key. However, you can maintain only one primary key or replica key in each AWS Region. All of the Regions must be in the same [AWS partition](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html).

You can have multiple sets of related multi-Region keys in the same or different AWS Regions. Although related multi-Region keys are interoperable, unrelated multi-Region keys are not interoperable.

### Primary key

A multi-Region *primary key* is a KMS key that can be replicated into other AWS Regions in the same partition. Each set of multi-Region keys has just one primary key.

A primary key differs from a replica key in the following ways:

- Only a primary key can be [replicated](https://docs.aws.amazon.com/kms/latest/developerguide/multi-region-keys-replicate.html).
- The primary key is the source for [shared properties](#mrk-replica-key) of its [replica keys](#mrk-replica-key), including the key material and key ID.
- You can enable and disable [automatic key rotation](https://docs.aws.amazon.com/kms/latest/developerguide/rotate-keys.html) only on a primary key.
- You can [schedule the deletion of a primary key](https://docs.aws.amazon.com/kms/latest/developerguide/deleting-keys.html#deleting-mrks) at any time. But AWS KMS will not delete a primary key until all of its replica keys are deleted.

However, primary and replica keys don't differ in any cryptographic properties. You can use a primary key and its replica keys interchangeably.

You are not required to replicate a primary key. You can use it just as you would any KMS key and replicate it if and when it is useful. However, because multi-Region keys have different security properties than single-Region keys, we recommend that you create a multi-Region key only when you plan to replicate it.

### Replica key

A multi-Region *replica key* is a KMS key that has the same [key ID](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-key-id) and key material as its [primary key](#mrk-primary-key) and related replica keys, but exists in a different AWS Region.

A replica key is a fully functional KMS key with it own key policy, grants, alias, tags, and other properties. It is not a copy of or pointer to the primary key or any other key. You can use a replica key even if its primary key and all related replica keys are disabled. You can also convert a replica key to a primary key and a primary key to a replica key. Once it is created, a replica key relies on its primary key only for [key rotation](https://docs.aws.amazon.com/kms/latest/developerguide/rotate-keys.html#multi-region-rotate) and [updating the primary Region](https://docs.aws.amazon.com/kms/latest/developerguide/multi-region-update.html).

Primary and replica keys don't differ in any cryptographic properties. You can use a primary key and its replica keys interchangeably. Data encrypted by a primary or replica key can be decrypted by the same key, or by any related primary or replica key.

### Replicate

You can *replicate* a multi-Region [primary key](#mrk-primary-key) into a different AWS Region in the same partition. When you do, AWS KMS creates a multi-Region [replica key](#mrk-replica-key) in the specified Region with the same [key ID](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-key-id) and other [shared properties](#mrk-sync-properties) as its primary key. For KMS keys with `AWS_KMS` origin, AWS KMS securely transports the key material across the Region boundary and associates it with the new replica key, all within AWS KMS. For KMS keys with `EXTERNAL` origin, you must import the same key material that you imported into the primary Region key into reach replica Region key individually.

### Shared properties

*Shared properties* are properties of a multi-Region primary key that are shared with its replica keys. AWS KMS creates the replica keys with the same shared property values as those of the primary key. Then, it periodically synchronizes the shared property values of the primary key to its replica keys. You cannot set these properties on a replica key.

The following are the shared properties of multi-Region keys.

- [Key ID](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-key-id) — (The `Region` element of the [key ARN](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-key-ARN) differs.)
- [Key material](https://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html#key-origin) — The primary and replica keys in a set of related multi-Region keys share the same key material. For multi-Region keys whose key material is generated by AWS KMS (`AWS_KMS` origin), AWS KMS securely transports all key materials from the primary to each replica when the replica is created or when new key material is created via automatic or on-demand rotation. For multi-Region keys with imported key material (`EXTERNAL` origin), AWS KMS synchronizes the key material identifier from the primary key but you must import key material into each replica key independently.
- [Key material origin](https://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html#key-origin)
- [Key spec](https://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html#key-spec) and encryption algorithms
- [Key usage](https://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html#key-usage)
- [Automatic key rotation](https://docs.aws.amazon.com/kms/latest/developerguide/rotating-keys-enable.html) — You can enable and disable automatic key rotation only on the primary key. New replica keys are created with all versions of the shared key material. For details, see [Rotating multi-Region keys](https://docs.aws.amazon.com/kms/latest/developerguide/rotate-keys.html#multi-region-rotate).
- [On-demand rotation](https://docs.aws.amazon.com/kms/latest/developerguide/rotating-keys-on-demand.html) — You can perform on-demand rotation only on the primary key. For multi-region keys whose key material is generated by AWS KMS (`AWS_KMS` origin), AWS KMS creates replica keys with all versions of the shared key material. For multi-region keys with imported key material (`EXTERNAL` origin), AWS KMS propagates the key material Id and key material description from the primary key to the replica keys, but not the key material. You must import the correct key material into each replica key individually. For details, see [Rotating multi-Region keys](https://docs.aws.amazon.com/kms/latest/developerguide/rotate-keys.html#multi-region-rotate).

You can also think of the primary and replica designations of related multi-Region keys as shared properties. When you [create new replica keys](#mrk-replica-key) or [update the primary key](https://docs.aws.amazon.com/kms/latest/developerguide/multi-region-update.html#update-primary-console), AWS KMS synchronizes the change to all related multi-Region keys. When these changes are complete, all related multi-Region keys list their primary key and replica keys accurately.

All other properties of multi-Region keys are *independent properties*, including the key description, [key policy](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html), [grants](https://docs.aws.amazon.com/kms/latest/developerguide/grants.html), [enabled and disabled key states](https://docs.aws.amazon.com/kms/latest/developerguide/enabling-keys.html), [aliases](https://docs.aws.amazon.com/kms/latest/developerguide/kms-alias.html), and [tags](https://docs.aws.amazon.com/kms/latest/developerguide/tagging-keys.html). You can set the same values for these properties on all related multi-Region keys, but if you change the value of an independent property, AWS KMS does not synchronize it.

You can track the synchronization of the shared properties of your multi-Region keys. In your AWS CloudTrail log, look for the [SynchronizeMultiRegionKey](https://docs.aws.amazon.com/kms/latest/developerguide/ct-synchronize-multi-region-key.html) event.