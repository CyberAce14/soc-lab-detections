# SOC-LAB Detection Rules & SOAR Pipeline

**Author:** Oscar Njumbe | Security Operations Engineer | Reading, UK  
**Stack:** Microsoft Sentinel · Wazuh · n8n SOAR · Elastic Stack · Splunk  
**Environment:** Production homelab — Proxmox, 13-VLAN architecture, 30+ VMs

---

## Overview

Production detection rules, Wazuh configurations, and SOAR automation pipeline assets deployed in a live enterprise-grade SOC homelab. All rules run against real telemetry generated through attack simulation using Kali Linux, Metasploitable2, and Mythic C2 — validated before promotion to production.

---

## Repository Structure

```
soc-lab-detections/
├── kql-detection-rules/     # Microsoft Sentinel KQL analytics rules (7 live)
├── wazuh-detection-rules/   # Custom Wazuh XML rules (4 deployed)
├── soar-pipeline/           # n8n SOAR pipeline — triggered by Elastic (ELK) SIEM (134+ alerts processed)
└── architecture/            # SOC-LAB environment overview
```

---

## Microsoft Sentinel — KQL Detection Rules

7 analytics rules live in Sentinel, mapped to MITRE ATT&CK:

| Rule File | Description | MITRE TTP |
|---|---|---|
| `T1110-brute-force-detection.kql` | Multiple failed sign-ins from same IP | T1110 |
| `T1059.001-powershell-abuse.kql` | Encoded commands, download cradles, bypass flags | T1059.001 |
| `T1078-new-ip-login-detection.kql` | Login from IP not seen in prior 14 days | T1078 |
| `T1543.003-new-suspicious-service.kql` | New service with suspicious binary path | T1543.003 |
| `T1021.002-lateral-movement-smb.kql` | SMB network logons to multiple targets | T1021.002 |
| `T1003.001-lsass-credential-access.kql` | LSASS memory access patterns | T1003.001 |
| `T1078.004-impossible-travel.kql` | Sign-ins from geographically impossible locations | T1078.004 |

All rules include severity tiers, MITRE mapping comments, and are tuned against 14+ days of historical logs to eliminate false positives before going live.

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

- **134+ alerts processed** with zero manual handling
- AI triage: MITRE ATT&CK mapping + severity scoring per alert
- Dedup logic + false-positive filtering built in
- Detection-to-case time: < 60 seconds

See `soar-pipeline/README.md` for architecture detail.

---

## Environment

| Layer | Technology |
|---|---|
| Hypervisor | Proxmox VE |
| Network Segmentation | pfSense — 13 VLAN zones |
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
