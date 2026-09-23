---
title: "Work with DHCP option sets - Amazon Virtual Private Cloud"
source: "https://docs.aws.amazon.com/vpc/latest/userguide/DHCPOptionSet.html"
author:
published:
created: 2026-09-19
description: "Use the following procedures to view and work with DHCP option sets. For more information about how DHCP option sets work, see ."
tags:
  - "clippings"
---
Work with DHCP option sets - Amazon Virtual Private Cloud

Use the following procedures to view and work with DHCP option sets. For more information about how DHCP option sets work, see [DHCP option set concepts](https://docs.aws.amazon.com/vpc/latest/userguide/DHCPOptionSetConcepts.html).

## Create a DHCP option set

A custom DHCP option set enables you to customize your VPC with your own DNS server, domain name, and more. You can create as many additional DHCP option sets as you want. However, you can only associate a VPC with one DHCP option set at a time.

###### Note

After you create a DHCP option set, you can't modify it. To update the DHCP options for your VPC, you must create a new DHCP option set and then associate it with your VPC.

###### To create a DHCP options set using the console

1. In the navigation pane, choose **DHCP option sets**.
2. Choose **Create DHCP options set**.
3. For **Tag settings**, optionally enter a name for the DHCP option set. If you enter a value, it automatically creates a Name tag for the DHCP option set.
4. For **DHCP options**, provide the configuration settings that you need.
	- **Domain name** (optional): Enter the domain name that a client should use when resolving hostnames using DNS. If you are not using AmazonProvidedDNS, your custom domain name servers must resolve the hostname as appropriate. If you use an Amazon Route 53 private hosted zone, you can use AmazonProvidedDNS. For more information, see [DNS attributes for your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-dns.html).
		###### Note
		Only use domain names that you fully control.
		Some Linux operating systems accept multiple domain names separated by spaces. However, Windows and other Linux operating systems treat the value as a single domain, which results in unexpected behavior. If your DHCP option set is associated with a VPC that has instances running operating systems that treat the value as a single domain, specify only one domain name.
	- **Domain name servers** (optional): Enter the DNS servers that will be used to resolve the IP address of a host from the host's name.
		You can enter either `AmazonProvidedDNS` or custom domain name servers. Using both might cause unexpected behavior. You can enter the IP addresses of up to four IPv4 domain name servers (or up to three IPv4 domain name servers and `AmazonProvidedDNS`) and four IPv6 domain name servers separated by commas. Although you can specify up to eight domain name servers, some operating systems might impose lower limits. For more information about **AmazonProvidedDNS** and the Amazon DNS server, see [Amazon DNS server](https://docs.aws.amazon.com/vpc/latest/userguide/AmazonDNS-concepts.html#AmazonDNS).
		###### Important
		If your VPC has an internet gateway, be sure to specify your own DNS server or an Amazon DNS server (AmazonProvidedDNS) for the **Domain name servers** value. Otherwise, the instances in the VPC won't have access to DNS, which disable internet access.
	- **NTP servers** (optional): Enter the IP addresses of up to eight Network Time Protocol (NTP) servers (four IPv4 addresses and four IPv6 addresses).
		NTP servers provide the time to your network. You can specify the Amazon Time Sync Service at IPv4 address `169.254.169.123` or IPv6 address `fd00:ec2::123`. Instances communicate with the Amazon Time Sync Service by default. Note that the IPv6 address is only accessible on [EC2 instances built on the Nitro System](https://docs.aws.amazon.com/ec2/latest/instancetypes/ec2-nitro-instances.html).
		For more information about the NTP servers option, see [RFC 2132](https://datatracker.ietf.org/doc/html/rfc2132#section-8.3). For more information about the Amazon Time Sync Service, see [Set the time for your instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/set-time.html) in the *Amazon EC2 User Guide*.
	- **NetBIOS name servers** (optional): Enter the IP addresses of up to four NetBIOS name servers.
		For EC2 instances running a Windows OS, the NetBIOS computer name is a friendly name assigned to the instance to identify it on the network. The NetBIOS name server maintains a list of mappings between NetBIOS computer names and network addresses for networks that use NetBIOS as their naming service.
	- **NetBIOS node type** (optional): Enter `1`, `2`, `4`, or `8`. We recommend that you specify `2` (point-to-point or P-node). Broadcast and multicast are not currently supported. For more information about these node types, see section 8.7 of [RFC 2132](https://tools.ietf.org/html/rfc2132) and section 10 of [RFC1001](https://tools.ietf.org/html/rfc1001).
		For EC2 instances running a Windows OS, this is the method that the instances use to resolve NetBIOS names to IP addresses. In the default options set, there is no value for NetBIOS node type.
	- **IPv6 Preferred Lease Time** (optional): A value (in seconds, minutes, hours, or years) for how frequently a running instance with an IPv6 assigned to it goes through DHCPv6 lease renewal. Acceptable values are between 140 and 2147483647 seconds (approximately 68 years). If no value is entered, the default lease time is 140 seconds. If you use long-term addressing for EC2 instances, you can increase the lease time and avoid frequent lease renewal requests. Lease renewal typically occurs when half of the lease time has elapsed.
5. Add **Tags**.
6. Choose **Create DHCP options set**. Note the name or ID of the new DHCP option set.
7. To configure a VPC to use the new option set, see [Change the option set associated with a VPC](#ChangingDHCPOptionsofaVPC).

###### To create a DHCP option set for your VPC using the command line

- [create-dhcp-options](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-dhcp-options.html) (AWS CLI)
- [New-EC2DhcpOption](https://docs.aws.amazon.com/powershell/latest/reference/items/New-EC2DhcpOption.html) (AWS Tools for Windows PowerShell)

## Change the option set associated with a VPC

After you create a DHCP option set, you can associate it with one or more VPCs. You can associate only one DHCP option set with a VPC at a time. If you do not associate a DHCP option set with a VPC, this disables domain name resolution in the VPC.

When you associate a new set of DHCP options with a VPC, any existing instances and all new instances that you launch in that VPC use the new options. You don't need to restart or relaunch your instances. Instances automatically pick up the changes within a few hours, depending on how frequently they renew their DHCP leases. If you prefer, you can explicitly renew the lease using the operating system on the instance.

###### To change the DHCP option set associated with a VPC using the console

1. In the navigation pane, choose **Your VPCs**.
2. Select the check box for the VPC, and then choose **Actions**, **Edit VPC settings**.
3. For **DHCP options set**, choose a new DHCP option set. Alternatively, choose **No DHCP option set** to disable domain name resolution for the VPC.
4. Choose **Save**.

## Delete a DHCP option set

When you no longer need a DHCP option set, use the following procedure to delete it. You can't delete a DHCP option set if it's in use. For each VPC associated with the DHCP option set to delete, you must associate a different DHCP option set with the VPC or configure the VPC to use no DHCP option set. For more information, see [Change the option set associated with a VPC](#ChangingDHCPOptionsofaVPC).

###### To delete a DHCP option set using the console

1. In the navigation pane, choose **DHCP option sets**.
2. Select the radio button for the DHCP option set, and then choose **Actions**, **Delete DHCP option set**.
3. When prompted for confirmation, enter `delete`, and then choose **Delete DHCP option set**.

###### To delete a DHCP option set using the command line

- [delete-dhcp-options](https://docs.aws.amazon.com/cli/latest/reference/ec2/delete-dhcp-options.html) (AWS CLI)
- [Remove-EC2DhcpOption](https://docs.aws.amazon.com/powershell/latest/reference/items/Remove-EC2DhcpOption.html) (AWS Tools for Windows PowerShell)