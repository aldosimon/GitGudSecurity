---
title: AWS Detection Catalogue
tags:
  - cloudsec
  - TTP
created: 2025-01-30
---
# AWS Detection Catalogue
This page list common AWS API calls (cloudtrail) and tries to map (loosely) them to MITRE ATTACK framework. If you are hunting or handling incidents, these can be your starting points. 
These list are collected from various article listed below.


## Initial Access

### Key Cloudtrail

- `ConsoleLogin` - Interactive AWS console access
- `PasswordRecoveryRequested` - Password reset attempts
- `AssumeRoleWithWebIdentity` - Federated authentication abuse
- `GetSessionToken` - Temporary credential requests

### Detection Examples

```
SELECT eventTime, sourceIPAddress, userIdentity.userName, errorCode, errorMessage  
FROM cloudtrail_logs  
WHERE eventName = 'ConsoleLogin'  
AND errorCode IN ('SigninFailure', 'Failed authentication')  
AND eventTime > NOW() - INTERVAL '1 hour'  
GROUP BY sourceIPAddress  
HAVING COUNT(*) > 5;
```

## Execution

### Key Cloudtrail

- `StartInstance` / `StartInstances` - EC2 instance execution
- `Invoke` - Lambda function execution
- `SendCommand` - Systems Manager command execution
- `RunInstances` - New EC2 instance creation

### Detection Examples

```
-- Monitor SSM command execution  
SELECT eventTime, userIdentity.userName, requestParameters.instanceIds,   
       requestParameters.parameters.commands  
FROM cloudtrail_logs   
WHERE eventName = 'SendCommand'  
  AND requestParameters.parameters.commands LIKE '%curl%'  
  AND requestParameters.parameters.commands LIKE '%bash%';
```
## Enumeration/ Discovery

