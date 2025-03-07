---
title: Cloud Detection Catalogue
tags:
  - cloudsec
  - TTP
created: 2025-01-30
---

## Enumeration Techniques 
### These enumeration from [[Following attackers’ (Cloud)trail in AWS Methodology and findings in the wild   Datadog Security Labs]]

| Enumeration API call              | Comment                                                                            | Attacker's question being answered                                         |
| --------------------------------- | ---------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| `sts:GetCallerIdentity`           | Returns the identity of the authenticated user                                     | "What are the credentials I compromised?"                                  |
| `ses:GetAccount`                  | Returns information about the SES account, including sending limits and past usage | "What's the volume of emails I can send through this account?"             |
| `ses:GetSendQuota`                | Returns SES sending limits                                                         | "What's the volume of emails I can send through this account?"             |
| `ses:ListIdentities`              | Lists verified SES senders                                                         | "Who can I impersonate?"                                                   |
| `sns:GetSMSAttributes`            | Returns SMS sending settings                                                       | "What's the SMS monthly spend limit?"                                      |
| `iam:ListUsers`                   | Returns IAM users in the account                                                   | "How many people are using this account?"                                  |
| `ec2:DescribeRegions`             | Describes the Regions that are enabled for your account, or all Regions.           | "What regions are enabled that infrastructure could be set up in?"         |
| `ec2:DescribeInstances`           | Describes the specified instances or all instances                                 | "What type of EC2 infrastructure is present in this account?"              |
| `ec2:DescribeVpcs`                | Describes one or more of your VPCs                                                 | "How is the network infrastructure set up in this account?"                |
| `lightsail:GetRegions`            | Returns a list of all valid regions for Amazon Lightsail                           | "What regions are enabled that infrastructure could be set up in?"         |
| `lightsail:GetInstances`          | Returns information about all Amazon Lightsail instances                           | "What type of Lightsail infrastructure is present in this account?"        |
| `route53:ListDomains`             | Returns domain names registered in the account                                     | "What's the name and domain names of the organization I have compromised?" |
| `route53:GetHostedZoneCount`      | Returns the number of hosted zones in the account                                  | "How large is this company?"                                               |
| `s3:ListBuckets`                  | Returns S3 buckets                                                                 | "Is there sensitive data available?"                                       |
| `servicequotas:ListServiceQuotas` | Returns service quotas in use for a specific service (e.g. EC2)                    | "How many resources can I spin up in that account?"                        |

ec2:DescribeInstances

