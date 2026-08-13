---
title: "Referencing access points with ARNs, access point aliases, or virtual-hosted–style URIs - Amazon Simple Storage Service"
source: "https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-naming.html"
author:
published:
created: 2026-07-19
description: "Referencing access points with ARNs, access point aliases, or virtual-hosted–style URIs based on use cases."
tags:
  - "clippings"
---
Referencing access points with ARNs, access point aliases, or virtual-hosted–style URIs - Amazon Simple Storage Service

After you create an access point you can use these endpoints to preform a number of operations. When referring to an access point you can use the Amazon Resource Names (ARNs), access point alias, or virtual-hosted–style URI.

## Access point ARNs

Access points have Amazon Resource Names (ARNs). Access point ARNs are similar to bucket ARNs, but they are explicitly typed and encode the access point's AWS Region and the AWS account ID of the access point's owner. For more information about ARNs, see [Identify AWS resources with Amazon Resource Names (ARNs)](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference-arns.html) in the *IAM User Guide*.

Access point ARNs use the following format:

```
arn:aws:s3:region:account-id:accesspoint/resource
```

- `` arn:aws:s3:`us-west-2`:`123456789012`:accesspoint/`test` `` represents the access point named `` `test` ``, owned by account `` `123456789012` `` in the Region `` `us-west-2` ``.
- ``arn:aws:s3:`us-west-2`:`123456789012`:accesspoint/*`` represents all access points under account `` `123456789012` `` in the Region `` `us-west-2` ``.

ARNs for objects accessed through an access point use the following format:

```
arn:aws:s3:region:account-id:accesspoint/access-point-name/object/resource
```

- `` arn:aws:s3:`us-west-2`:`123456789012`:accesspoint/`test`/object/`unit-01` `` represents the object `` `unit-01` ``, accessed through the access point named `` `test` ``, owned by account `` `123456789012` `` in the Region `` `us-west-2` ``.
- ``arn:aws:s3:`us-west-2`:`123456789012`:accesspoint/`test`/object/*`` represents all objects for the access point named `` `test` ``, in account `` `123456789012` `` in the Region `` `us-west-2` ``.
- ``arn:aws:s3:`us-west-2`:`123456789012`:accesspoint/`test`/object/`unit-01`/`finance`/*`` represents all objects under prefix `` `unit-01/finance/` `` for the access point named `` `test` ``, in account `` `123456789012` `` in the Region `` `us-west-2` ``.

## Access point aliases

When you create an access point, Amazon S3 automatically generates an alias that you can use instead of an Amazon S3 bucket name for data access. You can use this access point alias instead of an Amazon Resource Name (ARN) for access point data plane operations. For a list of these operations, see [Access point compatibility](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-service-api-support.html).

An access point alias name is created within the same namespace as an Amazon S3 bucket. This alias name is automatically generated and cannot be changed. An access point alias name meets all the requirements of a valid Amazon S3 bucket name and consists of the following parts:

`` `ACCESS POINT NAME`-METADATA-s3alias `` (for access points attached to an Amazon S3 bucket)

`` `ACCESS POINT NAME`-METADATA-ext-s3alias `` (for access points attached to an non-S3 bucket data source)

###### Note

The `-s3alias` and `-ext-s3alias` suffixes are reserved for access point alias names and can't be used for bucket or access point names. For more information about Amazon S3 bucket-naming rules, see [General purpose bucket naming rules](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucketnamingrules.html).

### Access points aliases use cases and limitations

When adopting access points, you can use access point alias names without requiring extensive code changes.

When you create an access point, Amazon S3 automatically generates an access point alias name, as shown in the following example. To run this command, replace the `` `user                 input placeholders` `` with your own information.

```
aws s3control create-access-point --bucket amzn-s3-demo-bucket1 --name my-access-point --account-id 111122223333
{
    "AccessPointArn": "arn:aws:s3:region:111122223333:accesspoint/my-access-point",
    "Alias": "my-access-point-aqfqprnstn7aefdfbarligizwgyfouse1a-s3alias"
}
```

You can use this access point alias name instead of an Amazon S3 bucket name in any data plane operation. For a list of these operations, see [Access point compatibility](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-service-api-support.html).

The following AWS CLI example for the `get-object` command uses the bucket's access point alias to return information about the specified object. To run this command, replace the `` `user input placeholders` `` with your own information.

```
aws s3api get-object --bucket my-access-point-aqfqprnstn7aefdfbarligizwgyfouse1a-s3alias --key dir/my_data.rtf my_data.rtf
            
{
    "AcceptRanges": "bytes",
    "LastModified": "2020-01-08T22:16:28+00:00",
    "ContentLength": 910,
    "ETag": "\"00751974dc146b76404bb7290f8f51bb\"",
    "VersionId": "null",
    "ContentType": "text/rtf",
    "Metadata": {}
}
```

#### Access point alias limitations

- Aliases cannot be configured by customers.
- Aliases cannot be deleted or modified or disabled on an access point.
- You can use this access point alias name instead of an Amazon S3 bucket name in some data plane operations. For a list of these operations, see [Access points compatibility with S3 operations](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-service-api-support.html#access-points-operations-support).
- You can't use an access point alias name for Amazon S3 control plane operations. For a list of Amazon S3 control plane operations, see [Amazon S3 Control](https://docs.aws.amazon.com/AmazonS3/latest/API/API_Operations_AWS_S3_Control.html) in the *Amazon Simple Storage Service API Reference*.
- You can't use S3 access point aliases as the source or destination for **Move** operations in the Amazon S3 console.
- Aliases cannot be used in AWS Identity and Access Management (IAM) policies.
- Aliases cannot be used as a logging destination for S3 server access logs.
- Aliases cannot be used as a logging destination for AWS CloudTrail logs.
- Amazon SageMaker AI GroundTruth does not support access point aliases.

## Virtual-hosted–style URI

Access points only support virtual-host-style addressing. In a virtual-hosted–style URI, the access point name, AWS account, and AWS Region is part of the domain name in the URL. For more information about virtual hosting, see [Virtual hosting of general purpose buckets](https://docs.aws.amazon.com/AmazonS3/latest/userguide/VirtualHosting.html).

Virtual-hosted–style URI for access points use the following format:

```
https://access-point-name-account-id.s3-accesspoint.region.amazonaws.com
```

###### Note

- If your access point name includes dash (-) characters, include the dashes in the URL and insert another dash before the account ID. For example, to use an access point named `` `finance-docs` `` owned by account `` `123456789012` `` in the Region `` `us-west-2` ``, the appropriate URL would be ``https://`finance-docs`-`123456789012`.s3-accesspoint.`us-west-2`.amazonaws.com``.
- S3 access points don't support access through HTTP. Access points support only secure access through HTTPS.