---
title: CloudWatch
created: 2025-02-12
description: 
tags: 
source:
---
# Related notes and references
[[aws-detect-and-response]]

# CloudWatch
[Almost all AWS services](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/aws-services-cloudwatch-metrics.html) support sending logs to CloudWatch. 

CloudWatch monitors your AWS resources and the applications that you run on AWS in real time. You can collect and track metrics, create customized dashboards, and set alarms. Making it **both a repository and a source for logs** from across AWS. 

With CloudWatch, you have the option to either manually export the logs to S3 storage or stream them directly to other places via other AWS services.
[[How to be IR Prepared in AWS - Cado Security  Cloud Forensics & Incident Response#CloudWatch]]

### What does it log? 
CloudWatch will log whatever logs other services are configured to send to it. [[How to be IR Prepared in AWS - Cado Security  Cloud Forensics & Incident Response#CloudWatch]]

#### IMDSv2 and CloudWatch
For IMDSv2 enforcement, you can review the CloudWatch Metric `MetadataNoToken` to see how many API calls still use the old system.

### Where does it log to?
CloudWatch can send logs to many different places, Manually there is the option to send the logs to an S3 bucket. If a subscription is configured, then there is the option to send the logs to Lambda, Kinesis Data Stream, or Kinesis Data Firehose Stream. [[How to be IR Prepared in AWS - Cado Security  Cloud Forensics & Incident Response#CloudWatch]]

### How do I enable it?
These logs are not enabled by default, all options either manual or subscription must be configured.
The method of configuring Logs to be sent to CloudWatch is dependent on the service you are looking to send logs from. A full list of services that support sending logs to CloudWatch can be found [here](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/aws-services-cloudwatch-metrics.html), along with documentation on configuring them.
[[How to be IR Prepared in AWS - Cado Security  Cloud Forensics & Incident Response#CloudWatch]]

### How do I access the logs?
To manually export logs to an S3 bucket, AWS provides [this guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/S3ExportTasksConsole.html). To configure real time access via subscriptions, AWS provides [this guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/Subscriptions.html).
[[How to be IR Prepared in AWS - Cado Security  Cloud Forensics & Incident Response#CloudWatch]]
