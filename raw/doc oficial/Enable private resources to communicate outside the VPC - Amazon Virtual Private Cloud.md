---
title: "Enable private resources to communicate outside the VPC - Amazon Virtual Private Cloud"
source: "https://docs.aws.amazon.com/vpc/latest/userguide/work-with-nat-instances.html"
author:
published:
created: 2026-09-19
description: "This section describes how to create and work with NAT instances to enable resources in a private subnet to communicate outside the virtual private cloud."
tags:
  - "clippings"
---
Enable private resources to communicate outside the VPC - Amazon Virtual Private Cloud

This section describes how to create and work with NAT instances to enable resources in a private subnet to communicate outside the virtual private cloud.

## 1\. Create a VPC for the NAT instance

Use the following procedure to create a VPC with a public subnet and a private subnet.

###### To create the VPC

1. Choose **Create VPC**.
2. For **Resources to create**, choose **VPC and more**.
3. For **Name tag auto-generation**, enter a name for the VPC.
4. To configure the subnets, do the following:
	1. For **Number of Availability Zones**, choose **1** or **2**, depending on your needs.
		2. For **Number of public subnets**, ensure that you have one public subnet per Availability Zone.
		3. For **Number of private subnets**, ensure that you have one private subnet per Availability Zone.
5. Choose **Create VPC**.

## 2\. Create a security group for the NAT instance

Create a security group with the rules described in the following table. These rules enable your NAT instance to receive internet-bound traffic from instances in the private subnet, as well as SSH traffic from your network. The NAT instance can also send traffic to the internet, which enables the instances in the private subnet to get software updates.

The following are the inbound recommended rules.

| Source | Protocol | Port range | Comments |
| --- | --- | --- | --- |
| `Private subnet CIDR` | TCP | 80 | Allow inbound HTTP traffic from servers in the private subnet |
| `Private subnet CIDR` | TCP | 443 | Allow inbound HTTPS traffic from servers in the private subnet |
| `Public IP address range of your network` | TCP | 22 | Allow inbound SSH access to the NAT instance from your network (over the internet gateway) |

The following are the recommended outbound rules.

| Destination | Protocol | Port range | Comments |
| --- | --- | --- | --- |
| 0.0.0.0/0 | TCP | 80 | Allow outbound HTTP access to the internet |
| 0.0.0.0/0 | TCP | 443 | Allow outbound HTTPS access to the internet |

###### To create the security group

1. In the navigation pane, choose **Security groups**.
2. Choose **Create security group**.
3. Enter a name and description for the security group.
4. For **VPC**, select the ID of the VPC for your NAT instance.
5. Add rules for inbound traffic under **Inbound rules** as follows:
	1. Choose **Add rule**. Choose **HTTP** for **Type** and enter the IP address range of your private subnet for **Source**.
		2. Choose **Add rule**. Choose **HTTPS** for **Type** and enter the IP address range of your private subnet for **Source**.
		3. Choose **Add rule**. Choose **SSH** for **Type** and enter the IP address range of your network for **Source**.
6. Add rules for outbound traffic under **Outbound rules** as follows:
	1. Choose **Add rule**. Choose **HTTP** for **Type** and enter 0.0.0.0/0 for **Destination**.
		2. Choose **Add rule**. Choose **HTTPS** for **Type** and enter 0.0.0.0/0 for **Destination**.
7. Choose **Create security group**.

For more information, see [Security groups](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html).

## 3\. Create a NAT AMI

A NAT AMI is configured to run NAT on an EC2 instance. You must create a NAT AMI and then launch your NAT instance using your NAT AMI.

If you plan to use an operating system other than Amazon Linux for your NAT AMI, refer to the documentation for this operating system to learn how to configure NAT. Be sure to save these settings so that they persist even after an instance reboot.

###### To create a NAT AMI for Amazon Linux

1. Launch an EC2 instance running AL2023 or Amazon Linux 2. Be sure to specify the security group that you created for the NAT instance.
2. Connect to your instance and run the following commands on the instance to enable iptables.
	```
	sudo yum install iptables-services -y
	sudo systemctl enable iptables
	sudo systemctl start iptables
	```
3. Do the following on the instance to enable IP forwarding such that it persists after reboot:
	1. Using a text editor, such as **nano** or **vim**, create the following configuration file: `/etc/sysctl.d/custom-ip-forwarding.conf`.
		2. Add the following line to the configuration file.
		```
		net.ipv4.ip_forward=1
		```
		3. Save the configuration file and exit the text editor.
		4. Run the following command to apply the configuration file.
		```
		sudo sysctl -p /etc/sysctl.d/custom-ip-forwarding.conf
		```
