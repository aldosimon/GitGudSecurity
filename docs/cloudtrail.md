---
title: CloudTrail
created: 2025-02-12
description: 
tags:
  - cloudsec
source:
---

# CloudTrail

CloudTrail monitors and logs the API activity. 

CloudTrail and IAM are linked, and understanding one helps with the other. Nearly Every API call to AWS is authenticated, and all API calls must be authorized. This is done via IAM.
[[Incident Response in AWS - Chris Farris]]

if you have [aws-organization](aws-protect.md#aws-organization) on, it is better to enable [organizational-cloudtrail](aws-protect.md#organization-cloudtrail)
## cloudtrail-best-practice

[cloudtrail-best-practice](https://aws.amazon.com/blogs/mt/aws-cloudtrail-best-practices/)

- [ ] to read cloudtrail best practice🔼 
## Organization Policy and CloudTrail

**Resource Control Policies**  need to enable **very expensive** **CloudTrail DataEvents** and review the vast data produced.  While **Service Control Policy** API calls is readily available
With **service control policies**, it is possible to review CloudTrail to determine who may be calling the specific actions that will be denied. [[Defining Security Invariants  PrimeHarbor]]
## Default settings 

Cloudtrail is always on by default, but only storing for 90d. this can be accessed via Event history. To store more set up a trail and use S3 (make sure **multi trail** or **organization** trail).

AWS CloudTrail (AWS's logging for API calls) **do not log data events**.  [AWS CloudTrail data management events](https://repost.aws/knowledge-center/cloudtrail-data-management-events)
**NOT LOGGED:**
- s3:PutObject
- s3:ListObjects
- s3:CopyObject
- s3:GetObject
However, when TA use S3:ListBuckets this is logged.

While it's possible to enable data event logging, this can get expensive due to additional charges applying for data events and the high-volume nature of data events.
[[Ransomware in AWS S3 SSE-C#Observations and Questions on Ransomware in S3]]


## What does it log? 
[[How to be IR Prepared in AWS - Cado Security  Cloud Forensics & Incident Response#AWS CloudTrail]]
CloudTrail records API activity such as the user agent, IP address, IAM user or role ARN, as well as any service-specific details about the request

Not all authenticated events are logged in CloudTrail. 
Only the management events/ control plane events. [logging management events](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-management-events-with-cloudtrail.html)
If you want to track access to specific Data sources, like S3, DynamoDB, or Lamba Functions, you need to enable Data Events for CLoudTrail [AWS IR](https://www.chrisfarris.com/post/aws-ir/)

Here is how to log data events [logging data events](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-data-events-with-cloudtrail.html)

Use [event selector](https://docs.aws.amazon.com/awscloudtrail/latest/APIReference/API_EventSelector.html) to specify `ReadOnly` or `WriteOnly` and [advance event selector](https://docs.aws.amazon.com/awscloudtrail/latest/APIReference/API_AdvancedEventSelector.html) to specify`eventSource`, `eventName` and `eventCategory`

## Where does it log to? 
[[How to be IR Prepared in AWS - Cado Security  Cloud Forensics & Incident Response#AWS CloudTrail]]

CloudTrail is **enabled by default** and will preserve **the past 90 days of activity**.  However, as best practice it should be configured to also send Logs to an S3 bucket for longer term storage. There is also the option to send logs to CloudWatch.

Data event is really voluminous, so don’t feed them into a SEIM. Instead, keep them in S3 because for when needed (occasional cost optimization research or in the event of an incident).

## How do I enable it? 
[[How to be IR Prepared in AWS - Cado Security  Cloud Forensics & Incident Response#AWS CloudTrail]]
CloudTrail logs are enabled by default. To enable the ability to send logs to an S3 bucket for longer term storage:
- First create an S3 bucket
- Then go to the AWS Management Console, and select the CloudTrail service.
- From the cloud trail dashboard go to _CloudTrail Insights > Create a trail,_ give the trail a name and configure its attributes; for storage select the S3 you created earlier.
- Finally choose your log events, management events and data events. Review and create your trail.

These events capture activity made through the AWS Management Console, AWS Command Line Interface, and AWS SDKs and APIs
## How do I access the logs? 
[[How to be IR Prepared in AWS - Cado Security  Cloud Forensics & Incident Response#AWS CloudTrail]]

Logs can be accessed and downloaded via the cloud trail console: [https://console.aws.amazon.com/cloudtrail/home/](https://console.aws.amazon.com/cloudtrail/home/).To view recent events, go to **Event History**.

To query CloudTrail events in S3, you can use Athena. I’ll point you at the [AWS Documentation for setting this up, they have an excellent explanation](https://docs.aws.amazon.com/athena/latest/ug/cloudtrail-logs.html).



The following AWS managed policies are available for CloudTrail:

* [AWSCloudTrail_FullAccess](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/AWSCloudTrail_FullAccess.html) – This policy provides full access to CloudTrail actions on all CloudTrail resources. This policy provides the required permissions to **create**, **update**, and **delete** CloudTrail trails, event data stores, and channels as well as permissions to manage the Amazon S3 bucket. It **doesn’t provide permissions to delete the Amazon S3 bucket**, the log group for CloudWatch Logs, or an Amazon SNS topic. Users with this role can turn off or reconfigure important auditing functions in their AWS accounts so use of this policy should be closely controlled and monitored.
* [AWSCloudTrail_ReadOnlyAccess](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/AWSCloudTrail_ReadOnlyAccess.html) – This policy grants permissions to view the CloudTrail console, including recent events and event history. This policy also allows you to view existing trails, event data stores, and channels.

## See also CloudTrail and centralized root management: [[aws-protect#AWS centralized root management|AWS protect]]

## aws-cloudtrail-limitation

there is quota limit for cloudtrail, you can see here [aws cloudtrail limit](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/WhatIsCloudTrail-Limits.html)
## References
[AWS-IR](https://www.chrisfarris.com/post/aws-ir/)
[aws-detect-and-response](aws-detect-and-response.md)
[[AWS CloudTrail Documentation]]