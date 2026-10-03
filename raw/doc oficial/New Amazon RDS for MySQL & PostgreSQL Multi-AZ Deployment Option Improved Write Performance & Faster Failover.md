---
title: "New Amazon RDS for MySQL & PostgreSQL Multi-AZ Deployment Option: Improved Write Performance & Faster Failover"
source: "https://aws.amazon.com/blogs/aws/amazon-rds-multi-az-db-cluster/"
author:
  - "[[Sébastien Stormacq]]"
published: 2022-03-02
created: 2026-10-02
description: "Today, we are announcing a new Amazon Relational Database Service (Amazon RDS) Multi-AZ deployment option with up to 2x faster transaction commit latency, automated failovers typically under 35 seconds, and readable standby instances. Amazon RDS offers two replication options to enhance availability and performance: Multi-AZ deployments gives high availability and automatic failover. Amazon RDS creates […]"
tags:
  - "clippings"
---
Today, we are announcing a new [Amazon Relational Database Service (Amazon RDS)](https://aws.amazon.com/rds/) Multi-AZ deployment option with up to 2x faster transaction commit latency, automated failovers typically under 35 seconds, and readable standby instances.

Amazon RDS offers two replication options to enhance availability and performance:

- [Multi-AZ deployments](https://aws.amazon.com/rds/features/multi-az/) gives high availability and automatic failover. Amazon RDS creates a storage-level replica of the database in a second Availability Zone. It then synchronously replicates data from the primary to the standby DB instance for high availability. The primary DB instance serves application requests, while the standby DB instance remains ready to take over in case of a failure. Amazon RDS manages all aspects of failure detection, failover, and repair actions so the applications using the database can be highly available.
- [Read replicas](https://aws.amazon.com/rds/features/read-replicas/) allow applications to scale their read operations across multiple database instances. The database engine replicates data asynchronously to the read replicas. The application sends the write requests (`INSERT`, `UPDATE`, and `DELETE`) to the primary database, and read requests (`SELECT`) can be load balanced across read replicas. In case of failure of the primary node, [you can manually promote a read replica to become the new primary database](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.html#USER_ReadRepl.Promote).

Multi-AZ deployments and read replicas serve different purposes. Multi-AZ deployments give your application high availability, durability, and automatic failover. Read replicas give your applications read scalability.

But what about applications that require both high availability with automatic failover and read scalability?

**Introducing the New Amazon RDS Multi-AZ Deployment Option With Two Readable Standby Instances.  
**Starting today, we’re adding a new option to deploy RDS databases. This option combines automatic failover and read replicas: Amazon RDS Multi-AZ with two readable standby instances. This deployment option is available for MySQL and PostgreSQL databases. This is a [database cluster](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/multi-az-db-clusters-concepts.html) with one primary and two readable standby instances. It provides up to 2x faster transaction commit latency and automated failovers, typically under 35 seconds.

The following diagram illustrates such a deployment:

[![Three AZ RDS databases](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2022/02/15/dbblog-1927-image001.png)](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2022/02/15/dbblog-1927-image001.png)

When the new Multi-AZ DB cluster deployment option is enabled, RDS configures a primary database and two read replicas in three distinct Availability Zones. It then monitors and enables failover in case of failure of the primary node.

Just like with traditional read replicas, the database engine replicates data between the primary node and the read replicas. And just like with the Multi-AZ one standby deployment option, RDS automatically detects and manages failover for high availability.

You do not have to choose between high availability or scalability; Multi-AZ DB cluster with two readable standby enables both.

**What Are the Benefits?  
**This new deployment option offers you four benefits over traditional multi-AZ deployments: improved commit latency, faster failover, readable standby instances, and optimized replications.

First, **write operations are faster** when using Multi-AZ DB cluster. The new Multi-AZ DB cluster instances leverage M6gd and R6gd instance types. These instances are powered by [AWS Graviton2 processors](https://aws.amazon.com/ec2/graviton). They are equipped with fast [NVMe](https://en.wikipedia.org/wiki/NVM_Express) SSD for local storage, ideal for high speed and low-latency storage. They deliver up to 40 percent better price performance and 50 percent more local storage GB per vCPU over comparable x86-based instances.

Multi-AZ DB instances use [Amazon Elastic Block Store (Amazon EBS)](https://aws.amazon.com/ebs/) to store the data and the transaction log. The new Multi-AZ DB cluster instances use local storage provided by the instances to store the transaction log. Local storage is optimized to deliver low-latency, high I/O operations per second (IOPS) to applications. Write operations are first written to the local storage transaction log, then flushed to permanent storage on database storage volumes.

Second, **failover operations are typically faster than in the Multi-AZ DB instance** scenario. The read replicas created by the new Multi-AZ DB cluster are full-fledged database instances. The system is designed to fail over as quickly as 35 seconds, plus the time to apply any pending transaction log. In case of failover, the system is fully automated to promote a new primary and reconfigure the old primary as a new reader instance.

Third, **the two standby instances are hot standbys**. Your applications may use the cluster reader endpoint to send their read requests (`SELECT`) to these standby instances. It allows your application to spread the database read load equally between the instances of the database cluster.

And finally, **leveraging local storage for transaction log optimizes replication**. The existing Multi-AZ DB instance replicates all changes at storage-level. The new Multi-AZ DB cluster replicates only the transaction log and uses [a quorum mechanism](https://en.wikipedia.org/wiki/Quorum_\(distributed_computing\)) to confirm at least one standby acknowledged the change. Database transactions are committed synchronously when one of the secondary instances conﬁrms the transaction log is written on its local disk.

**Migrating Existing Databases  
**For those of you having existing RDS databases and willing to take advantage of this new Multi-AZ DB cluster deployment option, you may [take a snapshot of your database](https://docs.aws.amazon.com/en_us/AmazonRDS/latest/UserGuide/USER_CreateSnapshot.html) to create a storage-level backup of your existing database instance. Once the snapshot is ready, you can create a new database cluster, with Multi-AZ DB cluster deployment option, based on this snapshot. Your new Multi-AZ DB cluster will be a perfect copy of your existing database.

**Let’s See It in Action  
**To get started, I point my browser to the [AWS Management Console](https://console.aws.amazon.com/) and navigate to RDS. The Multi-AZ DB cluster deployment option is available for MySQL version 8.0.28 or later and PostgreSQL version 13.4 R1. I select either database engine, and I ensure the version matches the minimum requirements. The rest of the procedure is the same as a standard Amazon RDS database launch.

Under **Deployment options**, I select **PostgreSQL**, version **13.4 R1**, and under **Availability and Durability**, I select **Multi-AZ DB cluster**.

[![Three AZ RDS launch console](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2022/02/15/2022-02-14_19-03-28.jpg)](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2022/02/15/2022-02-14_19-03-28.jpg)

If required, I may choose the set of Availability Zones RDS uses for the cluster. To do so, I create a [DB subnet group](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.WorkingWithRDSInstanceinaVPC.html#USER_VPC.Subnets) and assign the cluster to this subnet group.

Once launched, I verify that three DB instances have been created. I also take note of the two endpoints provided by Amazon RDS: the primary endpoint and one load-balanced endpoint for the two readable standby instances.

[![RDS Three AZ list of instances](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2022/02/15/2022-02-14_20-31-48-1024x549.png)](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2022/02/15/2022-02-14_20-31-48.png)

To test the new cluster, I create an Amazon Linux 2 EC2 instance in the same VPC, within the same security group as the database, and I make sure I attach an IAM role containing the [AmazonSSMManagedInstanceCore](https://console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore$jsonEditor) managed policy. This allows me [to connect to the instance using SSM](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/session-manager.html) instead of SSH.

Once the instance is started, I use SSM to connect to the instance. I install PostgreSQL client tools.

```
sudo amazon-linux-extras enable postgresql13
sudo yum clean metadata
sudo yum install postgresql
```

I connect to the primary DB. I create a table and INSERT a record.

```
psql -h awsnewsblog.cluster-c1234567890r.us-east-1.rds.amazonaws.com -U postgres

postgres=> create table awsnewsblogdemo (id int primary key, name varchar);
CREATE TABLE

postgres=> insert into awsnewsblogdemo (id,name) values (1, 'seb');
INSERT 0 1

postgres=> exit
```

To verify the replication works as expected, I connect to the read-only replica. Notice the `-ro-` in the endpoint name. I check the table structure and enter a `SELECT` statement to confirm the data have been replicated.

```
psql -h awsnewsblog.cluster-ro-c1234567890r.us-east-1.rds.amazonaws.com -U postgres

postgres=> \dt

              List of relations
 Schema |      Name       | Type  |  Owner
--------+-----------------+-------+----------
 public | awsnewsblogdemo | table | postgres
(1 row)

postgres=> select * from awsnewsblogdemo;
 id | name
----+------
  1 | seb
(1 row)

postgres=> exit
```

In the scenario of a failover, the application will be disconnected from the primary database instance. In that case, it is important that your application-level code try to reestablish network connection. After a short period of time, the DNS name of the endpoint will point to the standby instance, and your application will be able to reconnect.

To learn more about Multi-AZ DB clusters, you can refer to our [documentation](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/multi-az-db-clusters-concepts.html).

**Pricing and Availability  
**Amazon RDS Multi-AZ deployments with two readable standbys is generally available in the following Regions: US East (N. Virginia), US West (Oregon), and Europe (Ireland). We will add more regions to this list.

You can use it with MySQL version 8.0.28 or later or PostgreSQL version 13.4 R1.

Pricing depends on the instance type. In US regions, on-demand pricing starts at $0.522 per hour for M6gd instances and $0.722 per hour for R6gd instances. As usual, the Amazon RDS pricing page has the details for [MySQL](https://aws.amazon.com/rds/mysql/pricing/?pg=pr&loc=2) and [PostgreSQL](https://aws.amazon.com/rds/postgresql/pricing/?pg=pr&loc=3).

[You can start to use it today.](https://console.aws.amazon.com/rds/home?region=us-east-1#launch-dbinstance:gdb=false;s3-import=false)