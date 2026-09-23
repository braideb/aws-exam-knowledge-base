---
title: "Configure route tables - Amazon Virtual Private Cloud"
source: "https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html"
author:
published:
created: 2026-09-19
description: "Configure route tables to control where network traffic is directed."
tags:
  - "clippings"
---
Configure route tables - Amazon Virtual Private Cloud

A *route table* serves as the traffic controller for your virtual private cloud (VPC). Each route table contains a set of rules, called *routes*, that determine where network traffic from your subnet or gateway is directed. When you create a VPC, we also create the main route table for the VPC. You can create additional route tables for your VPC, so that you have more granular control over the network paths for your VPC.

You can use route tables to specify which networks your VPC can communicate with, such as other VPCs or on-premises networks. Each route specifies a destination (CIDR block or prefix list) and a target (such as an internet gateway, NAT gateway, VPC peering connection, or VPN connection). Traffic is routed to targets based on its destination IP address. Route tables enable you to create complex networking architectures that include public subnets, private subnets, VPN-only subnets, and isolated subnets.

###### Contents

- [Route table concepts](https://docs.aws.amazon.com/vpc/latest/userguide/RouteTables.html)
- [Subnet route tables](https://docs.aws.amazon.com/vpc/latest/userguide/subnet-route-tables.html)
- [Gateway route tables](https://docs.aws.amazon.com/vpc/latest/userguide/gateway-route-tables.html)
- [Route priority](https://docs.aws.amazon.com/vpc/latest/userguide/route-tables-priority.html)
- [Example routing options](https://docs.aws.amazon.com/vpc/latest/userguide/route-table-options.html)
- [Create a route table and routes](https://docs.aws.amazon.com/vpc/latest/userguide/create-vpc-route-table.html)
- [Manage subnet route tables](https://docs.aws.amazon.com/vpc/latest/userguide/WorkWithRouteTables.html)
- [Replace the main route table](https://docs.aws.amazon.com/vpc/latest/userguide/Route_Replacing_Main_Table.html)
- [Associate a route table with a gateway](https://docs.aws.amazon.com/vpc/latest/userguide/associate-route-table-gateway.html)
- [Replace or restore the target for a local route](https://docs.aws.amazon.com/vpc/latest/userguide/replace-local-route-target.html)
- [Advanced routing](https://docs.aws.amazon.com/vpc/latest/userguide/advanced-routing.html)
- [Troubleshoot reachability issues](https://docs.aws.amazon.com/vpc/latest/userguide/route-table-routes-troubleshoot.html)

Vpc › userguide

![](https://prod.us-west-2.tcx-beacon.docs.aws.dev/recommendation-beacon/similar/impressions/9402f4ea-1550-4850-b259-51033696c77b/ksTfa7FW0fSq633ylVilUgDC6J4UVsbK-ketSkL76TkuKH7MODSxug==/https:%7C%7Cdocs.aws.amazon.com%7Cvpc%7Clatest%7Cuserguide%7CVPC_Route_Tables.html/https:%7C%7Cdocs.aws.amazon.com%7Cvpc%7Clatest%7Cuserguide%7Csubnet-route-tables.html)

Learn how subnet route tables work in Amazon VPC, including main and custom route tables, route destinations and targets, and subnet associations.

*June 21, 2024*

Vpc › userguide

![](https://prod.us-west-2.tcx-beacon.docs.aws.dev/recommendation-beacon/similar/impressions/9402f4ea-1550-4850-b259-51033696c77b/ksTfa7FW0fSq633ylVilUgDC6J4UVsbK-ketSkL76TkuKH7MODSxug==/https:%7C%7Cdocs.aws.amazon.com%7Cvpc%7Clatest%7Cuserguide%7CVPC_Route_Tables.html/https:%7C%7Cdocs.aws.amazon.com%7Cvpc%7Clatest%7Cuserguide%7CWorkWithRouteTables.html)

Learn how to manage Amazon VPC route tables, including viewing subnet associations, adding or modifying routes, enabling route propagation, and changing subnet route table assignments.

*August 6, 2023*

Vpc › userguide

![](https://prod.us-west-2.tcx-beacon.docs.aws.dev/recommendation-beacon/similar/impressions/9402f4ea-1550-4850-b259-51033696c77b/ksTfa7FW0fSq633ylVilUgDC6J4UVsbK-ketSkL76TkuKH7MODSxug==/https:%7C%7Cdocs.aws.amazon.com%7Cvpc%7Clatest%7Cuserguide%7CVPC_Route_Tables.html/https:%7C%7Cdocs.aws.amazon.com%7Cvpc%7Clatest%7Cuserguide%7Ccreate-vpc-route-table.html)

Learn how to create a custom VPC route table, add routes for destination IP ranges, and associate subnets using the Amazon VPC console or AWS CLI.

*June 13, 2025*