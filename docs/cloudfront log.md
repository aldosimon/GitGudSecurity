---
title: CloudFront log
created: 2025-02-12
description: 
tags: 
source:
---
# Related notes and references
[[aws-detect-and-response]]
# CloudFront
CloudFront is **AWS’ content delivery network**. CloudFront has the option to log to an S3 bucket or real-time logging if needed via a Firehose stream.

**What does it log?** [[How to be IR Prepared in AWS - Cado Security  Cloud Forensics & Incident Response#CloudFront]]
CloudFront logs detailed information about every user request that CloudFront receives.

**Where does it log to?** [[How to be IR Prepared in AWS - Cado Security  Cloud Forensics & Incident Response#CloudFront]]
Logs are stored in an S3 bucket, or for real-time logs in the Kinesis Firehose data stream.

**How do I enable it?** [[How to be IR Prepared in AWS - Cado Security  Cloud Forensics & Incident Response#CloudFront]]
CloudFront logs are **not enabled by default**. To enable CloudFront access logs:
1. Access the [CloudFront console](https://console.aws.amazon.com/cloudfront/v3/home).
2. Choose the distribution you want to update.
3. On the General tab, under Settings, choose Edit.
4. For Standard logging, select On.
5. Choose the **S3 bucket where you want CloudFront** to deliver the log files. You can specify an optional prefix for the file names.
6. Choose Save changes.
To enable real-time logs:
1. Access the [CloudFront console](https://console.aws.amazon.com/cloudfront/v3/home).
2. From the left-hand navigation, select Logs.
3. Choose the Real-time configurations tab.
4. Choose Create configuration.
5. For Sampling rate, enter the percentage of requests for which you want to receive real-time log records.
6. For Fields, choose the specific fields that you want to receive in the log records. In the Choose options dropdown list, select any fields that you want to include in the configuration.
7. Choose one or more Kinesis data streams to receive real-time logs.Note: CloudFront real-time logs are delivered to the data stream of your choice in Amazon Kinesis Data Streams. To read and analyze your real-time logs, you can build your own [Kinesis data stream consumer](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/real-time-logs.html#real-time-log-consumer-guidance). Or, use [Amazon Kinesis Data Firehose](https://docs.aws.amazon.com/firehose/latest/dev/what-is-this-service.html) to send the log data to Amazon S3, [Amazon Redshift](https://docs.aws.amazon.com/redshift/latest/mgmt/welcome.html), [Amazon OpenSearch Service](https://aws.amazon.com/opensearch-service/the-elk-stack/what-is-opensearch/), or a third-party log processing service.
8. For IAM role, choose Create new service role for the console to create an IAM role for you. To use this option, you must have permission to create IAM roles.or-Use an existing IAM role.
9. (Optional) In the Distribution section, choose a CloudFront distribution and cache behavior to attach to the real-time log configuration.
10. Choose Create configuration.

**How do I access the logs?**[[How to be IR Prepared in AWS - Cado Security  Cloud Forensics & Incident Response#CloudFront]]
The standard logs can be downloaded from the S3 bucket. Accessing the real time logs is dependent on the endpoint of your Amazon Kinesis Data Streams.
