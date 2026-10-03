---
title: "Amazon Aurora endpoint connections"
source: "https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.Endpoints.html"
author:
published:
created: 2026-10-02
description: "Amazon Aurora typically involves a cluster of DB instances instead of a single instance. Each connection is handled by a specific DB instance. When you connect to an Aurora cluster, the host name and port that you specify point to an intermediate handler called an"
tags:
  - "clippings"
---
Amazon Aurora endpoint connections - Amazon Aurora

Amazon Aurora typically involves a cluster of DB instances instead of a single instance. Each connection is handled by a specific DB instance. When you connect to an Aurora cluster, the host name and port that you specify point to an intermediate handler called an *endpoint*. Aurora uses the endpoint mechanism to abstract these connections. Thus, you don't have to hardcode all the hostnames or write your own logic for balancing and rerouting connections when some DB instances aren't available.

For certain Aurora tasks, different instances or groups of instances perform different roles. For example, the primary instance handles all data definition language (DDL) and data manipulation language (DML) statements. Up to 15 Aurora Replicas handle read-only query traffic.

###### Topics

- [Types of Aurora endpoints](#Aurora.Overview.Endpoints.Types)
- [Viewing the endpoints for an Aurora cluster](#Aurora.Endpoints.Viewing)
- [How Aurora endpoints work with high availability](#Aurora.Overview.Endpoints.HA)
- [Cluster endpoints for Amazon Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Endpoints.Cluster.html)
- [Reader endpoints for Amazon Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Endpoints.Reader.html)
- [Instance endpoints for Amazon Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Endpoints.Instance.html)
- [Custom endpoints for Amazon Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Endpoints.Custom.html)

## Types of Aurora endpoints

Using endpoints, you can map each connection to the appropriate instance or group of instances based on your use case. For example, to perform DDL statements you can connect to whichever instance is the primary instance. To perform queries, you can connect to the reader endpoint, with Aurora automatically performing connection-balancing among all the Aurora Replicas. For clusters with DB instances of different capacities or configurations, you can connect to custom endpoints associated with different subsets of DB instances. For diagnosis or tuning, you can connect to a specific instance endpoint to examine details about a specific DB instance.

An endpoint is represented as an Aurora-specific URL that contains a host address and a port. The following types of endpoints are available from an Aurora DB cluster.

**Cluster endpoint**

Connect to the primary instance of your cluster to develop and test applications, and perform transformations like `INSERT` statements and DDL, DML, and ETL operations. Find the cluster endpoint location by using the AWS Management Console, AWS CLI, or Amazon RDS API, as described in [Viewing the endpoints for an Aurora cluster](#Aurora.Endpoints.Viewing).

For more information about cluster endpoints, see [Cluster endpoints for Amazon Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Endpoints.Cluster.html).

**Reader endpoint**

Perform queries. Aurora automatically performs connection-balancing among all the Aurora Replicas. Find the reader endpoint location by using the AWS Management Console, AWS CLI, or Amazon RDS API, as described in [Viewing the endpoints for an Aurora cluster](#Aurora.Endpoints.Viewing).

For more information about reader endpoints, see [Reader endpoints for Amazon Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Endpoints.Reader.html).

**Instance endpoint**

Examine details about a specific DB instance for diagnosis or tuning. You can find the instance endpoint location for each of your instances in the AWS Management Console only, on the instance detail page for your instance.

For more information about instance endpoints, see [Instance endpoints for Amazon Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Endpoints.Instance.html).

**Custom endpoint**

Connect to different subsets of DB instances on the DB cluster. This is useful when you have different instance capacities and configurations within your DB cluster. Find the custom endpoint locations by using the AWS Management Console, AWS CLI, or Amazon RDS API, as described in [Viewing the endpoints for an Aurora cluster](#Aurora.Endpoints.Viewing).

For more information about custom endpoints, see [Custom endpoints for Amazon Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Endpoints.Custom.html).

**Aurora Global Database writer endpoint**

Aurora Global Database has a special kind of endpoint that serves the same purpose as the cluster endpoint of a standalone Aurora cluster. It handles both write and read requests. When a secondary cluster becomes the new primary cluster due to a switchover or failover, Aurora automatically switches this endpoint to point to the cluster endpoint of the new primary cluster, in the other AWS Region. That way, you don't have to encode the AWS Region into the connection string for your application, and you don't have to change the connection string when the layout of the global database changes. Aurora creates this endpoint when you set up an Aurora Global Database, for example by choosing **Add Region** for an Aurora cluster in the AWS Management Console.

For information on how you can use this type of endpoint with Aurora Global Database, see [Connecting to Amazon Aurora Global Database](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database-connecting.html).

## Viewing the endpoints for an Aurora cluster

While you can only find the instance endpoint location on the instance detail page in the AWS Management Console, you can use the console, AWS CLI, or Amazon RDS API to find the locations of cluster, reader, and custom endpoints.

anchor anchor anchor

## How Aurora endpoints work with high availability

For clusters where high availability is important, use the cluster endpoint for read/write or general-purpose connections and the reader endpoint for read-only connections. The writer and reader endpoints manage DB instance failover better than instance endpoints do. Unlike the instance endpoints, the writer and reader endpoints automatically change which DB instance they connect to if a DB instance in your cluster becomes unavailable. For more information about cluster and reader endpoints, see [Cluster endpoints for Amazon Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Endpoints.Cluster.html) and [Reader endpoints for Amazon Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Endpoints.Reader.html).

If the primary DB instance of a DB cluster fails, Aurora automatically fails over to a new primary DB instance. It does so by either promoting an existing Aurora Replica to a new primary DB instance or creating a new primary DB instance. If a failover occurs, you can use the cluster endpoint to reconnect to the newly promoted or created primary DB instance, or use the reader endpoint to reconnect to one of the Aurora Replicas in the DB cluster. During a failover, the reader endpoint might direct connections to the new primary DB instance of a DB cluster for a short time after an Aurora Replica is promoted to the new primary DB instance.

If you design your own application logic to manage connections to instance endpoints, you can manually or programmatically discover the resulting set of available DB instances in the DB cluster. Use the [describe-db-clusters](https://docs.aws.amazon.com/cli/latest/reference/rds/describe-db-clusters.html) AWS CLI command or [DescribeDBClusters](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_DescribeDBClusters.html) RDS API operation to find the DB cluster and reader endpoints, DB instances, whether DB instances are readers, and their promotion tiers. You can then confirm their instance classes after failover and connect to an appropriate instance endpoint.

For more information about failovers, see [Fault tolerance for an Aurora DB cluster](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.AuroraHighAvailability.html#Aurora.Managing.FaultTolerance).

For more information about high availability in Amazon Aurora, see [High availability for Amazon Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.AuroraHighAvailability.html).