---
title: "Managing access to shared datasets with access points - Amazon Simple Storage Service"
source: "https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points.html"
author:
published:
created: 2026-07-19
description: "Simplify managing data access at scale for shared datasets by creating and using Amazon S3 access points."
tags:
  - "clippings"
---
Managing access to shared datasets with access points - Amazon Simple Storage Service

Amazon S3 access points simplify data access for any AWS service or customer application that stores data in S3. Access points are named network endpoints that are attached to a data source such as a bucket, Amazon FSx for NetApp ONTAP volume, or Amazon FSx for OpenZFS volume. For information about working with buckets, see [General purpose buckets overview](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingBucket.html). For information about working with FSx for NetApp ONTAP, see [What is Amazon FSx for NetApp ONTAP](https://docs.aws.amazon.com/fsx/latest/ONTAPGuide/what-is-fsx-ontap.html) in the *FSx for ONTAP User Guide*. For information about working with FSx for OpenZFS, see [What is Amazon FSx for OpenZFS](https://docs.aws.amazon.com/fsx/latest/OpenZFSGuide/what-is-fsx.html) in the *FSx for OpenZFS User Guide*.

You can use access points to perform S3 object operations, such as `GetObject` and `PutObject`. Each access point has distinct permissions and network controls that S3 applies for any request that is made through that access point. Each endpoint enforces a customized access point policy that allow you to control use by resource, user, or other conditions. If your access point is attached to a bucket the access point policy works in conjunction with the underlying bucket policy. You can configure any access point to accept requests only from a virtual private cloud (VPC) to restrict Amazon S3 data access to a private network. You can also configure custom block public access settings for each access point.

###### Note

You can only use access points to perform operations on objects. You can't use access points to perform other Amazon S3 operations, such as deleting buckets or creating S3 Replication configurations. For a complete list of S3 operations that support access points, see [Access point compatibility](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-service-api-support.html).

The topics in this section explain how to work with Amazon S3 access points. For topics on using access points with directory buckets see, [Managing access to shared datasets in directory buckets with access points](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-directory-buckets.html).

###### Topics

- [Access points naming rules, restrictions, and limitations](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-restrictions-limitations-naming-rules.html)
- [Referencing access points with ARNs, access point aliases, or virtual-hosted–style URIs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-naming.html)
- [Access point compatibility](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-service-api-support.html)
- [Configuring IAM policies for using access points](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-policies.html)
- [Monitoring and logging access points](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-monitoring-logging.html)
- [Creating an access point](https://docs.aws.amazon.com/AmazonS3/latest/userguide/creating-access-points.html)
- [Managing your Amazon S3 access points for general purpose buckets](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-manage.html)
- [Using Amazon S3 access points for general purpose buckets](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-access-points.html)
- [Using tags with S3 Access Points for general purpose buckets](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-tagging.html)