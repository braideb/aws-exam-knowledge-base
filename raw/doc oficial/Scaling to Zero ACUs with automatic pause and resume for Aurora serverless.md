---
title: "Scaling to Zero ACUs with automatic pause and resume for Aurora serverless"
source: "https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2-auto-pause.html"
author:
published:
created: 2026-10-02
description: "You can specify that Aurora serverless DB instances scale down to zero ACUs and automatically pause, if they don't have any connections initiated by user activity within a specified time period. You do so by specifying a minimum ACU value of zero for your DB cluster. You aren't charged for instance capacity while an instance is in the paused state. Enabling the automatic pause and resume feature (auto-pause) for Aurora clusters that are lightly used or have extended periods of inactivity can help you to manage costs for your database fleet."
tags:
  - "clippings"
---
Scaling to Zero ACUs with automatic pause and resume for Aurora serverless - Amazon Aurora

You can specify that Aurora serverless DB instances scale down to zero ACUs and automatically pause, if they don't have any connections initiated by user activity within a specified time period. You do so by specifying a minimum ACU value of zero for your DB cluster. You aren't charged for instance capacity while an instance is in the paused state. Enabling the automatic pause and resume feature (auto-pause) for Aurora clusters that are lightly used or have extended periods of inactivity can help you to manage costs for your database fleet.

###### Note

