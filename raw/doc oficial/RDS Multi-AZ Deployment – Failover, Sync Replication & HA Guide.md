---
title: "RDS Multi-AZ Deployment – Failover, Sync Replication & HA Guide"
source: "https://jayendrapatil.com/rds-multi-az-deployment/"
author:
  - "[[jayendrapatil]]"
published: 2023-12-17
created: 2026-10-02
description: "RDS Multi-AZ provides synchronous standby replication with automatic failover in 60-120 seconds. Learn how it works, supported engines, and Multi-AZ DB Cluster vs DB Instance."
tags:
  - "clippings"
---
## RDS Multi-AZ Deployment

- [RDS](https://jayendrapatil.com/aws-relational-database-service-rds/) Multi-AZ deployments provide high availability and automatic failover support for DB instances
- Multi-AZ helps improve the durability and availability of a critical system, enhancing availability during planned system maintenance, DB instance failure, and Availability Zone disruption.
- A [Multi-AZ DB instance deployment](https://jayendrapatil.com/aws-rds-multi-az-db-instance/)
	- has one standby DB instance that provides failover support but doesn’t serve read traffic.
		- There is only one row for the DB instance.
		- The value of **Role** is **Instance** or **Primary**.
		- The value of **Multi-AZ** is **Yes**.
- A [Multi-AZ DB cluster deployment](https://jayendrapatil.com/aws-rds-multi-az-db-cluster/)
	- has two standby DB instances that provide failover support and can also serve read traffic.
		- There is a cluster-level row with three DB instance rows under it.
		- For the cluster-level row, the value of **Role** is **Multi-AZ DB cluster**.
		- For each instance-level row, the value of **Role** is **Writer instance** or **Reader instance**.
		- For each instance-level row, the value of **Multi-AZ** is **3 Zones**.

## RDS Multi-AZ DB Instance Deployment

- RDS automatically creates a primary DB Instance and synchronously replicates the data to a standby instance in a different AZ.
- RDS performs an automatic failover to the standby, so that database operations can be resumed as soon as the failover is complete.
- RDS Multi-AZ deployment maintains the same endpoint for the DB Instance after a failover, so the application can resume database operation without the need for manual administrative intervention.
- Multi-AZ is a high-availability feature and NOT a scaling solution for read-only scenarios; a standby replica can’t be used to serve read traffic. To service read-only traffic, use a Multi-AZ DB cluster or a Read Replica.
- Failover times are typically 60–120 seconds for Multi-AZ DB instance deployments.
- Multi-AZ deployments for MariaDB, MySQL, Oracle, PostgreSQL, and RDS Custom for SQL Server DB instances use the Amazon failover technology.
- Microsoft SQL Server DB instances use SQL Server Database Mirroring (DBM), Always On Availability Groups (AGs), or block-level replication depending on the edition and version.
	- Database Mirroring is deprecated by Microsoft and will be removed in a future version of SQL Server. AWS recommends migrating to Always On AGs.
		- **New (Nov 2025):** RDS for SQL Server Web Edition now supports Multi-AZ using block-level replication, providing enterprise-grade availability without requiring Enterprise or Standard Edition licensing.
		- **New (Jan 2026):** RDS for SQL Server now supports differential and transaction log restores for Multi-AZ instances and instances with read replicas in the same region.

### ENA Express for Multi-AZ Replication (May 2026)

- Amazon RDS Multi-AZ instances now use **ENA Express** for replication traffic between Availability Zones.
- ENA Express uses AWS’s **Scalable Reliable Datagram (SRD)** protocol to optimize network performance.
- Delivers up to **25 Gbps single-flow bandwidth** for cross-AZ replication traffic (previously up to 5 Gbps).
- Leverages advanced congestion control and multi-pathing capabilities, reducing latency variability for Multi-AZ deployments.
- Results in increased write throughput and lower write latencies for write-intensive database workloads.
- Available at **no additional charge** for RDS for MariaDB, MySQL, PostgreSQL, Db2, and Oracle.
- Supported for db.m6i.8xlarge and above instance classes in a Multi-AZ configuration.
- To enable on existing instances, perform a stop/start or scale compute action.

## RDS Multi-AZ DB Cluster Deployment

- [RDS](https://jayendrapatil.com/aws-relational-database-service-rds/) Multi-AZ DB cluster deployment is a high-availability deployment mode of RDS with two readable standby DB instances.
- RDS Multi-AZ DB cluster has a writer DB instance and two reader DB instances in three separate AZs in the same AWS Region.
- With a Multi-AZ DB cluster, RDS semi-synchronously replicates data from the writer DB instance to both of the reader DB instances using the DB engine’s native replication capabilities.
- Multi-AZ DB clusters provide high availability, increased capacity for read workloads, and lower write latency when compared to Multi-AZ DB instance deployments.
- If an event of an outage, RDS manages failover from the writer DB instance to one of the reader DB instances. RDS does this based on which reader DB instance has the most recent change record.
- Failover times are **typically under 35 seconds** for Multi-AZ DB clusters, compared to 60-120 seconds for Multi-AZ DB instance deployments.
- Multi-AZ DB clusters support up to **2x faster transaction commit latency** compared to Multi-AZ DB instance deployments.
- Multi-AZ DB clusters are currently supported only for the **MySQL** (8.0, 8.4) and **PostgreSQL** DB engines.

![RDS Mulit-AZ DB Cluster](https://jayendrapatil.com/wp-content/uploads/2022/07/Multi-AZ-DB-Cluster.png)

### Multi-AZ DB Cluster – Recent Enhancements (2024-2026)

- **AWS Backup for Multi-AZ Clusters – 17 Additional Regions (Mar 2026):** AWS Backup expanded support for RDS Multi-AZ clusters with comprehensive data protection capabilities including automated lifecycle management and immutable backups via AWS Backup Vault Lock.
- **IAM Database Authentication (Oct 2024):** Multi-AZ DB clusters with two readable standbys now support AWS IAM database authentication, enabling centralized access management without storing password-based login credentials in the database.
- **Security Certificate Rotation (Mar 2024):** Supports database security certificate rotation for an extra layer of security by validating connections made to RDS database instances and clusters.
- **C6gd Instance Support (Mar 2024):** Multi-AZ DB clusters now support C6gd database instances for enhanced compute performance.
- **Minor Version Upgrades:** Downtime for minor version upgrades is typically under 35 seconds, and can be further reduced to under 1 second by using the AWS Advanced JDBC Wrapper Driver or RDS Proxy.

## Multi-AZ DB Instance vs Multi-AZ DB Cluster

![RDS Multi-AZ DB Instance vs DB Cluster](https://jayendrapatil.com/wp-content/uploads/2023/12/RDS-Multi-AZ-Comparison.jpg)

| Feature | Multi-AZ DB Instance | Multi-AZ DB Cluster |
| --- | --- | --- |
| Standby Instances | 1 (not readable) | 2 (readable) |
| Replication | Synchronous | Semi-synchronous (engine native) |
| Failover Time | 60-120 seconds | Typically under 35 seconds |
| Read Traffic | Not supported on standby | Supported on reader instances |
| Write Latency | Standard | Up to 2x faster transaction commits |
| Supported Engines | MySQL, PostgreSQL, MariaDB, Oracle, SQL Server | MySQL, PostgreSQL only |
| IAM DB Authentication | Supported | Supported (since Oct 2024) |
| ENA Express | Supported (since May 2026) | N/A (uses engine-native replication) |

## Improving Multi-AZ Failover with RDS Proxy and AWS Drivers

- **RDS Proxy** can reduce failover times by up to 66% by eliminating DNS propagation delays.
	- During failover, applications typically wait 35+ seconds for DNS CNAME changes to propagate.
		- RDS Proxy continuously monitors both primary and standby instances, bypassing DNS propagation entirely.
		- Supports Multi-AZ DB instance deployments for MySQL, MariaDB, and PostgreSQL.
		- Also supports Multi-AZ DB cluster deployments with two readable standbys.
- **AWS Advanced JDBC/Python Wrapper Drivers** provide topology-aware failover for Multi-AZ DB clusters.
	- The drivers monitor the DB cluster topology and detect writer changes automatically.
		- Can switch connections to a new writer in approximately **1 second or less** when there is no replica lag.
		- Reduces switchover and failover times to single-digit seconds, compared to tens of seconds for standard drivers.
		- Supports failover management, monitoring, IAM/Secrets Manager authentication, and load distribution plugins.

## RDS Multi-AZ vs Read Replicas

![RDS Mulit-AZ vs Multi-Region vs Read Replicas](https://jayendrapatil.com/wp-content/uploads/2020/08/RDS-Mulit-AZ-vs-Multi-Region-vs-Read-Replicas.png)

## RDS Extended Support and Multi-AZ

- Starting March 1, 2024, Amazon RDS automatically charges for **RDS Extended Support** when running a major version past its end of standard support date.
- Extended Support provides up to three additional years of critical security and bug fixes beyond a major version’s end of standard support date.
- Key end-of-standard-support dates:
	- PostgreSQL 11 – February 29, 2024
		- PostgreSQL 12 – February 28, 2025
		- PostgreSQL 13 – February 28, 2026
		- MySQL 5.7 – February 29, 2024
- Extended Support pricing increases over time (Year 1 → Year 2 → Year 3), making timely upgrades important for Multi-AZ deployments where charges apply to both primary and standby instances.

## AWS Certification Exam Practice Questions

> - Questions are collected from Internet and the answers are marked as per my knowledge and understanding (which might differ with yours).
> - AWS services are updated everyday and both the answers and questions might be outdated soon, so research accordingly.
> - AWS exam questions are not updated to keep up the pace with AWS updates, so even if the underlying feature has changed the question might not be updated
> - Open to further feedback, discussion and correction.

1. A company is deploying a new two-tier web application in AWS. The company has limited staff and requires high availability, and the application requires complex queries and table joins. Which configuration provides the solution for the company’s requirements?
	1. MySQL Installed on two Amazon EC2 Instances in a single Availability Zone (does not provide High Availability out of the box)
		2. **Amazon RDS for MySQL with Multi-AZ**
		3. Amazon ElastiCache (Just a caching solution)
		4. Amazon DynamoDB (Not suitable for complex queries and joins)
2. What would happen to an RDS (Relational Database Service) multi-Availability Zone deployment if the primary DB instance fails?
	1. IP of the primary DB Instance is switched to the standby DB Instance.
		2. A new DB instance is created in the standby availability zone.
		3. **The canonical name record (CNAME) is changed from primary to standby.**
		4. The RDS (Relational Database Service) DB instance reboots.
3. Will my standby RDS instance be in the same Availability Zone as my primary?
	1. Only for Oracle RDS types
		2. Yes
		3. Only if configured at launch
		4. **No**
4. Is creating a Read Replica of another Read Replica supported?
	1. Only in certain regions
		2. **Only with MySQL based RDS** \[*Note: As of 2022+, cascading Read Replicas are now also supported for PostgreSQL (version 14.1+). Both MySQL and PostgreSQL support up to three levels of cascading replicas.*\]
		3. Only for Oracle RDS types
		4. No
5. A user is planning to set up the Multi-AZ feature of RDS. Which of the below mentioned conditions won’t take advantage of the Multi-AZ feature?
	1. Availability zone outage
		2. A manual failover of the DB instance using Reboot with failover option
		3. **Region outage**
		4. When the user changes the DB instance’s server type
6. When you run a DB Instance as a Multi-AZ deployment, the “\_\_\_\_\_” serves database writes and reads
	1. secondary
		2. backup
		3. stand by
		4. **primary** \[*Note: For Multi-AZ DB clusters, reader instances can also serve read traffic.*\]
7. When running my DB Instance as a Multi-AZ deployment, can I use the standby for read or write operations?
	1. Yes
		2. Only with MSSQL based RDS
		3. Only for Oracle RDS instances
		4. **No** \[*Note: This applies to Multi-AZ DB instance deployments. Multi-AZ DB cluster deployments DO allow reader instances to serve read traffic.*\]
8. Read Replicas require a transactional storage engine and are only supported for the \_\_\_\_\_\_\_\_\_ storage engine
	1. OracleISAM
		2. MSSQLDB
		3. **InnoDB**
		4. MyISAM
9. A user is configuring the Multi-AZ feature of an RDS DB. The user came to know that this RDS DB does not use the AWS technology, but uses server mirroring to achieve replication. Which DB is the user using right now?
	1. MySQL
		2. Oracle
		3. **MS SQL** \[*Note: SQL Server now also supports Always On Availability Groups and block-level replication for Multi-AZ, depending on the edition and version. Database Mirroring is deprecated by Microsoft.*\]
		4. PostgreSQL
10. If you have chosen Multi-AZ deployment, in the event of a planned or unplanned outage of your primary DB Instance, Amazon RDS automatically switches to the standby replica. The automatic failover mechanism simply changes the \_\_\_\_\_\_ record of the main DB Instance to point to the standby DB Instance.
	1. DNAME
		2. **CNAME**
		3. TXT
		4. MX
11. When automatic failover occurs, Amazon RDS will emit a DB Instance event to inform you that automatic failover occurred. You can use the \_\_\_\_\_ to return information about events related to your DB Instance
	1. FetchFailure
		2. DescriveFailure
		3. **DescribeEvents**
		4. FetchEvents
12. The new DB Instance that is created when you promote a Read Replica retains the backup window period.
	1. **TRUE**
		2. FALSE
13. Will I be alerted when automatic failover occurs?
	1. **Only if SNS configured**
		2. No
		3. Yes
		4. Only if Cloudwatch configured
14. Can I initiate a “forced failover” for my MySQL Multi-AZ DB Instance deployment?
	1. Only in certain regions
		2. Only in VPC
		3. **Yes**
		4. No
15. A user is accessing RDS from an application. The user has enabled the Multi-AZ feature with the MS SQL RDS DB. During a planned outage how will AWS ensure that a switch from DB to a standby replica will not affect access to the application?
	1. RDS will have an internal IP which will redirect all requests to the new DB
		2. **RDS uses DNS to switch over to standby replica for seamless transition**
		3. The switch over changes Hardware so RDS does not need to worry about access
		4. RDS will have both the DBs running independently and the user has to manually switch over
16. Which of the following is part of the failover process for a Multi-AZ Amazon Relational Database Service (RDS) instance?
	1. The failed RDS DB instance reboots.
		2. The IP of the primary DB instance is switched to the standby DB instance.
		3. **The DNS record for the RDS endpoint is changed from primary to standby.**
		4. A new DB instance is created in the standby availability zone.
17. Which of these is not a reason a Multi-AZ RDS instance will failover?
	1. An Availability Zone outage
		2. A manual failover of the DB instance was initiated using Reboot with failover
		3. To autoscale to a higher instance class (Refer [link](https://aws.amazon.com/rds/features/multi-az/))
		4. **Master database corruption occurs**
		5. The primary DB instance fails
18. How does Amazon RDS multi Availability Zone model work?
	1. **A second, standby database is deployed and maintained in a different availability zone from master, using synchronous replication.** (Refer [link](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZSingleStandby.html))
		2. A second, standby database is deployed and maintained in a different availability zone from master using asynchronous replication.
		3. A second, standby database is deployed and maintained in a different region from master using asynchronous replication.
		4. A second, standby database is deployed and maintained in a different region from master using synchronous replication.
19. A user is using a small MySQL RDS DB. The user is experiencing high latency due to the Multi AZ feature. Which of the below mentioned options may not help the user in this situation?
	1. Schedule the automated back up in non-working hours
		2. Use a large or higher size instance
		3. Use PIOPS
		4. **Take a snapshot from standby Replica**
20. What is the charge for the data transfer incurred in replicating data between your primary and standby?
	1. **No charge. It is free.**
		2. Double the standard data transfer charge
		3. Same as the standard data transfer charge
		4. Half of the standard data transfer charge
21. A user has enabled the Multi AZ feature with the MS SQL RDS database server. Which of the below mentioned statements will help the user understand the Multi AZ feature better?
	1. In a Multi AZ, AWS runs two DBs in parallel and copies the data asynchronously to the replica copy
		2. In a Multi AZ, AWS runs two DBs in parallel and copies the data synchronously to the replica copy
		3. **In a Multi AZ, AWS runs just one DB but copies the data synchronously to the standby replica**
		4. AWS MS SQL does not support the Multi AZ feature
22. A company needs a highly available database solution with the lowest possible failover time. The application uses MySQL. Which RDS deployment option provides the fastest failover? \[*New Question*\]
	1. Multi-AZ DB instance deployment (Failover takes 60-120 seconds)
		2. **Multi-AZ DB cluster deployment (Failover typically under 35 seconds)**
		3. Single-AZ with Read Replicas (No automatic failover)
		4. Cross-Region Read Replica (Manual promotion required)
23. Which of the following is a benefit of RDS Multi-AZ DB cluster deployment over Multi-AZ DB instance deployment? (Choose TWO) \[*New Question*\]
	1. **Reader instances can serve read traffic**
		2. Supports all RDS database engines
		3. **Up to 2x faster transaction commit latency**
		4. Uses synchronous replication
		5. Supports Oracle and SQL Server engines
24. A company runs a write-intensive MySQL workload on RDS Multi-AZ. They notice high replication latency between the primary and standby instances. What recent AWS feature can help reduce this latency? \[*New Question*\]
	1. Enable Read Replicas for write offloading
		2. **Enable ENA Express for Multi-AZ replication which uses AWS SRD protocol for up to 25 Gbps single-flow bandwidth**
		3. Switch to asynchronous replication
		4. Enable Enhanced Monitoring
25. A company wants to minimize application downtime during RDS Multi-AZ failover events. The current failover takes 60-120 seconds due to DNS propagation. What should they implement? \[*New Question*\]
	1. Enable Multi-AZ DB cluster deployment
		2. Use a shorter DNS TTL on the application side
		3. **Use Amazon RDS Proxy which eliminates DNS propagation delay by continuously monitoring both primary and standby instances**
		4. Implement application-level retry logic with exponential backoff