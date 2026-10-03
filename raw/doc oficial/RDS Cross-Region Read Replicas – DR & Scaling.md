---
title: "RDS Cross-Region Read Replicas – DR & Scaling"
source: "https://jayendrapatil.com/amazon-rds-cross-region-read-replicas/"
author:
  - "[[jayendrapatil]]"
published: 2024-02-13
created: 2026-10-02
description: "RDS Cross-Region Read Replicas create an asynchronously replicated read-only DB instance in a secondary AWS Region."
tags:
  - "clippings"
---
## RDS Cross-Region Read Replicas

![](https://www.youtube.com/watch?v=O3xpmqgn5UE)

- RDS Cross-Region Read Replicas create an asynchronously replicated read-only DB instance in a secondary AWS Region.
- Supported for MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, and Db2
- Cross-Region Read Replicas help to improve
	- disaster recovery capabilities (reduces RTO and RPO),
		- scale read operations into a region closer to end users,
		- migration from a data center in one region to another region
- You can create up to 15 in-Region and cross-Region read replicas combined per source DB instance.
- Due to the limit on the number of access control list (ACL) entries for the source VPC, RDS can’t guarantee more than five cross-Region read replica DB instances.

## RDS Cross-Region Read Replicas Process

- RDS configures the source DB instance as a replication source and setups the specified read replica in the destination AWS Region.
- RDS creates an automated DB snapshot of the source DB instance in the source AWS Region.
- RDS begins a cross-Region snapshot copy for the initial data transfer.
- RDS then uses the copied DB snapshot for the initial data load on the read replica. When the load is complete the DB snapshot copy is deleted.
- RDS starts by replicating the changes made to the source instance since the start of the create read replica operation.

## RDS Cross-Region Read Replicas Considerations

- A source DB instance can have cross-region read replicas in multiple AWS Regions.
- You can create up to 15 in-Region and cross-Region read replicas combined per source DB instance (applies to MySQL, MariaDB, PostgreSQL, Oracle, SQL Server, and Db2).
- Due to the limit on the number of access control list (ACL) entries for the source VPC, RDS can’t guarantee more than five cross-Region read replica DB instances.
- Replica lags are higher for Cross-region replicas. This lag time comes from the longer network channels between regional data centers.
- Read Replica uses the default DB parameter group and DB option group for the specified DB engine when configured from AWS console. For MySQL and Oracle DB engines, you can specify a custom parameter group via CLI/API. For Db2, you must specify a custom parameter group that includes your IBM Site ID and Customer ID.
- Read Replica uses the default security group.
- Cross-Region RDS read replica can be created from a source RDS DB instance that is not a read replica of another RDS DB instance for Db2, Microsoft SQL Server, Oracle, and PostgreSQL DB instances (versions lower than 14.1). This limitation doesn’t apply to MariaDB, MySQL, and PostgreSQL 14.1+ DB instances which support cascading cross-region read replicas.
- Deleting the source for a cross-region read replica will result in
	- read replica promotion for Db2, MariaDB, MySQL, and Oracle DB instances
		- no automatic read replica promotion for PostgreSQL DB instances — the replication status of the read replica is set to `terminated`. You must promote the read replica manually or delete it.
- An encrypted read replica in a different AWS Region requires the source DB instance to be encrypted. A KMS key in the destination Region must be specified.
- You can replicate between the GovCloud (US-East) and GovCloud (US-West) Regions, but not into or out of GovCloud (US).

## Cross-Region Cascading Read Replicas

- MySQL and MariaDB support creating read replicas from other read replicas (cascading), including cross-Region scenarios. This can reduce cross-Region data transfer costs.
- PostgreSQL 14.1+ supports cross-Region cascading read replicas with up to three levels of cascading.
	- You can create a cross-Region replica from the source, then create same-Region replicas from it.
		- You can also create a same-Region replica from the source, then create cross-Region replicas from it.
- Cascading replicas help reduce data transfer costs — only the first replica in the chain incurs cross-Region transfer charges.
- SQL Server, Oracle, and Db2 do **not** support cascading (creating cross-Region replicas from another replica).

## Cross-Region Replication Costs

- Data transferred for cross-Region replication incurs Amazon RDS data transfer charges.
- Cross-Region replication actions that generate data transfer out charges:
	- Initial snapshot transfer when creating the read replica
		- Ongoing data modification replication from source to read replica region
- For MySQL and MariaDB, cascading read replicas can reduce costs — create one cross-Region replica and then create additional replicas from it within the same destination Region (no cross-Region transfer charges for same-Region replication).

### Cross-Region Automated Backup Replication

- RDS supports replicating automated backups (snapshots and transaction logs) to a different AWS Region.
- Enables point-in-time recovery in the secondary Region with RPO of minutes.
- You can replicate up to 20 backups to each destination Region per account.
- Supported for MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, and Db2 (encrypted databases).
- Not supported for Multi-AZ DB clusters.
- Expanded to five additional AWS Regions in April 2025.

### Cross-Region and Cross-Account Snapshot Copy (September 2025)

- Amazon RDS now supports copying RDS and Aurora snapshots across Regions and accounts in a single step.
- Previously required sequential copying (first cross-Region, then cross-account, or vice versa).
- Protects against ransomware attacks and Region outages by enabling isolated backup environments.

### RDS for Db2 Cross-Region Standby Replicas (June 2025)

- RDS for Db2 introduced cross-Region standby replicas for disaster recovery.
- Standby replicas don’t accept user connections but provide faster failover for cross-Region DR scenarios.
- Complements read replicas which are used for read scaling.

### RDS for PostgreSQL Delayed Read Replicas (August 2025)

- Allows specifying a minimum time period that a replica lags behind the source using the `recovery_min_apply_delay` parameter.
- Creates a time buffer to protect against accidental data loss (e.g., table drops, unintended modifications).
- Available with PostgreSQL versions 14.19, 15.14, 16.10, 17.6, and later.
- Can be promoted to become the new primary for quick recovery within minutes.
- Works with both in-Region and cross-Region replicas, including cascaded read replicas.

### RDS for Oracle Cross-Region Replicas with Additional Storage Volumes (January 2026)

- Cross-Region replicas for Oracle now support additional storage volumes.
- Customers can add up to three storage volumes (each up to 64 TiB) in addition to the primary storage volume, totaling up to 256 TiB.
- Enables scaling Oracle workloads without architecture changes while maintaining cross-Region DR.

## AWS Certification Exam Practice Questions

> - Questions are collected from Internet and the answers are marked as per my knowledge and understanding (which might differ with yours).
> - AWS services are updated everyday and both the answers and questions might be outdated soon, so research accordingly.
> - AWS exam questions are not updated to keep up the pace with AWS updates, so even if the underlying feature has changed the question might not be updated
> - Open to further feedback, discussion and correction.

1. Your company has HQ in Tokyo and branch offices worldwide and is using logistics software with a multi-regional deployment on AWS in Japan, Europe, and US. The logistic software has a 3-tier architecture and uses MySQL 5.6 for data persistence. Each region has deployed its database. In the HQ region, you run an hourly batch process reading data from every region to compute cross-regional reports that are sent by email to all offices this batch process must be completed as fast as possible to optimize logistics quickly. How do you build the database architecture to meet the requirements?
	1. **For each regional deployment, use RDS MySQL with a master in the region and a read replica in the HQ region**
		2. For each regional deployment, use MySQL on EC2 with a master in the region and send hourly EBS snapshots to the HQ region
		3. For each regional deployment, use RDS MySQL with a master in the region and send hourly RDS snapshots to the HQ region
		4. For each regional deployment, use MySQL on EC2 with a master in the region and use S3 to copy data files hourly to the HQ region
2. A company has a production database running on RDS for PostgreSQL in us-east-1. They need to provide read access to users in eu-west-1 with minimal latency, and also want the ability to quickly recover from a regional failure. They want to minimize cross-region data transfer costs while having three read replicas available in eu-west-1. What is the most cost-effective architecture?
	1. Create three cross-Region read replicas directly from the source in us-east-1 to eu-west-1
		2. Use Aurora Global Database with three reader instances in eu-west-1
		3. **Create one cross-Region read replica in eu-west-1, then create two cascading read replicas from it in the same Region (requires PostgreSQL 14.1+)**
		4. Enable cross-Region automated backup replication and restore in eu-west-1 when needed
3. A solutions architect needs to design a disaster recovery strategy for an RDS for PostgreSQL database. The strategy must allow recovery from accidental data deletions within a 2-hour window while maintaining a separate cross-Region replica for regional failover. Which combination of features provides this capability? (Select TWO)
	1. Enable Multi-AZ deployment in the primary Region
		2. **Configure a delayed read replica with recovery\_min\_apply\_delay set to 2 hours**
		3. Enable cross-Region automated backup replication
		4. **Create a cross-Region read replica for regional failover**
		5. Use Aurora Global Database with write forwarding
4. Which of the following statements about RDS cross-Region read replicas is correct? (Select TWO)
	1. RDS for SQL Server supports cascading cross-Region read replicas
		2. **RDS for PostgreSQL 14.1+ supports creating cross-Region read replicas from another read replica (cascading)**
		3. When the source is deleted, PostgreSQL cross-Region read replicas are automatically promoted
		4. **You can create up to 15 in-Region and cross-Region read replicas combined per source instance**
		5. Cross-Region read replicas are synchronously replicated
5. A company wants to minimize RTO for their RDS for Db2 database during a regional disaster. They need the replica to be ready for immediate failover without serving read traffic. Which approach should they use?
	1. Create a cross-Region read replica and promote it during failover
		2. **Create a cross-Region standby replica for faster failover**
		3. Enable cross-Region automated backup replication and perform point-in-time recovery
		4. Use AWS DMS for continuous replication to the secondary Region