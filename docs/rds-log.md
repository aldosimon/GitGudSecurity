---
title: RDS log
created: 2025-02-12
description: 
tags: 
source:
---
# Related notes and references
[[aws-detect-and-response]]

# RDS log
Amazon RDS (Relational Database Service) has supported its own intel instance level logging for some time now, but it also has the additional capability to have those logs fed into CloudWatch.

### What does it log?
[[How to be IR Prepared in AWS - Cado Security  Cloud Forensics & Incident Response#RDS]]
RDS **instance level-logs** record general, error, audit, and slow query database events.

**Where does it log to?**[[How to be IR Prepared in AWS - Cado Security  Cloud Forensics & Incident Response#RDS]]
RDS logs are stored internal to the RDS service but it can be **optionally configured** to send the logs to CloudWatch.

**How do I enable it?**[[How to be IR Prepared in AWS - Cado Security  Cloud Forensics & Incident Response#RDS]]
RDS logs are **enabled by default**, but the CloudWatch integration needs to be manually configured. To do this:
- Open the Amazon RDS console at [https://console.aws.amazon.com/rds/](https://console.aws.amazon.com/rds/).
- Go to Databases, and choose the DB instance that you want to modify.
- In the Log exports section, choose the logs that you want to start publishing to CloudWatch Logs.
- Choose Continue, and then choose Modify DB Instance on the summary page.

**How do I access the logs?**[[How to be IR Prepared in AWS - Cado Security  Cloud Forensics & Incident Response#RDS]]
The Internal RDS service logs are accessible via the RDS console or API. If the CloudWatch integration is enabled, then logs can also be viewed in the CloudWatch dashboard.