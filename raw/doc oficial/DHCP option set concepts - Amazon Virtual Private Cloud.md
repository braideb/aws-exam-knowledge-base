---
title: "DHCP option set concepts - Amazon Virtual Private Cloud"
source: "https://docs.aws.amazon.com/vpc/latest/userguide/DHCPOptionSetConcepts.html"
author:
published:
created: 2026-09-19
description: "A DHCP option set is a group of network settings used by resources in your VPC, such as EC2 instances, to communicate over your virtual network."
tags:
  - "clippings"
---
DHCP option set concepts - Amazon Virtual Private Cloud

A *DHCP option set* is a group of network settings used by resources in your VPC, such as EC2 instances, to communicate over your virtual network.

Each Region has a default DHCP option set. Each VPC uses the default DHCP option set for its Region unless you either create and associate a custom DHCP option set with the VPC or configure the VPC with no DHCP option set.

If your VPC has no DHCP option set configured:

- For [EC2 instances built on the Nitro System](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html#instance-hypervisor-type), AWS configures `169.254.169.253` as the default domain name server.
- For [EC2 instances built on Xen](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html#instance-hypervisor-type), no domain name servers are configured and, because instances in the VPC have no access to a DNS server, they can't access the internet.

You can associate a DHCP option set with multiple VPCs, but each VPC can have only one associated DHCP option set.

If you delete a VPC, the DHCP option set that is associated with the VPC is disassociated from the VPC.

## Default DHCP option set

The default DHCP option set contains the following settings:

- **Domain name servers**: The DNS servers that your network interfaces use for domain name resolution. For a default DHCP option set, this is always `AmazonProvidedDNS`. For more information, see [Amazon DNS server](https://docs.aws.amazon.com/vpc/latest/userguide/AmazonDNS-concepts.html#AmazonDNS).
- **Domain name**: The domain name that a client should use when resolving hostnames using the Domain Name System (DNS). For more information about the domain names used for EC2 instances, see [Amazon EC2 instance hostnames](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-naming.html).
- **IPv6 Preferred Lease Time**: How frequently a running instance with an IPv6 assigned to it goes through DHCPv6 lease renewal. The default lease time is 140 seconds. Lease renewal typically occurs when half of the lease time has elapsed.

When you use a default DHCP options set, the following settings are not used, but there are defaults for EC2 instances:

- **NTP servers**: By default, EC2 instances use the [Amazon Time Sync Service](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/set-time.html) to retrieve the time.
- **NetBIOS name servers**: For EC2 instances running Windows, the NetBIOS computer name is a friendly name assigned to the instance to identify it on the network. The NetBIOS name server maintains a list of mappings between NetBIOS computer names and network addresses for networks that use NetBIOS as their naming service.
- **NetBIOS node type**: For EC2 instances running Windows, this is the method that the instances use to resolve NetBIOS names to IP addresses.

When you use the default option set, the Amazon DHCP server uses the network settings in the default option set. When you launch instances in your VPC, they do the following, as shown in the diagram: (1) interact with the DHCP server, (2) interact with the Amazon DNS server, and (3) connect to other devices in the network through the router for your VPC. The instances can interact with the Amazon DHCP server at any time to get their IP address lease and additional network settings.

![Default DHCP option set.](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/dhcp-default-update-new.png)

## Custom DHCP option set

You can create a custom DHCP option set with the following settings, and then associate it with a VPC:

- **Domain name servers**: The DNS servers that your network interfaces use for domain name resolution.
- **Domain name**: The domain name that a client uses when resolving hostnames using the Domain Name System (DNS).
- **NTP servers**: The NTP servers that provide the time to the instances.
- **NetBIOS name servers**: For EC2 instances running Windows, the NetBIOS computer name is a friendly name assigned to the instance to identify it on the network. A NetBIOS name server maintains a list of mappings between NetBIOS computer names and network addresses for networks that use NetBIOS as their naming service.
- **NetBIOS node type**: For EC2 instances running Windows, the method that the instances use to resolve NetBIOS names to IP addresses.
- **IPv6 Preferred Lease Time** (optional): A value (in seconds, minutes, hours, or years) for how frequently a running instance with an IPv6 assigned to it goes through DHCPv6 lease renewal. Acceptable values are between 140 and 4294967295 seconds (approximately 138 years). If no value is entered, the default lease time is 140 seconds. If you use long-term addressing for EC2 instances, you can increase the lease time and avoid frequent lease renewal requests. Lease renewal typically occurs when half of the lease time has elapsed.

When you use a custom option set, instances launched into your VPC do the following, as shown in the diagram: (1) use the network settings in the custom DHCP option set, (2) interact with the DNS, NTP, and NetBIOS servers specified in the custom DHCP option set, and (3) connect to other devices in the network through the router for your VPC.

![Custom DHCP option set.](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/dhcp-custom-update-new.png)