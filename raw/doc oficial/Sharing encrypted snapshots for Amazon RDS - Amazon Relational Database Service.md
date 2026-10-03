---
title: "Sharing encrypted snapshots for Amazon RDS - Amazon Relational Database Service"
source: "https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/share-encrypted-snapshot.html"
author:
published:
created: 2026-10-02
description: "Share an encrypted DB snapshot."
tags:
  - "clippings"
---
Sharing encrypted snapshots for Amazon RDS - Amazon Relational Database Service

You can share DB snapshots that have been encrypted "at rest" using the AES-256 encryption algorithm, as described in [Encrypting Amazon RDS resources](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.Encryption.html).

The following restrictions apply to sharing encrypted snapshots:

- You can't share encrypted snapshots as public.
- You can't share Oracle or Microsoft SQL Server snapshots that are encrypted using Transparent Data Encryption (TDE).
- You can't share a snapshot that has been encrypted using the default KMS key of the AWS account that shared the snapshot.
	For more information about AWS KMS key management for Amazon RDS, see [AWS KMS key management](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.Encryption.Keys.html).

To work around the default KMS key issue, perform the following tasks:

## Create a customer managed key and give access to it

First you create a custom KMS key in the same AWS Region as the encrypted DB snapshot. While creating the customer managed key, you give access to it for another AWS account.

###### Note

You can also use a KMS key from another AWS account when the key policy grants access to the source and target accounts.

###### To create a customer managed key and give access to it

1. Sign in to the AWS Management Console from the source AWS account.
2. Open the AWS KMS console at [https://console.aws.amazon.com/kms](https://console.aws.amazon.com/kms).
3. To change the AWS Region, use the Region selector in the upper-right corner of the page.
4. In the navigation pane, choose **Customer managed keys**.
5. Choose **Create key**.
6. On the **Configure key** page:
	1. For **Key type**, select **Symmetric**.
		2. For **Key usage**, select **Encrypt and decrypt**.
		3. Expand **Advanced options**.
		4. For **Key material origin**, select **KMS**.
		5. For **Regionality**, select **Single-Region key**.
		6. Choose **Next**.
7. On the **Add labels** page:
	1. For **Alias**. enter a display name for your KMS key, for example `share-snapshot`.
		2. (Optional) Enter a description for your KMS key.
		3. (Optional) Add tags to your KMS key.
		4. Choose **Next**.
8. On the **Define key administrative permissions** page, choose **Next.**
9. On the **Define key usage permissions** page:
	1. For **Other AWS accounts**, choose **Add another AWS account**.
		2. Enter the ID of the AWS account to which you want to give access.
		You can give access to multiple AWS accounts.
		3. Choose **Next**.
10. Review your KMS key, then choose **Finish**.

## Copy and share the snapshot from the source account

Next you copy the source DB snapshot to a new snapshot using the customer managed key. Then you share it with the target AWS account.

## Copy the shared snapshot in the target account

Now you can copy the shared snapshot in the target AWS account.