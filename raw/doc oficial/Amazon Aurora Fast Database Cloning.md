---
title: "Amazon Aurora Fast Database Cloning"
source: "https://aws.amazon.com/blogs/aws/amazon-aurora-fast-database-cloning/"
author:
  - "[[Randall Hunt]]"
published: 2017-08-30
created: 2026-10-02
description: "Today, I want to quickly show off a feature of Amazon Aurora that I find incredibly useful: Fast Database Cloning. By taking advantage of Aurora’s underlying distributed storage engine you’re able to quickly and cheaply create a copy-on-write clone of your database. In my career I’ve frequently spent time waiting on some representative sample of […]"
tags:
  - "clippings"
---
Today, I want to quickly show off a feature of [Amazon Aurora](https://aws.amazon.com/rds/aurora/) that I find incredibly useful: [Fast Database Cloning](https://www.youtube.com/watch?v=gdhSG25gSZA). By taking advantage of Aurora’s underlying [distributed storage engine](https://aws.amazon.com/blogs/database/introducing-the-aurora-storage-engine/) you’re able to quickly and cheaply create a copy-on-write clone of your database.

In my career I’ve frequently spent time waiting on some representative sample of data to use in development, experiments, or analytics. If I had a 2TB database it could take hours just waiting for a copy of the data to be ready before I could perform my tasks. Even within RDS MySQL, I would still have to wait several hours for a snapshot copy to complete before I was able to test a schema migration or perform some analytics. Aurora solves this problem in a very interesting way.

The distributed storage engine for Aurora allows us to do things which are normally not feasible or cost-effective with a traditional database engine. By creating pointers to individual pages of data the storage engine enables fast database cloning. Then, when you make changes to the data in the source or the clone, a copy-on-write protocol creates a new copy of that page and updates the pointers. This means my 2TB snapshot restore job that used to take an hour is now ready in about 5 minutes – and most of that time is spent provisioning a new RDS instance.

The time it takes to create the clone is independent of the size of the database since we’re pointing at the same storage. It also makes cloning a very cost-effective operation since I only pay storage costs for the changed pages instead of an entire copy. The database clone is still a regular Aurora Database Cluster with all the same durability guarantees.

Let’s clone a database. First, I’ll select an Aurora (MySQL) instance and select “create-clone” from the Instance Actions.

![](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2017/08/28/Screenshot-2017-08-28-12.29.13.png)

Next I’ll name our clone `dolly-the-sheep` and provision it.

![](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2017/08/28/Screenshot-2017-08-28-14.09.37.png)

It took about 5 minutes and 30 seconds for my clone to become available and I started making some large schema changes and saw no performance impact. The schema changes themselves completed faster than they would have on traditional MySQL due to improvements the Aurora team made to enable [faster DDL](https://aws.amazon.com/blogs/database/amazon-aurora-under-the-hood-fast-ddl/) operations. I could subsequently create a clone-of-a-clone or even a clone-of-a-clone-of-a-clone (and so on) if I wanted to have another team member perform some tests on my schema changes while I continued to make changes of my own. It’s important to note here that clones are first class databases from the perspective of RDS. I still have all of the features that every other Aurora database supports: snapshots, backups, monitoring and more.

I hope this feature will allow you and your teams to save a lot of time and money on experimenting and developing applications based on Amazon Aurora. You can read more about this feature in the [Amazon Aurora User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Aurora.Managing.Clone.html) and I strongly suggest following the [AWS Database Blog](https://aws.amazon.com/blogs/database/). Anurag Gupta’s posts on quorums and [Amazon Aurora storage](https://aws.amazon.com/blogs/database/amazon-aurora-under-the-hood-quorum-and-correlated-failure/) are particularly interesting.

Have follow-up questions or feedback? Ping us at [aurora-pm@amazon.com](mailto:aurora-pm@amazon.com), or leave a comment here. We’d love to get your thoughts and suggestions.

– [Randall](https://twitter.com/jrhunt/)