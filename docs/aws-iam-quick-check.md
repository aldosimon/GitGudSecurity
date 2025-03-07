---
title: AWS IAM Quick Check
created: 2025-02-18
description: 
tags: 
source:
---
# about

this guide how to do aws iam quick check, by first generating credential report and checking againts aws iam top 10

## generate-credential-report

Recommended to do this periodically (via lambda). Generate credential report via CLI
```
aws iam generate-credential-report
```

## check-top-10

### has the root user been accessed recently?
Check in 90 days period

### Does the root account have multi-factor authentication (MFA) enabled?

### Does the root account have access keys enabled?
Root should not need access keys

### Are there any users with access keys and console credentials?

User should have either keys or console, but not both.

### Check for inactive users and users that have never logged in.

Find and delete any accounts where the users have passwords but have never logged in and accounts that haven't been used in last 90 days.

### Find any users with passwords that have not been rotated
Match this period with your policy

### Is MFA enabled for all users?


### Are access keys being rotated?
Match this period with your policy

### Can you find any users with unused access keys?
Deactivate

If you need a practical way of doing this, you can use steampipe, script is over [here](https://steampipe.io/blog/aws-iam-credential-report-top-10-checks) or [here](https://github.com/aldosimon/SteampipeCollection)
## References and related notes
[steampipe top 10 AWS IAM ](https://steampipe.io/blog/aws-iam-credential-report-top-10-checks)
[aws-protect](aws-protect.md)

