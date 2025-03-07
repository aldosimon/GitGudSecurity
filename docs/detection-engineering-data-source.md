---
title: Detection Engineering Data Source
created: 2025-02-10
description: 
tags: 
source:
---
# Related notes: 
[[detection-engineering]]
[practical threat detection engineering](https://ieeexplore.ieee.org/document/10251369):

# Summary
This note list data sources for SIEM/ SOAR

# Types of assets
## Windows
Common windows list of data sources:
- Application log
- Security log
- System log
- Powershell log
- Sysmon log

----

PowerShell event logs only include full command lines if PowerShell Script Block Logging is enabled. It is disabled by default and must be enabled via a Group Policy Object or the Registry. 

-----

## Linux
Linux logs commonly used:
- /var/log/syslog or / var/log/messages: system activty data
- /var/log/auth.log or / var/log/secure: security log
- /var/log/cron: scheduled task
- /var/log/faillog: failed logins
- /var/log/audit/audit.log: auditd rules based log

## Network
- Packet captures: expensive, low priority. e.g. using packetbeat
- Network devices: logs from network devices, commonly used syslog to ship. e.g. log are flow data; another one is alerts from network security appliances

## Application
Application logs may vary by name, but commonly app logs have these function:
- Access, authentication and authorization logs: who logged in/out and when, whether it was a successful login attempt, and what operations were performed
- Change log: e.g. config change, permission change
- Error log
- Availability log

## Cloud
read more on AWS: [[aws-detect-and-response]]

## Security tooling

- Endpoint solution (EDR, endpoint protection) logs: alert log based off rules
- Network protection (Zeek, suricata, see also [[detection-engineering-data-source#Network]]): alert based off rules. these appliances also have detection development capability, thus detection engineer usually can also tinker with the rule that fire alert

# Data sources challenges

### Completeness
We do not want to waste storage resources and bandwidth on data sources that won’t add value to our investigation due to the data they expose
For example, if a system provides logs showing a network connection was established but there are no details on the source/destination of the connection with contextless timestamps and ambiguous time zones, there is likely not much that can be used from that to develop a quality detection.

### Quality
data should be reliable and relevant. another aspect is the format of the data.

### Timeliness
this covers delays of sending data, if the data is pre-process that cause delay?

### Coverage
data should provide cover for what detection we aim to build

### Understanding your data sources
Some points to help understand your sources:
- Does the data source cover our detection needs? we should already have the technical needs for our detection when we ask these questions. Read more on [[detection-engineering#Investigate]]
- What method is required to retrieve logs from the data source? API, Syslog, or something else? 
- What format does the data source provide logs in? 
- What time zone is the data source using for their timestamps?
- Who is the point of contact for the data source that we need logs from? 
- What are the retention policies of the data source? 
- Does the detection engineering team have access to retrieve logs from the source? 
- What is the delay between an event occurring and it being received in the detection?
- Can the logging configuration be modified to improve the data being received?

---
note about retention:  sometimes the data source owner fail to understand that the goal of detection is not storing log, and this blur the line between their log retention policy with detection needs retention policy. be concise and clear to differentiate this when communicating data needs.

----
