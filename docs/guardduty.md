---
title: GuardDuty
created: 2025-02-12
description: 
tags:
  - AWS
source:
---
# Related notes
[[aws-detect-and-response]]
# GuardDuty

Amazon GuardDuty is AWS’s managed threat detection service It leverages three primary data feeds:
- CloudTrail
- VPC Flow Logs
- DNS Query Logs 
The findings it produces are either around an Identity or a Network-based device. The findings can also be **explicit** or **anomaly-based**. With Delegated Admin capabilities, GuardDuty can easily manage and deployed via AWS Organizations.
[[Incident Response in AWS - Chris Farris]]


GuardDuty **can be access from AWS RDSDB**, this is seen as part of defense evasion from  invictus-IR article [[cloud-detection-catalogue#defense-evasion techniques]]


## GuardDuty coverage

a list of GuardDuty [finding types](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-active.html)


## Cost

As with other Security investment, it is ROI sensitive.

GuardDuty cost can be challenging due to the rates are based on amount of analyzed data from various, unpredictable sources like CloudTrail, Flow Logs, DNS.

## Latency

Another problem with GuardDuty is the finding will obviously have delay.
Read more on this from [tracebit](https://tracebit.com/blog/a-hard-look-at-guardduty-shortcomings#footnote-6)
## Tools
[amazon-guardduty-tester](https://github.com/awslabs/amazon-guardduty-tester): This repository contains scripts and guidance that can be used as a proof-of-concept to generate Amazon GuardDuty findings related to real AWS resources

## References
[tracebit](https://tracebit.com/blog/a-hard-look-at-guardduty-shortcomings#footnote-2)