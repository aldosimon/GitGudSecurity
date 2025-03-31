---
title: Amazon VPC Log
created: 2025-02-12
description: 
tags: 
source:
---
# Related notes and references
[aws-detect-and-response](aws-detect-and-response.md)

# Amazon VPC

Amazon-VPC (Virtual Private Cloud) provides AWS customers a logically isolated section of Amazon Web Services Cloud. Allowing them to access the Amazon Elastic Compute Cloud over an IPsec based virtual private network.

**What does it log?** [](../01-source/How%20to%20be%20IR%20Prepared%20in%20AWS%20-%20Cado%20Security%20%20Cloud%20Forensics%20&%20Incident%20Response.md#Amazon%20VPC)
Amazon VPC records **Network Flow Logs** from across the Virtual Private Cloud. Flow log data for a monitored network interface is recorded as flow log records, which are log events consisting of fields that describe the traffic flow.

**Where does it log to?**[](../01-source/How%20to%20be%20IR%20Prepared%20in%20AWS%20-%20Cado%20Security%20%20Cloud%20Forensics%20&%20Incident%20Response.md#Amazon%20VPC)
Amazon VPC can **send logs to CloudWatch, S3 storage buckets** and/or **Kinesis firehose**.

**How do I enable it?**[](../01-source/How%20to%20be%20IR%20Prepared%20in%20AWS%20-%20Cado%20Security%20%20Cloud%20Forensics%20&%20Incident%20Response.md#Amazon%20VPC)
VPC Flow logs **are not enabled** by default. To create a flow log, you must specify the resource for which to create the flow log, the type of traffic to capture and the destinations to which you want to publish the flow log data. AWS provides three guides on setting up VPC flow logs for [S3](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs-s3.html), [CloudWatch](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs-cwl.html) and [FireHose](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs-firehose.html).
- [ ] research about firehouse

**How do I access the logs?**[](../01-source/How%20to%20be%20IR%20Prepared%20in%20AWS%20-%20Cado%20Security%20%20Cloud%20Forensics%20&%20Incident%20Response.md#Amazon%20VPC)
To view flow logs, take o one of the following steps:
- Open the [Amazon EC2 console](https://console.aws.amazon.com/ec2). In the navigation pane, choose Network Interfaces. Select the checkbox for the network interface.
- Open the [Amazon VPC console](https://console.aws.amazon.com/vpc/). In the navigation pane, choose Your VPCs. Select the checkbox for the VPC.
- Open the [Amazon VPC console](https://console.aws.amazon.com/vpc/). In the navigation pane, choose Subnets. Select the checkbox for the subnet.
Then choose Flow Logs.