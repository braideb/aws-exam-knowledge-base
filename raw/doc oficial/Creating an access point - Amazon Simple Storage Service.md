---
title: "Creating an access point - Amazon Simple Storage Service"
source: "https://docs.aws.amazon.com/AmazonS3/latest/userguide/creating-access-points.html"
author:
published:
created: 2026-07-19
description: "Create up to 10,000 access points to simplify managing data access at scale in Amazon S3."
tags:
  - "clippings"
---
Creating an access point - Amazon Simple Storage Service

You can create S3 access points by using the AWS Management Console, AWS Command Line Interface (AWS CLI), AWS SDKs, or Amazon S3 REST API. Access points are named network endpoints that are attached to a data source such as a bucket, Amazon FSx for ONTAP volume, or Amazon FSx for OpenZFS volume.

By default, you can create up to 10,000 access points per Region for each of your AWS accounts. If you need more than 10,000 access points for a single account in a single Region, you can request a service quota increase. For more information about service quotas and requesting an increase, see [AWS Service Quotas](https://docs.aws.amazon.com/general/latest/gr/aws_service_limits.html) in the *AWS General Reference*.

## Creating access points with S3 buckets

An access point is associated with exactly one Amazon S3 general purpose bucket. If you want to use a bucket in your AWS account, you must first create a bucket. For more information about creating buckets, see [Creating, configuring, and working with Amazon S3 general purpose buckets](https://docs.aws.amazon.com/AmazonS3/latest/userguide/creating-buckets-s3.html).

You can also create a cross-account access point that's associated with a bucket in another AWS account, as long as you know the bucket name and the bucket owner's account ID. However, creating cross-account access points doesn't grant you access to data in the bucket until you are granted permissions from the bucket owner. The bucket owner must grant the access point owner's account (your account) access to the bucket through the bucket policy. For more information, see [Granting permissions for cross-account access points](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-policies.html#access-points-cross-account).

###### To create an access point

1. Sign in to the AWS Management Console and open the Amazon S3 console at [https://console.aws.amazon.com/s3/](https://console.aws.amazon.com/s3/).
2. In the navigation bar, choose the name of the currently displayed AWS Region. Next, choose the Region in which you want to create an access point. The access point must be created in the same Region as the associated bucket.
3. In the left navigation pane, choose **Access Points**.
4. On the **Access Points** page, choose **Create access point**.
5. In the **Access point name** field, enter the name for the access point. For more information about naming access points, see [Naming rules for access points](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-restrictions-limitations-naming-rules.html#access-points-names).
6. For **Data source**, specify the S3 bucket that you want to use with the access point.
	To use a bucket in your account, choose **Choose a bucket in this account**, and enter or browse for the bucket name.
	To use a bucket in a different AWS account, choose **Specify a bucket in another account**, and enter the AWS account ID and name of the bucket. If you're using a bucket in a different AWS account, the bucket owner must update the bucket policy to authorize requests from the access point. For an example bucket policy, see [Granting permissions for cross-account access points](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-policies.html#access-points-cross-account).
	###### Note
	For information about using an FSx for OpenZFS volume as a data source, see [Creating access points with Amazon FSx](#create-access-points-with-fsx).
7. Choose a **Network origin**, either **Internet** or **virtual private cloud (VPC)**. If you choose **virtual private cloud (VPC)**, enter the **VPC ID** that you want to use with the access point.
	For more information about network origins for access points, see [Creating access points restricted to a virtual private cloud](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-vpc.html).
8. Under **Block Public Access settings for this Access Point**, select the block public access settings that you want to apply to the access point. All block public access settings are enabled by default for new access points. We recommend that you keep all settings enabled unless you know that you have a specific need to disable any of them.
	###### Note
	After you create an access point, you can't change its block public access settings.
	For more information about using Amazon S3 Block Public Access with access points, see [Managing public access to access points for general purpose buckets](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-bpa-settings.html).
9. (Optional) Under **Access Point policy - *optional***, specify the access point policy. Before you save your policy, make sure to resolve any security warnings, errors, general warnings, and suggestions. For more information about specifying an access point policy, see [Policy examples for access points](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-policies.html#access-points-policy-examples).
10. Choose **Create access point**.

## Using the S3 console

###### To create an access point

1. Sign in to the AWS Management Console and open the Amazon S3 console at [https://console.aws.amazon.com/s3/](https://console.aws.amazon.com/s3/).
2. In the navigation bar, choose the name of the currently displayed AWS Region. Next, choose the Region in which you want to create an access point. The access point must be created in the same Region as the associated bucket.
3. In the left navigation pane, choose **Access Points**.
4. On the **Access Points** page, choose **Create access point**.
5. In the **Access point name** field, enter the name for the access point. For more information about naming access points, see [Naming rules for access points](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-restrictions-limitations-naming-rules.html#access-points-names).
6. For **Data source**, specify the S3 bucket that you want to use with the access point.
	To use a bucket in your account, choose **Choose a bucket in this account**, and enter or browse for the bucket name.
	To use a bucket in a different AWS account, choose **Specify a bucket in another account**, and enter the AWS account ID and name of the bucket. If you're using a bucket in a different AWS account, the bucket owner must update the bucket policy to authorize requests from the access point. For an example bucket policy, see [Granting permissions for cross-account access points](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-policies.html#access-points-cross-account).
	###### Note
	For information about using an FSx for OpenZFS volume as a data source, see [Creating access points with Amazon FSx](#create-access-points-with-fsx).
7. Choose a **Network origin**, either **Internet** or **virtual private cloud (VPC)**. If you choose **virtual private cloud (VPC)**, enter the **VPC ID** that you want to use with the access point.
	For more information about network origins for access points, see [Creating access points restricted to a virtual private cloud](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-vpc.html).
8. Under **Block Public Access settings for this Access Point**, select the block public access settings that you want to apply to the access point. All block public access settings are enabled by default for new access points. We recommend that you keep all settings enabled unless you know that you have a specific need to disable any of them.
	###### Note
	After you create an access point, you can't change its block public access settings.
	For more information about using Amazon S3 Block Public Access with access points, see [Managing public access to access points for general purpose buckets](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-bpa-settings.html).
9. (Optional) Under **Access Point policy - *optional***, specify the access point policy. Before you save your policy, make sure to resolve any security warnings, errors, general warnings, and suggestions. For more information about specifying an access point policy, see [Policy examples for access points](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-policies.html#access-points-policy-examples).
10. Choose **Create access point**.

The following example command creates an access point named `` `example-ap` `` for the bucket `` `amzn-s3-demo-bucket` `` in the account `` `111122223333` ``. To create the access point, you send a request to Amazon S3 that specifies the following:

- The access point name. For information about naming rules, see [Naming rules for access points](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-restrictions-limitations-naming-rules.html#access-points-names).
- The name of the bucket that you want to associate the access point with.
- The account ID for the AWS account that owns the access point.

```
aws s3control create-access-point --name example-ap --account-id 111122223333 --bucket amzn-s3-demo-bucket
```

When you're creating an access point by using a bucket in a different AWS account, include the `--bucket-account-id` parameter. The following example command creates an access point in the AWS account `` `111122223333` ``, using the bucket `` `amzn-s3-demo-bucket2` ``, which is in the AWS account `` `444455556666` ``.

```
aws s3control create-access-point --name example-ap --account-id 111122223333 --bucket amzn-s3-demo-bucket --bucket-account-id 444455556666
```

## Using the AWS CLI

The following example command creates an access point named `` `example-ap` `` for the bucket `` `amzn-s3-demo-bucket` `` in the account `` `111122223333` ``. To create the access point, you send a request to Amazon S3 that specifies the following:

- The access point name. For information about naming rules, see [Naming rules for access points](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-restrictions-limitations-naming-rules.html#access-points-names).
- The name of the bucket that you want to associate the access point with.
- The account ID for the AWS account that owns the access point.

```
aws s3control create-access-point --name example-ap --account-id 111122223333 --bucket amzn-s3-demo-bucket
```

When you're creating an access point by using a bucket in a different AWS account, include the `--bucket-account-id` parameter. The following example command creates an access point in the AWS account `` `111122223333` ``, using the bucket `` `amzn-s3-demo-bucket2` ``, which is in the AWS account `` `444455556666` ``.

```
aws s3control create-access-point --name example-ap --account-id 111122223333 --bucket amzn-s3-demo-bucket --bucket-account-id 444455556666
```

You can use the REST API to create an access point. For more information, see [CreateAccessPoint](https://docs.aws.amazon.com/AmazonS3/latest/API/API_control_CreateAccessPoint.html) in the *Amazon Simple Storage Service API Reference*.

## Using the REST API

You can use the REST API to create an access point. For more information, see [CreateAccessPoint](https://docs.aws.amazon.com/AmazonS3/latest/API/API_control_CreateAccessPoint.html) in the *Amazon Simple Storage Service API Reference*.

## Creating access points with Amazon FSx

You can create and attach an access point to an FSx for OpenZFS volume using the Amazon FSx console, AWS CLI, or API. Once attached, you can use the S3 object APIs to access your file data. Your data continues to reside on the Amazon FSx file system and continues to be directly accessible for your existing workloads. You continue to manage your storage using all the FSx for OpenZFS storage management capabilities, including backups, snapshots, user and group quotas, and compression.

For instructions on creating an access point and attaching it to an FSx for OpenZFS volume see, [Creating an access point](https://docs.aws.amazon.com/fsx/latest/OpenZFSGuide/create-access-points.html) in the *FSx for OpenZFS User Guide*.