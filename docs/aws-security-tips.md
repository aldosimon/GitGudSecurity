---
title: AWS Security Tips
created:
description:
tags:
source:
---
# AWS Security Tips

The categorization of domains are based off [CSMM](https://www.iansresearch.com/resources/cloud-security-maturity-model) to make it easier to tackle
## Foundational Domain

Foundationals are: Core, critical categories to ensure availability of a secure baseline.

### Governance
### Organization

#### root user

#### create-admin

use MFA!
#### limit root account

also see [#protect-ou](#protect-ou)
##### delegated administrator

delegate management account privilege with this. use for IAM, billing, so you don't have to check billing using high privilege account day to day basis

read more about delegated administrator at [aws protect](aws-protect.md)
more here: [slaw](https://slaw.securosis.com/p/enable-delegated-administrator-identity-center-cloudtrail) and [aws](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_delegate_policies.html)
#### create aws organization
##### set up org contact
security contact is important to set up, aside from that  create: 
- Billing information
- Operational information
- Security information
also see security account at [aws detect & response](aws-detect.md)

how to? 
- enable trusted access for aws account management
- change email account at the contact info of the management account


#### create ou

At a minimum you want OUs for governance/security, infrastructure/operations, and workloads; as well as places to handle special cases such as incident response, mergers and acquisitions, and accounts you are deprovisioning.

one example, you need these accounts : security, infra, workload, IR, nursery, onboarding, sandbox, suspended, exceptions. also see security account at [aws detect & response](aws-detect.md)


| type | name             | function |
| ---- | ---------------- | -------- |
| OU   | Security         |          |
| OU   | Workload/Prod    |          |
| OU   | Infrastructure   |          |
| OU   | Exception        |          |
| OU   | Suspended        |          |
| OU   | Oboarding        |          |
#### protect ou

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

#### policies
##### invariants 

invariants are: enforce/ alert on things that should **always** or **never** be true. idea is if there is no need for context (these things should always or never), no need for sec team time

Here are some examples of invariants you can apply in your policies:
- Configure account-wide security defaults, including S3 block public access, EBS and all other default encryption.
##### deny unused region

if your org do not use a certain region, just block that. reduce attack surface.

when you are writing the policy, remember this: **Deny statements override allow statements, but allow statements in different policies all add up together.**

_**We use double negatives in AWS policies because they enable us to say “block everything except this”. This keeps us secure when new services, permissions (e.g., IAM user policies), actions, or regions are added!**_

also see other scp examples on [](aws-protect.md#scp-samples)

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

### Identity

Identity perimeters from [cloud security](cloud-security.md):

1. management plane access model: you should have information how management plane access happens, and understand risk of them. in the future aim for single, auditable mechanism e.g. AWS SSO with your IdP (with phish resistant tech.)
   Remove insecure mechanism such as Access Key/Secret Key, any use of IAM Users directly, unused users and roles, and ensure secure configurations are used for cross-account access for humans (MFA) and services (ExternalID).
2. server access model: understand what are currently use and the risk, mitigate/remediate and try to aim for best practice in the long run. SSH w/ key (no password!) > bastion/ jump host > AWS SSM.
   in short term, get compensating control fail2ban, bastion but aim for the ideal cloud native offering(SSM)
3. IAM Security: do not use/secure root; clean up unused roles -> see access analyzer result; use IAM roles instead of users; understand risk of cross-account trust;
	- [IAM Credential Report](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_getting-report.html): Identify unused users and roles, and well as authentication patterns, such as MFA usage.
	- [IAM Access Analyzer](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html): Identify resources in your accounts shared with external entities.
	- [Trusted Advisor](https://aws.amazon.com/premiumsupport/technology/trusted-advisor/best-practice-checklist/) (free): Multi-factor authentication on root account, AWS IAM use.
	- [AWS Config](https://aws.amazon.com/config/) and/or [Security Hub](https://aws.amazon.com/security-hub/), if in use.

##### account

- as a guidance have at minimum these account, also read about [#delegated-administration](#delegated-administration)
- Ensure security visibility and break-glass access to all accounts.


| OU       | name          | desc.                                                 | info                                                                                             |
| -------- | ------------- | ----------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| Security | SecurityAudit | delegated administrator for cloudtrail function       | SecurityAudit and LogArchive is separated. so ops can access log without messing with cloudtrail |
| Security | LogArchive    | create, access, maintain storing of cloudtrail in s3. | SecurityAudit and LogArchive is separated. so ops can access log without messing with cloudtrail |
| Security | IAM           | delegated administrator for IAM function              |                                                                                                  |
| Security | SecOps        |                                                       |                                                                                                  |

#### roles

#### security permission

- Ensure security visibility and break-glass access to all accounts.

set up permission role for **security** with this pattern:
- A _full-read role_ with a read-only policy.
- A _full admin_ role that can do anything (for break glass, incident response)
- An _IAM role_
- additional cross-account role

set up by following the steps:
create permission set -> create group (if needed) ->  assign account the group and permission set.

more here at [slaw](https://slaw.securosis.com/p/creating-security-team-permissions-iam-identity-center) and [aws](https://docs.aws.amazon.com/singlesignon/latest/userguide/permissionsetsconcept.html)

### Security Monitoring

#### setup organization cloudtrail

- Enable Cloudtrail in all accounts; turn on optional security features, including encryption at-rest and file validation; centralize and back up logs.
- setup organization cloud trail, make sure it is multi trail. use LogArchive user to set it up. the s3 where it store should have lifecycle and versioning (send to glacier if needed).
- use your log archive account -> go to s3 -> go to org cloudtrail -> management -> lifecycle rule -> choose apply to all object -> tick expire current version of object -> put in how many days at the bottom

#### guardduty

- enable GuardDuty and delegate admin to your SecurityAudit account. 
- enable GuardDuty in all accounts, and centralize alerts.
- also remember to turn on auto enable for all account, check in account tab in guardduty see if all account is enabled.
- also remember to enable in all your region.

#### aws security hub

enable securityhub, but if you are looking just to aggregate findings (without config and standards), do not use aws config and standard, because this can get costly. more on [](aws-protect.md#aws-security-hub)

you can later connect security hub to siem using cloudwatch or load from s3 bucket. more at [aws](https://docs.aws.amazon.com/securityhub/latest/partnerguide/prepare-receive-findings.html) and also [aws in using securityhub and opensearch](https://aws.amazon.com/blogs/security/how-to-use-aws-security-hub-and-amazon-opensearch-service-for-siem/)

## Structural Domain

Structural Domains are Categories to protect the building blocks of your cloud environment.

### network

enumerate and understand the risk of your network perimeter (from [cloudsec](cloud-security.md)):

1. Public resources in managed services: understand resources that allow to be public thorugh RCP, sharing API, network access. [read more](https://github.com/SummitRoute/aws_exposable_resources)
2. Public network access to hosted services: leverage findings from tooling or trusted advisor, security groups that are open etc.
3. Default, insecure resources

### workload

### application

### data

## Procedural Domain
Procedural Domains are Categories to highlight the processes needed to protect your cloud (and keep it protected)

### Risk assessment

### Resilience

### Compliance and Audit
### Incident Response
## References

[slaw](https://slaw.securosis.com/)
[aws protect](aws-protect.md)
[aws detect and response](aws-detect.md)
[org-kickstart](https://github.com/primeharbor/org-kickstart)
[CSMM](https://www.iansresearch.com/resources/cloud-security-maturity-model?utm_source=slaw.securosis.com&utm_medium=referral&utm_campaign=stage-check-org-and-iam-foundation)
[cloudsec orienteering](https://tldrsec.com/p/blog-cloud-security-orienteering)