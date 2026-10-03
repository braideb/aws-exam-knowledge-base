---
title: "Amazon Aurora MySQL database clusters now support up to 256 TiB of storage volume"
source: "https://aws.amazon.com/about-aws/whats-new/2025/07/amazon-aurora-mysql-database-clusters-256-tib-storage/"
author:
  - "[[Amazon Web Services]]"
published: 2025-07-30
created: 2026-10-02
description: "Discover more about what's new at AWS with Amazon Aurora MySQL database clusters now support up to 256 TiB of storage volume"
tags:
  - "clippings"
---
Amazon Aurora MySQL-Compatible Edition now supports a maximum storage limit of 256 TiB, doubling the previous limit of 128 TiB. This enhancement allows customers to store and manage even larger datasets within a single Aurora database cluster simplifying data management for large-scale applications and supporting the growing data needs of modern applications. Customers only pay for the storage they use, with no need for upfront provisioning of the full 256 TiB.  
  
To access the increased storage limit, upgrade your cluster to supported database versions. Once upgraded, Aurora storage will automatically scale up to 256 TiB capacity based on the amount of data in the cluster volume. Visit [technical documentation](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/CHAP_Limits.html#RDS_Limits.FileSize.Aurora) to learn more about supported versions. This new storage volume capacity is available in all [AWS regions](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.RegionsAndAvailabilityZones.html#Aurora.Overview.Availability) where Aurora MySQL and Aurora PostgreSQL is available.  
  
Amazon Aurora is designed for unparalleled high performance and availability at global scale with full MySQL and PostgreSQL compatibility. It provides built-in security, continuous backups, serverless compute, up to 15 read replicas, automated multi-Region replication, and integrations with other AWS services. To get started with Amazon Aurora, take a look at our [getting started page](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/CHAP_GettingStartedAurora.html).