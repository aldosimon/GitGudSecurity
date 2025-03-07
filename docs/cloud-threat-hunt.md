---
title: Cloud Threat Hunt
created: 
description: 
tags: 
source:
---
# about

## principles

- [ ] to be populated🔽 

## hunt-ideas
#### Amazon IAM [[Following attackers’ (Cloud)trail in AWS Methodology and findings in the wild   Datadog Security Labs | datadog 2023 TH]]
 - High volumes of `access denied` errors from a specific identity
- IAM user creation events from EC2 instances (where the role session name starts with `i-`)
- `Access denied` errors that occurred when creating an IAM user, especially when the same IAM user name was attempted to be created across multiple environments
- IAM user creation events from identities that had never created IAM users in the past
- IAM user names with grammatical errors or slight deviations of a common word

## reference-and-related
[[infosec-compendiums|infosec-compendiums]]
