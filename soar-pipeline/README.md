# SOAR Pipeline — n8n Automation

## Overview

End-to-end SOC alert automation pipeline built on **n8n v2.23.3**, processing Microsoft Sentinel alerts with zero manual handling. Designed to eliminate routine triage toil and reduce detection-to-case time to under 60 seconds.

---

## Pipeline Architecture

```
Microsoft Sentinel Alert
        │
        ▼
  n8n Webhook Trigger
        │
        ▼
  Dedup + FP Filter ──── Known FP? ──► Discard + Log
        │ No
        ▼
  GPT-4.1 Mini Triage
  ┌─────────────────────────────────┐
  │ • MITRE ATT&CK TTP mapping      │
  │ • Severity score (1-10)         │
  │ • Analyst summary (1-2 lines)   │
  └─────────────────────────────────┘
        │
        ▼
  AbuseIPDB Enrichment
  (Reputation score + ISP + country)
        │
        ▼
  IRIS Case Auto-Creation
  (Title, severity, IOCs, AI summary)
        │
        ▼
  Slack Notification
  (Alert summary → #soc-alerts channel)
```

---

## Stats

| Metric | Value |
|---|---|
| Alerts processed | 134+ |
| Manual handling required | 0 |
| Average detection-to-case time | < 60 seconds |
| False positive filter effectiveness | ~18% of alerts suppressed |

---

## Components

| Component | Purpose |
|---|---|
| **n8n v2.23.3** | Orchestration engine — webhook trigger, routing, API calls |
| **GPT-4.1 Mini** | AI triage — MITRE mapping, severity scoring, analyst summary |
| **AbuseIPDB** | IP reputation enrichment — confidence score, ISP, country |
| **IRIS** | Case management — auto-creation with IOCs and AI summary |
| **Slack** | Notification — real-time alert to #soc-alerts |

---

## Pipeline JSON

Export your n8n workflow:  
*n8n → Workflows → [workflow name] → ⋮ → Export → Download JSON*

Save as `n8n_soar_pipeline.json` in this directory and commit.

---

## Environment Variables Required

```
OPENAI_API_KEY=          # GPT-4.1 Mini (OpenAI API)
ABUSEIPDB_API_KEY=       # AbuseIPDB API key (free tier: 1000 req/day)
IRIS_API_KEY=            # IRIS case management API key
IRIS_BASE_URL=           # Your IRIS instance URL
SLACK_WEBHOOK_URL=       # Slack incoming webhook URL
```

> **Never commit API keys.** Use n8n's built-in credentials store or environment variables.
