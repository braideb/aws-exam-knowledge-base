---
title: "Configuring and managing a Multi-AZ deployment for Amazon RDS - Amazon Relational Database Service"
source: "https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html"
author:
published:
created: 2026-10-02
description: "Get high availability and failover support for your DB instances with Amazon RDS using a Multi-AZ deployment."
tags:
  - "clippings"
---
Configuring and managing a Multi-AZ deployment for Amazon RDS - Amazon Relational Database Service

Multi-AZ deployments can have one standby or two standby DB instances. When the deployment has one standby DB instance, it's called a *Multi-AZ DB instance deployment*. A Multi-AZ DB instance deployment has one standby DB instance that provides failover support, but doesn't serve read traffic. When the deployment has two standby DB instances, it's called a *Multi-AZ DB cluster deployment*. A Multi-AZ DB cluster deployment has standby DB instances that provide failover support and can also serve read traffic.

You can use the AWS Management Console to determine whether a Multi-AZ deployment is a Multi-AZ DB instance deployment or a Multi-AZ DB cluster deployment. In the navigation pane, choose **Databases**, and then choose a **DB identifier**.

- A Multi-AZ DB instance deployment has the following characteristics:
	- There is only one row for the DB instance.
	- The value of **Role** is **Instance** or **Primary**.
	- The value of **Multi-AZ** is **Yes**.
- A Multi-AZ DB cluster deployment has the following characteristics:
	- There is a cluster-level row with three DB instance rows under it.
	- For the cluster-level row, the value of **Role** is **Multi-AZ DB cluster**.
	- For each instance-level row, the value of **Role** is **Writer instance** or **Reader instance**.
	- For each instance-level row, the value of **Multi-AZ** is **3 Zones**.

In addition, the following topics apply to both DB instances and Multi-AZ DB clusters.