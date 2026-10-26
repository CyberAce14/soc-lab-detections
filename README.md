# SOC-LAB Detection Rules & SOAR Pipeline

**Author:** Oscar Njumbe | Security Operations Engineer | Reading, UK  
**Stack:** Microsoft Sentinel · Wazuh · n8n SOAR · Elastic Stack · Splunk  
**Environment:** Production-style homelab — Proxmox, 10 VLAN-segmented zones (pfSense), 30 VMs

---

## Overview

Detection rules, Wazuh configurations, and SOAR automation pipeline assets from a production-style SOC homelab. Rules are exercised with attack simulation using Kali Linux, Metasploitable2, and Mythic C2.

---

## Repository Structure

```
soc-lab-detections/
├── kql-detection-rules/     # Microsoft Sentinel KQL rules: live/ (7 enabled), reference/ (designs, not deployed)
├── elk-detection-rules/     # Elastic SIEM EQL detection rules (4 deployed — SOAR webhook source)
├── wazuh-detection-rules/   # Custom Wazuh XML rules (4 deployed)
├── soar-pipeline/           # n8n SOAR pipeline — triggered by Elastic (ELK) SIEM (100+ alerts processed)
└── architecture/            # SOC-LAB environment overview
```

---

## Microsoft Sentinel — KQL Detection Rules

7 scheduled analytics rules are enabled in the lab's Sentinel workspace (`kql-detection-rules/live/`), mapped to MITRE ATT&CK. Windows events arrive via Azure Monitor Agent into the `Event` table, so each rule parses the raw `EventData` XML.

| Rule File | Description | Severity | MITRE TTP |
|---|---|---|---|
| `T1110-failed-logon-spike.kql` | 5+ failed logons per account/IP in 1h | Medium | T1110 |
| `T1110-successful-logon-after-failures.kql` | Successful logon after 3+ failures | High | T1110 |
| `T1059.001-suspicious-powershell.kql` | Encoded commands, download cradles, bypass flags | High | T1059.001 |
| `T1136-new-local-account.kql` | New user account created | High | T1136 |
| `T1078-user-added-to-privileged-group.kql` | Account added to a privileged group | High | T1078 |
| `T1059-shell-spawned-by-system.kql` | cmd/PowerShell/script host spawned by SYSTEM | High | T1059 |
| `T1543.003-new-service-installed.kql` | New Windows service installed | Medium | T1543.003 |

`kql-detection-rules/reference/` holds further rule designs (SMB lateral movement, LSASS access, new-IP login, impossible travel) written for `SecurityEvent` and `SigninLogs`. They are not deployed in the lab and have not been run against lab data.

---

## Elastic SIEM — EQL Detection Rules

4 custom EQL rules deployed in Kibana Security, generating alerts that trigger the n8n SOAR pipeline via webhook:

| Rule File | Description | MITRE TTP |
|---|---|---|
| `T1110-brute-force-ssh.eql` | SSH brute force sequence — 5 failures then success | T1110.001 |
| `T1059.001-powershell-suspicious-exec.eql` | Suspicious PowerShell flags and download cradles | T1059.001 |
| `T1071-c2-beacon-detection.eql` | Periodic outbound C2 beacon pattern via Zeek | T1071.001 |
| `T1055-process-injection.eql` | Unusual parent-child process relationships | T1055 |

---

## Wazuh — Custom Detection Rules

4 custom rules extending the Wazuh default ruleset, deployed to the Wazuh manager:

| Rule ID | Description | MITRE TTP | Level |
|---|---|---|---|
| 100001 | SSH brute force — 8 failures in 120s | T1110.001 | 10 |
| 100002 | Unauthorised sudo escalation attempt | T1548.003 | 12 |
| 100003 | Executable script staged in /tmp | T1059 / T1105 | 8 |
| 100004 | New local user account created | T1136.001 | 10 |

---

## SOAR Pipeline — n8n Automation

End-to-end alert automation pipeline built on n8n v2.23.3:

```
Elastic SIEM Alert → n8n Webhook → FP Filter → Dedup Check
    → GPT-4.1 Mini Triage (+ AbuseIPDB Tool) → Severity Mapper
    → Slack Notification → IRIS Case Auto-Creation
```

- **100+ alerts processed** automatically; analysts review and escalate in IRIS
- AI triage: MITRE ATT&CK mapping + severity scoring per alert
- Dedup logic + false-positive filtering built in

See `soar-pipeline/README.md` for architecture detail.

---

## Environment

| Layer | Technology |
|---|---|
| Hypervisor | Proxmox VE |
| Network Segmentation | pfSense — 10 VLAN-segmented zones |
| Cloud SIEM | Microsoft Sentinel + Azure Arc + AMA/DCR pipelines |
| On-Prem SIEM | Elastic Stack · Splunk v9.1.1 · Wazuh |
| NDR | Zeek + Security Onion |
| EDR | Windows Defender + Sysmon |
| Identity Detection | Microsoft Defender for Identity (MDI v3.0.6.405) |
| SOAR | n8n v2.23.3 + GPT-4.1 Mini |
| Red Team | Kali Linux · Metasploitable2 · Mythic C2 |
| Malware Analysis | REMnux |
| Vulnerability Management | OpenVAS / Nessus |
| Active Directory | Windows Server 2022 DC + BloodHound CE (Docker) |

---

## Contact

**LinkedIn:** [linkedin.com/in/oscar-njumbe-36106139](https://linkedin.com/in/oscar-njumbe-36106139)  
**Email:** oscar.njumbe01@gmail.com  
**Certifications:** CompTIA Security+ (SY0-701, 2025)
