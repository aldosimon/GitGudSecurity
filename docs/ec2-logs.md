---
title: EC2 Logs
created: 
description: 
tags: 
source:
---
# Related notes and references
[[aws-detect-and-response]]

# EC2 Log

EC2 is AWS’ Elastic compute offering, it supports systems level logging, allowing logs directly from the OS on the instance **to be collected via the CloudWatch agent and stored in CloudWatch**.

### What does it Log?
[[How to be IR Prepared in AWS - Cado Security  Cloud Forensics & Incident Response#EC2]]
EC2 collects different system level logs depending on the operating system of the Instance. Below the [default logs](https://docs.aws.amazon.com/managedservices/latest/userguide/access-to-logs-ec2.html) that the CloudWatch agent in an EC2 instance collects:

For Amazon Linux / Red Hat Linux / Centos Linux / Ubuntu / SUSE Linux:
```Plain
/var/log/amazon/ssm/amazon-ssm-agent.log
/var/log/amazon/ssm/errors.log
/var/log/audit/audit.log
/var/log/cloud-init-output.log
/var/log/cfn-init.log
/var/log/cfn-init-cmd.log
/var/log/cloud-init.log (Amazon Linux 1 / Amazon Linux 2 only)
/var/log/cron
/var/log/maillog
/var/log/messages
/var/log/secure
/var/log/spooler
/var/log/yum.log
/var/log/aws/ams/bootstrap.log
/var/log/aws/ams/build.log
/var/log/syslog
/var/log/dpkg.log
/var/log/auth.log
/var/log/zypper.log
```


For Windows:
```Plain
SecurityEventLog
SystemEventLog
AmazonSSMAgentLog
MicrosoftWindowsAppLockerMSIAndScriptEventLog
MicrosoftWindowsAppLockerEXEAndDLLEventLog
AmazonCloudWatchAgentLog
EC2ConfigServiceEventLog (Windows Server 2012 R2 Only)
ApplicationEventLog
AmazonCloudFormationLog
MicrosoftWindowsGroupPolicyOperationalEventLog
AmazonSSMErrorLog
```

### Where does it log to?
[[How to be IR Prepared in AWS - Cado Security  Cloud Forensics & Incident Response#EC2]]
Logs from EC2 instances are sent to CloudWatch, and stored in a CloudWatch Log group with the **same name** as the instance.

### How do I enable it?
[[How to be IR Prepared in AWS - Cado Security  Cloud Forensics & Incident Response#EC2]]
To enable the logging, CloudWatch agent must be installed. AWS provides a guide on this [here](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/install-CloudWatch-Agent-on-EC2-Instance.html).

### How do I access the logs? 
[[How to be IR Prepared in AWS - Cado Security  Cloud Forensics & Incident Response#EC2]]
Logs can be accessed via CloudWatch

