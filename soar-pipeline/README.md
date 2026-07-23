# SOAR Pipeline — n8n Automation

## Overview

End-to-end SOC alert automation pipeline built on **n8n v2.23.3**, triggered via webhook from **Elastic (ELK) SIEM** and processing alerts with zero manual handling. Reduces detection-to-case time to under 60 seconds.

---

## Pipeline Architecture

```
Elastic (ELK) SIEM Alert (POST)
        │
        ▼
┌─────────────────┐
│    Webhook      │  Receives alert payload from Elastic SIEM via HTTP POST
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   FP Filter     │  Suppresses known false positives — rule-based discard
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Dedup Check    │  Prevents duplicate cases for the same alert signature
└────────┬────────┘
         │
         ▼
┌──────────────────────────────────────┐
│       GPT Message Model              │  GPT-4.1 Mini
│  • MITRE ATT&CK TTP classification   │
│  • Severity assessment               │  ┌──────────────────────┐
│  • Analyst summary (1-2 lines)       │◄─┤  AbuseIPDB Tool      │
│                                      │  │  IP reputation lookup│
└────────┬─────────────────────────────┘  │  (GET api.abuseipdb) │
         │                                └──────────────────────┘
         ▼
┌─────────────────┐
│ Severity Mapper │  Maps GPT output to standardised severity levels (1-4)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Send a Message  │  Slack notification → #soc-alerts
│ (Slack)         │  Includes: alert title, severity, TTP, IP reputation
└────────┬────────┘
         │
         ▼
┌─────────────────────────────────┐
│     Create IRIS Case            │  POST → https://192.168.50.70 (IRIS)
│  Auto-creates case with:        │  Case management with full alert context,
│  • AI-generated summary         │  IOCs, severity, and MITRE mapping
│  • MITRE TTP                    │
│  • IP enrichment data           │
└─────────────────────────────────┘
```

---

## Key Design Decisions

**AbuseIPDB as a GPT Tool (not a sequential node):** The IP enrichment is invoked *by* the GPT model as a tool call — meaning GPT decides when to look up an IP based on the alert context, rather than enriching every alert blindly. This reduces unnecessary API calls and keeps enrichment contextually relevant.

**FP Filter before GPT:** Known false positives are discarded before reaching the AI triage stage — keeping GPT-4.1 Mini API costs low and preventing noise from polluting the case management system.

**Dedup before triage:** Deduplication runs before AI processing, so repeated alerts from the same source within a short window don't generate multiple cases.

---

## Stats

| Metric | Value |
|---|---|
| Alerts processed | 134+ |
| Manual handling required | 0 |
| Average detection-to-case time | < 60 seconds |
| False positives suppressed | ~18% of alert volume |

---

## Components

| Node | Type | Purpose |
|---|---|---|
| Webhook | Trigger | Receives Elastic SIEM alert payload via HTTP POST |
| FP Filter | Code | Rule-based suppression of known false positives |
| Dedup Check | Code | Prevents duplicate cases for repeated alerts |
| GPT Message Model | AI (GPT-4.1 Mini) | MITRE mapping, severity scoring, analyst summary |
| AbuseIPDB-Enrichment | Tool (HTTP GET) | IP reputation lookup — called by GPT as needed |
| Severity Mapper | Code | Normalises GPT severity output to 1-4 scale |
| Send a Message | Slack | Real-time notification to #soc-alerts channel |
| Create IRIS Case | HTTP POST | Auto-creates case in IRIS with full alert context |

---

## Pipeline Export

To add the workflow JSON to this repo:

1. Open n8n → Workflows → SOAR Pipeline
2. Click ⋮ (three dots) → **Export** → **Download**
3. Save as `soar-pipeline/n8n_soar_pipeline.json`
4. Commit and push

---

## Environment Variables Required

```
OPENAI_API_KEY=          # GPT-4.1 Mini (OpenAI API)
ABUSEIPDB_API_KEY=       # AbuseIPDB (free tier: 1,000 req/day)
IRIS_BASE_URL=           # Your IRIS instance URL (e.g. https://192.168.50.70)
IRIS_API_KEY=            # IRIS API key
SLACK_WEBHOOK_URL=       # Slack incoming webhook URL
```

> **Never commit API keys.** Use n8n's built-in credentials store or `.env` with `.gitignore`.