The auto-pause feature is available for Aurora serverless with both Aurora PostgreSQL and Aurora MySQL. You might need to upgrade your Aurora database engine version to take advantage of this feature. For the engine versions where a minimum capacity of 0 ACUs is available, see [Aurora serverless capacity](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.how-it-works.html#aurora-serverless-v2.how-it-works.capacity).

## Overview of the Aurora serverless auto-pause feature

Aurora serverless DB instances can automatically pause after a period with no user connections, and automatically resume when a connection request arrives. The Aurora serverless automatic pause/resume feature helps to manage costs for systems that don't have a stringent service level objective (SLO). For example, you might enable this feature for clusters used for development and testing, or for internal applications where a brief pause is acceptable while the database resumes. If your workload has periods of inactivity and can tolerate slight delays in connecting while the instance resumes, consider using auto-pause with your Aurora serverless instances to reduce costs.

You control this behavior by specifying whether the Aurora serverless DB instances in a cluster can automatically pause or not, and how long each instance must be idle before it pauses. To enable the auto-pause behavior for all the Aurora serverless DB instances in an Aurora cluster, you set the minimum capacity value for the cluster to zero ACUs.

The cost-savings benefits of the auto-pause feature are similar to using the stop/start cluster feature. Auto-pause for Aurora serverless has the additional benefits of a faster resume than starting a stopped cluster, and automating the process of determining when to pause and resume each DB instance.

The auto-pause feature also provides additional granularity in controlling costs for compute resources within your cluster. You can enable some reader instances to pause even while the writer instance and other readers in the cluster remain active at all times. You do so by assigning the reader instances that can pause independently of other instances a failover priority in the range 2-15.

The writer instances and all reader instances with failover priority 0 and 1 always pause and resume at the same time. Thus, the instances in this group pause after none of them have any connections for the specified time interval.

Aurora DB clusters can contain a combination of writer and reader DB instances and provisioned and Aurora serverless DB instances. Therefore, to use this feature effectively, it's helpful to understand the following aspects of the auto-pause mechanism:

- The circumstances when a DB instance might automatically pause.
- When a DB instance might be prevented from pausing. For example, enabling some Aurora features or performing certain kinds of operations on the cluster might prevent instances from pausing, even without any connections to those instances.
- The consequences for monitoring and billing while an instance is paused.
- What actions cause a DB instance to resume processing.
- How the capacity of a DB instance changes around the time of the pause and resume events.
- How to control the idle interval before a DB instance pauses.
- How to code application logic to handle the period while a DB instance is resuming processing.

## Prerequisites and Limitations for the Aurora serverless auto-pause feature

Before using the auto-pause feature, check which engine versions to use. Also, check whether auto-pause works in combination with the other Aurora features you intend to use. You can't turn on auto-pause if you're using an engine version that doesn't support it. For incompatible features, you won't get any error if you use them in combination with auto-pause. If the cluster is using any incompatible features or settings, the Aurora serverless instances won't automatically pause.

- If you're using Aurora PostgreSQL, the database engine must be running at least version 16.3, 15.7, 14.12, or 13.15.
- If you're using Aurora MySQL, the database engine must be running version 3.08.0 or higher.
- For the full list of engine versions and AWS Regions where this feature is available, see [Supported Regions and Aurora DB engines for Aurora serverless](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.Aurora_Fea_Regions_DB-eng.Feature.ServerlessV2.html).
- When an Aurora serverless instance resumes, its capacity might be lower than it was when the instance was paused.

Certain conditions or settings prevent Aurora serverless instances from automatically pausing. For more information, see [Situations where Aurora serverless doesn't auto-pause](#auto-pause-whynot).

## Turning the auto-pause feature on and off

You can turn the auto-pause feature on and off at the cluster level. To do so, you use the same procedures as when you adjust the minimum and maximum capacity for the cluster. The auto-pause feature is represented by a minimum capacity of 0 ACUs.

### Turning on auto-pause for Aurora serverless instances in a cluster

Follow the procedure in [Setting the Aurora serverless capacity range for a cluster](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2-administration.html#aurora-serverless-v2-setting-acus). For the minimum capacity, choose 0 ACUs. When you choose a minimum capacity of 0 ACUs, you can also specify the length of time for the instance to be idle before it's automatically paused.

The following CLI example shows how you might create an Aurora cluster with the auto-pause feature enabled and the auto-pause interval set to ten minutes (600 seconds).

```
aws rds create-db-cluster \
    --db-cluster-identifier my-serverless-v2-cluster \
    --region eu-central-1 \
    --engine aurora-mysql \
    --engine-version 8.0 \
    --serverless-v2-scaling-configuration MinCapacity=0,MaxCapacity=4,SecondsUntilAutoPause=600 \
    --master-username myuser \
    --manage-master-user-password
```

The following CLI example shows how you might turn on the auto-pause feature for an existing Aurora cluster. This example sets the auto-pause interval to one hour (3600 seconds).

```
aws rds modify-db-cluster --db-cluster-identifier serverless-v2-cluster \
  --serverless-v2-scaling-configuration MinCapacity=0,MaxCapacity=80,SecondsUntilAutoPause=3600

aws rds describe-db-clusters --db-cluster-identifier serverless-v2-cluster \
  --query 'DBClusters[*].ServerlessV2ScalingConfiguration|[0]'
{
    "MinCapacity": 0,
    "MaxCapacity": 80.0,
    "SecondsUntilAutoPause": 3600
}
```

### Configurable Aurora serverless auto-pause timeout interval

The timeout interval is represented in the `ServerlessV2ScalingConfiguration` attribute of your Aurora cluster. You can specify this interval in the AWS Management Console when creating or modifying an Aurora cluster, if the minimum capacity is set to zero ACUs. You can specify it in the AWS CLI by using the `--serverless-v2-scaling-configuration` parameter when creating or modifying an Aurora cluster. You can specify it in the RDS API by using the `ServerlessV2ScalingConfiguration` parameter when creating or modifying an Aurora cluster.

The minimum interval that you can set is 300 seconds (five minutes). That's the default if you don't specify an interval. The maximum interval that you can set is 86,400 seconds (one day).

Suppose that you turn off the auto-pause feature for a cluster by changing the cluster's minimum capacity to a nonzero value. In that case, the interval property is removed from the `ServerlessV2ScalingConfiguration` attribute. The absence of that property provides an extra confirmation that the auto-pause feature is turned off for that cluster. If you later turn auto-pause back on, you can specify any custom interval again at that time.

### Resuming an auto-paused Aurora serverless instance

- When you connect to a paused Aurora serverless instance, it automatically resumes and accepts the connection.
- A connection attempt that doesn't include valid credentials still causes the DB instance to resume.
- If you connect through the writer endpoint, Aurora resumes the writer DB instance if it's auto-paused. At the same time, Aurora resumes any auto-paused reader instances that have failover priority 0 or 1, meaning their capacity is tied to the capacity of the writer instance.
- If you connect through the reader endpoint, Aurora chooses a reader instance randomly. If that reader instance is paused, Aurora resumes it. Aurora also resumes the writer instance first, because the writer instance must always be active if any reader instances are active. When Aurora resumes that writer instance, that also causes any reader instances in failover promotion tiers zero and one to resume.
- If you send a request to your cluster through the RDS Data API, Aurora resumes the writer instance if it's paused. Then Aurora processes the Data API request.
- When you change the value of a configuration parameter in a DB cluster parameter group, Aurora automatically resumes any paused Aurora serverless instances in all clusters that use that cluster parameter group. Similarly, when you change a parameter value in a DB parameter group, Aurora automatically resumes any paused Aurora serverless instances that use that DB parameter group. The same automatic resume behavior applies when you modify a cluster to assign a different cluster parameter group, or when you modify an instance to assign a different DB parameter group.
- Performing a backtrack request automatically resumes the Aurora serverless writer instance if it's paused. Aurora processes the backtrack request after the writer instance resumes. You can backtrack to a time during which an Aurora serverless instance was paused.
- Taking a cluster snapshot or deleting a snapshot doesn't cause any Aurora serverless instances to resume.
- Creating an Aurora clone causes Aurora to resume the writer instance of the cluster that's being cloned.
- If a paused instance receives a large number of connection requests before it finishes resuming, some sessions might be unable to connect. We recommend implementing retry logic for connections to Aurora clusters that have some Aurora serverless instances with auto-pause enabled. For example, you might retry any failed connection three times.
- Aurora can perform some types of minor internal maintenance without waking up an instance. However, some types of maintenance that happen during the cluster's maintenance window do require Aurora to resume the instance. When the maintenance is finished, the instance is automatically paused again if there's no more activity after the specified interval.
	###### Note
	Aurora doesn't automatically resume a paused instance for engine-specific scheduled jobs, such as those in the PostgreSQL `pg_cron` extension or the MySQL event scheduler. To ensure that such jobs run, initiate a connection manually to the instance before the scheduled time. Aurora doesn't queue any jobs where the scheduled time occurs while the DB instance is paused. Such jobs are skipped when the instance resumes later.
- If the Aurora cluster undergoes a failover while an Aurora serverless instance is auto-paused, Aurora might resume an instance and then promote that instance to be the writer. The same might happen if one or more DB instances are removed from the cluster while an instance is paused. In this case, the instance becomes the writer immediately when it's resumed.
- Operations that change properties of the cluster also cause any auto-paused Aurora serverless instances to resume. For example, an auto-paused instance resumes for operations such as the following:
	- Changing the scaling range of the cluster.
	- Upgrading the engine version of the cluster.
	- Describing or downloading log files from a paused instance. You can examine historical log data from paused instances by enabling log uploads to CloudWatch and analyzing the logs through CloudWatch.

### Turning off auto-pause for Aurora serverless instances in a cluster

Follow the procedure in [Setting the Aurora serverless capacity range for a cluster](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2-administration.html#aurora-serverless-v2-setting-acus). For the minimum capacity, choose a value of 0.5 or greater. When you turn off the auto-pause feature, the interval for the instance to be idle is reset. If you turn auto-pause on again, you specify a new timeout interval.

The following CLI example shows how you might turn off the auto-pause feature for an existing Aurora cluster. The `describe-db-clusters` output shows that the `SecondsUntilAutoPause` attribute is removed when the minimum capacity is set to a nonzero value.

```
aws rds modify-db-cluster --db-cluster-identifier serverless-v2-cluster \
  --serverless-v2-scaling-configuration MinCapacity=2,MaxCapacity=80

aws rds describe-db-clusters --db-cluster-identifier serverless-v2-cluster \
  --query 'DBClusters[*].ServerlessV2ScalingConfiguration|[0]'
{
    "MinCapacity": 2,
    "MaxCapacity": 80.0
}
```

## How the Aurora serverless auto-pause feature works

You can use the following information to plan your usage of the auto-pause feature. Understanding the circumstances where instances pause, resume, or stay active can help you balance the tradeoffs between availability, responsiveness, and cost savings.

### What happens when Aurora serverless instances pause

When an Aurora serverless DB instance pauses after a period with no connections:

- Aurora begins pausing the instance after the specified interval elapses with no connections to the instance, regardless of how many ACUs the instance has at the time.
- The pause mechanism isn't instantaneous. An Aurora serverless instance that's about to be auto-paused might wait briefly to catch up with all the changes to Aurora storage.
- The instance charges for that instance are put on hold. The `ServerlessV2Usage` metric has a value of 0 while the instance is paused.
- The status value for the instance doesn't change. The status is still shown as "available".
- The instance stops writing to the database log files. It stops sending metrics to CloudWatch, other than registering zero percent for `CPUUtilization` and `ACUUtilization`, and zero for `ServerlessDatabaseCapacity`.
- Aurora emits events when an Aurora serverless DB instance begins pausing, finishes pausing, and if the pause mechanism is interrupted or is unsuccessful. For details about these events, see [DB instance events](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/USER_Events.Messages.html#USER_Events.Messages.instance).

### What happens when auto-paused Aurora serverless instances resume

When an Aurora serverless DB instance resumes after being automatically paused, the following conditions apply:

- Any parameter changes that are in `pending-reboot` changes are applied when the instance resumes.
- Aurora emits instance-level events when each Aurora serverless DB instance begins resuming, finishes resuming, and if the instance can't resume for some reason. For details about these events, see [DB instance events](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/USER_Events.Messages.html#USER_Events.Messages.instance).
- Any requested connections are established after the DB instance finishes resuming. Because the typical time to resume might be approximately 15 seconds, we recommend that you adjust any client timeout settings to be longer than 15 seconds. For example, in your JDBC driver settings you might adjust the values for the `connectTimeout` and `sslResponseTimeout` settings to be longer than 15 seconds.

###### Note

If an Aurora serverless instance remains paused more than 24 hours, Aurora can put the instance into a deeper sleep that takes longer to resume. In that case, the resume time can be 30 seconds or longer, roughly equivalent to doing a reboot of the instance. If your database has periods of inactivity that last longer than a day, we recommend setting connection timeouts to 30 seconds or more.

### How instance billing works for auto-paused Aurora serverless clusters

While an Aurora serverless instance is auto-paused, its instance charge is zero. The `ServerlessV2Usage` metric is zero during that period. AWS still charges for Aurora storage and other aspects of the cluster that aren't tied to that specific DB instance.

### Situations where Aurora serverless doesn't auto-pause

- If the minimum capacity value for your DB cluster is higher than zero ACUs, the Aurora serverless instances in the cluster don't automatically pause. If you have existing clusters with Aurora serverless instances from before the auto-pause feature was available, the lowest minimum capacity setting was 0.5 ACUs. To use the auto-pause feature with such clusters, modify the minimum capacity setting to zero ACUs.
- If any user-initiated connections are open to an Aurora serverless instance, the instance won't pause. The instance also won't pause while activities such as patching and upgrades are in progress. Administrative connections that Aurora uses for health checks aren't counted as activity and don't prevent the instance from pausing.
- In an Aurora PostgreSQL cluster has logical replication enabled, or an Aurora MySQL cluster that has binlog replication enabled, the writer instance and any reader instances in failover promotion tiers zero and one don't automatically pause. Aurora performs a constant minimal amount of activity for checking the health of the replication connection.
	For clusters with replication enabled, you can still have Aurora reader instances in the source cluster that auto-pause. To do so, set their failover priority to a value other than zero or one. That way, they can be paused independently of the writer instance.
- If your Aurora cluster has an associated RDS Proxy, the proxy maintains an open connection to each DB instance in the cluster. Thus, any Aurora serverless instances in such a cluster won't automatically pause.
- If your cluster is the primary cluster in an Aurora global database, Aurora doesn't automatically pause the Aurora serverless writer instance. That's because a constant level of activity is needed on the writer instance to manage the other clusters in the global database. Because the writer instance remains active, any Aurora serverless reader instances with failover priority zero or one also don't auto-pause.
- Aurora serverless instances in the secondary clusters of an Aurora Global Database don't automatically pause. If a DB cluster is promoted to a standalone cluster, then the auto-pause feature becomes effective if all the other conditions are met.
- In a cluster with an associated zero-ETL integration to Redshift, the writer instance and any reader instances in failover promotion tiers zero and one don't automatically pause.
- In addition to activity on the main database port for the instance, if an Aurora PostgreSQL instance has the Babelfish feature enabled, any connections and activity on the T-SQL port prevent the instance from being auto-paused.

### How auto-pause works with the cluster stop/start feature

You can stop and start an Aurora cluster when the auto-pause feature is enabled. It doesn't matter if some instances are paused. When you start the cluster again, any paused Aurora serverless instances are automatically resumed.

While an Aurora cluster is stopped, any paused Aurora serverless instances don't automatically resume based on attempts to connect. Once the cluster is started again, the usual mechanisms for pausing and resuming Aurora serverless instances apply.

### How maintenance and upgrades work for auto-paused Aurora serverless clusters

- While an Aurora serverless instance is auto-paused, if you attempt to upgrade the Aurora cluster, Aurora resumes the instance and upgrades it.
- Aurora periodically resumes any auto-paused Aurora serverless instances to perform maintenance such as minor version upgrades and changes to properties such as parameter groups.
- After an Aurora serverless instance wakes up for an administrative operation such as an upgrade or applying maintenance, Aurora waits at least 20 minutes before pausing that instance again. That's to allow any background operations to finish. The twenty-minute period also avoids pausing and resuming the instance multiple times if the instance undergoes multiple administrative operations in succession.

## How Aurora serverless auto-pause works for different types of Aurora clusters

The considerations for the auto-pause feature depend on how many instances are in your Aurora cluster, the failover promotion tiers of the reader instances, and whether all the instances are Aurora serverless or a combination of Aurora serverless and provisioned.

### Recommended Aurora cluster layouts when using auto-pause

When the auto-pause feature is enabled, you can arrange your Aurora cluster for the right balance of high availability, fast response, and scalability to suit your use case. You do so by choosing the combination of Aurora serverless instances, provisioned instances, and failover promotion tiers for the DB instances in your cluster.

The following types of configurations demonstrate different tradeoffs between high availability and cost optimization for your cluster:

- For a development and test system, you can set up a single-AZ DB cluster with an Aurora serverless DB instance. The single instance serves all read and write requests. When the cluster isn't used for significant intervals of time, the DB instance pauses. At that point, the DB compute costs for your cluster are also paused.
- For a system running an application where high availability is a priority, but the cluster still has periods where it's entirely idle, you can set up a multi-AZ cluster where both the writer and reader DB instances are Aurora serverless. Set the reader instance to failover priority zero or one, so that the writer and reader instance both pause and resume at the same time. Now you get the benefit of fast failover while the cluster is active. When the cluster remains idle for longer than the auto-pause threshold, the DB instance charges for both of the instances are paused. When the cluster resumes processing, the first database session takes a brief time to connect.
- Suppose your cluster is constantly active with some minimal amount of activity, and requires fast response for any connection. In that case, you can create a cluster with more than one Aurora serverless reader instance, and decouple the capacities of some reader instances from the writer. Specify failover priority zero or one for the writer instance and one reader instance. Specify a priority greater than one for the other reader instances. That way, the reader instances in the higher priority tiers can auto-pause, even while the writer and one of the readers remain active.
	In this case, you can employ some other techniques to ensure that the cluster stays continuously available while still scaling down to a low capacity during idle periods:
	- You can use provisioned instances for the writer and the priority 0 or 1 reader. That way, two DB instances never auto-pause and are always available to serve database traffic and to perform failovers.
	- You can set up a custom endpoint that includes the Aurora serverless instances in the higher priority tiers, but not the writer or the promotion tier 0 or 1 readers. That way, you can direct read-only sessions that aren't sensitive to latency to the readers that might be auto-paused. You can avoid using the reader endpoint for such requests, because Aurora might direct reader endpoint connections to the always-awake reader instance, or to one of the auto-paused instances. Using the custom endpoint lets you direct connections to different groups of instances based on your preference for fast response or extra scaling capacity.

### How Aurora serverless auto-pause works for the writer instance in a DB cluster

When an Aurora DB cluster contains only a single DB instance, the mechanism for auto-pausing and resuming the DB instance is straightforward. It depends only on activity on the writer instance. You might have such a configuration for clusters used for development and testing, or for running applications where high availability isn't crucial. Note that in a single-instance cluster, Aurora directs connections through the reader endpoint to the writer DB instance. Thus, for a single-instance DB cluster, attempting to connect to the reader endpoint causes the auto-paused writer DB instance to resume.

The following additional factors apply to Aurora clusters with multiple DB instances:

- Within an Aurora DB cluster, the writer DB instance is typically accessed frequently. Therefore, you might find that the writer DB instance remains active even while one or more of the reader DB instances automatically pauses.
- Certain activities on the reader DB instances require that writer DB instance to be available. Therefore, the writer DB instances can't be paused until all the reader instances are also paused. Resuming any reader instance automatically resumes the writer instance, even if your application isn't accessing the writer instance directly.
- Aurora serverless reader instances in failover promotion tiers zero and one scale to keep their capacity in synch with the writer instance. Thus, when an Aurora serverless writer instance resumes, so do any Aurora serverless reader instances that are in promotion tiers zero or one.

### How Aurora serverless auto-pause works for multi-AZ clusters

Within an Aurora DB cluster containing both a writer and one or more reader DB instances, some Aurora serverless DB instances might be paused while other DB instances are active. The writer instance and any reader instances with failover priority 0 and 1 always pause and resume all at the same time. Reader instances with priority other than 0 or 1 can pause and resume independently of the other instances.

When you use this feature for clusters with multiple reader instances, you can manage costs without sacrificing high availability. The writer instance and another one or two reader instances can remain active at all times, while additional reader instances can pause when they're not needed to handle high-volume read traffic.

Whether a reader instance can automatically pause depends on whether its capacity can scale independently, or whether the capacity is tied to that of the writer DB instance. That scaling property depends on the failover priority of the reader DB instance. When the priority of the reader is zero or one, the capacity of the reader tracks the capacity of the writer DB instance. Therefore, to allow reader DB instances to automatically pause in the broadest range of situations, set their priority to a value higher than one.

The time to resume a reader instance might be slightly longer than to resume a writer instance. For the fastest response if instances might be paused, connect to the cluster endpoint.

### How Aurora serverless auto-pause works for clusters with provisioned instances

Any provisioned DB instances in your Aurora DB cluster won't automatically pause. Only Aurora serverless DB instances, with the `db.serverless` instance class, can use the auto-pause feature.

When your Aurora cluster contains any provisioned DB instances, any Aurora serverless writer instance doesn't automatically pause. That's because of the requirement that the writer instance remains available while any reader instances are active. The fact that the Aurora serverless writer remains active also means that any Aurora serverless reader instances with failover priority 0 and 1 won't auto-pause in a hybrid cluster containing any provisioned instances.

## Monitoring Aurora clusters that use auto-pause

To monitor Aurora, you should already be familiar with the monitoring procedures in [Monitoring Amazon Aurora metrics with Amazon CloudWatch](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/monitoring-cloudwatch.html) and the CloudWatch metrics listed in [Metrics reference for Amazon Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/metrics-reference.html). Be aware that there are some special considerations when you monitor Aurora clusters that use the auto-pause feature:

- There can be periods of time when Aurora serverless instances aren't recording log data and most metrics because the instances are paused. The only metrics sent to CloudWatch while an instance is paused are zero percent for `CPUUtilization` and `ACUUtilization`, and zero for `ServerlessDatabaseCapacity`.
- You can check whether Aurora serverless instances are pausing more or less frequently than you expect. To do so, check how often the `ServerlessDatabaseCapacity` metric changes from a nonzero value to zero, and how long it stays zero. If the instances don't remain paused as long as you expect, you don't save as much as you could on costs. If the instances pause and resume more frequently than you intend, your cluster might have unnecessary latency when responding to connection requests. For information about the factors that affect whether and how often Aurora serverless instances can pause, see [Prerequisites and Limitations for the Aurora serverless auto-pause feature](#auto-pause-prereqs), [Situations where Aurora serverless doesn't auto-pause](#auto-pause-whynot), and [Troubleshooting for Aurora serverless auto-pause](#auto-pause-troubleshooting).
- You can also examine a log file that records automatic pause and resume operations for an Aurora serverless instance. If an instance didn't pause after the timeout interval expired, this log file also includes the reason why auto-pause didn't happen. For more information, see [Monitoring Aurora serverless pause and resume activity](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2-administration.html#autopause-logging-instance-log).

### Checking if an Aurora serverless instance is paused

To determine whether an Aurora serverless instance is in the paused state, you can observe the `ACUUtilization` metric for the instance. That metric has a value of zero while the instance is paused.

While an Aurora serverless instance is paused, its status value is still listed as **Available**. The same applies while a paused Aurora serverless instance is in the process of resuming. That's because you can successfully connect to such an instance, even if the connection experiences a slight delay.

Any metrics related to the availability of Aurora instances consider the period while the instance is paused as time that the instance was available.

### Events for auto-pause and auto-resume operations

Aurora emits events for Aurora serverless instances when auto-pause and auto-resume operations start, finish, or are cancelled. The events related to the auto-pause feature are `RDS-EVENT-0370` through `RDS-EVENT-0374`. For details about these events, see [DB instance events](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/USER_Events.Messages.html#USER_Events.Messages.instance).

### How auto-pause works with Performance Insights and Enhanced Monitoring

While an Aurora serverless instance is paused, Aurora doesn't collect monitoring information for that instance through either Performance Insights or Enhanced Monitoring. When the instance is resumed, there might be a brief delay before Aurora resumes collecting such monitoring information.

### How the Aurora serverless auto-pause feature interacts with Aurora metrics

While an Aurora serverless instance is paused, it doesn't emit most CloudWatch metrics or write any information to its database logs. The only metrics sent to CloudWatch while an instance is paused are zero percent for `CPUUtilization` and `ACUUtilization`, and zero for `ServerlessDatabaseCapacity`.

When CloudWatch is computing statistics related to instance or cluster availability and uptime, Aurora serverless instances are considered to be available during the time that they're paused.

When you initiate an AWS CLI or RDS API action to describe or download the logs for a paused Aurora serverless instance, the instance resumes automatically to make the log information available.

#### Example of CloudWatch metrics

The following AWS CLI examples show how you might observe the capacity of an instance changing over time. During the time period, the instance auto-pauses and then resumes. While it's paused, the `ServerlessDatabaseCapacity` metric reports a value of zero. To determine whether the instance was paused at any point during the time period, we check if the minimum capacity during that time period was zero.

The following Linux example represents an Aurora serverless instance that has been automatically paused for some time. We sample the `ServerlessDatabaseCapacity` each minute, over a period of three minutes. The minimum ACU value of 0.0 confirms that the instance was paused at some point during each minute.

```
$ aws cloudwatch get-metric-statistics \
  --metric-name "ServerlessDatabaseCapacity" \
  --start-time "$(date -d '3 minutes ago')" --end-time "$(date -d 'now')" --period 60 \
  --statistics Minimum \
  --namespace "AWS/RDS" --dimensions Name=DBInstanceIdentifier,Value=my-autopause-instance \
  --output text | sort -k 3

ServerlessDatabaseCapacity
DATAPOINTS    0.0    2024-08-02T22:11:00+00:00    None
DATAPOINTS    0.0    2024-08-02T22:12:00+00:00    None
DATAPOINTS    0.0    2024-08-02T22:13:00+00:00    None
```

Next, we attempt to make a connection to the paused Aurora serverless instance. In this example, we intentionally use an incorrect password so that the connection attempt doesn't succeed. Despite the failure, the connection attempt causes Aurora to resume the paused instance.

```
$ mysql -h my_cluster_endpoint.rds.amazonaws.com -u admin -pwrong-password

ERROR 1045 (28000): Access denied for user 'admin'@'ip_address' (using password: YES)
```

The following Linux example demonstrates that the paused instance resumed, remained idle for approximately five minutes, and then went back into its paused state. The instance resumed at a capacity of 2.0 ACUs. Then it scaled up slightly, for example to perform some internal cleanup. Because it didn't receive any user connection attempts within the five-minute timeout period, it went from 4.0 ACUs directly into the paused state.

```
$ aws cloudwatch get-metric-statistics \
  --metric-name "ServerlessDatabaseCapacity" \
  --start-time "$(date -d '8 minutes ago')" --end-time "$(date -d 'now')" --period 60 \
  --statistics Minimum \
  --namespace "AWS/RDS" --dimensions Name=DBInstanceIdentifier,Value=my-autopause-instance \
  --output text | sort -k 3

ServerlessDatabaseCapacity
DATAPOINTS    0.0    2024-08-02T22:13:00+00:00    None
DATAPOINTS    2.0    2024-08-02T22:14:00+00:00    None
DATAPOINTS    3.0    2024-08-02T22:15:00+00:00    None
DATAPOINTS    3.0    2024-08-02T22:16:00+00:00    None
DATAPOINTS    4.0    2024-08-02T22:17:00+00:00    None
DATAPOINTS    4.0    2024-08-02T22:18:00+00:00    None
DATAPOINTS    4.0    2024-08-02T22:19:00+00:00    None
DATAPOINTS    0.0    2024-08-02T22:20:00+00:00    None
```

If the report was intended to show how quickly the instance scaled up to handle the workload, we might specify `Maximum` for the statistic instead of `Minimum`. For capacity planning and cost estimation, we might specify a longer time period and use the `Average` statistic. That way, we could determine typical capacity values during periods of high activity, low activity, and paused state. To examine behavior during the precise times around pausing and resuming, we might specify a period of one second and examine a shorter time interval. The timestamp values in the output, such as `2024-08-02T22:13:00+00:00`, demonstrate the format to specify precise parameters for the `--start-time` and `--end-time` options.

## Troubleshooting for Aurora serverless auto-pause

If you find that Aurora serverless instances aren't pausing as often as you expect, check for the following possible causes:

- Confirm that the Aurora version you're running does support a minimum capacity of zero ACUs. For the capacity ranges of different Aurora versions, see [Aurora serverless capacity](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.how-it-works.html#aurora-serverless-v2.how-it-works.capacity).
- Confirm that the minimum capacity value for the cluster is set to zero ACUs.
- Confirm that the instance in question is actually using the Aurora serverless instance class `db.serverless`, not one of the provisioned instance classes.
- Confirm that the cluster isn't using any of the incompatible features or settings from [Prerequisites and Limitations for the Aurora serverless auto-pause feature](#auto-pause-prereqs).
- Examine the log file showing when Aurora serverless instances were paused, resumed, or Aurora wasn't able to pause or resume an instance for some reason. For more information, see [Monitoring Aurora serverless pause and resume activity](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2-administration.html#autopause-logging-instance-log).
- Check if any clients or applications are keeping connections open for long periods of time. Conversely, check if any applications that use RDS Data API or Lambda functions are sending frequent requests so that the instance is never idle long enough to pause. You can examine the CloudWatch metrics such as `ConnectionAttempts` and `DatabaseConnections`. For more information, see [Instance-level metrics for Amazon Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.AuroraMonitoring.Metrics.html#Aurora.AuroraMySQL.Monitoring.Metrics.instances).
- If a reader instance rarely if ever pauses, check its failover priority. If the reader is used for read scaling and not as a standby in case of failover, set it to a priority in the range 2-15.
- If the writer instance rarely if ever pauses, check your usage of the reader instances. The writer can only pause if the entire cluster is idle. That's because a connection to any reader instance causes the writer to resume.
- If your application receives timeouts during connection requests while Aurora serverless instances are resuming, consider lengthening the timeout interval used by your application code or the underlying database framework. Longer connection timeouts reduce the possibility of failed connections while Aurora serverless instances are resuming. However, the longer timeouts can also make your application slower to detect availability issues in your cluster.
	Conversely, consider lengthening the auto-pause interval so that applications don't encounter paused instances as often.
	If there isn't a logical balance between the auto-pause behavior and cluster responsiveness for your application, that cluster might not be a good candidate for using the auto-pause feature.
- If you estimate how long your Aurora serverless instances will be paused, be aware that there are factors that make it impractical to make precise predictions.
	- Instances might resume periodically to perform maintenance, minor version upgrades, or apply changes to parameter groups.
	- For multi-AZ clusters, there are situations where resuming one instance causes other instances to resume also. Resuming any reader always causes the writer to resume. Resuming the writer always causes any readers with failover priority 0 and 1 to resume.
	We recommend measuring the actual paused time over a period of several days, with a realistic workload. Then use those measurements to set a baseline for what proportion of time you can expect an instance to be paused.
- You might find that internal operations such as MySQL purge, MySQL event scheduler, PostgreSQL autovacuum, or PostgreSQL jobs scheduled through the `pg_cron` extension aren't running or aren't completing. The instance doesn't automatically resume to perform such operations that don't involve a user connection to the database. If such internal operations are underway when the auto-pause timeout expires, maintenance tasks such as MySQL purge and PostgreSQL autovacuum are canceled. Scheduled jobs from the MySQL event scheduler or the PostgreSQL `pg_cron` extension are also canceled if they're in progress when Aurora initiates the pause operation.
	If you need to ensure that the instance is periodically awake to perform scheduled operations, you can initiate a connection to resume the instance before the start time of the job. You can also increase the auto-pause timeout interval so that operations such as autovacuum can run for longer after user activity is finished. You can also use mechanisms such as Lambda functions to perform database operations on a schedule, in a way that automatically resumes the instance if necessary.

## Application design considerations for the Aurora serverless auto-pause feature

When an Aurora serverless DB instance resumes after being automatically paused, it begins with a relatively small capacity and scales up from there. This starting capacity applies even if the DB instance had some higher capacity immediately before it was automatically paused.

Use this feature with applications that can tolerate an interval of approximately 15 seconds while establishing a connection. That accounts for the typical case where an Aurora serverless instance resumes due to one or a small number of incoming connections. If the instance is paused for longer than 24 hours, the time to resume might be longer.

If you are using Aurora serverless in combination with the Aurora stop/start cluster feature, the resumption time for auto-paused Aurora serverless instances is typically much shorter than the time to start a cluster that's stopped.

When coding connection logic in your application, retry the connection if the first attempt return an error that has a transient cause. (If the error is due to authentication failure, correct the credentials before retrying.) An error that happens immediately after resuming might be a timeout, or some error related to database limits. Retrying can handle issues in the rarer case where an Aurora serverless instance is resumed due to a high number of simultaneous connection requests. In that case, some connections might take longer than usual to be processed, or might exceed the limit of simultaneous connections on the first attempt.

During application development and debugging, don't leave client sessions or programming tools with connections open to the database. Aurora won't pause an instance if there are any user-initiated connections open, regardless of whether the connections aren't running any SQL statements or transactions. When one Aurora serverless instance in an Aurora cluster can't pause, other instances in the cluster might also be prevented from pausing. For more information, see [Situations where Aurora serverless doesn't auto-pause](#auto-pause-whynot).

Aurora emits events when an Aurora serverless DB instance begins resuming, finishes resuming, and if the instance can't resume for some reason. For details about these events, see [DB instance events](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/USER_Events.Messages.html#USER_Events.Messages.instance).