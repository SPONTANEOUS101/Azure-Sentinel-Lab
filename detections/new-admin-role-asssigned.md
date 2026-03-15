# Detection: New Admin Role Assigned

## Purpose
Detects when a user is added to a privileged directory role.

## Data Source
AuditLogs (Entra ID)

## KQL Query
AuditLogs
| where ActivityDisplayName == "Add member to role"
| extend Role = tostring(TargetResources[0].displayName)
| extend AddedUser = tostring(TargetResources[1].userPrincipalName)
| project TimeGenerated, Role, AddedUser, InitiatedBy = tostring(InitiatedBy.user.userPrincipalName)

## Rule Settings
- Schedule: Every 5 minutes
- Lookback: 5 minutes
- Severity: High
- Tactics: Privilege Escalation, Persistence
- Incident grouping: AddedUser + Role