Describes the specified instances or all instances
"What type of EC2 infrastructure is present in this account?"
one of the way to do it: [account id from ec2](https://hackingthe.cloud/aws/enumeration/account_id_from_ec2/)

s3:ListObjects 
[[Ransomware in AWS S3 SSE-C#Explaining the S3 SSE-C Ransomware Approach]]
- [ ] research and complete this block

### These enumeration from [[The curious case of DangerDev@protonmail.me]]
- ListBuckets 
- ListGroupsForUser 
- ListInstanceProfiles 
- ListSSHPublicKeys 
- SimulatePrincipalPolicy The AWS Policy Simulator allows users to test an existing policy recorded in the policySourceArn field against a set of actions recorded in the actionNames field. This helps answer the question can I perform action X with policy Y.

### Example detection with IP
from [[Incident Response in AWS - Chris Farris]]
```
index=cloudtrail eventName=GetCallerIdentity OR ListBuckets OR DescribeInstances 
| iplocation sourceIPAddress 
| table userIdentity.arn, sourceIPAddress, City, Country 
| sort -City, Country
```



-----

## persistence

+ **creation of a EC2 key pair**, is known technique [[Following attackers’ (Cloud)trail in AWS Methodology and findings in the wild   Datadog Security Labs]]
+ **Creating root user** [[Following attackers’ (Cloud)trail in AWS Methodology and findings in the wild   Datadog Security Labs]]
+ `create security group` [[Following attackers’ (Cloud)trail in AWS Methodology and findings in the wild   Datadog Security Labs]]
+ `CreateUser` from [[The curious case of DangerDev@protonmail.me]]
+ `CreateLoginProfile` call which is used to give a user the ability to login through the AWS management console from [[The curious case of DangerDev@protonmail.me]]
+ Allow user to AssumeRole to a role that is privileged and preferably come from AWS default role. For example [cat flap](https://rootcat.de/blog/thecatflap) that allow user to AssumeRole for a `AWSControlTowerExecution` role.
### example detection
from [[Incident Response in AWS - Chris Farris]]

```index=cloudtrail
eventName="CreateUser"
| iplocation sourceIPAddress
| search Country!="United States"
| table userIdentity.arn, sourceIPAddress,
  City, Country
```
----

## privilege-escalation

### These priv-esc from [[The curious case of DangerDev@protonmail.me]]: 
+ The AdministratorAccess policy was attached to the newly created account with AttachUserPolicy that provides full access to AWS services and resources. 
+ AssumeRole using external account from [[The curious case of DangerDev@protonmail.me]]. Attacker create malicious role, malicious role allows AssumeRole form external account.

### Root access [[Hands-On Security Tips For Centralize Root Access In AWS(AssumeRoot)#CDR Tips]]
1. **Monitor for New Permissions**: Keep an eye out for any addition of permissions like `sts:AssumeRoot` to identities in your organization account. Any identity with this permission will be capable of assuming root access, which can pose a significant security risk.
2. **Attack Scenario — Re-Enabling Root Login**: Imagine a scenario where an attacker knows the root password of a member account, but root login for that account has been disabled. If the attacker has already compromised a principal in the organization’s AWS account that can assume root access (admittedly, an unusual situation), they might try to re-enable root login by using `AssumeRoot` with the `IAMCreateRootUserPassword` policy. They could then create a root login profile by assuming root privileges.
3. **Watch for Suspicious Actions**: A compromised identity with permission to assume root access could create significant risks for member accounts by executing suspicious actions, such as `aws iam create-login-profile`. To mitigate this risk, monitor any `AssumeRoot` actions with the `IAMCreateRootUserPassword` policy and closely watch for any root `CreateLoginProfile` actions within member accounts. Such actions are highly unusual and should always be investigated.
4. **Frequent** `**AssumeRoot**` **Logs**: If you notice frequent `AssumeRoot` actions in your logs, **don't be alarmed right away**. These actions are likely generated by CSPM tools that are verifying root configuration across all member accounts.
- [ ] research more on these detection opportunity of root access
----
## defense-evasion techniques

### **from [[The curious case of DangerDev@protonmail.me]]**
- Removing IAM users with `DeleteUser`
- Cleaning up policies with `DetachUserPolicy` and `DeleteUserPolicy`
- Deactivating long term access keys with `UpdateAccessKey`
- Cleaning up long term access keys with `DeleteAccessKey`
- Inspecting GuardDuty findings with `ListFindings` and `GetFindings` this is unique because Amazon Relational Database Service (RDS) console is used RDSDBinstance to access GuardDuty. 
- [ ] research more on this
- [x] check why createinstances is defense evasion? ✅ 2025-01-31

**from [[AWS Detection Engineering - Sekoia.io Blog]]:**
- For GuardDuty ``DeleteDetector`` ``UpdateDetector`` ``CreateIPSet`` are nice events to look at. The first two have explicit names but the last one can be trickier and forgotten. This event allows monitoring when someone is updating the list of the GuardDuty trusted IP addresses. An attacker can use this to **whitelist its C2’s IP addresses** to avoid detection.

From [[Incident Response in AWS - Chris Farris]]
```
index=cloudtrail 
eventName=StopLogging OR DeleteTrail OR PutEventSelectors OR DeleteDetector
| iplocation sourceIPAddress 
| table userIdentity.arn, sourceIPAddress, City, Country
```


----

## Impact

`s3:PutObject`
`s3:GetObject`

### s3 cp / `CopyObject` : 
Creates a copy of an object that is already stored in Amazon S3.
this is used in where ransomware operator encrypt S3 using own key (SSE-C) [[Ransomware in AWS S3 SSE-C#Explaining the S3 SSE-C Ransomware Approach]]
You can copy individual objects between general purpose buckets, between directory buckets, and between general purpose buckets and directory buckets.
Both the Region that you want to copy the object from and the Region that you want to copy the object to must be enabled for your account.

AWS CLI used: [[Ransomware in AWS S3 SSE-C#Explaining the S3 SSE-C Ransomware Approach]]
```bash
aws s3 cp s3://<bucket_with_data>/<file_name> s3://<bucket_with_data>/<file_name> \
--sse-c AES256 \
--sse-c-key <customer_provided_key_here> 
```

**Permissions**
**General purpose bucket permissions**: read object with s3:GetObject and copy object with `s3:PutObject`
**Directory bucket permissions**: s3express: CreateSession to read/write
read more: https://docs.aws.amazon.com/AmazonS3/latest/API/API_CopyObject.html


`s3:PutLifecycleConfiguration`  ransomware operator used to auto delete after a certain time [[Ransomware in AWS S3 SSE-C#Observations and Questions on Ransomware in S3]]

`s3:DeleteObject`
`s3:DeleteObjectVersion`

## Lateral Movement

Examples from [[Incident Response in AWS - Chris Farris]]
- sts:AssumeRole (Cloud to Cloud)
- ssm:StartSession (Cloud to Ground)
- ssm:SendCommand (Cloud to Ground)
- ec2-instance-connect:SendSSHPublicKey (Cloud to Ground)
- ec2:AuthorizeSecurityGroupIngress (Cloud to Ground, or Ground to Ground)
- VPC Flow Logs (Ground to Ground)
### Anomalies

### Error message from anomaly IP

[[Incident Response in AWS - Chris Farris]]

`index=cloudtrail errorMessage=* | iplocation sourceIPAddress | stats count by City, Country | sort -City, Country`


## sensitive-iam-actions

why worry on a whole lot api calls, these are the most sensitive according to 
[sensitive iam](https://www.chrisfarris.com/post/sensitive_iam_actions/) 
- CredentialExposure 
- DataAccess 
- PrivEsc 
- ResourceExposure

## references-and-related
[cloud-security](./cloud-security.md)
[sensitive iam](https://www.chrisfarris.com/post/sensitive_iam_actions/) 
