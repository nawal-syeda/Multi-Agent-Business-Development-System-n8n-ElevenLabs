# Automated Business Development System

AI-powered n8n workflow for automated lead generation, outreach, CRM management, and demo booking.

---

## Workflow Overview

```text
Business Development Manager (Orchestrator)
│
├── MCP Prospecting Server
│     └── Prospecting Sub-agent
│           ├── Web scraping
│           └── Email enrichment
│
├── RevOps Sub-agent
│     └── CRM lead creation + deduplication
│
├── SDR Agent
│     └── Personalized email drafting
│
└── Account Executive
      ├── Deal Recording Agent
      │     └── ElevenLabs voice → CRM
      │
      └── Demo Booking Agent
            └── Google Calendar integration

↓
Push Notifications
```

---

## Components

### Business Development Manager

Main orchestrator controlling the full workflow lifecycle.

### MCP Prospecting Server

Handles structured prospecting requests and context management.

### Prospecting Sub-agent

* Scrapes company data
* Filters by ICP
* Enriches contact emails

**Tools**

* Firecrawl / Apify
* Hunter.io

### RevOps Sub-agent

* Detects duplicates
* Creates CRM leads
* Assigns sales reps

**CRM**

* Pipedrive

### SDR Agent

* Generates personalized cold emails using LLMs
* Saves Gmail drafts
* Updates CRM statuses

### Account Executive

#### Deal Recording Agent

* Handles inbound calls via ElevenLabs
* Creates CRM deals
* Logs transcripts

#### Demo Booking Agent

* Checks calendar availability
* Books demos
* Sends invites

---

## Tech Stack

### Core

* n8n
* MCP (Model Context Protocol)

### Prospecting

* Firecrawl
* Apify
* Hunter.io

### CRM & Communication

* Pipedrive
* Gmail API
* Google Calendar

### AI & Voice

* Claude / GPT
* ElevenLabs

---

## File Structure

```text
MAS BD/
├── Account Executive/
│   ├── Deal recording sub-agent.json
│   └── Demo booking subagent.json
│
├── Business Development/
│   ├── Business Development Manager.json
│   ├── RevOps subagent.json
│   └── SDR.json
│
└── mcp/
    ├── prospecting mcp.json
    └── prospecting sub-agent.json
```

---

## Setup

### Required APIs

* Hunter.io
* Firecrawl / Apify
* Pipedrive
* Google Workspace
* Claude/OpenAI
* ElevenLabs

### Import Order

1. prospecting mcp.json
2. prospecting sub-agent.json
3. RevOps subagent.json
4. SDR.json
5. Deal recording sub-agent.json
6. Demo booking subagent.json
7. Business Development Manager.json

---

## Features

* Automated ICP-based lead generation
* CRM deduplication
* AI-personalized outreach
* Voice-based inbound sales handling
* Calendar-aware demo scheduling
* Real-time notifications
* End-to-end workflow automation

---

## Error Handling

* Retry failed API calls
* Exponential backoff for rate limits
* Duplicate lead detection
* Backup logging for failures
* Workflow execution monitoring

---

## Example Workflow

```text
1. Load ICP from Google Drive
2. Trigger prospecting
3. Scrape + enrich leads
4. Create CRM entries
5. Generate outreach emails
6. Handle inbound calls
7. Book demos
8. Send notifications
9. MCP server used as custom Claude connector to generate leads by describing ICP to claude in simple terms
```
