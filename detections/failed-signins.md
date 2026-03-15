# Detection: Multiple Failed Sign-ins (Password Spray Indicator)

## Purpose
Detects repeated failed sign-in attempts from the same IP within 5 minutes.

## Data Source
SignInLogs

## KQL Query
SigninLogs
| where ResultType != 0
| summarize FailedCount = count() by IPAddress, UserPrincipalName, bin(TimeGenerated, 5m)
| where FailedCount >= 5

## Rule Settings
- Schedule: Every 5 minutes
- Lookback: 5 minutes
- Severity: Medium
- Tactics: Credential Access
- Incident grouping: IPAddress + UserPrincipalName
