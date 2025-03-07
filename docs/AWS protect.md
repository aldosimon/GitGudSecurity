---
title: AWS protect
created: 2025-01-31
tags:
  - protect
  - AWS
description: protect AWS asset using these methods from various source
---

# AWS Protect

## S3 Bucket

using policy to protect from. or scope down permission of IAM role from these action. 
read more on [org. policies](#organizational-policies)

- `s3:DeleteObject`
- `s3:DeleteObjectVersion`
- `s3:PutLifecycleConfiguration`
- `s3:PutObject`
- `s3:GetObject`
- `s3:PutLifecycleConfiguration`
- `s3:ListObjects`
- `s3:ListBuckets` (theorized)

reference: [fogsecurity.io](https://www.fogsecurity.io/blog/ransomware-in-aws-s3-sse-c)

### Bucket Policies

- protect from SSE-C ([fogsecurity.io](https://www.fogsecurity.io/blog/ransomware-in-aws-s3-sse-c))
	- deny object uploads with SSE-C encryption
	- denies non-KMS (AWS Managed Key or CMK)
	+ requires a specific CMK

### AWS Block Public Access for S3

[read more](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html)

## IAM

### Reducing usage of static creds, long-term credentials/ tokens

Access Keys in AWS are typically tied to long-term credentials such as IAM users. In AWS, security best practices recommend moving away from long-term credentials to short term credentials. Instead of IAM Users, IAM roles are preferred.-- [fogsecurity.io](https://www.fogsecurity.io/blog/ransomware-in-aws-s3-sse-c)

For more details about temporary credentials in AWS, [AWS has extensive documentation and AWS STS](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html).

### Rotate long-term keys to minimize exposure

### Scan code for stored credentials in CI/CD pipelines and repositories

### Require MFA for human access
Use MFA if you think the identity is important 

### Migrate administrators and highly-privileged access to Just in Time access and/or use hardware tokens

### Permission Reduction and Least Privilege
[**Least Privilege**](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege)
have one role per resources or per app, follow the least privilege principle

### AWS centralized root management
with AWS Organizations, **each AWS account** in your organization comes with its own root user. These root users have unrestricted administrative access to everything in their AWS accounts. controls for this account before centralized root management is typical e.g. MFA, password rotate, limit permission via SCP [Or Aspir](https://medium.com/@oraspir/hands-on-security-tips-for-centralize-root-access-in-aws-assumeroot-5d315de423cd)

Centralized root access: [Or Aspir](https://medium.com/@oraspir/hands-on-security-tips-for-centralize-root-access-in-aws-assumeroot-5d315de423cd)
- centralize their management
- delete root user credentials
- reduced attack surface (with fewer root to manage)
- new account won't have root credentials by default
- With managed policies, such as _`IAMAuditRootUserCredentials`_, _`IAMCreateRootUserPassword`_, _`IAMDeleteRootUserCredentials`_, `S3UnlockBucketPolicy`, and `SQSUnlockQueuePolicy`, you are strictly limited to using only those policies.
- If root access is needed, it can be temporarily recovered, used for the specific task, and then removed again.

Assuming root would look like this
`aws sts assume-root --target-principal=123456789 --task-policy-arn arn=arn:aws:iam::aws/root-task/S3UnlockBucketPolicy`

CloudTrail would show `AssumeRoot` event type see here for example [Or Aspir](https://medium.com/@oraspir/hands-on-security-tips-for-centralize-root-access-in-aws-assumeroot-5d315de423cd)

#### related API with centralized root management
These are related API with centralized root management that might be of interest when you design detection/ protect [Or Aspir](https://medium.com/@oraspir/hands-on-security-tips-for-centralize-root-access-in-aws-assumeroot-5d315de423cd)
- check if feature is enabled. `ListOrganizationsFeatures`
- Identify Accounts with Root Access. No simple API, but use these steps:
	- assume root with `IAMAuditRootUserCredentials ` 
		`aws sts assume-root --target-principal=123456789 --task-policy-arn arn=arn:aws:iam::aws/root-task/IAMAuditRootUserCredentials` 
	- `GetLoginProfile` API command without providing the `UserName`
		`aws iam get-login-profile`
	- If the root account doesn’t have a login profile, you should get the following error: 
		_An error occurred (NoSuchEntity) when calling the GetLoginProfile operation: Login Profile for User null cannot be found._
	- Also check for MFA and signed certs -> delete this also

#### related detection and response: [[AWS detect and response - stub]]

### credential report
https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_getting-report.html

```awscli
aws iam generate-credential-report --profile <aws-cli-profile>
```

### IAM tools
 - [iamlive](https://github.com/iann0036/iamlive) 
 - [Policy Sentry](https://policy-sentry.readthedocs.io/en/stable/)
 - [IAM Zero](https://iamzero.dev/)

## Network

### Monitor network perimeter

### Know your exposed asset

### put guardrails when creating new VPC

### auto-remediation of VPC if guardrail is impossible

## Organizational policies

### Service Control Policies (SCPs)
the OG of Security Invariants- define the maximum permissions of identities in your organization.
With **service control policies**, it is possible to review CloudTrail to determine who may be calling the specific actions that will be denied. [PrimeHarbor](https://www.primeharbor.com/blog/security-invariants/#:~:text=A%20security%20invariant%20is%20a,for%20your%20business%20and%20applications.)

### Resource Control Policies (RCPs)

a new type of Organization Policy that defines the maximum permissions of *resources* in your organization. 
Resource Control Policies apply to the resources in your organization and can control any principal, even those outside of your control. [Only a few services support RCPs](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_rcps.html#rcp-supported-services) at this time: S3, KMS, SQS, SecretsManager, and IAM Roles (via STS).
[- PrimeHarbor](https://www.primeharbor.com/blog/security-invariants/#:~:text=A%20security%20invariant%20is%20a,for%20your%20business%20and%20applications.)

**Resource Control Policies** are more complicated to preview because you’ll need to enable very expensive **CloudTrail DataEvents** and review the vast data produced. For some RCPs, you can probably use: [IAM Access Analyser](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html#what-is-access-analyzer-resource-identification)
[- PrimeHarbor](https://www.primeharbor.com/blog/security-invariants/#:~:text=A%20security%20invariant%20is%20a,for%20your%20business%20and%20applications.)

#### RCP samples
[samples of RCP](https://github.com/aws-samples/resource-control-policy-examples)

You should not attach RCPs without thoroughly testing the impact that the policy has on resources in your accounts. Once you have a policy ready that you would like to implement, we recommend testing in a separate organization or OU that can represent your production environment. Once tested, you should deploy changes to test OUs and then progressively deploy the changes to a broader set of OUs over time.



### Declarative Policies (DPs)
These policies exist outside of IAM evaluation and enforce **specific controls** at the AWS Service.
You can determine the impactof DP with a simple CSPM review for public images and snapshots. For IMDSv2 enforcement, you can review the CloudWatch Metric `MetadataNoToken` to see how many API calls still use the old system. 
[PrimeHarbor](https://www.primeharbor.com/blog/security-invariants/#:~:text=A%20security%20invariant%20is%20a,for%20your%20business%20and%20applications.)
[read more on AWS](https://aws.amazon.com/blogs/security/get-the-full-benefits-of-imdsv2-and-disable-imdsv1-across-your-aws-infrastructure/)


### Permissions Boundaries
These **IAM Policies** don’t grant permissions but rather define the maximum permissions of the principal (**IAM User or Role**) to which they are attached.


### examples of implementing OP

#### prime harbor security invariants
- [PrimeHarbor](https://www.primeharbor.com/blog/security-invariants/#:~:text=A%20security%20invariant%20is%20a,for%20your%20business%20and%20applications.)

#### collection of AWS OP
- [prime harbor AWS OP github](https://github.com/primeharbor/aws-organizational-policies)
- [infralicious](https://github.com/infralicious/awesome-service-control-policies)

#### Lesson learned from reports 
- protect from SSE-C ([fogsecurity.io](https://www.fogsecurity.io/blog/ransomware-in-aws-s3-sse-c))
	- deny object uploads with SSE-C encryption 
	- Requires KMS Encryption (which can be either an AWS Managed Key or Customer Managed Key)

### Avoid the use of AWS managed policies 
[datadog](https://securitylabs.datadoghq.com/articles/securely-integrating-with-customers-aws-accounts/#minimize-and-document-permissions-that-your-integration-requires)
While policies like `ReadOnlyAccess` might seem like an easy way to grant limited permissions, AWS managed policies are typically over-privileged and allow overly-sensitive, unnecessary actions. For example:

- [`ReadOnlyAccess`](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/ReadOnlyAccess.html) allows full read access to all data in S3 buckets and DynamoDB tables in the account, as well as read access to all SSM parameters, which typically (and rightly) contain secrets.
- [`SecurityAudit`](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/SecurityAudit.html) contains the `ec2:DescribeInstanceAttribute` permission, which grants privileges to retrieve user data configuration from all EC2 instances, often containing sensitive data or configuration secrets. It also grants `lambda:ListFunctions`, which permits retrieval of the value of all environment variables for all Lambda functions in the account.

In addition, AWS can add or remove permissions from these policies at any time without notice, and they do so on a [daily basis](https://github.com/zoph-io/MAMIP/commits/master/), granting you no control over its lifecycle.

## Backup policy
AWS backup encryption
https://docs.aws.amazon.com/aws-backup/latest/devguide/encryption.html



## Best practice to allow 3P access to AWS account
[[stub]]

## Configuration Management
[[stub]]

#### scan with AWS config



## AWS Security Hub
AWS brand of CSPM



## AWS Risk Management

### Enable Amazon Detective
