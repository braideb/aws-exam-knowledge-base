---
title: "DNS attributes for your VPC - Amazon Virtual Private Cloud"
source: "https://docs.aws.amazon.com/vpc/latest/userguide/vpc-dns.html"
author:
published:
created: 2026-09-19
description: "DNS translates hostnames to IP addresses, enabling internet and internal network communication. Use the Amazon-provided DNS resolver or configure custom DHCP options for your VPC."
tags:
  - "clippings"
---
DNS attributes for your VPC - Amazon Virtual Private Cloud

Domain Name System (DNS) is a standard by which names used on the internet are resolved to their corresponding IP addresses. A DNS hostname is a name that uniquely and absolutely names a computer; it's composed of a host name and a domain name. DNS servers resolve DNS hostnames to their corresponding IP addresses.

Public IPv4 addresses enable communication over the internet, while private IPv4 addresses enable communication within the network of the instance. For more information, see [IP addressing for your VPCs and subnets](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-ip-addressing.html).

Amazon provides a DNS server ([the Amazon Route 53 Resolver](https://docs.aws.amazon.com/vpc/latest/userguide/AmazonDNS-concepts.html#AmazonDNS)) for your VPC. To use your own DNS server instead, create a new set of DHCP options for your VPC. For more information, see [DHCP option sets in Amazon VPC](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_DHCP_Options.html).

Vpc › userguide

![](https://prod.us-west-2.tcx-beacon.docs.aws.dev/recommendation-beacon/similar/impressions/9402f4ea-1550-4850-b259-51033696c77b/6qZD4rNNCZO-p2tOL37X9apWQVuRBL6TFYA9IPyPYZ5wnYSaosJ0LQ==/https:%7C%7Cdocs.aws.amazon.com%7Cvpc%7Clatest%7Cuserguide%7Cvpc-dns.html/https:%7C%7Cdocs.aws.amazon.com%7Cvpc%7Clatest%7Cuserguide%7CAmazonDNS-concepts.html)

Learn how the Amazon DNS server (Route 53 Resolver) works in a VPC, including IP addresses, DNS hostname types, VPC DNS attributes, quotas, and private hosted zones.

*June 21, 2024*

Vpc › userguide

![](https://prod.us-west-2.tcx-beacon.docs.aws.dev/recommendation-beacon/similar/impressions/9402f4ea-1550-4850-b259-51033696c77b/6qZD4rNNCZO-p2tOL37X9apWQVuRBL6TFYA9IPyPYZ5wnYSaosJ0LQ==/https:%7C%7Cdocs.aws.amazon.com%7Cvpc%7Clatest%7Cuserguide%7Cvpc-dns.html/https:%7C%7Cdocs.aws.amazon.com%7Cvpc%7Clatest%7Cuserguide%7Cvpc-dns-updating.html)

Learn how to view and update DNS support attributes for your Amazon VPC using the console or AWS CLI, including DNS hostnames and DNS resolution settings.

*July 11, 2024*

Route53 › DeveloperGuide

![](https://prod.us-west-2.tcx-beacon.docs.aws.dev/recommendation-beacon/similar/impressions/9402f4ea-1550-4850-b259-51033696c77b/6qZD4rNNCZO-p2tOL37X9apWQVuRBL6TFYA9IPyPYZ5wnYSaosJ0LQ==/https:%7C%7Cdocs.aws.amazon.com%7Cvpc%7Clatest%7Cuserguide%7Cvpc-dns.html/https:%7C%7Cdocs.aws.amazon.com%7CRoute53%7Clatest%7CDeveloperGuide%7Chosted-zones-private.html)

Learn how Amazon Route 53 private hosted zones work, including VPC associations, DNS query resolution, and how VPC Resolver routes traffic within private namespaces.

*August 6, 2023*