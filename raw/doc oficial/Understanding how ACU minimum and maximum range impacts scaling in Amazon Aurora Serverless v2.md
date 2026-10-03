---
title: "Understanding how ACU minimum and maximum range impacts scaling in Amazon Aurora Serverless v2"
source: "https://aws.amazon.com/blogs/database/understanding-how-acu-minimum-and-maximum-range-impacts-scaling-in-amazon-aurora-serverless-v2/"
author:
  - "[[Priyanka Sanghvi\"]]"
  - "[[\"Venu Koneru]]"
published: 2024-11-25
created: 2026-10-02
description: "In Part 1 of this two-part blog post series, we focused on understanding how certain Amazon Aurora Serverless v2 database parameters influence the scaling of Aurora capacity units (ACUs) to its minimum and maximum amounts. This post is Part 2, and it focuses on understanding how the minimum and maximum configuration of ACUs impacts scaling behavior in Aurora Serverless v2 and how fast scaling occurs after it starts."
tags:
  - "clippings"
---
In [Part 1](https://aws.amazon.com/blogs/database/understanding-how-certain-database-parameters-impact-scaling-in-amazon-aurora-serverless-v2/) of this two-part blog post series, we focused on understanding how certain [Amazon Aurora Serverless v2](https://aws.amazon.com/rds/aurora/serverless/) database parameters influence the scaling of [Aurora capacity units](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.how-it-works.html) (ACUs) to its minimum and maximum amounts. This post is Part 2, and it focuses on understanding how the minimum and maximum configuration of ACUs impacts scaling behavior in Aurora Serverless v2 and how fast scaling occurs after it starts.

Aurora Serverless v2 is an on-demand, auto scaling configuration for [Amazon Aurora](https://aws.amazon.com/rds/aurora/). It automatically starts up, shuts down, and scales capacity based on your application’s needs. The unit of measure for Aurora Serverless v2 is the ACU. As it scales, Aurora Serverless adjusts capacity in fine-grained ACU increments, providing the right amount of resources. It supports various workloads, from development and test environments, websites, and applications with infrequent or unpredictable workloads to demanding, business-critical applications requiring high scale and availability. Each workload has unique minimum and maximum ACU requirements. Finding the right ACU configuration and understanding the factors influencing Aurora Serverless v2 scaling is essential.

For this post, we defined a test environment with Aurora Serverless v2 with PostgreSQL compatibility, and demonstrated a few test cases to help understand ACU scaling and provide recommendations for configuring the minimum and maximum ACU.

## Understanding ACU and how scaling works

Aurora Serverless v2 [automatically scales](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.how-it-works.html#aurora-serverless-v2.how-it-works.scaling) the capacity of your database up and down in fine-grained increments called ACUs. The ACU isn’t tied to DB instance classes used for provisioned clusters. Each ACU combines approximately 2 GiB of memory, CPU, and networking resources. When using Aurora Serverless v2, you specify a [capacity range](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.setting-capacity.html) (minimum and maximum ACU values) for each database (DB) cluster. The `ServerlessDatabaseCapacity` and `ACUUtilization` metrics help track the actual capacity usage within this range.

The capacity of each Aurora Serverless v2 DB writer or reader is represented as a floating-point ACU value that scales up or down as needed. The minimum capacity can be set as low as 0.5 ACUs, and the maximum is capped at 256 ACUs. [Now you can also enable automatic pause](https://aws.amazon.com/blogs/database/introducing-scaling-to-0-capacity-with-amazon-aurora-serverless-v2/) behavior by specifying a minimum capacity of 0 ACUs if you don’t have any connections initiated by user activity within a specified time period, as explained in [Scaling to Zero ACUs with automatic pause and resume for Aurora Serverless v2](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2-auto-pause.html).  
Depending on the additional features you enable on your DB cluster, you might need the minimum ACU to be higher than 0.5. For example, [Amazon RDS Performance Insights](https://aws.amazon.com/rds/performance-insights/) needs the minimum ACU to be at least 2 and when the Aurora cluster is paused, it doesn’t collect monitoring information for that instance through either Amazon RDS Performance Insights or [Enhanced Monitoring](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_Monitoring.OS.html). For more details, refer to [Choosing the minimum Aurora Serverless v2 capacity setting for a cluster](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.setting-capacity.html#aurora-serverless-v2.min_capacity_considerations).

For this post, we won’t be focusing on the automatic pause feature.

Choosing the appropriate minimum and maximum ACU values is crucial, considering factors such as the application’s working set size and peak load requirements. Monitoring the capacity values over time helps identify if adjustments are needed, such as changing the ACU range or optimizing the application to better utilize resources.

The scaling is based on the actual workload on your database. The [scaling process](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.how-it-works.html#aurora-serverless-v2.how-it-works.scaling) works as follows:

1. You define a capacity range (minimum and maximum ACUs) for your Aurora Serverless v2 DB cluster.
2. Aurora Serverless v2 continually monitors the workload on your database.
3. If the workload increases, Aurora Serverless v2 automatically scales up the capacity of your database by adding more ACUs up to the maximum capacity you defined.
4. If the workload decreases, Aurora Serverless v2 automatically scales down the capacity of your database by removing ACUs down to the minimum capacity you defined.

The scaling process is designed to be seamless and transparent, without disruption to your database operations or connections. Additionally, Aurora Serverless v2 supports independent scaling for the writer instance and reader instances in a Multi-AZ deployment. You can configure the reader instances to [scale independently](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2-administration.html#aurora-serverless-v2-choosing-promotion-tier:~:text=Aurora%20Serverless%20v2%20reader%20DB%20instances%20in%20tiers%202%E2%80%9315%20don%27t%20have%20the%20same%20constraint%20on%20their%20minimum%20capacity.%20When%20they%20are%20idle%2C%20they%20can%20scale%20down%20to%20the%20minimum%20Aurora%20capacity%20unit%20\(ACU\)%20value%20specified%20in%20the%20cluster%27s%20capacity%20range.) from the writer instance, or tie their capacity to the writer instance’s capacity.

## Prerequisites

For this post, we use an Aurora Serverless v2 DB cluster with PostgreSQL compatibility. We use [HammerDB](https://www.hammerdb.com/) to generate some load on the database, to evaluate how the Aurora Serverless v2 DB cluster scales with each of our test cases. Before you get started, make sure you complete the following prerequisites:

1. Create or have access to an [AWS account](https://signin.aws.amazon.com/).
2. [Create an Aurora Serverless v2 DB cluster](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.create.html) with PostgreSQL compatibility with an ACU range of 0.5–64. PostgreSQL 13.6 and higher supports Aurora Serverless v2. We chose PostgreSQL 16.1 for our testing.
3. Create a bastion host using [Amazon Elastic Compute Cloud](https://aws.amazon.com/ec2) (Amazon EC2) with Amazon Linux 2023, which you can use to access Aurora Serverless v2 in a private subnet from your machine’s IP address or from a range of IP addresses, if you’re connecting from your corporate network. For instructions to select the necessary network configuration for a bastion host, refer to [Creating and connecting to an Aurora PostgreSQL DB cluster](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/CHAP_GettingStartedAurora.CreatingConnecting.AuroraPostgreSQL.html).
4. Install the following packages needed to support PostgreSQL connectivity and HammerDB on your bastion host:
	```bash
	sudo dnf update
	sudo yum -y install postgresql15
	sudo yum -y install python3-psycopg2
	--Download HammerDB
	wget https://github.com/TPC-Council/HammerDB/releases/download/v4.10/HammerDB-4.10-Linux.tar.gz
	--Unzip
	tar -zxvf HammerDB-4.10-Linux.tar.gz
	cd HammerDB*/
	./hammerdbcli
	--this verifies all libraries needed
	hammerdbcli> librarycheck
	-- Verify you see below message for PostgreSQL:
	Checking database library for PostgreSQL
	Success ... loaded library Pgtcl for PostgreSQL
	```
5. [Create a schema using HammerDB](https://catalog.us-east-1.prod.workshops.aws/workshops/2a5fc82d-2b5f-4105-83c2-91a1b4d7abfe/en-US/3-intermediate/graviton/task1) with 100 warehouses and 50 virtual users.
6. [Generate load](https://catalog.us-east-1.prod.workshops.aws/workshops/2a5fc82d-2b5f-4105-83c2-91a1b4d7abfe/en-US/3-intermediate/graviton/task7) on the database, using the following HammerDB parameters:
	- Virtual users: 50
		- Ramp-up time: 2 minutes
		- Total test duration: 5 minutes

## Test cases

We are using a preconfigured test environment, [generated a load](https://catalog.us-east-1.prod.workshops.aws/workshops/2a5fc82d-2b5f-4105-83c2-91a1b4d7abfe/en-US/3-intermediate/graviton/task7) using Hammer DB, and observed how Aurora Serverless v2 scales the following test cases:

- Handling irregular workloads
- Consistent high-volume workloads
- Cost-sensitive applications with moderate loads

### Test case 1: Handling irregular workloads

For our first test case, we consider a business with highly unpredictable workloads that can spike at any time, such as a retail website during flash sales or holiday seasons.

We set the minimum and maximum ACUs to a value of 0.5 and 10, respectively. The following figure shows our scaling metrics.

[![Handling irregular workloads](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2024/11/13/Picture1-5.jpg)](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2024/11/13/Picture1-5.jpg)

This setting allows for low idle capacity while still accommodating moderate spikes. The system effectively scaled up during peak demand and reduced the capacity gradually within approximately 10 minutes as the workload decreased, reflecting a smoother transition.

### Test case 2: Consistent high-volume workloads

For this use case, we assume a data analytics company is running batch processing jobs that require consistent high performance and have predictable peak usage times, such as end-of- month report generation.

We set the minimum and maximum ACUs to a value of 4 and 10, respectively. The following figure shows these metrics.

With a higher minimum ACU setting, the database maintained a higher baseline capacity, allowing for quick scaling during peak times, but the scaling down process appeared more abrupt for sustained high volume workflows and completed within approximately 5 minutes.

### Test case 3: Cost-sensitive applications with moderate loads

For this test case, we assume a startup with budget constraints is running a customer-facing application that experiences moderate but steady traffic, such as a software as a service (SaaS) application with a growing user base.

We set the minimum and maximum ACUs to a value of 4 and 20, respectively. The following figure shows these metrics.

The higher maximum ACU setting allowed the database to handle a larger peak load effectively. The scaling down process was relatively slower, compared to the second test case, but this setting offered the highest capacity for handling extreme loads.

## Observations from test cases

Based on our test cases, we made the following observations:

- **Responsive scaling** – In all scenarios, Aurora Serverless v2 demonstrated the ability to scale up quickly in response to increased demand
- **Baseline capacity** – Higher minimum ACU settings (test cases 2 and 3) provided a higher baseline capacity, which may be beneficial for workloads with consistent or predictable loads, reducing latency during initial scaling
- **Scaling down** – The scaling down process appeared to be more gradual when higher ACU limits were set, providing stability and avoiding abrupt capacity reductions that might affect performance

## Recommendations

Based on our findings, we recommend the following:

- **Set a balanced minimum ACU** – Start with a balanced [minimum ACU](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.setting-capacity.html#aurora-serverless-v2.min_capacity_considerations) of 2 or higher to maintain performance without incurring high costs. For details about how to tune the minimum ACUs, refer to [Choosing the minimum Aurora Serverless v2 capacity setting for a cluster](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.setting-capacity.html#aurora-serverless-v2.min_capacity_considerations).
- **Set a scalable maximum ACU** – Configure a scalable maximum ACU (such as 10 ACUs) to accommodate future growth and occasional traffic spikes. For details about how to tune the maximum ACUs, refer to [Choosing the maximum Aurora Serverless v2 capacity setting for a cluster](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.setting-capacity.html#aurora-serverless-v2.max_capacity_considerations).
- **Optimize queries** – Regularly review and optimize the database queries to reduce the load and improve the performance, minimizing the need for higher ACUs.
- **Run performance tests** – Regularly perform load testing to verify that the ACU settings can handle the peak loads without degradation.
- **Use Performance Insights** – Use [Performance Insights](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/USER_PerfInsights.html) to monitor database performance and make data-driven adjustments to the ACU settings.

## Conclusion

Aurora Serverless v2 offers a robust solution for businesses seeking flexible and cost-effective database management. In this post, we highlighted how analyzing workload patterns and fine-tuning the minimum and maximum ACU settings can help you achieve efficient database scaling and maintain performance while managing costs.

We welcome your feedback. Please share your experiences and questions in the comments.

---

### About the Authors

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2024/11/14/priyanka.jpg) **Priyanka** is a Database Specialist Solutions Architect at AWS, focusing on Amazon RDS and Aurora PostgreSQL. She is passionate about designing solutions for customers aiming to modernize their applications and databases in the cloud. She is based in Seattle, Washington. Outside work, Priyanka enjoys reading books and exploring new destinations during her travels.

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2024/11/13/Venu_Koneru_Headshot.jpg) **Venu Koneru** is a Database Specialist Solutions Architect at Amazon Web Services (AWS). His multifaceted career has spanned roles including web developer, data modeler, database administrator, and database architect across the education, travel, and finance sectors. With a wealth of experience in database management and architecture, Venu now leverages his skills at AWS to guide customers through their cloud database modernization journeys.