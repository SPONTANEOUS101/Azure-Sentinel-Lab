# Azure Sentinel Architecture

## Overview

This document walks through the architecture of my Azure Sentinel lab — how I set it up, what data I connected, how detections work, and how incidents flow through the system.  
The goal was to build a simple but realistic SOC pipeline focused on identity security using Entra ID logs.

This setup helped me understand how Sentinel ties everything together: data ingestion, analytics rules, incidents, and automation.

# 1. High-Level Architecture

At a high level, the environment works like this:

Entra ID → Log Analytics Workspace → Microsoft Sentinel → Incidents → Automation → Response

Sentinel sits on top of a Log Analytics Workspace and uses the data stored there to detect suspicious activity. Once something is detected, Sentinel creates an incident, and automation can take over from there.

# 2. Core Components

## Log Analytics Workspace

This is the backbone of the entire setup. Every log I collect ends up here.

The workspace is responsible for:

- Storing all incoming logs  
- Running KQL queries  
- Powering Sentinel’s analytics rules  
- Acting as the central data lake for the SOC  

# 3. Data Ingestion

## Connected Logs
I connected two key Entra ID log types:

### AuditLogs
These logs track changes inside the directory, such as:

- Admin role assignments  
- User creation  
- Group membership changes  
- Privileged operations  

These are essential for detecting privilege escalation.

### SignInLogs
These logs capture authentication activity:

- Successful and failed sign-ins  
- MFA status  
- IP address and location  
- Device details  

## How the Logs Flow

Here’s the path the logs take:

Entra ID  
   ↓  
Diagnostic Settings  
   ↓  
Log Analytics Workspace  
   ↓  
Microsoft Sentinel

Once the logs were connected, I verified ingestion with simple KQL queries like:

AuditLogs | take 10  
SigninLogs | take 10

Seeing data appear confirmed everything was wired correctly.

# 4. Detection Architecture

I built two main detections to start with — both focused on identity security.

## Detection 1: Multiple Failed Sign-ins
- Source: SignInLogs  
- Purpose: Catch password spray attempts  
- MITRE tactic: Credential Access  

## Detection 2: New Admin Role Assigned
- Source: AuditLogs  
- Purpose: Detect privilege escalation  
- MITRE tactics: Privilege Escalation, Persistence  

Both detections use KQL queries that run every few minutes.

## How Detections Work

The flow looks like this:

KQL Query  
   ↓  
Analytics Rule  
   ↓  
Alert  
   ↓  
Incident

I also added incident grouping so related alerts don’t create noise.  
For example, grouping by user or IP helps keep incidents clean and easy to investigate.

# 5. Incident Management
When a detection fires, Sentinel creates an alert, and alerts are grouped into an incident.
Inside the incident, I can see:

- The user involved  
- The IP address  
- The timeline of events  
- Any related alerts  
- Evidence collected by Sentinel  

This gives a clear picture of what happened and how serious it is.

### Incident Lifecycle

Alert → Incident → Investigation → Response → Closure

# 6. Automation & Response
To make the system more proactive, I added automation.

## Automation Rule
- Trigger: When an alert is created  
- Condition: Analytics rule name contains “New Admin Role Assigned”  
- Action: Run a playbook  

## Playbook (Logic App)

The playbook does one simple thing:
- Sends me an email whenever someone is added to an admin role  

The email includes:
- The role assigned  
- The user who received the role  
- Who performed the action  
- The timestamp  

## Automation Flow

Incident Created  
   ↓  
Automation Rule  
   ↓  
Playbook  
   ↓  
Email Notification

# 7. Testing the Architecture

To make sure everything worked end-to-end, I tested:
- KQL queries  
- Analytics rules  
- Incident creation  
- Automation rules  
- Playbook execution  

After assigning a test admin role, I received the email alert — confirming the entire pipeline works.

# 8. Architecture Diagram (Text Version)

                +---------------------------+
                |     Entra ID (Azure AD)   |
                |  - AuditLogs              |
                |  - SignInLogs             |
                +-------------+-------------+
                              |
                              v
                +---------------------------+
                | Log Analytics Workspace   |
                |  (Stores all logs)        |
                +-------------+-------------+
                              |
                              v
                +---------------------------+
                |     Microsoft Sentinel    |
                |  - Analytics Rules        |
                |  - Incidents              |
                |  - Investigation          |
                +-------------+-------------+
                              |
                              v
                +---------------------------+
                |   Automation Rules        |
                |  (Trigger playbooks)      |
                +-------------+-------------+
                              |
                              v
                +---------------------------+
                |       Playbooks           |
                |  (Email, Teams)   |
                +---------------------------+


# 9. Summary

This architecture gives me a complete, working Sentinel environment that covers:

- Log ingestion  
- Detection engineering  
- Incident creation  
- Automation  
- Response  
