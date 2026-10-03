---
title: "Using Aurora serverless"
source: "https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.html"
author:
published:
created: 2026-10-02
description: "Work with Aurora serverless, an on-demand autoscaling configuration for Amazon Aurora."
tags:
  - "clippings"
---
Using Aurora serverless - Amazon Aurora

Aurora serverless is an on-demand, autoscaling configuration for Amazon Aurora. Aurora serverless helps to automate the processes of monitoring the workload and adjusting the capacity for your databases. Capacity is adjusted automatically based on application demand. You're charged only for the resources that your DB clusters consume. Thus, Aurora serverless can help you to stay within budget and avoid paying for computer resources that you don't use.

This type of automation is especially valuable for multitenant databases, distributed databases, development and test systems, and other environments with highly variable and unpredictable workloads.

###### Topics

- [Aurora serverless use cases](#aurora-serverless-v2.use-cases)
- [Advantages of Aurora serverless](#aurora-serverless-v2.advantages)
- [How Aurora serverless works](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.how-it-works.html)
- [Requirements and limitations for Aurora serverless](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.requirements.html)
- [Creating a DB cluster that uses Aurora serverless](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.create.html)
- [Managing Aurora serverless DB clusters](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2-administration.html)
- [Performance and scaling for Aurora serverless](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.setting-capacity.html)
- [Scaling to Zero ACUs with automatic pause and resume for Aurora serverless](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2-auto-pause.html)

## Aurora serverless use cases

Aurora serverless supports many types of database workloads. These range from development and testing environments, to websites and applications that have unpredictable workloads, to the most demanding, business-critical applications that require high scale and availability.

Aurora serverless is especially useful for the following use cases:

- **Variable workloads** – You're running workloads that have sudden and unpredictable increases in activity. An example is a traffic site that sees a surge of activity when it starts raining. Another is an e-commerce site with increased traffic when you offer sales or special promotions. With Aurora serverless, your database automatically scales capacity to meet the needs of the application's peak load and scales back down when the surge of activity is over. With Aurora serverless, you no longer need to provision for peak or average capacity. You can specify an upper capacity limit to handle the worst-case situation, and that capacity isn't used unless it's needed.
	The granularity of scaling in Aurora serverless helps you to match capacity closely to your database's needs. For a provisioned cluster, scaling up requires adding a whole new DB instance. Aurora serverless can add half an ACU when only a little more capacity is needed. It can add 0.5, 1, 1.5, 2, or additional half-ACUs based on the additional capacity needed to handle an increase in workload. And it can remove 0.5, 1, 1.5, 2, or additional half-ACUs when the workload decreases and that capacity is no longer needed.
- **Multi-tenant applications** – With Aurora serverless, you don't have to individually manage database capacity for each application in your fleet. Aurora serverless manages individual database capacity for you.
	You can create a cluster for each tenant. That way, you can use features such as cloning, snapshot restore, and Aurora global databases to enhance high availability and disaster recovery as appropriate for each tenant.
	Each tenant might have specific busy and idle periods depending on the time of day, time of year, promotional events, and so on. Each cluster can have a wide capacity range. That way, clusters with low activity incur minimal DB instance charges. Any cluster can quickly scale up to handle periods of high activity.
- **New applications** – You're deploying a new application and you're unsure about the DB instance size you need. By using Aurora serverless, you can set up a cluster with one or many DB instances and have the database autoscale to the capacity requirements of your application.
- **Mixed-use applications** – Suppose that you have an online transaction processing (OLTP) application, but you periodically experience spikes in query traffic. By specifying promotion tiers for the Aurora serverless DB instances in a cluster, you can configure your cluster so that the reader DB instances can scale independently of the writer DB instance to handle the additional load. When the usage spike subsides, the reader DB instances scale back down to match the capacity of the writer DB instance.
- **Capacity planning** – Suppose that you usually adjust your database capacity, or verify the optimal database capacity for your workload, by modifying the DB instance classes of all the DB instances in a cluster. With Aurora serverless, you can avoid this administrative overhead. You can determine the appropriate minimum and maximum capacity by running the workload and checking how much the DB instances actually scale.
	You can modify existing DB instances from provisioned to Aurora serverless or from Aurora serverless to provisioned. You don't need to create a new cluster or a new DB instance in such cases.
	With an Aurora global database, you might not need as much capacity for the secondary clusters as in the primary cluster. You can use Aurora serverless DB instances in the secondary clusters. That way, the cluster capacity can scale up if a secondary region is promoted and takes over your application's workload.
- **Development and testing** – In addition to running your most demanding applications, you can also use Aurora serverless for development and testing environments. With Aurora serverless, you can create DB instances with a low minimum capacity instead of using burstable db.t\* DB instance classes. You can set the maximum capacity high enough that those DB instances can still run substantial workloads without running low on memory. When the database isn't in use, all of the DB instances scale down to avoid unnecessary charges.
	###### Tip
	To make it convenient to use Aurora serverless in development and test environments, the AWS Management Console provides the **Easy create** shortcut when you create a new cluster. If you choose the **Dev/Test** option, Aurora creates a cluster with an Aurora serverless DB instance and a capacity range that's typical for a development and test system.

### Using Aurora serverless for existing provisioned workloads

Suppose that you already have an Aurora application running on a provisioned cluster. You can check how the application would work with Aurora serverless by adding one or more Aurora serverless DB instances to the existing cluster as reader DB instances. You can check how often the reader DB instances scale up and down. You can use the Aurora failover mechanism to promote an Aurora serverless DB instance to be the writer and check how it handles the read/write workload. That way, you can switch over with minimal downtime and without changing the endpoint that your client applications use. For details on the procedure to convert existing clusters to Aurora serverless, see [Converting a provisioned writer or reader to Aurora serverless](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2-administration.html#aurora-serverless-v2-converting-from-provisioned).

## Advantages of Aurora serverless

Aurora serverless is intended for variable or "spiky" workloads. With such unpredictable workloads, you might have difficulty planning when to change your database capacity. You might also have trouble making capacity changes quickly enough using the familiar mechanisms such as adding DB instances or changing DB instance classes. Aurora serverless provides the following advantages to help with such use cases:

- **Simpler capacity management than provisioned** – Aurora serverless reduces the effort for planning DB instance sizes and resizing DB instances as the workload changes. It also reduces the effort for maintaining consistent capacity for all the DB instances in a cluster.
- **Faster and easier scaling during periods of high activity** – Aurora serverless scales compute and memory capacity as needed, with no disruption to client transactions or your overall workload. The ability to use reader DB instances with Aurora serverless helps you to take advantage of horizontal scaling in addition to vertical scaling. The ability to use Aurora global databases means that you can spread your Aurora serverless read workload across multiple AWS Regions. This capability is more convenient than the scaling mechanisms for provisioned clusters.
- **Cost-effective during periods of low activity** – Aurora serverless helps you to avoid overprovisioning your DB instances. Aurora serverless adds resources in granular increments when DB instances scale up. You pay only for the database resources that you consume. Aurora serverless resource usage is measured on a per-second basis. That way, when a DB instance scales down, the reduced resource usage is registered right away.
- **Greater feature parity with provisioned** – You can use many Aurora features with Aurora serverless oFor example, with Aurora serverless you can use reader DB instances, global databases, AWS Identity and Access Management (IAM) database authentication, and Performance Insights.
	In particular, with Aurora serverless you can take advantage of the following features from provisioned clusters:
	- **Reader DB instances** – Aurora serverless can take advantage of reader DB instances to scale horizontally. When a cluster contains one or more reader DB instances, the cluster can fail over immediately in case of problems with the writer DB instance.
	- **Multi-AZ clusters** – You can distribute the Aurora serverless DB instances of a cluster across multiple Availability Zones (AZs). Setting up a Multi-AZ cluster helps to ensure business continuity even in the rare case of issues that affect an entire AZ.
	- **Global databases** – You can use Aurora serverless in combination with Aurora global databases to create additional read-only copies of your cluster in other AWS Regions for disaster recovery purposes.
	- **RDS Proxy** – You can use Amazon RDS Proxy to allow your applications to pool and share database connections to improve their ability to scale.
- **Faster, more granular, less disruptive scaling** – Aurora serverless can scale up and down faster. Scaling can change capacity by as little as 0.5 ACUs, instead of doubling or halving the number of ACUs. Scaling typically happens with no pause in processing at all. Scaling doesn't involve an event that you have to be aware of. Scaling can happen while SQL statements are running and transactions are open, without the need to wait for a quiet point.