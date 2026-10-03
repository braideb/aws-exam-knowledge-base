---
title: "Using the Amazon RDS Data API"
source: "https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/data-api.html"
author:
published:
created: 2026-10-02
description: "Learn how to use the Amazon RDS Data API with Aurora DB clusters."
tags:
  - "clippings"
---
Using the Amazon RDS Data API - Amazon Aurora

By using RDS Data API (Data API), you can work with a web-services interface to your Aurora DB cluster. Data API doesn't require a persistent connection to the DB cluster. Instead, it provides a secure HTTP endpoint and integration with AWS SDKs. You can use the endpoint to run SQL statements without managing connections.

Users don't need to pass credentials with calls to Data API, because Data API uses database credentials stored in AWS Secrets Manager. To store credentials in Secrets Manager, users must be granted the appropriate permissions to use Secrets Manager, and also Data API. For more information about authorizing users, see [Authorizing access to the Amazon RDS Data API](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/data-api.access.html).

You can also use Data API to integrate Amazon Aurora with other AWS applications such as AWS Lambda, AWS AppSync, and AWS Cloud9. Data API provides a more secure way to use AWS Lambda. It enables you to access your DB cluster without your needing to configure a Lambda function to access resources in a virtual private cloud (VPC). For more information, see [AWS Lambda](https://aws.amazon.com/lambda/), [AWS AppSync](https://aws.amazon.com/appsync/), and [AWS Cloud9](https://aws.amazon.com/cloud9/).

You can enable Data API when you create the Aurora DB cluster. You can also modify the configuration later. For more information, see [Enabling the Amazon RDS Data API](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/data-api.enabling.html).

After you enable Data API, you can also use the query editor to run ad hoc queries without configuring a query tool to access Aurora in a VPC. For more information, see [Using the Aurora query editor](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/query-editor.html).

###### Topics

- [Region and version availability for the Amazon RDS Data API](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/data-api.regions.html)
- [Using IPv6 with Amazon RDS Data API](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/data-api.ipv6.html)
- [Limitations for the Amazon RDS Data API](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/data-api.limitations.html)
- [Authorizing access to the Amazon RDS Data API](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/data-api.access.html)
- [Enabling the Amazon RDS Data API](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/data-api.enabling.html)
- [Creating an Amazon VPC endpoint for the Amazon RDS Data API (AWS PrivateLink)](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/data-api.vpc-endpoint.html)
- [Calling the Amazon RDS Data API](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/data-api.calling.html)
- [Using the Java client library for RDS Data API](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/data-api.java-client-library.html)
- [Processing Amazon RDS Data API query results in JSON format](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/data-api-json.html)
- [Troubleshooting Amazon RDS Data API](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/data-api.troubleshooting.html)
- [Logging Amazon RDS Data API calls with AWS CloudTrail](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/logging-using-cloudtrail-data-api.html)
- [Monitoring RDS Data API queries with Performance Insights](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/monitoring-using-performance-insights-data-api.html)