4. Run the following command on the instance, and note the name of the primary network interface. You'll need this information for the next step.
	```
	netstat -i
	```
	In the following example output, `docker0` is a network interface created by docker, `eth0` is the primary network interface, and `lo` is the loopback interface.
	```
	Iface      MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
	docker0   1500        0      0      0 0             0      0      0      0 BMU
	eth0      9001  7276052      0      0 0       5364991      0      0      0 BMRU
	lo       65536   538857      0      0 0        538857      0      0      0 LRU
	```
	In the following example output, the primary network interface is `enX0`.
	```
	Iface      MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
	enX0      9001     1076      0      0 0          1247      0      0      0 BMRU
	lo       65536       24      0      0 0            24      0      0      0 LRU
	```
	In the following example output, the primary network interface is `ens5`.
	```
	Iface      MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
	ens5      9001    14036      0      0 0          2116      0      0      0 BMRU
	lo       65536       12      0      0 0            12      0      0      0 LRU
	```
5. Run the following commands on the instance to configure NAT. If the primary network interface is not `eth0`, replace `eth0` with the primary network interface that you noted in the previous step.
	```
	sudo /sbin/iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
	sudo /sbin/iptables -F FORWARD
	sudo service iptables save
	```
6. Create a NAT AMI from the EC2 instance. For more information, see [Create a Linux AMI from an instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-an-ami-ebs.html#how-to-create-ebs-ami) in the *Amazon EC2 User Guide*.

## 4\. Launch a NAT instance

Use the following procedure to launch a NAT instance using the VPC, security group, and NAT AMI that you created.

###### To launch a NAT instance

1. Open the Amazon EC2 console at [https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/).
2. On the dashboard, choose **Launch instance**.
3. For **Name**, enter a name for your NAT instance.
4. For **Application and OS Images**, select your NAT AMI (choose **Browse more AMIs**, **My AMIs**).
5. For **Instance type**, choose an instance type that provides the compute, memory, and storage resources that your NAT instance needs.
6. For **Key pair**, select an existing key pair or choose **Create new key pair**.
7. For **Network settings**, do the following:
	1. Choose **Edit**.
		2. For **VPC**, choose the VPC that you created.
		3. For **Subnet**, choose the public subnet that you created.
		4. For **Auto-assign public IP**, choose **Enable**. Alternatively, after you launch the NAT instance, allocate an Elastic IP address and assign it to the NAT instance.
		5. For **Firewall**, choose **Select existing security group** and then choose the security group that you created.
8. Choose **Launch instance**. Choose the instance ID to open the instance details page. Wait for the instance state to change to **Running** and for the status checks to succeed.
9. Disable source/destination checks for the NAT instance (see [5\. Disable source/destination checks](#EIP_Disable_SrcDestCheck)).
10. Update the route table to send traffic to the NAT instance (see [6\. Update the route table](#nat-routing-table)).

## 5\. Disable source/destination checks

Each EC2 instance performs source/destination checks by default. This means that the instance must be the source or destination of any traffic it sends or receives. However, a NAT instance must be able to send and receive traffic when the source or destination is not itself. Therefore, you must disable source/destination checks on the NAT instance.

###### To disable source/destination checking

1. Open the Amazon EC2 console at [https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/).
2. In the navigation pane, choose **Instances**.
3. Select the NAT instance.
4. Choose **Actions**, **Networking**, **Change source/destination check**.
5. For **Source/destination checking**, select **Stop**.
6. Choose **Save**.
7. If the NAT instance has a secondary network interface, choose it from **Network interfaces** on the **Networking** tab. Choose the interface ID to go to the network interfaces page. Choose **Actions**, **Change source/dest. check**, clear **Enable**, and choose **Save**.

## 6\. Update the route table

The route table for the private subnet must have a route that sends internet traffic to the NAT instance.

###### To update the route table

1. In the navigation pane, choose **Route tables**.
2. Select the route table for the private subnet.
3. On the **Routes** tab, choose **Edit routes** and then choose **Add route**.
4. Enter 0.0.0.0/0 for **Destination** and the instance ID of the NAT instance for **Target**.
5. Choose **Save changes**.

For more information, see [Configure route tables](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html).

## 7\. Test your NAT instance

After you have launched a NAT instance and completed the configuration steps above, you can test whether an instance in your private subnet can access the internet through the NAT instance by using the NAT instance as a bastion server.

### Step 1: Update the NAT instance security group

To allow instances in your private subnet to send ping traffic to the NAT instance, add a rule to allow inbound and outbound ICMP traffic. To allow the NAT instance to serve as a bastion server, add a rule to allow outbound SSH traffic to the private subnet.

###### To update your NAT instance security group

1. In the navigation pane, choose **Security groups**.
2. Select the check box for the security group associated with your NAT instance.
3. On the **Inbound rules** tab, choose **Edit inbound rules**.
4. Choose **Add rule**. Choose **All ICMP - IPv4** for **Type**. Choose **Custom** for **Source** and enter the IP address range of your private subnet. Choose **Save rules**.
5. On the **Outbound rules** tab, choose **Edit outbound rules**.
6. Choose **Add rule**. Choose **SSH** for **Type**. Choose **Custom** for **Destination** and enter the IP address range of your private subnet.
7. Choose **Add rule**. Choose **All ICMP - IPv4** for **Type**. Choose **Anywhere - IPv4** for **Destination**. Choose **Save rules**.

### Step 2: Launch a test instance in the private subnet

Launch an instance into your private subnet. You must allow SSH access from the NAT instance, and you must use the same key pair that you used for the NAT instance.

###### To launch a test instance in the private subnet

1. Open the Amazon EC2 console at [https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/).
2. On the dashboard, choose **Launch instance**.
3. Select your private subnet.
4. Do not assign a public IP address to this instance.
5. Ensure that the security group for this instance allows inbound SSH access from your NAT instance, or from the IP address range of your public subnet, and outbound ICMP traffic.
6. Select the same key pair that you used for the NAT instance.

### Step 3: Ping an ICMP-enabled website

To verify that the test instance in your private subnet can use your NAT instance to communicate with the internet, run the **ping** command.

###### To test the internet connection from your private instance

1. From your local computer, configure SSH agent forwarding, so that you can use the NAT instance as a bastion server.
	anchor anchor
	```bash
	ssh-add key.pem
	```
2. From your local computer, connect to your NAT instance.
	anchor anchor
	```bash
	ssh -A ec2-user@nat-instance-public-ip-address
	```
3. From the NAT instance, run the **ping** command, specifying a website that is enabled for ICMP.
	```bash
	[ec2-user@ip-10-0-4-184]$ ping ietf.org
	```
	To confirm that your NAT instance has internet access, verify that you received output such as the following, and then press **Ctrl+C** to cancel the **ping** command. Otherwise, verify that the NAT instance is in a public subnet (its route table has a route to an internet gateway).
	```
	PING ietf.org (104.16.45.99) 56(84) bytes of data.
	64 bytes from 104.16.45.99 (104.16.45.99): icmp_seq=1 ttl=33 time=7.88 ms
	64 bytes from 104.16.45.99 (104.16.45.99): icmp_seq=2 ttl=33 time=8.09 ms
	64 bytes from 104.16.45.99 (104.16.45.99): icmp_seq=3 ttl=33 time=7.97 ms
	...
	```
4. From your NAT instance, connect to your instance in your private subnet by using its private IP address.
	```bash
	[ec2-user@ip-10-0-4-184]$ ssh ec2-user@private-server-private-ip-address
	```
5. From your private instance, test that you can connect to the internet by running the **ping** command.
	```bash
	[ec2-user@ip-10-0-135-25]$ ping ietf.org
	```
	To confirm that your private instance has internet access through the NAT instance verify that you received output such as the following, and then press **Ctrl+C** to cancel the **ping** command.
	```
	PING ietf.org (104.16.45.99) 56(84) bytes of data.
	64 bytes from 104.16.45.99 (104.16.45.99): icmp_seq=1 ttl=33 time=8.76 ms
	64 bytes from 104.16.45.99 (104.16.45.99): icmp_seq=2 ttl=33 time=8.26 ms
	64 bytes from 104.16.45.99 (104.16.45.99): icmp_seq=3 ttl=33 time=8.27 ms
	...
	```

###### Troubleshooting

If the **ping** command fails from the server in the private subnet, use the following steps to troubleshoot the issue:

- Verify that you pinged a website that has ICMP enabled. Otherwise, your server can't receive reply packets. To test this, run the same **ping** command from a command line terminal on your own computer.
- Verify that the security group for your NAT instance allows inbound ICMP traffic from your private subnet. Otherwise, your NAT instance can't receive the **ping** command from your private instance.
- Verify that you disabled source/destination checking for your NAT instance. For more information, see [5\. Disable source/destination checks](#EIP_Disable_SrcDestCheck).
- Verify that you configured your route tables correctly. For more information, see [6\. Update the route table](#nat-routing-table).

### Step 4: Clean up

If you no longer require the test server in the private subnet, terminate the instance so that you are no longer billed for it. For more information, see [Terminate your instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/terminating-instances.html) in the *Amazon EC2 User Guide*.

If you no longer require the NAT instance, you can stop or terminate it, so that you are no longer billed for it. If you created a NAT AMI, you can create a new NAT instance whenever you need one.