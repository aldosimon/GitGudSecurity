---
title: AWS Protect
created: 2025-01-31
tags:
  - protect
  - AWS
description: protect AWS asset using these methods from various source
---
# AWS Protect

is compiled notes, concept explanation about protecting your AWS assets
there is no order to these notes, if you want step by step go here 
## aws-s3

### s3-security-guidelines
guidance and best practice
- [ ] read up on aws s3 best practice🔼 
[s3-best-practice](https://docs.aws.amazon.com/AmazonS3/latest/userguide/security-best-practices.html#security-best-practices-prevent)
[s3-control-securityhub](https://docs.aws.amazon.com/securityhub/latest/userguide/s3-controls.html)
[top-10](https://aws.amazon.com/blogs/security/top-10-security-best-practices-for-securing-data-in-amazon-s3/)
- Block public S3 buckets at the organization level
- Use bucket policies to verify all access granted is restricted and specific
- Ensure that any identity-based policies don’t use wildcard actions
- Enable S3 protection in GuardDuty to detect suspicious activities
- Use Macie to scan for sensitive data outside of designated areas
- Encrypt your data in S3
- Protect data in S3 from accidental deletion using S3 Versioning and S3 Object Lock
- Enable logging for S3 using CloudTrail and S3 server access logging. also see [cloudtrail](cloudtrail.md), and [s3-logging](#s3-logging) for comparison
- Backup your data in S3
- Monitor S3 using Security Hub and CloudWatch Logs

here is a practical guide on s3 protect, from [ramimac-s3-logging](https://ramimac.me/s3-logging#about-s3-logging)
1. enforce IaaC (i.e. terraform), have standard modules for core services like S3. 
2. For S3, you can configure your module to enable S3 Access Logs by default. Take care, as you’ll need to handle each account:region pair where you will be hosting S3 buckets.
3. With your buckets represented as code, you can use a SAST tool or linter to: detect use of non hardened s3 bucket, detect when cloudtrail data events enabled.  
4. on buckets with data events, write  detections for unapproved or unexpected access.
5. as mentioned above, S3 access log need lambda for detection (see here [a Lambda trigger](https://docs.aws.amazon.com/lambda/latest/dg/with-s3-example.html) ) for the bucket. The Lambda will need to parse the log file and implement detection logic over the logs.
6. use CSPM to enforce invariants


using policy to protect from or scope down permission of IAM role from these action. 
read on [organizational-policy](#organizational-policy)
- `s3:DeleteObject`
- `s3:DeleteObjectVersion`
- `s3:PutLifecycleConfiguration`
- `s3:PutObject`
- `s3:GetObject`
- `s3:PutLifecycleConfiguration`
- `s3:ListObjects`
- `s3:ListBuckets` (theorized)
[](../01-source/Ransomware%20in%20AWS%20S3%20SSE-C.md#Securing%20Data%20in%20S3%20and%20Preventing%20S3%20SSE-C%20Ransomware)
- [ ] research public access blocks
### bucket-policies
- protect from SSE-C [](../01-source/Ransomware%20in%20AWS%20S3%20SSE-C.md#Securing%20Data%20in%20S3%20and%20Preventing%20S3%20SSE-C%20Ransomware)
	- deny object uploads with SSE-C encryption
	- denies non-KMS (AWS Managed Key or CMK)
	+ requires a specific CMK
- [ ] read more on bucket policy vs s3 policy?

### bucket-lifecycle

- [ ] read more on bucket lifecycle🔼 

### block-public-access
[read more](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html)

### s3-versioning

from [slaw](https://slaw.securosis.com/p/meaning-lifecycles-versions-ransomware)
We can enable S3 versions, which keep versions of files as they are changed and deleted. The API call to delete a version is different than the one to delete a current object, so we can use IAM policies or SCPs to block version deletion.

The problem is that gets a bit expensive. So we can use lifecycle policies to delete old versions, similar to how we will use it today to delete objects older than 90 days.

Or if we **really** don’t want to lose data, we can move it to AWS Glacier, very cheap storage which offers lock policies that prevent deletion for a set period of time. The downside of Glacier is that, depending on the storage class, it can take a day to get data back.

### s3-logging
also see [cloudtrail](cloudtrail.md)
- Only the management events/ control plane events. [logging management events](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-management-events-with-cloudtrail.html)
- however data events can be enabled. Here is how to log data events [logging data events](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-data-events-with-cloudtrail.html)
- There is also [server log](https://docs.aws.amazon.com/AmazonS3/latest/userguide/ServerLogs.html), to see request logs. It can only be configured with another S3 bucket as the target for log delivery. 
- use [event selector](https://docs.aws.amazon.com/awscloudtrail/latest/APIReference/API_EventSelector.html) to specify `ReadOnly` or `WriteOnly` and [advance event selector](https://docs.aws.amazon.com/awscloudtrail/latest/APIReference/API_AdvancedEventSelector.html) to specify`eventSource`, `eventName` and `eventCategory`
- set up the log using [organization trail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/creating-trail-organization.html)
- guide to [logging-s3-cloudtrail](https://docs.aws.amazon.com/AmazonS3/latest/userguide/cloudtrail-logging.html)
#### s3-server-vs-cloudtrail-logging

see here for comparison [s3 server logging vs aws cloudtrail](https://docs.aws.amazon.com/AmazonS3/latest/userguide/logging-with-S3.html)

| Log properties                                                                                       |                                            AWS CloudTrail                                            |                  Amazon S3 server logs                   |
| ---------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------: | :------------------------------------------------------: |
| Can be forwarded to other systems (Amazon CloudWatch Logs, Amazon CloudWatch Events)                 |                                                 Yes                                                  |                            No                            |
| Deliver logs to more than one destination (for example, send the same logs to two different buckets) |                                                 Yes                                                  |                            No                            |
| Turn on logs for a subset of objects (prefix)                                                        |                                                 Yes                                                  |                            No                            |
| Cross-account log delivery (target and source bucket owned by different accounts)                    |                                                 Yes                                                  |                            No                            |
| Integrity validation of log file by using digital signature or hashing                               |                                                 Yes                                                  |                            No                            |
| Default or choice of encryption for log files                                                        |                                                 Yes                                                  |                            No                            |
| Object operations (by using Amazon S3 APIs)                                                          |                                                 Yes                                                  |                           Yes                            |
| Bucket operations (by using Amazon S3 APIs)                                                          |                                                 Yes                                                  |                           Yes                            |
| Searchable UI for logs                                                                               |                                                 Yes                                                  |                            No                            |
| Fields for Object Lock parameters, Amazon S3 Select properties for log records                       |                                                 Yes                                                  |                            No                            |
| Fields for `Object Size`, `Total Time`, `Turn-Around Time`, and `HTTP Referer` for log records       |                                                  No                                                  |                           Yes                            |
| Lifecycle transitions, expirations, restores                                                         |                                                  No                                                  |                           Yes                            |
| Logging of keys in a batch delete operation                                                          |                                                  No                                                  |                           Yes                            |
| Authentication failures1                                                                             |                                                  No                                                  |                           Yes                            |
| Accounts where logs get delivered                                                                    |                                     Bucket owner2, and requester                                     |                    Bucket owner only                     |
| **Performance and Cost**                                                                             |                                          **AWS CloudTrail**                                          |                **Amazon S3 Server Logs**                 |
| Price                                                                                                | Management events (first delivery) are free; data events incur a fee, in addition to storage of logs |       No other cost in addition to storage of logs       |
| Speed of log delivery                                                                                |                   Data events every 5 minutes; management events every 15 minutes                    |                    Within a few hours                    |
| Log format                                                                                           |                                                 JSON                                                 | Log file with space-separated, newline-delimited records |


or from rami mac guide on [s3 logging](https://ramimac.me/s3-logging#about-s3-logging)

| Data Events                                                                                                                                                   | S3 Server Access Logs                                                                                  |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| Support [granular Event Selectors](https://aws.amazon.com/about-aws/whats-new/2020/12/aws-cloudtrail-provides-more-granular-control-of-data-event-logging/)   | Offer detail that is not available in Data Events, like `HTTP Referer`                                 |
| N/A                                                                                                                                                           | Log the Object Size and HTTP Referer                                                                   |
| Configured via Cloudtrail, and can easily be [managed centrally](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/creating-trail-organization.html) | Have to be configured on each bucket                                                                   |
| Logs can be routed to various and multiple systems                                                                                                            | Can only log to another S3 bucket in the same region and account                                       |
| Reliable and ~fast delivery (under 5 minutes, often much sooner)                                                                                              | Delivery is best effort and is “within a few hours”                                                    |
| Generally, are well supported as a data source in your SIEM of choice                                                                                         | Require a Lambda for detective controls, date partioning can enable querying (once enabled, post 2023) |
| Cost per Data Event, for storage (both in AWS and your SIEM), and for SIEM ingestion (often)                                                                  | Cost for storage in S3                                                                                 |
planning [s3 logging](https://ramimac.me/s3-logging#about-s3-logging)

consider characteristics of each bucket: read/write volume expected; risk (unauthorized write/read); frequency of user access. Then, based on those parameters, go:
- Turn on S3 Access Logs for any non-public bucket. Save them for a rainy day, at which point you can query them with Athena, somewhat painfully at scale
- Use Cloudtrail Data Events with Event Selectors, on buckets of reasonable volume **where you require detective controls on reads or writes**
- If you have a high volume bucket that **needs detective controls**, you’ll want to deploy a Lambda to process S3 Access Logs as they’re delivered
- To actually allow querying of S3 Access Logs, invest in post processing

## iam

also see [AWS IAM quick check](aws-iam-quick-check.md)
also see [](.md#aws-organization)

iam api call happens in N. Virginia.
### understand-how-user-use-iam

this should be one of the first step, understanding how user interact with aws services. after this then push to more secure route, by putting guardrail, not gate keeping.

### less-static-long-term-creds

Access Keys in AWS are typically tied to long-term credentials such as IAM users. In AWS, security best practices recommend moving away from long-term credentials to short term credentials. Instead of IAM Users, IAM roles are preferred.
[](../01-source/Ransomware%20in%20AWS%20S3%20SSE-C.md#Securing%20Data%20in%20S3%20and%20Preventing%20S3%20SSE-C%20Ransomware)

For more details about temporary credentials in AWS, [AWS has extensive documentation and AWS STS](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html).
- [ ] research more on temp creds AWS

#### AKIA-ASIA

IAM credentials with the `AKIA` prefix is long term access keys. These are associated with IAM users. These credentials can potentially be exposed and used by attackers, because they do not expire by default.

IAM credentials with the `ASIA` prefix belong to short lived access keys which were generated using [STS](https://docs.aws.amazon.com/STS/latest/APIReference/welcome.html). These credentials last for a limited time. 
### Rotate long-term keys to minimize exposure

### Scan code for stored credentials in CI/CD pipelines and repositories

### Require MFA for human access
Use MFA if you think the identity is important 

### Migrate administrators and highly-privileged access to Just in Time access and/or use hardware tokens

### Permission Reduction and Least Privilege
[**Least Privilege**](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege)
have one role per resources or per app, follow the least privilege principle
- [ ] research more on least privilege 

### root-account
Do not use root account. Use IAM user at least, or IAM role. better yet use aws org and accounts.

with AWS Organizations, **each AWS account** in your organization comes with its own root user. These root users have unrestricted administrative access to everything in their AWS accounts. controls for this account before centralized root management is typical e.g. MFA, password rotate, limit permission via SCP [Hands-On Security Tips For Centralize Root Access In AWS(AssumeRoot)](01-source/Hands-On%20Security%20Tips%20For%20Centralize%20Root%20Access%20In%20AWS(AssumeRoot).md)

centralized root access: [Hands-On Security Tips For Centralize Root Access In AWS(AssumeRoot)](01-source/Hands-On%20Security%20Tips%20For%20Centralize%20Root%20Access%20In%20AWS(AssumeRoot).md)
- centralize their management
- delete root user credentials
- reduced attack surface (with fewer root to manage)
- new account won't have root credentials by default
- With managed policies, such as _`IAMAuditRootUserCredentials`_, _`IAMCreateRootUserPassword`_, _`IAMDeleteRootUserCredentials`_, `S3UnlockBucketPolicy`, and `SQSUnlockQueuePolicy`, you are strictly limited to using only those policies.
- If root access is needed, it can be temporarily recovered, used for the specific task, and then removed again.

Assuming root would look like this
`aws sts assume-root --target-principal=123456789 --task-policy-arn arn=arn:aws:iam::aws/root-task/S3UnlockBucketPolicy`

CloudTrail would show `AssumeRoot` event type see here for example [](01-source/Hands-On%20Security%20Tips%20For%20Centralize%20Root%20Access%20In%20AWS(AssumeRoot).md#CloudTrail%20Logging%20How%20(Assume)Root%20Actions%20Look%20Now)

#### related API with centralized root management
These are related API with centralized root management that might be of interest when you design detection/ protect [Hands-On Security Tips For Centralize Root Access In AWS(AssumeRoot)](01-source/Hands-On%20Security%20Tips%20For%20Centralize%20Root%20Access%20In%20AWS(AssumeRoot).md)
- check if feature is enabled. `ListOrganizationsFeatures`
- Identify Accounts with Root Access. No simple API, but use these steps:
	- assume root with `IAMAuditRootUserCredentials ` 
		`aws sts assume-root --target-principal=123456789 --task-policy-arn arn=arn:aws:iam::aws/root-task/IAMAuditRootUserCredentials` 
	- `GetLoginProfile` API command without providing the `UserName`
		`aws iam get-login-profile`
	- If the root account doesn’t have a login profile, you should get the following error: 
		_An error occurred (NoSuchEntity) when calling the GetLoginProfile operation: Login Profile for User null cannot be found._
	- Also check for MFA and signed certs -> delete this also

### related detection and response: [aws-detect-and-response](aws-detect-and-response.md)

### credential report
https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_getting-report.html

```bash
aws iam generate-credential-report --profile <aws-cli-profile>
```

### IAM tools
 - [iamlive](https://github.com/iann0036/iamlive) 
 - [Policy Sentry](https://policy-sentry.readthedocs.io/en/stable/)
 - [IAM Zero](https://iamzero.dev/)
 - [ ] put this on infosec compendium projects

## network

### Monitor network perimeter

### Know your exposed asset

### put guardrails when creating new VPC

### auto-remediation of VPC if guardrail is impossible


## ec2

### inventory-of-internet-exposed-ec2

have this, because more risky.

## aws-policy

### policy-primer

these are types or policy
- identity based policies (inline and managed)
- instances policies
- resources based policies
- organizational service control policies
- organizational resources control policies
- permission boundaries
- ACL
- session policies


- [ ] read more on org policies, RCP - SCP

Almost all mid-to-large sized AWS environments make use of [multi-account](https://docs.aws.amazon.com/whitepapers/latest/organizing-your-aws-environment/organizing-your-aws-environment.html) architecture. Using multiple AWS accounts offers a number of [benefits](https://docs.aws.amazon.com/whitepapers/latest/organizing-your-aws-environment/benefits-of-using-multiple-aws-accounts.html) and is considered a best practice. To help organize and manage those accounts, AWS offers a service called [AWS Organizations](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html).

Policies in AWS Organizations enable you to apply additional types of management to the AWS accounts in your organization

#### policy-basic-principles
- default deny: They are default deny, and can only "remove deny". If you see an _allow_ permission that doesn’t mean something is being granted to someone— it just means the SCP won’t block it.
- logical sum 
- deny override allow

### Service Control Policies (SCPs)

SCPs **DO NOT** work on a management account

SCP is the OG of Security Invariants- define the maximum permissions of identities in your organization.

With **service control policies**, it is possible to review CloudTrail to determine who may be calling the specific actions that will be denied. [](../01-source/Defining%20Security%20Invariants%20%20PrimeHarbor.md#Safely%20leveraging%20these%20controls)

**SCPs do NOT grant anyone permissions, they only restrict what actions are allowed in an account!** Service Control Policies are guardrails which restrict what can happen in an account — they cannot grant permissions to anyone or anything in an account.

SCPs do not affect your management account, even when applied to your Organization’s root. This is another good reason to limit use of management accounts. - SCPs can restrict the root (but not management) account.

SCP can’t restrict access to resources from entities outside your account (e.g. if you create public s3, the SCP won’t evaluate that API call — because it happens outside your account). It can however, **block creation of public s3**, but previously created public s3 will still accessible.

SCPs can’t restrict service linked roles (like the one created when we enabled organizations).

SCPs are applied at the Organizational Unit or account level.

SCP samples: see [](.md#rcp-samples)

### Resource Control Policies (RCPs)
a new type of Organization Policy that defines the maximum permissions of *resources* in your organization. 
Resource Control Policies apply to the resources in your organization and can control any principal, even those outside of your control. [Only a few services support RCPs](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_rcps.html#rcp-supported-services) at this time: S3, KMS, SQS, SecretsManager, and IAM Roles (via STS). 
[](../01-source/Defining%20Security%20Invariants%20%20PrimeHarbor.md#Resource%20Control%20Policies)

**Resource Control Policies** are more complicated to preview because you’ll need to enable very expensive **CloudTrail DataEvents** and review the vast data produced. For some RCPs, you can probably use [IAM Access Analyser](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html#what-is-access-analyzer-resource-identification) [](../01-source/Defining%20Security%20Invariants%20%20PrimeHarbor.md#Safely%20leveraging%20these%20controls)

**RCP samples**: see [](.md#rcp-samples)

### iam-policies

references on [aws](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html)
AWS managed policies exist in every AWS account. Customer managed policies only exist in the account where they are created, so AWS needs to provision those policies into accounts for it all to work.
### declarative-policies
These policies exist outside of IAM evaluation and enforce **specific controls** at the AWS Service.
You can determine the impactof DP with a simple CSPM review for public images and snapshots. For IMDSv2 enforcement, you can review the CloudWatch Metric `MetadataNoToken` to see how many API calls still use the old system. 
[](../01-source/Defining%20Security%20Invariants%20%20PrimeHarbor.md#Safely%20leveraging%20these%20controls)
[read more on AWS](https://aws.amazon.com/blogs/security/get-the-full-benefits-of-imdsv2-and-disable-imdsv1-across-your-aws-infrastructure/)
- [ ] read more on IMDSv2

### permission-boundaries
These **IAM Policies** don’t grant permissions but rather define the maximum permissions of the principal (**IAM User or Role**) to which they are attached.
- [x] read security invariants ✅ 2025-02-02
### examples-of-implementing-policies

note on samples: You should not attach RCPs without thoroughly testing the impact that the policy has on resources in your accounts. 
Once you have a policy ready that you would like to implement, we recommend testing in a separate organization or OU that can represent your production environment. 
Once tested, you should deploy changes to test OUs and then progressively deploy the changes to a broader set of OUs over time.
#### prime harbor security invariants
- [Defining Security Invariants  PrimeHarbor](../01-source/Defining%20Security%20Invariants%20%20PrimeHarbor.md)
- enforce/ alert on things that should **always** or **never** be true. idea is if there is no need for context (these things should always or never), no need for sec team time
#### scp-samples
- [prime harbor AWS OP github](https://github.com/primeharbor/aws-organizational-policies)
- [infralicious](https://github.com/infralicious/awesome-service-control-policies)
- [rami scps](https://rami.wiki/scps/)
- [aws examples](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_examples.html)

#### rcp-samples
[samples of RCP](https://github.com/aws-samples/resource-control-policy-examples)
#### lesson-learned-from-incidents 
- protect from SSE-C ([](../01-source/Ransomware%20in%20AWS%20S3%20SSE-C.md#Securing%20Data%20in%20S3%20and%20Preventing%20S3%20SSE-C%20Ransomware))
	- deny object uploads with SSE-C encryption 
	- Requires KMS Encryption (which can be either an AWS Managed Key or Customer Managed Key)

### note-on-aws-managed-policies 
[A SaaS provider's guide to securely integrating with customers' AWS accounts   Datadog Security Labs](../01-source/A%20SaaS%20provider's%20guide%20to%20securely%20integrating%20with%20customers'%20AWS%20accounts%20%20%20Datadog%20Security%20Labs.md)
While policies like `ReadOnlyAccess` might seem like an easy way to grant limited permissions, AWS managed policies are typically **over-privileged** and **allow overly-sensitive**, unnecessary actions. For example:

do not use aws managed policy, but use aws managed policy as a starter to build your own.

- [`ReadOnlyAccess`](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/ReadOnlyAccess.html) allows full read access to all data in S3 buckets and DynamoDB tables in the account, as well as read access to all SSM parameters, which typically (and rightly) contain secrets.
- [`SecurityAudit`](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/SecurityAudit.html) contains the `ec2:DescribeInstanceAttribute` permission, which grants privileges to retrieve user data configuration from all EC2 instances, often containing sensitive data or configuration secrets. It also grants `lambda:ListFunctions`, which permits retrieval of the value of all environment variables for all Lambda functions in the account.

In addition, AWS can add or remove permissions from these policies at any time without notice, and they do so on a [daily basis](https://github.com/zoph-io/MAMIP/commits/master/), granting you no control over its lifecycle.

## aws-organizations

this is the way to scale up your aws deployment. make sure you check this before going organizational:
- Never ever ever enable Organizations (or deploy Control Tower) from an account which is running any workloads!!!
- have at least one iam account, **do not** use your root.
- use MFA
- Protect the management account, minimize who has access, and minimize what you deploy there!!!! 
- activate trail (make sure it is multi region)
- activate billing alert. use sns if needs be

### magic-roles

these are roles that related to creation of aws organization
_Service Linked Role_: This is a unique type of IAM role directly linked to an AWS service. It’s predefined by the service and includes all the permissions the service needs to perform actions on your behalf. These roles simplify setup since you don’t need to manually add permissions for the service
_AWSServiceRolesForOrganizations_ : This role is used by AWS Organizations to enable trusted access for other AWS services. It allows these services to perform tasks across all accounts in your organization, such as managing resources or configurations
_OrganizationAccountAccessRole_ : allow root to assume member account

### delegated-administrator

Users/roles in a delegated account can administer the service. Each service is a bit different in terms of how much is delegated. For example Identity Center doesn’t let the delegated admin change any permission sets that were created directly in the management account, which helps prevent a delegated admin from making themselves an admin in the management account itself. But nearly all other capabilities are fair game. remember this when you are creating account.

this help with least privilege and reducing blast radius

if you are delegating IAM identity center, permission **AWSSSOMemberAccountAdministrator** should also be present in the delegated account

for use cases of read more at [wiz](https://www.wiz.io/blog/use-cases-for-delegated-administrator-for-aws-organizations)

## backup-policy

## AWS backup encryption
https://docs.aws.amazon.com/aws-backup/latest/devguide/encryption.html



## configuration-management
[aws-security-best-practices-cheat-sheet](../01-source/aws-security-best-practices-cheat-sheet.pdf)

### scan with AWS config
- [ ] read more on aws config


## aws-security-hub

Security hub serve three purposes:
- Collects events and results from nearly _any other AWS security service,_ all in one place. and Security Hub normalizes findings from those services into a standard format. (i.e. AWS Security Findings Format - ASFF)
- Work as a Cloud Security Posture Management (CSPM) tool. Think of CSPM as a vulnerability scanner for your cloud.
- Consolidate events and findings across accounts and regions in an Organization

AWS security hub enable AWS config and security standards. These are expensive, if you are looking to consolidate stuff from other security tool, disable these.

read more at [security hub](https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html)
- [x] read more on AWS security hubs ✅ 2025-03-06


## aws-risk-management

amazon detective
- [ ] read on Amazon detective
- [ ] read on AWS risk management


## tooling

### basic

| name                                              | description                    |
| ------------------------------------------------- | ------------------------------ |
| [aws-cli](https://github.com/aws/aws-cli)         | the aws-cli, what else         |
| [aws-shell](https://github.com/awslabs/aws-shell) | interactive version of aws-cli |

### inventory

| name                                                               | description                                                                                                                                  |
| ------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------- |
| [aws-inventory](https://github.com/nccgroup/aws-inventory)         | tool that tries to discover all [AWS resources](https://docs.aws.amazon.com/general/latest/gr/glos-chap.html#resource) created in an account |
| [Resource Counter](https://github.com/disruptops/resource-counter) | this command line tool counts the number of resources in different categories across Amazon regions.                                         |
| [aws_public_ips](https://github.com/arkadiyt/aws_public_ips)       | aws_public_ips is a tool to fetch all public IP addresses (both IPv4/IPv6) associated with an AWS account.                                   |

### audit
| name                                                                        | description                                                                                                                                                                    |
| --------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [CS-Suite](https://github.com/SecurityFTW/cs-suite)                         | This is a suite of cloud security tools designed to automate various security assessments and tasks                                                                            |
| [CloudSploit](https://github.com/cloudsploit/scans)                         | CloudSploit is an open-source cloud security scanner. It's designed to detect security risks in cloud infrastructure, including AWS, Azure, Google Cloud, and others.          |
| [AWS Security Benchmark](https://github.com/awslabs/aws-security-benchmark) | This project provides scripts and resources to assess AWS environments against security benchmarks,                                                                            |
| [S3Scan](https://github.com/bear/s3scan)                                    | S3Scan is specifically designed for scanning Amazon S3 (Simple Storage Service) buckets.                                                                                       |
| [CloudMapper](https://github.com/duo-labs/cloudmapper)                      | **Note** the Network Visualization functionality (command `prepare`) is no longer maintained.<br><br>CloudMapper helps you analyze your Amazon Web Services (AWS) environments |
| [PMapper](https://github.com/nccgroup/PMapper)                              | PMapper focuses on analyzing IAM (Identity and Access Management) policies in AWS.                                                                                             |
| [Prowler](https://github.com/toniblyx/prowler)                              | Prowler is a security assessment, auditing, hardening and forensics tool.                                                                                                      |

### incident response

## aws-organization

### organizational-unit
from [slaw](https://slaw.securosis.com/p/aws-lego-organizing-org)
At a minimum you want OUs for governance/security, infrastructure/operations, and workloads; as well as places to handle special cases such as incident response, mergers and acquisitions, and accounts you are deprovisioning.

Separation of functions. Always keep in mind _who_ needs to use the account and _what_ the account needs to do.

examples: [chris farris ou template terraform](https://github.com/primeharbor/org-kickstart/blob/main/ous.tf?utm_source=slaw.securosis.com&utm_medium=referral&utm_campaign=aws-lego-organizing-the-org) and [aws reference arch](https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/architecture.html?utm_source=slaw.securosis.com&utm_medium=referral&utm_campaign=aws-lego-organizing-the-org)


### accounts
from [slaw](https://slaw.securosis.com/p/aws-lego-organizing-org)
When deciding what accounts to create and where, try to use these guidelines:

- One of the best security tools in our hierarchy is Service Control Policies (SCPs). For example if you want different ones for development vs. production environments, you should probably have dev and prod OUs.
- If two things are in different security contexts — perhaps because one has a lot of compliance requirements, but the other is hosting pictures of your cats — move them into separate accounts.
- If application stacks are managed by different teams, put them in different accounts. This makes IAM easier because you can restrict access to the relevant teams.
- If two applications need to talk to each other and it’s a **lot** of data, then… wait until we get to that lab. This is the hardest decision point because, for cost reasons, you may need to put different workloads managed by different teams into the same account.
- Separate accounts are also awesome for assigning to different cost centers. **Much** easier than trying to use tags.

### organization-cloudtrail

why use organization-cloudtrail :
- Creates a trail in every account and region in the Org, all using the same configuration.
- Centralizes all the logs _across accounts and regions_ into one S3 bucket.
- Handles all the paths and naming for that bucket so you can figure out which account and region a log came from (it uses the account ID and region in the log path).
- Within an account you can’t disable or delete the organization trail, which really pisses off attackers.

## Best practice to allow 3P access to AWS account
[Best-practice-to-allow-3P to access AWS accounts](Best-practice-to-allow-3P%20to%20access%20AWS%20accounts)

## references-and-related
- [hackingthe.clud - AWS orgs](https://hackingthe.cloud/aws/general-knowledge/aws_organizations_defaults/)
- [cloud-security](cloud-security.md)
- [hackingthe.cloud - IAM identifiers](https://hackingthe.cloud/aws/general-knowledge/iam-key-identifiers/)
- [ghost of  cloudsec yet to come](https://www.chrisfarris.com/post/ghost-of-cloudsec-yet-to-come/)
- [marco lancini](https://blog.marcolancini.it/2018/blog-arsenal-cloud-native-security-tools/#aws)
- [slaw](https://slaw.securosis.com/p/give-account-security-blanket-scps)
- [iam references aws](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html)
- [aws-respond](aws-respond.md)

