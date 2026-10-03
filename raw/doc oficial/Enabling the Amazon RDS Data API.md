---
title: "Enabling the Amazon RDS Data API"
source: "https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/data-api.enabling.html"
author:
published:
created: 2026-10-02
description: "Use this section to enable the Amazon RDS Data API."
tags:
  - "clippings"
---
Enabling the Amazon RDS Data API - Amazon Aurora

To use the Amazon RDS Data API (Data API), enable it for your Aurora DB cluster. You can enable Data API when you create or modify the DB cluster.

###### Note

Whether Data API is available for your cluster depends on your Aurora version, database engine, and AWS Region. For Aurora, Data API works with clusters that use both provisioned and Aurora serverless instances. To check whether your cluster can use Data API, see [Supported Regions and Aurora DB engines for RDS Data API](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.Aurora_Fea_Regions_DB-eng.Feature.Data_API.html).

## Enabling RDS Data API when you create a database

While you are creating a database that supports RDS Data API (Data API), you can enable this feature. The following procedures describe how to do so when you use the AWS Management Console, the AWS CLI, or the RDS API.

To enable Data API when you create a DB cluster, select the **Enable the RDS Data API** checkbox in the **Connectivity** section of the **Create database** page, as in the following screenshot.

![The Connectivity section on the Create database page, with the Enable the RDS Data API checkbox selected.](https://docs.aws.amazon.com/images/AmazonRDS/latest/AuroraUserGuide/images/data-api-enable-on-create.png)

For instructions on how to create an Aurora DB cluster that can use the RDS Data API, see the following:

- For Aurora serverless and provisioned clusters – [Creating an Amazon Aurora DB cluster](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.CreateInstance.html)

## Console

To enable Data API when you create a DB cluster, select the **Enable the RDS Data API** checkbox in the **Connectivity** section of the **Create database** page, as in the following screenshot.

![The Connectivity section on the Create database page, with the Enable the RDS Data API checkbox selected.](https://docs.aws.amazon.com/images/AmazonRDS/latest/AuroraUserGuide/images/data-api-enable-on-create.png)

For instructions on how to create an Aurora DB cluster that can use the RDS Data API, see the following:

- For Aurora serverless and provisioned clusters – [Creating an Amazon Aurora DB cluster](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.CreateInstance.html)

To enable Data API while you're creating an Aurora DB cluster, run the [create-db-cluster](https://docs.aws.amazon.com/cli/latest/reference/rds/create-db-cluster.html) AWS CLI command with the `--enable-http-endpoint` option.

The following example creates an Aurora PostgreSQL DB cluster with Data API enabled.

For Linux, macOS, or Unix:

```
aws rds create-db-cluster \
        --db-cluster-identifier my_pg_cluster \
        --engine aurora-postgresql \
        --enable-http-endpoint
```

For Windows:

```
aws rds create-db-cluster ^
        --db-cluster-identifier my_pg_cluster ^
        --engine aurora-postgresql ^
        --enable-http-endpoint
```

## AWS CLI

To enable Data API while you're creating an Aurora DB cluster, run the [create-db-cluster](https://docs.aws.amazon.com/cli/latest/reference/rds/create-db-cluster.html) AWS CLI command with the `--enable-http-endpoint` option.

The following example creates an Aurora PostgreSQL DB cluster with Data API enabled.

For Linux, macOS, or Unix:

```
aws rds create-db-cluster \
        --db-cluster-identifier my_pg_cluster \
        --engine aurora-postgresql \
        --enable-http-endpoint
```

For Windows:

```
aws rds create-db-cluster ^
        --db-cluster-identifier my_pg_cluster ^
        --engine aurora-postgresql ^
        --enable-http-endpoint
```

To enable Data API while you're creating an Aurora DB cluster, use the [CreateDBCluster](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_CreateDBCluster.html) operation with the value of the `EnableHttpEndpoint` parameter set to `true`.

## RDS API

To enable Data API while you're creating an Aurora DB cluster, use the [CreateDBCluster](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_CreateDBCluster.html) operation with the value of the `EnableHttpEndpoint` parameter set to `true`.

## Enabling or disabling RDS Data API on an existing database

You can modify a DB cluster that supports RDS Data API (Data API) to enable or disable this feature.

### Enabling or disabling Data API (Aurora serverless and provisioned)

Use the following procedures to enable or disable Data API on Aurora serverless and provisioned databases.

You can enable or disable Data API by using the RDS console for a DB cluster that supports this feature. To do so, open the cluster details page of the database on which you want to enable or disable Data API, and select **Enable RDS Data API** or **Disable RDS Data API** from the **Actions** dropdown menu. This will allow you to enable or disable the RDS Data API for your cluster.

The following screenshot shows that the **RDS Data API** isn't enabled.

![The RDS Data API section on the Connectivity and security tab showing disabled status.](https://docs.aws.amazon.com/images/AmazonRDS/latest/AuroraUserGuide/images/data-api-enable-from-details.png)

## Console

You can enable or disable Data API by using the RDS console for a DB cluster that supports this feature. To do so, open the cluster details page of the database on which you want to enable or disable Data API, and select **Enable RDS Data API** or **Disable RDS Data API** from the **Actions** dropdown menu. This will allow you to enable or disable the RDS Data API for your cluster.

The following screenshot shows that the **RDS Data API** isn't enabled.

![The RDS Data API section on the Connectivity and security tab showing disabled status.](https://docs.aws.amazon.com/images/AmazonRDS/latest/AuroraUserGuide/images/data-api-enable-from-details.png)

To enable or disable Data API on an existing database, run the [enable-http-endpoint](https://docs.aws.amazon.com/cli/latest/reference/rds/enable-http-endpoint.html) or [disable-http-endpoint](https://docs.aws.amazon.com/cli/latest/reference/rds/disable-http-endpoint.html) AWS CLI command, and specify the ARN of your DB cluster.

The following example enables Data API.

For Linux, macOS, or Unix:

```
aws rds enable-http-endpoint \
        --resource-arn cluster_arn
```

For Windows:

```
aws rds enable-http-endpoint ^
        --resource-arn cluster_arn
```

## AWS CLI

To enable or disable Data API on an existing database, run the [enable-http-endpoint](https://docs.aws.amazon.com/cli/latest/reference/rds/enable-http-endpoint.html) or [disable-http-endpoint](https://docs.aws.amazon.com/cli/latest/reference/rds/disable-http-endpoint.html) AWS CLI command, and specify the ARN of your DB cluster.

The following example enables Data API.

For Linux, macOS, or Unix:

```
aws rds enable-http-endpoint \
        --resource-arn cluster_arn
```

For Windows:

```
aws rds enable-http-endpoint ^
        --resource-arn cluster_arn
```

To enable or disable Data API on an existing database, use the [EnableHttpEndpoint](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_EnableHttpEndpoint.html) and [DisableHttpEndpoint](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_DisableHttpEndpoint.html) operations.

## RDS API

To enable or disable Data API on an existing database, use the [EnableHttpEndpoint](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_EnableHttpEndpoint.html) and [DisableHttpEndpoint](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_DisableHttpEndpoint.html) operations.