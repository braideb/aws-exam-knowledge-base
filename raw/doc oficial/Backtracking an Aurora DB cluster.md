---
title: "Backtracking an Aurora DB cluster"
source: "https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Managing.Backtrack.html"
author:
published:
created: 2026-10-02
description: "With Amazon Aurora MySQL-Compatible Edition, you can backtrack a DB cluster to a specific time, without restoring data from a backup."
tags:
  - "clippings"
---
Backtracking an Aurora DB cluster - Amazon Aurora

With Amazon Aurora MySQL-Compatible Edition, you can backtrack a DB cluster to a specific time, without restoring data from a backup.

###### Important

Backtracking is supported in Aurora MySQL version 2, version 3, and version 8.4.

###### Contents

- [Overview of backtracking](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Managing.Backtrack.html#AuroraMySQL.Managing.Backtrack.Overview)
- [Region and version availability](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Managing.Backtrack.html#AuroraMySQL.Managing.Backtrack.Availability)
- [Upgrade considerations for backtrack-enabled clusters](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Managing.Backtrack.html#AuroraMySQL.Managing.Backtrack.Upgrade)
- [Configuring backtracking a Aurora MySQL DB cluster](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Managing.Backtrack.Configuring.html)
- [Performing a backtrack for an Aurora MySQL DB cluster](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Managing.Backtrack.Performing0.html)
- [Monitoring backtracking for an Aurora MySQL DB cluster](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Managing.Backtrack.Monitoring.html)
- [Subscribing to a backtrack event with the console](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Managing.Backtrack.html#AuroraMySQL.Managing.Backtrack.Event.Console)
- [Retrieving existing backtracks](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Managing.Backtrack.html#AuroraMySQL.Managing.Backtrack.Retrieving)
- [Disabling backtracking for an Aurora MySQL DB cluster](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Managing.Backtrack.Disabling.html)

## Overview of backtracking

Backtracking "rewinds" the DB cluster to the time you specify. Backtracking is not a replacement for backing up your DB cluster so that you can restore it to a point in time. However, backtracking provides the following advantages over traditional backup and restore:

- You can easily undo mistakes. If you mistakenly perform a destructive action, such as a DELETE without a WHERE clause, you can backtrack the DB cluster to a time before the destructive action with minimal interruption of service.
- You can backtrack a DB cluster quickly. Restoring a DB cluster to a point in time launches a new DB cluster and restores it from backup data or a DB cluster snapshot, which can take hours. Backtracking a DB cluster doesn't require a new DB cluster and rewinds the DB cluster in minutes.
- You can explore earlier data changes. You can repeatedly backtrack a DB cluster back and forth in time to help determine when a particular data change occurred. For example, you can backtrack a DB cluster three hours and then backtrack forward in time one hour. In this case, the backtrack time is two hours before the original time.

###### Note

For information about restoring a DB cluster to a point in time, see [Overview of backing up and restoring an Aurora DB cluster](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Managing.Backups.html).

### Backtrack window

With backtracking, there is a target backtrack window and an actual backtrack window:

- The *target backtrack window* is the amount of time you want to be able to backtrack your DB cluster. When you enable backtracking, you specify a *target backtrack window*. For example, you might specify a target backtrack window of 24 hours if you want to be able to backtrack the DB cluster one day.
- The *actual backtrack window* is the actual amount of time you can backtrack your DB cluster, which can be smaller than the target backtrack window. The actual backtrack window is based on your workload and the storage available for storing information about database changes, called *change records.*

As you make updates to your Aurora DB cluster with backtracking enabled, you generate change records. Aurora retains change records for the target backtrack window, and you pay an hourly rate for storing them. Both the target backtrack window and the workload on your DB cluster determine the number of change records you store. The workload is the number of changes you make to your DB cluster in a given amount of time. If your workload is heavy, you store more change records in your backtrack window than you do if your workload is light.

You can think of your target backtrack window as the goal for the maximum amount of time you want to be able to backtrack your DB cluster. In most cases, you can backtrack the maximum amount of time that you specified. However, in some cases, the DB cluster can't store enough change records to backtrack the maximum amount of time, and your actual backtrack window is smaller than your target. Typically, the actual backtrack window is smaller than the target when you have extremely heavy workload on your DB cluster. When your actual backtrack window is smaller than your target, we send you a notification.

When backtracking is enabled for a DB cluster, and you delete a table stored in the DB cluster, Aurora keeps that table in the backtrack change records. It does this so that you can revert back to a time before you deleted the table. If you don't have enough space in your backtrack window to store the table, the table might be removed from the backtrack change records eventually.

### Backtracking time

Aurora always backtracks to a time that is consistent for the DB cluster. Doing so eliminates the possibility of uncommitted transactions when the backtrack is complete. When you specify a time for a backtrack, Aurora automatically chooses the nearest possible consistent time. This approach means that the completed backtrack might not exactly match the time you specify, but you can determine the exact time for a backtrack by using the [describe-db-cluster-backtracks](https://docs.aws.amazon.com/cli/latest/reference/rds/describe-db-cluster-backtracks.html) AWS CLI command. For more information, see [Retrieving existing backtracks](#AuroraMySQL.Managing.Backtrack.Retrieving).

### Backtracking limitations

The following limitations apply to backtracking:

- Backtracking is only available for DB clusters that were created with the Backtrack feature enabled. You can't modify a DB cluster to enable the Backtrack feature. You can enable the Backtrack feature when you create a new DB cluster or restore a snapshot of a DB cluster.
- The limit for a backtrack window is 72 hours.
- Backtracking affects the entire DB cluster. For example, you can't selectively backtrack a single table or a single data update.
- You can't create cross-Region read replicas from a backtrack-enabled cluster, but you can still enable binary log (binlog) replication on the cluster. If you try to backtrack a DB cluster for which binary logging is enabled, an error typically occurs unless you choose to force the backtrack. Any attempts to force a backtrack will break downstream read replicas and interfere with other operations such as blue/green deployments.
- You can't backtrack a database clone to a time before that database clone was created. However, you can use the original database to backtrack to a time before the clone was created. For more information about database cloning, see [Cloning a volume for an Amazon Aurora DB cluster](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Managing.Clone.html).
- Backtracking causes a brief DB instance disruption. You must stop or pause your applications before starting a backtrack operation to ensure that there are no new read or write requests. During the backtrack operation, Aurora pauses the database, closes any open connections, and drops any uncommitted reads and writes. It then waits for the backtrack operation to complete.
- You can't restore a cross-Region snapshot of a backtrack-enabled cluster in an AWS Region that doesn't support backtracking.
- If you perform an in-place upgrade for a backtrack-enabled cluster from Aurora MySQL version 2 to version 3, or from version 3 to version 8.4, you can't backtrack to a point in time before the upgrade happened.

## Region and version availability

Backtrack is not available for Aurora PostgreSQL.

Following are the supported engines and Region availability for Backtrack with Aurora MySQL.

| Region | Aurora MySQL version 8.4 | Aurora MySQL version 3 | Aurora MySQL version 2 |
| --- | --- | --- | --- |
| US East (N. Virginia) | All versions | All versions | All versions |
| US East (Ohio) | All versions | All versions | All versions |
| US West (N. California) | All versions | All versions | All versions |
| US West (Oregon) | All versions | All versions | All versions |
| Africa (Cape Town) | – | – | – |
| Asia Pacific (Hong Kong) | – | – | – |
| Asia Pacific (Jakarta) | – | – | – |
| Asia Pacific (Malaysia) | – | – | – |
| Asia Pacific (Melbourne) | – | – | – |
| Asia Pacific (Mumbai) | All versions | All versions | All versions |
| Asia Pacific (New Zealand) | – | – | – |
| Asia Pacific (Osaka) | All versions | All versions | Version 2.07.3 and higher |
| Asia Pacific (Seoul) | All versions | All versions | All versions |
| Asia Pacific (Singapore) | All versions | All versions | All versions |
| Asia Pacific (Sydney) | All versions | All versions | All versions |
| Asia Pacific (Taipei) | – | – | – |
| Asia Pacific (Thailand) | – | – | – |
| Asia Pacific (Tokyo) | All versions | All versions | All versions |
| Canada (Central) | All versions | All versions | All versions |
| Canada West (Calgary) | – | – | – |
| China (Beijing) | – | – | – |
| China (Ningxia) | – | – | – |
| Europe (Frankfurt) | All versions | All versions | All versions |
| Europe (Ireland) | All versions | All versions | All versions |
| Europe (London) | All versions | All versions | All versions |
| Europe (Milan) | – | – | – |
| Europe (Paris) | All versions | All versions | All versions |
| Europe (Spain) | – | – | – |
| Europe (Stockholm) | – | – | – |
| Europe (Zurich) | – | – | – |
| Israel (Tel Aviv) | – | – | – |
| Mexico (Central) | – | – | – |
| Middle East (Bahrain) | – | – | – |
| Middle East (UAE) | – | – | – |
| South America (São Paulo) | – | – | – |
| AWS GovCloud (US-East) | – | – | – |
| AWS GovCloud (US-West) | – | – | – |

## Upgrade considerations for backtrack-enabled clusters

You can upgrade a backtrack-enabled DB cluster from Aurora MySQL version 2 to version 3, or from version 3 to version 8.4, because all minor versions of Aurora MySQL version 3 and version 8.4 are supported for Backtrack.

## Subscribing to a backtrack event with the console

The following procedure describes how to subscribe to a backtrack event using the console. The event sends you an email or text notification when your actual backtrack window is smaller than your target backtrack window.

###### To view backtrack information using the console

1. Sign in to the AWS Management Console and open the Amazon RDS console at [https://console.aws.amazon.com/rds/](https://console.aws.amazon.com/rds/).
2. Choose **Event subscriptions**.
3. Choose **Create event subscription**.
4. In the **Name** box, type a name for the event subscription, and ensure that **Yes** is selected for **Enabled**.
5. In the **Target** section, choose **New email topic**.
6. For **Topic name**, type a name for the topic, and for **With these recipients**, enter the email addresses or phone numbers to receive the notifications.
7. In the **Source** section, choose **Instances** for **Source type**.
8. For **Instances to include**, choose **Select specific instances**, and choose your DB instance.
9. For **Event categories to include**, choose **Select specific event categories**, and choose **backtrack**.
	Your page should look similar to the following page.
	![Backtrack event subscription.](https://docs.aws.amazon.com/images/AmazonRDS/latest/AuroraUserGuide/images/aurora-backtrack-event.png)
10. Choose **Create**.

## Retrieving existing backtracks

You can retrieve information about existing backtracks for a DB cluster. This information includes the unique identifier of the backtrack, the date and time backtracked to and from, the date and time the backtrack was requested, and the current status of the backtrack.

###### Note

Currently, you can't retrieve existing backtracks using the console.

The following procedure describes how to retrieve existing backtracks for a DB cluster using the AWS CLI.

###### To retrieve existing backtracks using the AWS CLI

- Call the [describe-db-cluster-backtracks](https://docs.aws.amazon.com/cli/latest/reference/rds/describe-db-cluster-backtracks.html) AWS CLI command and supply the following values:
	- `--db-cluster-identifier` – The name of the DB cluster.
	The following example retrieves existing backtracks for `sample-cluster`.
	For Linux, macOS, or Unix:
	```
	aws rds describe-db-cluster-backtracks \
	    --db-cluster-identifier sample-cluster
	```
	For Windows:
	```
	aws rds describe-db-cluster-backtracks ^
	    --db-cluster-identifier sample-cluster
	```

## AWS CLI

The following procedure describes how to retrieve existing backtracks for a DB cluster using the AWS CLI.

###### To retrieve existing backtracks using the AWS CLI

- Call the [describe-db-cluster-backtracks](https://docs.aws.amazon.com/cli/latest/reference/rds/describe-db-cluster-backtracks.html) AWS CLI command and supply the following values:
	- `--db-cluster-identifier` – The name of the DB cluster.
	The following example retrieves existing backtracks for `sample-cluster`.
	For Linux, macOS, or Unix:
	```
	aws rds describe-db-cluster-backtracks \
	    --db-cluster-identifier sample-cluster
	```
	For Windows:
	```
	aws rds describe-db-cluster-backtracks ^
	    --db-cluster-identifier sample-cluster
	```

To retrieve information about the backtracks for a DB cluster using the Amazon RDS API, use the [DescribeDBClusterBacktracks](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_DescribeDBClusterBacktracks.html) operation. This operation returns information about backtracks for the DB cluster specified in the `DBClusterIdentifier` value.

## RDS API

To retrieve information about the backtracks for a DB cluster using the Amazon RDS API, use the [DescribeDBClusterBacktracks](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_DescribeDBClusterBacktracks.html) operation. This operation returns information about backtracks for the DB cluster specified in the `DBClusterIdentifier` value.