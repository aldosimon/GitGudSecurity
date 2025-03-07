---
title: Application Security
created: 
description: 
tags: 
source:
---
# about
### AppSec program [BSidesSF 2020 - How to 10X Your Company’s Security - clint gibbler](https://www.youtube.com/@BSidesSF)

## principals
- automate as much as possible
- guardrails not gatekeeping
- high signal low noise tools and alerting: okay not to catch everything, but when it catch something, it is impactful
- developers are security customer: 
	- make secure with less friction
- self service security
	- tools and service that user can use with less security team interaction


## app-security-program
### fundamentals (short-term)
Vulnerability Management
Continuous Scanning
Asset Inventory: 
	- code, cloud (compute, services, secrets, api, roles, network acl), database, network endpoint, endpoint
	- which RDS without encryption, which compute endpoint is exposed to internet
	

### scaling (medium-term)
threat modeling: focus effort on risk, determine risk with developer help
	- self service security questioner for developer
	- light weight threat model inside SDLC
security engineering: build framework, library, templates that are secure by default
detection and response:
	- when fishy events occurs, send alert to dev. escalate if user did not initiate action.
	- another POV is automating SOC runbook

### long-term
automate least privilege
	- [policy_sentry](https://github.com/salesforce/policy_sentry), repokid
targeting vuln class
	 -time audit, use result to focus on certain vuln class by secure default
enforce invariants: enforce/ alert on things that should **always** or **never** be true. idea is if there is no need for context (these things should always or never), no need for sec team time
	 - ec2 should **never** open all port to public -> enforce with lambda
	 - manual changes should **never** be made through aws console -> detect with cloudtrail
	 - never spin up instance in x regions -> detect/ enforce
## reference-and-related
[[infosec-compendiums|infosec-compendiums]]
