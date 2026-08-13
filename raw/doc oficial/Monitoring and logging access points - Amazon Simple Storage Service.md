---
title: "Monitoring and logging access points - Amazon Simple Storage Service"
source: "https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-monitoring-logging.html"
author:
published:
created: 2026-07-19
description: "Monitor S3 access points using Amazon CloudWatch request metrics or Amazon S3 server access logs and AWS CloudTrail."
tags:
  - "clippings"
---
Monitoring and logging access points - Amazon Simple Storage Service

Amazon S3 logs requests made through access points and requests made to the API operations that manage access points, such as `CreateAccessPoint` and `GetAccessPointPolicy`. To monitor and manage usage patterns, you can also configure Amazon CloudWatch Logs request metrics for access points.

## CloudWatch request metrics

To understand and improve the performance of applications that are using access points, you can use CloudWatch for Amazon S3 request metrics. Request metrics help you monitor Amazon S3 requests to quickly identify and act on operational issues.

By default, request metrics are available at the bucket level. However, you can define a filter for request metrics using a shared prefix, object tags, or an access point. When you create an access point filter, the request metrics configuration includes requests to the access point that you specify. You can receive metrics, set alarms, and access dashboards to view real-time operations performed through this access point.

You must opt in to request metrics by configuring them in the console or by using the Amazon S3 API. Request metrics are available at 1-minute intervals after some latency for processing. Request metrics are billed at the same rate as CloudWatch custom metrics. For more information, see [Amazon CloudWatch pricing](https://aws.amazon.com/cloudwatch/pricing/).

To create a request metrics configuration that filters by access point, see [Creating a metrics configuration that filters by prefix, object tag, or access point](https://docs.aws.amazon.com/AmazonS3/latest/userguide/metrics-configurations-filter.html).

## AWS CloudTrail logs for requests made through access points

You can log requests made through access points and requests made to the APIs that manage access points, such as `CreateAccessPoint` and `GetAccessPointPolicy,` by using server access logging and AWS CloudTrail.

CloudTrail log entries for requests made through access points include the access point ARN in the `resources` section of the log.

For example, suppose you have the following configuration:

- A bucket named `` `amzn-s3-demo-bucket1` `` in Region `` `us-west-2` `` that contains an object named `` `my-image.jpg` ``
- An access point named `` `my-bucket-ap` `` that is associated with ``` `` `amzn-s3-demo-bucket1` `` ```
- An AWS account ID of `` `123456789012` ``

The following example shows the `resources` section of a CloudTrail log entry for the preceding configuration:

```bash
"resources": [
        {"type": "AWS::S3::Object",
            "ARN": "arn:aws:s3:::amzn-s3-demo-bucket1/my-image.jpg"
        },
        {"accountId": "123456789012",
            "type": "AWS::S3::Bucket",
            "ARN": "arn:aws:s3:::amzn-s3-demo-bucket1"
        },
        {"accountId": "123456789012",
            "type": "AWS::S3::AccessPoint",
            "ARN": "arn:aws:s3:us-west-2:123456789012:accesspoint/my-bucket-ap"
        }
    ]
```

If you are using an access point attached to a volume on an Amazon FSx file system, the `resources` section of a CloudTrail log entry will look different. For example:

```bash
"resources": [
        {
            "accountId": "123456789012",
            "type": "AWS::FSx::Volume",
            "ARN": "arn:aws:fsx:us-east-1:123456789012:volume/fs-0123456789abcdef9/fsvol-01234567891112223"
        }
    ]
```

For more information about S3 Server Access Logs, see [Logging requests with server access logging](https://docs.aws.amazon.com/AmazonS3/latest/userguide/ServerLogs.html). For more information about AWS CloudTrail, see [What is AWS CloudTrail?](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html) in the *AWS CloudTrail User Guide*.