# ELK Detection Rules — Elastic SIEM

Custom detection rules deployed in Elastic Security (Kibana), written in **EQL (Event Query Language)** and **KQL (Kibana Query Language)**. These rules run against log sources ingested into the Elastic Stack and generate alerts that trigger the n8n SOAR pipeline via webhook.

---

## Rules Overview

| Rule File | Description | Language | MITRE TTP |
|---|---|---|---|
| `T1110-brute-force-ssh.eql` | SSH brute force sequence detection | EQL | T1110.001 |
| `T1059.001-powershell-suspicious-exec.eql` | Suspicious PowerShell execution | EQL | T1059.001 |
| `T1071-c2-beacon-detection.eql` | Periodic C2 beacon pattern via Zeek | EQL | T1071.001 |
| `T1055-process-injection.eql` | Process injection via unusual parent-child | EQL | T1055 |

---

## Platform

- **Elastic Stack** — log ingestion via Filebeat, Winlogbeat, Zeek integration
- **Kibana Security** — rule management, alert generation
- **Alert output** — HTTP POST webhook → n8n SOAR pipeline

---

## Deploying Rules

Export/import via Kibana:
1. Kibana → Security → Rules → Detection Rules
2. **Import:** Upload `.ndjson` rule export file
3. **Export:** Select rules → Export → saves as `.ndjson`
