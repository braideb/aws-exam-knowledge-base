---
title: "Logging requests with server access logging - Amazon Simple Storage Service"
source: "https://docs.aws.amazon.com/AmazonS3/latest/userguide/ServerLogs.html"
author:
published:
created: 2026-07-18
description: "Enable Amazon S3 server access logging to track requests for access to your S3 buckets."
tags:
  - "clippings"
---
Logging requests with server access logging - Amazon Simple Storage Service

Server access logging provides detailed records for the requests that are made to a bucket. Server access logs can be used for operational analysis, to understand your security posture, and to charge back users.

Amazon S3 periodically collects access log records, consolidates them, and delivers them to your chosen destination. You can deliver logs to Amazon CloudWatch Logs or to an Amazon S3 general purpose bucket.

###### Note

Server access logs don't record information about wrong-Region redirect errors for Regions that launched after March 20, 2019. Wrong-Region redirect errors occur when a request for an object or bucket is made outside the Region in which the bucket exists.

## Delivery options

You can deliver server access logs to two destinations. You can use one or both.

- **Amazon CloudWatch Logs** – Delivers logs in structured format to a CloudWatch Logs log group. You can query logs with CloudWatch Logs Insights, aggregate logs across accounts and Regions, and encrypt logs with AWS KMS. From the log group, you can also deliver logs to Amazon S3 in JSON or Apache Parquet format, or mirror logs to S3 Tables in Apache Iceberg format for SQL analytics. CloudWatch vended logs ingestion rates apply.
- **Amazon S3 general purpose bucket** – Delivers logs in space-delimited text format to an Amazon S3 bucket you specify. There is no charge for log delivery. You pay only for the storage of the log files. The destination bucket must be in the same AWS Region and AWS account as the source bucket.

The following table compares the two delivery options.

| Capability | Amazon CloudWatch Logs | Amazon S3 general purpose bucket |
| --- | --- | --- |
| Destinations | CloudWatch Logs log group, Amazon S3 (JSON or Parquet), or Amazon Data Firehose. The CloudWatch Logs log group delivery can be extended to mirror logs to S3 Tables in Iceberg format. | Amazon S3 bucket (space-delimited text) |
| Log format | Structured JSON (log group), JSON or Apache Parquet (Amazon S3), Apache Iceberg (S3 Tables) | Space-delimited text |
| Querying | CloudWatch Logs Insights (log group), Amazon Athena or Spark (S3 Tables) | Amazon Athena |
| SQL analytics via S3 Tables | Yes (automatic Iceberg mirror) | No (requires ETL) |
| Cross-account aggregation | Yes (Amazon S3 and Firehose destinations support cross-account delivery) | No (same account only) |
| Cross-Region aggregation | Yes (Amazon S3 and Firehose destinations support cross-Region delivery) | No (same Region only) |
| AWS KMS encryption | Yes | No (SSE-S3 only) |
| Setup | CloudWatch Logs APIs or Amazon S3 console | Amazon S3 API (`PutBucketLogging`) or console |
| Delivery latency | Within a few hours | Within a few hours |

## Best-effort server log delivery

Server access log records are delivered on a best-effort basis. Most requests for a bucket that is properly configured for logging result in a delivered log record. Most log records are delivered within a few hours of the time that they are recorded, but they can be delivered more frequently.

The completeness and timeliness of server logging is not guaranteed. The log record for a particular request might be delivered long after the request was actually processed, or *it might not be delivered at all*. It is possible that you might even see a duplication of a log record. The purpose of server logs is to give you an idea of the nature of traffic against your bucket. Although log records are rarely lost or duplicated, be aware that server logging is not meant to be a complete accounting of all requests.

Because of the best-effort nature of server logging, your usage reports might include one or more access requests that do not appear in a delivered server log. You can find these usage reports under **Cost & usage reports** in the AWS Billing and Cost Management console.

## Bucket logging status changes take effect over time

Changes to the logging status of a bucket take time to actually affect the delivery of log files. For example, if you enable logging for a bucket, some requests made in the following hour might be logged, and others might not. Suppose that you change the destination bucket for logging from bucket A to bucket B. For the next hour, some logs might continue to be delivered to bucket A, whereas others might be delivered to the new destination bucket B. Similarly, if you change the CloudWatch Logs log group used for delivery, some logs might continue to be delivered to the previous log group during the transition. In all cases, the new settings eventually take effect without any further action on your part.

## Pricing

The cost of server access logging depends on which delivery option you use.

- **Amazon CloudWatch Logs** – You pay for ingestion at CloudWatch vended logs rates, with volume-based tiered pricing. CloudWatch Logs compresses logs before storage, so stored volume is significantly less than ingested volume. If you enable the S3 Tables mirror, there is no additional charge for storage or table maintenance. You pay only for query requests at S3 Tables pricing. For current rates, see [CloudWatch pricing](https://aws.amazon.com/cloudwatch/pricing/).
- **Amazon S3 general purpose bucket** – There is no charge for log delivery. You pay only for the storage of the log files at standard Amazon S3 storage rates. We do not assess data-transfer charges for log file delivery, but we do charge the normal data-transfer rate for accessing the log files. For current rates, see [Amazon S3 pricing](https://aws.amazon.com/s3/pricing/).

For more information about delivering and using server access logs, see the following sections.