# SOC-LAB Architecture Overview

**Environment:** Production-grade enterprise SOC homelab  
**Hypervisor:** Proxmox VE | **VMs:** 30+ | **Network Zones:** 13 VLAN-segmented via pfSense

---

## Network Architecture

```
ISP Router (192.168.1.1)
        │
  Proxmox Host (hv-pmx-01)
        │
   pfSense VM ──── WAN: 192.168.1.96
        │
  ┌─────┴──────────────────────────────────────────┐
  │                 VLAN Segments                   │
  ├──── INFRA_CORE      192.168.10.0/24  (DC, DNS) │
  ├──── NET_SECURITY    192.168.20.0/24  (IDS/NDR)  │
  ├──── ANALYST         192.168.30.0/24  (SOC WS)   │
  ├──── SOC_SIEM        192.168.40.0/24  (SIEM VMs) │
  ├──── MGMT            192.168.50.0/24  (Mgmt)     │
  ├──── VULN_MALWARE    192.168.60.0/24  (OpenVAS)  │
  ├──── ATTACK_SIM      192.168.70.0/24  (Red Team)  │
  ├──── SANDBOX         192.168.80.0/24  (Malware)  │
  ├──── DMZ_LAB         192.168.90.0/24  (DMZ)      │
  └──── GNS3_TRANSIT    192.168.254.0/24 (Network)  │
  └─────────────────────────────────────────────────┘
```

---

## Detection Stack

### Cloud SIEM — Microsoft Sentinel
- Azure Monitor Agent (AMA) + Data Collection Rules (DCR)
- 30+ log sources onboarded
- Azure Arc for non-Azure VM onboarding
- Microsoft Defender for Identity (MDI v3.0.6.405)
- 7 KQL analytics rules (see `kql-detection-rules/`)

### On-Premise SIEM
- **Elastic Stack** — log ingestion + Kibana dashboards
- **Splunk v9.1.1** — detection + analytics
- **Wazuh** — host-based IDS + 4 custom rules (see `wazuh-detection-rules/`)

### NDR
- **Zeek** — network traffic analysis + protocol logs
- **Security Onion** — network monitoring
- Zeek → Wazuh pipeline for network-based detection

### EDR & Identity
- **Sysmon** — Windows telemetry (process creation, network, file, registry)
- **Windows Defender** — endpoint protection
- **Microsoft Defender for Identity** — identity-based attack detection
- **BloodHound CE** (Docker) — AD attack path enumeration (4 privilege escalation paths identified)

---

## Red Team / Attack Simulation

| Tool | Purpose |
|---|---|
| Kali Linux | Primary attacker VM |
| Metasploitable2 | Vulnerable target |
| Mythic C2 | Command & control framework |
| Nessus / OpenVAS | Vulnerability scanning |

Attack scenarios run: ransomware simulation, lateral movement, credential dumping, privilege escalation — all validated against live detection rules before production promotion.

---

## SOAR

- **n8n v2.23.3** — orchestration
- **GPT-4.1 Mini** — AI triage (MITRE mapping, severity scoring)
- **AbuseIPDB** — IP reputation enrichment
- **IRIS** — case management
- **Slack** — analyst notification

See `soar-pipeline/README.md` for full pipeline detail.
