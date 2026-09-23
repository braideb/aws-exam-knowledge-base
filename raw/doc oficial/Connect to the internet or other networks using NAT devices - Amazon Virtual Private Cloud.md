---
title: "Connect to the internet or other networks using NAT devices - Amazon Virtual Private Cloud"
source: "https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat.html"
author:
published:
created: 2026-09-19
description: "Enable access to the internet or other VPCs from private subnets using NAT devices."
tags:
  - "clippings"
---
Connect to the internet or other networks using NAT devices - Amazon Virtual Private Cloud

You can use a NAT device to allow resources in private subnets to connect to the internet, other VPCs, or on-premises networks. These instances can communicate with services outside the VPC, but they cannot receive unsolicited connection requests.

For example, the following diagram shows a NAT device in a public subnet that allows the EC2 instances in a private subnet to connect to the internet through an internet gateway. The NAT device replaces the source IPv4 address of the instances with the address of the NAT device. When sending response traffic to the instances, the NAT device translates the addresses back to the original source IPv4 addresses.

![A NAT device that allows EC2 instances in a private subnet to connect to the internet.](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/nat-device-overview.png)

###### Important

- We use the term *NAT* in this documentation to follow common IT practice, though the actual role of a NAT device is both address translation and port address translation (PAT).
- You can use a managed NAT device offered by AWS, called a *NAT gateway*, or you can create your own NAT device on an EC2 instance, called a *NAT instance*. We recommend that you use NAT gateways because they provide better availability and bandwidth and require less effort on your part to administer.

Vpc › userguide

![](https://prod.us-west-2.tcx-beacon.docs.aws.dev/recommendation-beacon/similar/impressions/9402f4ea-1550-4850-b259-51033696c77b/ksBl0VIPwaWxhOHZ6hrqXEoGGikGZoF3mggrSk3-mgapbCbLSD2WmA==/https:%7C%7Cdocs.aws.amazon.com%7Cvpc%7Clatest%7Cuserguide%7Cvpc-nat.html/https:%7C%7Cdocs.aws.amazon.com%7Cvpc%7Clatest%7Cuserguide%7Cwork-with-nat-instances.html)

Learn how to set up a NAT instance in Amazon VPC to enable private subnet resources to access the internet, including creating a NAT AMI, security groups, and route tables.

*June 21, 2024*

Vpc › userguide

![](https://prod.us-west-2.tcx-beacon.docs.aws.dev/recommendation-beacon/similar/impressions/9402f4ea-1550-4850-b259-51033696c77b/ksBl0VIPwaWxhOHZ6hrqXEoGGikGZoF3mggrSk3-mgapbCbLSD2WmA==/https:%7C%7Cdocs.aws.amazon.com%7Cvpc%7Clatest%7Cuserguide%7Cvpc-nat.html/https:%7C%7Cdocs.aws.amazon.com%7Cvpc%7Clatest%7Cuserguide%7CVPC_NAT_Instance.html)

Learn about NAT instances in Amazon VPC, including how they enable outbound internet traffic from private subnets and the basics of NAT instance architecture.

*August 6, 2023*

Vpc › userguide

![](https://prod.us-west-2.tcx-beacon.docs.aws.dev/recommendation-beacon/similar/impressions/9402f4ea-1550-4850-b259-51033696c77b/ksBl0VIPwaWxhOHZ6hrqXEoGGikGZoF3mggrSk3-mgapbCbLSD2WmA==/https:%7C%7Cdocs.aws.amazon.com%7Cvpc%7Clatest%7Cuserguide%7Cvpc-nat.html/https:%7C%7Cdocs.aws.amazon.com%7Cvpc%7Clatest%7Cuserguide%7Cnat-gateway-scenarios.html)

Learn about NAT gateway use cases in Amazon VPC, including accessing the internet from a private subnet, using allow-listed IPs for on-premises access, and enabling communication between overlapping networks.

*August 6, 2023*