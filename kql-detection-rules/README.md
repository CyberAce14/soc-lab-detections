# KQL detection rules

## `live/` - the 7 analytics rules enabled in the lab's Microsoft Sentinel workspace
Windows Security/System events arrive through Azure Monitor Agent into the `Event` table, so each rule parses the raw `EventData` XML with `extract()`.

| Rule file | Severity | MITRE |
|---|---|---|
| `T1110-failed-logon-spike.kql` | Medium | T1110 |
| `T1110-successful-logon-after-failures.kql` | High | T1110 |
| `T1059.001-suspicious-powershell.kql` | High | T1059.001 |
| `T1136-new-local-account.kql` | High | T1136 |
| `T1078-user-added-to-privileged-group.kql` | High | T1078 |
| `T1059-shell-spawned-by-system.kql` | High | T1059 |
| `T1543.003-new-service-installed.kql` | Medium | T1543.003 |

## `reference/` - additional rule designs (not deployed)
Written for the `SecurityEvent` and `SigninLogs` tables (Sysmon-in-SecurityEvent, Entra sign-in logs). The lab does not currently ingest those tables, so these have **not** been run against lab data. Treat them as starting points.
