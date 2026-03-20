---
name: n8n-build-protocol
description: Rules and architecture patterns for building n8n workflows in the AI Marketing Intelligence Engine. Use this skill whenever building, extending, or modifying any n8n automation workflow — including new platform modules (Google Ads, LinkedIn), optimization engines, alert systems, or multi-client routers. Always consult this skill before writing any n8n node configuration, workflow JSON, or Code node logic.
---

# SKILL: n8n Marketing Automation Build Protocol
**Version:** 1.0 | **Project:** AI Marketing Intelligence Engine

## Purpose
This skill defines how to build every n8n workflow in the AI Marketing Intelligence Engine.
Apply these rules to every new workflow module without exception.

## Core Architecture Rules

### 1. Always Use Modular Workflows
Never build one monolithic workflow.
Each module is a separate workflow with a single responsibility.

Current modules:
- `meta_ads_performance_engine` — Meta Ads data + AI report ✅ Built
- `campaign_optimization_engine` — reads report, pushes changes to Meta API 🔜
- `multi_client_router` — loops clients, triggers platform workflows 🔜
- `alert_engine` — monitors KPIs, fires Slack/email alerts 🔜
- `google_ads_performance_engine` — Google Ads data + AI report 🔜
- `linkedin_ads_performance_engine` — LinkedIn Ads data + AI report 🔜

### 2. Standard Node Order
```
Cron Trigger / Webhook → Client Registry → Split Clients → Platform API Request → Transform & Normalize → Build AI Prompt → Claude AI Analysis → Build Report → Convert to Binary → Save to Google Drive
```

### 3. Only Use These n8n Node Types
- `n8n-nodes-base.scheduleTrigger`
- `n8n-nodes-base.set`
- `n8n-nodes-base.code`
- `n8n-nodes-base.httpRequest`
- `n8n-nodes-base.googleDrive`
- `n8n-nodes-base.if`
- `n8n-nodes-base.merge`
- `n8n-nodes-base.executeworkflow`

### 4. Anthropic API Call Structure
- Method: POST
- URL: https://api.anthropic.com/v1/messages
- Model: claude-sonnet-4-20250514
- Max tokens: 4096
- anthropic-version: 2023-06-01

### 5. Google Drive Upload
Always two nodes: Convert to Binary (Code node) → Google Drive node
Folder field: bare folder ID only, never full URL

### 6. Separation of Concerns
- API requests and AI analysis are NEVER in the same node
- Data transformation is NEVER mixed with API calls
- Each node does exactly one thing

### 7. Performance Flags
- HIGH_FREQUENCY_FATIGUE: Frequency ≥ 3.5
- LOW_CTR: CTR < 0.8% with >5,000 impressions
- HIGH_CPA: CPA > $100
- LOW_ROAS: ROAS < 1.5x
- LOW_CVR: CVR < 1% with >200 clicks