### Key Cloudtrail
- ec2:DescribeInstances
	- Describes the specified instances or all instances "What type of EC2 infrastructure is present in this account?" one of the way to do it: [account id from ec2](https://hackingthe.cloud/aws/enumeration/account_id_from_ec2/)
- s3:ListObjects - Ransomware in AWS S3 SSE-C#Explaining the S3 SSE-C Ransomware Approach
	- [ ] research and complete this block
- `ListSSHPublicKeys 
- `SimulatePrincipalPolicy`: The AWS Policy Simulator allows users to test an existing policy recorded in the policySourceArn field against a set of actions recorded in the actionNames field. This helps answer the question can I perform action X with policy Y.
- `ListUsers` - IAM user enumeration
- `ListRoles` - IAM role enumeration
- `ListIdentities` - SES identity listing
- `ListAccessKeys` - Access key discovery
- `ListServiceQuotas` - Service limit discovery
- `ListInstanceProfiles` - EC2 service role discovery
- `ListBuckets` - S3 bucket enumeration
- `ListGroups` - IAM group discovery
- `GetSendQuota` - SES sending limits
- `GetCallerIdentity` - Current identity verification
- `DescribeInstances` - EC2 instance discovery
- `GetBucketAcl` - S3 permissions discovery
- `GetBucketVersioning` - S3 versioning status
- `GetAccountAuthorizationDetails` - Comprehensive account enumeration
- From - Following attackers’ (Cloud)trail in AWS Methodology and findings in the wild   Datadog Security Labs

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

### Detection Examples

- from [Incident Response in AWS - Chris Farris](../Clippings/Incident%20Response%20in%20AWS%20-%20Chris%20Farris.md)
```
index=cloudtrail eventName=GetCallerIdentity OR ListBuckets OR DescribeInstances 
| iplocation sourceIPAddress 
| table userIdentity.arn, sourceIPAddress, City, Country 
| sort -City, Country
```

```
-- Identify reconnaissance activity  
WITH discovery_events AS (  
  SELECT eventTime, userIdentity.userName, sourceIPAddress, eventName  
  FROM cloudtrail_logs   
  WHERE eventName IN ('ListUsers', 'ListRoles', 'ListBuckets', 'DescribeInstances', 'GetCallerIdentity')  
    AND eventTime > NOW() - INTERVAL '1 hour'  
)  
SELECT userName, sourceIPAddress,   
       COUNT(DISTINCT eventName) as unique_discovery_actions,  
       COUNT(*) as total_discovery_calls  
FROM discovery_events  
GROUP BY userName, sourceIPAddress  
HAVING COUNT(DISTINCT eventName) > 5  
ORDER BY unique_discovery_actions DESC;
```
## Persistence

### Key Cloudtrail
- `CreateAccessKey` - New programmatic access credentials
- `CreateUser` - New IAM user creation
- `CreateNetworkAclEntry` - Network-level persistence
- `CreateRoute` - Routing table modifications
- `CreateLoginProfile` - Console access setup
- `AuthorizeSecurityGroupEgress` / `AuthorizeSecurityGroupIngress` - Firewall rule changes
- `CreateVirtualMFADevice` - MFA device setup
- `CreateConnection` - Database/service connections
+ **creation of a EC2 key pair**, is known technique - Following attackers’ (Cloud)trail in AWS Methodology and findings in the wild   Datadog Security Labs
+ **Creating root user** - Following attackers’ (Cloud)trail in AWS Methodology and findings in the wild   Datadog Security Labs
+ `create security group` - Following attackers’ (Cloud)trail in AWS Methodology and findings in the wild   Datadog Security Labs
+ `CreateUser` from - The curious case of DangerDev@protonmail.me
+ `CreateLoginProfile` call which is used to give a user the ability to login through the AWS management console from [The curious case of DangerDev@protonmail.me](../01-source/The%20curious%20case%20of%20DangerDev@protonmail.me.md)
+ Allow user to AssumeRole to a role that is privileged and preferably come from AWS default role. For example [cat flap](https://rootcat.de/blog/thecatflap) that allow user to AssumeRole for a `AWSControlTowerExecution` role.
### Detection Examples
from [Incident Response in AWS - Chris Farris](../Clippings/Incident%20Response%20in%20AWS%20-%20Chris%20Farris.md)

```
index=cloudtrail
eventName="CreateUser"
| iplocation sourceIPAddress
| search Country!="United States"
| table userIdentity.arn, sourceIPAddress,
  City, Country
```

```
-- Monitor user creation and access key generation  
SELECT eventTime, userIdentity.userName, requestParameters.userName as new_user,  
       responseElements.accessKey.accessKeyId  
FROM cloudtrail_logs   
WHERE eventName IN ('CreateUser', 'CreateAccessKey', 'CreateLoginProfile')  
  AND userIdentity.type = 'IAMUser'  
ORDER BY eventTime DESC;
```

## Privilege Escalation

### Key Cloudtrail

+ The `AdministratorAccess` policy was attached to the newly created account with `AttachUserPolicy` that provides full access to AWS services and resources. 
+ `AssumeRole` using external account from [The curious case of DangerDev@protonmail.me](../01-source/The%20curious%20case%20of%20DangerDev@protonmail.me.md). Attacker create malicious role, malicious role allows AssumeRole form external account.
- `CreateGroup` - Administrative group creation
- `CreateRole` - IAM role creation
- `UpdateAccessKey` - Access key status changes
- `PutGroupPolicy` - Inline group policy assignment
- `PutRolePolicy` - Inline role policy assignment
- `PutUserPolicy` - Inline user policy assignment
- `AttachUserPolicy` - Managed policy attachment to users
- `AttachRolePolicy` - Managed policy attachment to roles
- `AddRoleToInstanceProfile` - EC2 service role assignment
- `AddUserToGroup` - Group membership changes
- `PutAccountDetails`

	read more: [wiz](https://www.wiz.io/blog/wiz-discovers-cloud-email-abuse-campaign)
	
	Amazon Simple Email Service (SES) is AWS’s cloud-based bulk email platform. By default, accounts operate in a restricted [“sandbox” mode](https://docs.aws.amazon.com/ses/latest/dg/manage-sending-quotas.html), where emails can only be sent to verified addresses and volumes are capped to 200 messages per day, at a maximum rate of one message per second. An account can be moved into the unrestricted “production” mode, in which emails can be sent to arbitrary recipients and the quota is raised, typically to 50,000 emails per day. The transition requires submitting account details for AWS review through the `PutAccountDetails` API, and customers who need even higher volumes can request additional capacity through a support ticket.
	
	`CreateCase` was also used: 
	
	But the attacker wasn’t content with the default 50,000-emails-per-day quota. They tried to open a support ticket programmatically through the `CreateCase` API, asking AWS to further raise their limits _(ATT&CK:_ [_T1098_](https://attack.mitre.org/techniques/T1098/)_)_, an attempt that failed due to insufficient permissions

- Root access 
  
	read more: [Hands-On Security Tips For **Centralize Root Access** In AWS](https://medium.com/@oraspir/hands-on-security-tips-for-centralize-root-access-in-aws-assumeroot-5d315de423cd)

	1. Monitor for New Permissions: Keep an eye out for any addition of permissions like `sts:AssumeRoot` to identities in your organization account. Any identity with this permission will be capable of assuming root access, which can pose a significant security risk.
	2. Attack Scenario — Re-Enabling Root Login: Imagine a scenario where an attacker knows the root password of a member account, but root login for that account has been disabled. If the attacker has already compromised a principal in the organization’s AWS account that can assume root access (admittedly, an unusual situation), they might try to re-enable root login by using `AssumeRoot` with the `IAMCreateRootUserPassword` policy. They could then create a root login profile by assuming root privileges.
	 3. Watch for Suspicious Actions: A compromised identity with permission to assume root access could create significant risks for member accounts by executing suspicious actions, such as `aws iam create-login-profile`. To mitigate this risk, monitor any `AssumeRoot` actions with the `IAMCreateRootUserPassword` policy and closely watch for any root `CreateLoginProfile` actions within member accounts. Such actions are highly unusual and should always be investigated.
	4. Frequent `**AssumeRoot**` Logs: If you notice frequent `AssumeRoot` actions in your logs, **don't be alarmed right away**. These actions are likely generated by CSPM tools that are verifying root configuration across all member accounts.
- [ ] research more on these detection opportunity of root access

### Detection Examples

```
-- Monitor privilege escalation activities  
SELECT eventTime, userIdentity.userName, eventName,   
       requestParameters.groupName, requestParameters.roleName,  
       requestParameters.policyDocument, requestParameters.policyArn  
FROM cloudtrail_logs   
WHERE eventName IN ('PutUserPolicy', 'PutRolePolicy', 'PutGroupPolicy', 'AttachUserPolicy', 'AttachRolePolicy', 'AddUserToGroup')  
  AND (requestParameters.policyDocument LIKE '%"*"%'   
       OR requestParameters.policyArn LIKE '%AdministratorAccess%'  
       OR requestParameters.policyArn LIKE '%PowerUserAccess%'  
       OR requestParameters.groupName LIKE '%Admin%'  
       OR requestParameters.groupName LIKE '%Power%');
```
## Defense Evasion
### Key Cloudtrail
- `StopLogging` - CloudTrail logging disruption
- `DeleteTrail` - Audit trail removal
- `UpdateTrail` - Audit configuration changes
- `PutEventSelectors` - Logging scope modification
- `DeleteFlowLogs` - VPC flow log removal
- `DeleteDetector` - GuardDuty detector deletion
- `DeleteMembers` - Security service membership removal
- `DeleteSnapshot` - Evidence destruction
- `DeactivateMFADevice` - MFA bypass
- `DeleteCertificate` - SSL certificate removal
- Removing IAM users with `DeleteUser`
- Cleaning up policies with `DetachUserPolicy` and `DeleteUserPolicy`
- Deactivating long term access keys with `UpdateAccessKey`
- Cleaning up long term access keys with `DeleteAccessKey`
- Inspecting GuardDuty findings with `ListFindings` and `GetFindings` this is unique because Amazon Relational Database Service (RDS) console is used RDSDBinstance to access GuardDuty
- For GuardDuty ``DeleteDetector`` ``UpdateDetector`` ``CreateIPSet`` are nice events to look at. The first two have explicit names but the last one can be trickier and forgotten. This event allows monitoring when someone is updating the list of the GuardDuty trusted IP addresses. An attacker can use this to **whitelist its C2’s IP addresses** to avoid detection

### Detection Examples

```
-- Monitor security control tampering  
SELECT eventTime, userIdentity.userName, eventName, requestParameters.name,  
       requestParameters.detectorId, requestParameters.trailName  
FROM cloudtrail_logs   
WHERE eventName IN ('StopLogging', 'DeleteTrail', 'DeleteDetector', 'DeleteFlowLogs')  
  AND userIdentity.type != 'AWSService'  
ORDER BY eventTime DESC;
```

```
index=cloudtrail 
eventName=StopLogging OR DeleteTrail OR PutEventSelectors OR DeleteDetector
| iplocation sourceIPAddress 
| table userIdentity.arn, sourceIPAddress, City, Country
```


## Credential Access
### Key Cloudtrail
`GetSecretValue` - Secrets Manager access
`PutSecretValue` - Secrets modification
`CreateSecret` - New secret creation
`DeleteSecret` - Secret removal
`GetPasswordData` - EC2 Windows password retrieval
`RequestCertificate` - SSL certificate requests
`UpdateAssumeRolePolicy` - Trust relationship changes

### Detection Examples
```
-- Monitor credential access patterns  
SELECT eventTime, userIdentity.userName, requestParameters.secretId,  
       requestParameters.instanceId, sourceIPAddress  
FROM cloudtrail_logs   
WHERE eventName IN ('GetSecretValue', 'PutSecretValue', 'CreateSecret', 'DeleteSecret', 'GetPasswordData')  
  AND eventTime > NOW() - INTERVAL '24 hours'  
GROUP BY userIdentity.userName, sourceIPAddress  
HAVING COUNT(*) > 10;
```
## Impact

### Key Cloudtrail
- `s3:PutObject`
- `s3:GetObject`
- s3 cp / `CopyObject` 
  
  Creates a copy of an object that is already stored in Amazon S3. this is used in where ransomware operator encrypt S3 using own key (SSE-C). You can copy individual objects between general purpose buckets, between directory buckets, and between general purpose buckets and directory buckets. Both the Region that you want to copy the object from and the Region that you want to copy the object to must be enabled for your account.
  
```bash
--command used
aws s3 cp s3://<bucket_with_data>/<file_name> s3://<bucket_with_data>/<file_name> \
--sse-c AES256 \
--sse-c-key <customer_provided_key_here> 
```

- `PutBucketVersioning` - S3 versioning manipulation
- `DeleteObject` - S3 object deletion
- `RunInstances` - Resource creation for impact
- `DeleteAccountPublicAccessBlock` - S3 public access enabling
- `DeleteDBInstance` - Database deletion
- `ModifyDBInstance` - Database modification
- `s3:DeleteObjectVersion`
- `s3:PutLifecycleConfiguration`  ransomware operator used to auto delete after a certain time

### Detection Examples
```
-- Monitor high-impact activities  
SELECT eventTime, userIdentity.userName, eventName,  
       requestParameters.instanceType, requestParameters.minCount,  
       requestParameters.versioningConfiguration, requestParameters.bucketName,  
       requestParameters.dbInstanceIdentifier  
FROM cloudtrail_logs   
WHERE (eventName = 'RunInstances' AND requestParameters.instanceType IN ('c5.xlarge', 'c5.2xlarge', 'c5.4xlarge'))  
   OR (eventName = 'PutBucketVersioning' AND requestParameters.versioningConfiguration LIKE '%Suspended%')  
   OR eventName = 'DeleteAccountPublicAccessBlock'  
   OR eventName = 'DeleteObject'  
   OR eventName = 'DeleteDBInstance'  
ORDER BY eventTime DESC;
```
## Lateral Movement

### Key Cloudtrail
- sts:AssumeRole (Cloud to Cloud) -- Role assumption for lateral access
- ssm:StartSession (Cloud to Ground)
- ssm:SendCommand (Cloud to Ground)
- ec2-instance-connect:SendSSHPublicKey (Cloud to Ground)
- ec2:AuthorizeSecurityGroupIngress (Cloud to Ground, or Ground to Ground)
- VPC Flow Logs (Ground to Ground)
- `SwitchRole` - Console role switching
### Detection Examples

```
-- Monitor cross-account access  
SELECT eventTime, userIdentity.arn, requestParameters.roleArn,  
       requestParameters.roleSessionName, sourceIPAddress  
FROM cloudtrail_logs   
WHERE eventName = 'AssumeRole'  
  AND requestParameters.roleArn NOT LIKE '%:123456789012:%'  -- Your account ID  
ORDER BY eventTime DESC;
```

## Exfiltration
### Key Cloudtrail

- `GetObject` - S3 object retrieval
- `CopyObject` - S3 object copying
- `CreateSnapshot` - EBS snapshot creation
- `ModifySnapshotAttributes` - Snapshot sharing configuration
- `ModifyImageAttribute` - AMI sharing setup
- `SharedSnapshotCopyInitiated` - Cross-region snapshot copying
- `SharedSnapshotVolumeCreated` - Snapshot volume creation
- `ModifyDBSnapshotAttribute` - RDS snapshot sharing
- `CreateDBSnapshot` - Database snapshot creation
- `PutBucketPolicy` - S3 bucket policy modification
- `PutBucketAcl` - S3 bucket ACL changes

### Detection Examples

```
-- Monitor data exfiltration activities  
SELECT eventTime, userIdentity.userName, eventName,  
       requestParameters.snapshotId, requestParameters.imageId,  
       requestParameters.userIds, requestParameters.valuesToAdd,  
       requestParameters.bucketName, requestParameters.key  
FROM cloudtrail_logs   
WHERE (eventName IN ('ModifySnapshotAttribute', 'ModifyImageAttribute', 'ModifyDBSnapshotAttribute')  
       AND (requestParameters.userIds IS NOT NULL OR requestParameters.valuesToAdd IS NOT NULL))  
   OR (eventName = 'GetObject' AND sourceIPAddress NOT LIKE '10.%' AND sourceIPAddress NOT LIKE '172.%' AND sourceIPAddress NOT LIKE '192.168.%')  
ORDER BY eventTime DESC;
```
## Other

### Anomalies

#### Error message from anomaly IP

[Incident Response in AWS - Chris Farris](../Clippings/Incident%20Response%20in%20AWS%20-%20Chris%20Farris.md)

`index=cloudtrail errorMessage=* | iplocation sourceIPAddress | stats count by City, Country | sort -City, Country`

### Sensitive IAM actions

why worry on a whole lot api calls, these are the most sensitive according to 
[sensitive iam](https://www.chrisfarris.com/post/sensitive_iam_actions/) 
- CredentialExposure 
- DataAccess 
- PrivEsc 
- ResourceExposure

## References
1. [cloud-security](./cloud-security.md)
2. [sensitive iam](https://www.chrisfarris.com/post/sensitive_iam_actions/) 
3. [cloudtrail cheatsheet](https://aws.plainenglish.io/aws-cloudtrail-event-cheatsheet-a-detection-engineers-guide-to-critical-api-calls-part-1-04fb1588556f)
4. [The curious case of DangerDev@protonmail.me](https://www.invictus-ir.com/news/the-curious-case-of-dangerdev-protonmail-me)
5. [Following attackers’ (Cloud)trail in AWS Methodology and findings in the wild   Datadog Security Labs](https://securitylabs.datadoghq.com/articles/following-attackers-trail-in-aws-methodology-findings-in-the-wild/#most-common-enumeration-techniques)
6. [Hands-On Security Tips For **Centralize Root Access** In AWS](https://medium.com/@oraspir/hands-on-security-tips-for-centralize-root-access-in-aws-assumeroot-5d315de423cd)
7. [AWS Detection Engineering](https://blog.sekoia.io/aws-detection-engineering/)
8. [Chris Farris's AWS IR ](https://www.chrisfarris.com/post/aws-ir/)
9. [Ransomware in Amazon S3: SSE-C](https://www.fogsecurity.io/blog/ransomware-in-aws-s3-sse-c)