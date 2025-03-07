---
title: AWS Security Baby Steps
created: 
description: 
tags: 
source:
---
# aws-security-baby-steps

these are steps and security consideration from cloud slaw and other source (being added as of now)

## steps

### root-user

#### create-admin

use MFA!
#### limit-root-account

also see [protect-ou](aws-security-baby-steps.md#protect-ou)
##### delegated-administration

delegate management account privilege with this. use for IAM, billing, so you don't have to check billing using high privilege account day to day basis

Users/roles in a delegated account can administer the service. Each service is a bit different in terms of how much is delegated. For example Identity Center doesn’t let the delegated admin change any permission sets that were created directly in the management account, which helps prevent a delegated admin from making themselves an admin in the management account itself. But nearly all other capabilities are fair game. remember this when you are creating account.

this help with least privilege and reducing blast radius

if you are delegating IAM identity center, permission **AWSSSOMemberAccountAdministrator** should also be present in the delegated account

more here: [slaw](https://slaw.securosis.com/p/enable-delegated-administrator-identity-center-cloudtrail) and [aws](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_delegate_policies.html)
### create-organization
#### set-up-org-contact
security contact is important to set up, aside from that  create: 
- Billing information
- Operational information
- Security information
also see [security-account](./aws-detect-and-response.md#security-account)

how to? 
- enable trusted access for aws account management
- change email account at the contact info of the management account

### setup-cloudtrail

setup organization cloud trail, make sure it is multi trail. use LogArchive user to set it up.
the s3 where it store should have lifecycle and versioning (send to glacier if needed).

use your log archive account -> go to s3 -> go to org cloudtrail -> management -> lifecycle rule -> choose apply to all object -> tick expire current version of object -> put in how many days at the bottom
### create-ou

At a minimum you want OUs for governance/security, infrastructure/operations, and workloads; as well as places to handle special cases such as incident response, mergers and acquisitions, and accounts you are deprovisioning.

one example, you need these accounts : security, infra, workload, IR, nursery, onboarding, sandbox, suspended, exceptions. also see [security-account](./aws-detect-and-response.md#security-account)

| type | name             | function |
| ---- | ---------------- | -------- |
| OU   | Security         |          |
| OU   | Workload/Prod    |          |
| OU   | Infrastructure   |          |
| OU   | Exception        |          |
| OU   | Suspended        |          |
| OU   | Oboarding        |          |
#### protect-ou

these OU should have policy to deny these things, except the **exception OU**!
1. leave org.
2. deny root any action -> or kill root account or use root central management
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "organizations:LeaveOrganization"
      ],
      "Resource": "*",
      "Effect": "Deny"
    },
    {
      "Action": "*",
      "Resource": "*",
      "Effect": "Deny",
      "Condition": {
        "StringLike": {
          "aws:PrincipalArn": [
            "arn:aws:iam::*:root"
          ]
        }
      }
    }
  ]
}
```

### identity

as a guidance have at minimum these account, also read about [delegated-administration](aws-security-baby-steps.md#delegated-administration)

##### account

| OU       | name          | desc.                                                 | info                                                                                             |
| -------- | ------------- | ----------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| Security | SecurityAudit | delegated administrator for cloudtrail function       | SecurityAudit and LogArchive is separated. so ops can access log without messing with cloudtrail |
| Security | LogArchive    | create, access, maintain storing of cloudtrail in s3. | SecurityAudit and LogArchive is separated. so ops can access log without messing with cloudtrail |
| Security | IAM           | delegated administrator for IAM function              |                                                                                                  |
| Security | SecOps        |                                                       |                                                                                                  |

#### roles

#### security-permission
set up permission role for **security** with this pattern:
- A _full-read role_ with a read-only policy.
- A _full admin_ role that can do anything (for break glass, incident response)
- An _IAM role_
- additional cross-account role

set up by following the steps:
create permission set -> create group (if needed) ->  assign account the group and permission set.

more here at [slaw](https://slaw.securosis.com/p/creating-security-team-permissions-iam-identity-center) and [aws](https://docs.aws.amazon.com/singlesignon/latest/userguide/permissionsetsconcept.html)
### policies
#### deny-unused-region

if your org do not use a certain region, just block that. reduce attack surface.

when you are writing the policy, remember this: **Deny statements override allow statements, but allow statements in different policies all add up together.**

_**We use double negatives in AWS policies because they enable us to say “block everything except this”. This keeps us secure when new services, permissions (e.g., IAM user policies), actions, or regions are added!**_

also see other scp examples on [aws-protect](aws-protect.md#scp-samples)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyAllOutsideRegions",
      "Effect": "Deny",
      "NotAction": [
        "a4b:*",
        "acm:*",
        "aws-marketplace-management:*",
        "aws-marketplace:*",
        "aws-portal:*",
        "budgets:*",
        "ce:*",
        "chime:*",
        "cloudfront:*",
        "config:*",
        "cur:*",
        "directconnect:*",
        "ec2:DescribeRegions",
        "ec2:DescribeTransitGateways",
        "ec2:DescribeVpnGateways",
        "fms:*",
        "globalaccelerator:*",
        "health:*",
        "iam:*",
        "importexport:*",
        "kms:*",
        "mobileanalytics:*",
        "networkmanager:*",
        "organizations:*",
        "pricing:*",
        "route53:*",
        "route53domains:*",
        "route53-recovery-cluster:*",
        "route53-recovery-control-config:*",
        "route53-recovery-readiness:*",
        "s3:GetAccountPublic*",
        "s3:ListAllMyBuckets",
        "s3:ListMultiRegionAccessPoints",
        "s3:PutAccountPublic*",
        "shield:*",
        "sts:*",
        "support:*",
        "trustedadvisor:*",
        "waf-regional:*",
        "waf:*",
        "wafv2:*",
        "wellarchitected:*"
      ],
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": [
            "us-east-1",
            "us-west-2"
          ]
        }
      }
    }
  ]
}



```




details [here](https://slaw.securosis.com/p/meaning-lifecycles-versions-ransomware)

### guardduty

enable GuardDuty and delegate admin to your SecurityAudit account.
also remember to turn on auto enable for all account, check in account tab in guardduty see if all account is enabled.
also remember to enable in all your region.

### aws-security-hub

enable securityhub, but if you are looking just to aggregate findings (without config and standards), do not use aws config and standard, because this can get costly. more on [aws-protect](./aws-protect#aws-security-hub)

you can later connect security hub to siem using cloudwatch or load from s3 bucket. more at [aws](https://docs.aws.amazon.com/securityhub/latest/partnerguide/prepare-receive-findings.html) and also [aws in using securityhub and opensearch](https://aws.amazon.com/blogs/security/how-to-use-aws-security-hub-and-amazon-opensearch-service-for-siem/)
## references-and-related

[slaw](https://slaw.securosis.com/)

