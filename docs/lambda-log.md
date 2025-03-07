---
title: Lambda log
created: 2025-02-12
description: 
tags: 
source:
---
# Related notes and references
[[aws-detect-and-response]]

# Lambda

AWS Lambda is a compute service that lets you **run code without provisioning** or managing servers. It supports Logging of application logs from the running code.
_Lambda Logs Viewed in CloudWatch_
**What does it log?** [[How to be IR Prepared in AWS - Cado Security  Cloud Forensics & Incident Response#Lambda]]
Lambda logs any application logs sent to **stdout/stderr** from your running code.

**Where does it log to?** [[How to be IR Prepared in AWS - Cado Security  Cloud Forensics & Incident Response#Lambda]]
Logs are sent to CloudWatch.

**How do I enable it?** [[How to be IR Prepared in AWS - Cado Security  Cloud Forensics & Incident Response#Lambda]]
They are enabled by default, however you must ensure that your function has **permission** to create log groups and/or streams

**How do I access the logs?**[[How to be IR Prepared in AWS - Cado Security  Cloud Forensics & Incident Response#Lambda]]
Logs can be accessed via the CloudWatch console:
1. Open the [Functions](https://console.aws.amazon.com/lambda/home#/functions) page of the Lambda console.
2. Choose a function.
3. Choose Monitor.
4. Choose View Logs in CloudWatch.