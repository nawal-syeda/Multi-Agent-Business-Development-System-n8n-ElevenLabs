# Automated Business Development System

An end-to-end n8n workflow orchestrating AI-powered agents for automated lead generation, qualification, outreach, and deal management. This system automates the entire sales pipeline from ICP-driven prospecting to demo booking with real-time notifications.

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────┐
│              Business Development Manager               │
│                   (Main Orchestrator)                   │
└──────────────────┬──────────────────────────────────────┘
                   │
                   ├─→ Calls MCP Prospecting Agent
                   │        ↓
                   │   ┌─────────────────────────┐
                   │   │  Prospecting Sub-agent  │
                   │   │  (Web Scraping + Email  │
                   │   │     Enrichment)         │
                   │   └──────────┬──────────────┘
                   │              │
                   │              ↓
                   ├─→ ┌──────────────────────┐
                   │   │   RevOps Sub-agent   │
                   │   │  (CRM Lead Creation  │
                   │   │  + Deduplication)    │
                   │   └──────────┬───────────┘
                   │              │
                   │              ↓
                   ├─→ ┌──────────────────────┐
                   │   │     SDR Agent        │
                   │   │ (Personalized Email  │
                   │   │    Drafting)         │
                   │   └──────────────────────┘
                   │
                   └─→ ┌──────────────────────────────────┐
                       │      Account Executive           │
                       ├──────────────────────────────────┤
                       │  ├─ Deal Recording Sub-agent     │
                       │  │   (ElevenLabs Voice → CRM)    │
                       │  │                                │
                       │  └─ Demo Booking Sub-agent       │
                       │      (Calendar Integration)      │
                       └──────────────────────────────────┘
                                     ↓
                       ┌─────────────────────────┐
                       │   Push Notifications    │
                       │   (Success/Failure)     │
                       └─────────────────────────┘
```

**Workflow Structure:**
- `MAS BD/` - Main workflow directory
  - `Account Executive/`
    - `Deal recording sub-agent.json`
    - `Demo booking subagent.json`
  - `Business Development/`
    - `Business Development Manager.json` (Orchestrator)
    - `RevOps subagent.json`
    - `SDR.json`
  - `mcp/` (MCP Server Integration)
    - `prospecting mcp.json` (MCP server connector)
    - `prospecting sub-agent.json` (Actual prospecting logic)

---

## 📋 Table of Contents

- [Overview](#overview)
- [System Architecture](#system-architecture)
- [Workflow Components](#workflow-components)
  - [Business Development Manager (Orchestrator)](#1-business-development-manager-orchestrator)
  - [MCP Prospecting Server](#2-mcp-prospecting-server)
  - [Prospecting Sub-agent](#3-prospecting-sub-agent)
  - [RevOps Sub-agent](#4-revops-sub-agent)
  - [SDR Agent](#5-sdr-agent)
  - [Account Executive](#6-account-executive)
- [Tech Stack](#tech-stack)
- [Setup Instructions](#setup-instructions)
- [File Structure](#file-structure)
- [Configuration](#configuration)
- [Notifications](#notifications)
- [Error Handling](#error-handling)

---

## Overview

This n8n workflow implements a complete AI-powered business development pipeline:

1. **Lead Discovery**: Scrapes the web for prospects matching your ICP
2. **Lead Enrichment**: Finds verified email addresses
3. **CRM Integration**: Creates leads in Pipedrive with deduplication
4. **Outreach Automation**: Drafts personalized sales emails
5. **Deal Management**: Records inbound calls and books demos
6. **Real-time Notifications**: Push alerts on success/failure at every stage

**Key Features:**
- Fully automated lead generation from ICP definition
- Duplicate detection to prevent CRM pollution
- LLM-powered personalized email drafting
- Voice-enabled deal recording with ElevenLabs
- Calendar-aware demo booking
- End-to-end notification system

---

## Workflow Components

### 1. Business Development Manager (Orchestrator)

**File:** `Business Development/Business Development Manager.json`

**Purpose:** Central controller that coordinates all sub-agents and manages the complete business development lifecycle.

**Process Flow:**
```
1. Read ICP definition from Google Drive
2. Call MCP Prospecting Server with ICP parameters
3. MCP server triggers Prospecting Sub-agent
4. Monitor lead creation by RevOps Sub-agent
5. Trigger SDR Agent for email drafting
6. Monitor Account Executive activities
7. Send push notifications on success/failure
```

**Inputs:**
- ICP definition document (Google Drive)
  - Target industries
  - Company size range
  - Geographic regions
  - Technology stack requirements
  - Budget indicators

**Outputs:**
- Orchestration logs
- Success/failure notifications
- Pipeline metrics (leads generated, emails drafted, demos booked)

**Integrations:**
- Google Drive (ICP document retrieval)
- MCP server (prospecting coordination)
- n8n webhook triggers for sub-agent coordination
- Push notification service

**Error Handling:**
- Retries failed MCP calls (max 3 attempts)
- Fallback notifications on complete workflow failure
- Logs all agent interactions for debugging

---

### 2. MCP Prospecting Server

**File:** `mcp/prospecting mcp.json`

**Purpose:** MCP (Model Context Protocol) server that acts as an interface between the Business Development Manager and the Prospecting Sub-agent. Provides standardized tool calling and context management for prospecting operations.

**MCP Tools Exposed:**
- `search_prospects`: Searches for companies matching ICP criteria
- `enrich_contact_data`: Finds email addresses for discovered prospects
- `validate_leads`: Pre-validates leads before sending to RevOps

**Process Flow:**
```
1. Receive ICP parameters from Business Development Manager
2. Initialize prospecting context
3. Call Prospecting Sub-agent with structured parameters
4. Manage state between scraping iterations
5. Return enriched lead batch to orchestrator
```

**Benefits of MCP Architecture:**
- Standardized tool interface for prospecting operations
- Better error handling and retry logic
- Context preservation across multi-step prospecting
- Easier testing and debugging of prospecting logic

**Integration:**
- Connected to Business Development Manager
- Triggers Prospecting Sub-agent
- Returns results via MCP response format

---

### 3. Prospecting Sub-agent

**File:** `mcp/prospecting sub-agent.json`

### 3. Prospecting Sub-agent

**File:** `mcp/prospecting sub-agent.json`

**Purpose:** Discovers potential leads matching the ICP by scraping the web and enriching contact data. Called by MCP Prospecting Server.

**Process Flow:**
```
1. Receive ICP definition from Business Dev Manager
2. Scrape web for prospects using Firecrawl/Apify
   - Target: Company websites, LinkedIn, industry directories
   - Filters: Company size, industry, location, tech stack
3. Extract company names and domains
4. Find verified email addresses via Hunter.io
5. Send enriched lead data to RevOps Sub-agent
```

**Data Collected:**
- Company name
- Domain
- Industry
- Company size
- Location
- Contact email (verified)
- LinkedIn profile URL
- Tech stack (when available)

**Integrations:**
- **Firecrawl/Apify**: Web scraping for lead discovery
- **Hunter.io**: Email finding and verification
- n8n webhook to RevOps Sub-agent

**Error Handling:**
- Skips leads with no valid email found
- Logs scraping failures for manual review
- Rate limiting for API compliance

---

### 4. RevOps Sub-agent

**File:** `Business Development/RevOps subagent.json`

**Purpose:** Manages CRM operations including lead creation and duplicate detection.

**Process Flow:**
```
1. Receive enriched lead data from Prospecting Sub-agent
2. Check if lead already exists in Pipedrive CRM
   - Search by: email, company name, domain
3. If duplicate found:
   - Skip creation
   - Log duplicate entry
4. If new lead:
   - Create lead in Pipedrive
   - Tag with source: "Automated Prospecting"
   - Assign to appropriate sales rep
   - Add to nurture sequence
5. Return success/failure status
```

**Duplicate Detection Logic:**
- Email match (primary)
- Domain match + company name similarity (secondary)
- Manual review queue for ambiguous cases

**Integrations:**
- **Pipedrive CRM**: Lead creation and management
- n8n data store for tracking processed leads

**CRM Fields Populated:**
- Lead name (company)
- Contact person (if available)
- Email
- Phone (if available)
- Industry
- Company size
- Source: "Automated - n8n Workflow"
- Status: "New - Awaiting Outreach"

---

### 5. SDR Agent

**File:** `Business Development/SDR.json`

**Purpose:** Drafts personalized sales outreach emails for newly created CRM leads.

**Process Flow:**
```
1. Query Pipedrive for leads with status "New - Awaiting Outreach"
2. For each lead:
   - Retrieve company information from CRM
   - Generate personalized email using LLM (Claude/GPT)
     * Research company's recent activities
     * Identify pain points relevant to your solution
     * Craft value proposition
     * Include clear CTA
3. Save draft email in Gmail inbox
4. Update lead status in Pipedrive: "Outreach Draft Ready"
5. Notify Business Dev Manager of completion
```

**Email Personalization Elements:**
- Company-specific pain points
- Recent news or funding announcements
- Industry trends relevant to prospect
- Tailored value proposition
- Specific CTA (demo, call, resource download)

**LLM Prompt Strategy:**
```
System: You are an experienced SDR writing personalized cold emails.

Context:
- Company: {company_name}
- Industry: {industry}
- Company size: {size}
- Recent activities: {news_summary}

Task: Write a 3-paragraph cold email:
1. Personalized hook based on recent company activity
2. Value proposition addressing likely pain points
3. Soft CTA for demo booking

Tone: Professional, consultative, not salesy
Length: Under 150 words
```

**Integrations:**
- **Pipedrive CRM**: Lead retrieval and status updates
- **LLM API** (Claude/GPT): Email generation
- **Gmail API**: Draft creation
- Web search for company research (optional)

---

### 6. Account Executive

**Files:** 
- `Account Executive/Deal recording sub-agent.json`
- `Account Executive/Demo booking subagent.json`

**Purpose:** Handles inbound calls and demo bookings with voice AI and calendar integration.

The Account Executive consists of two interconnected sub-agents:

#### 6.1 Deal Recording Sub-agent

**File:** `Account Executive/Deal recording sub-agent.json`

**Purpose:** Processes inbound calls via ElevenLabs and creates deals in CRM.

**Process Flow:**
```
1. Receive inbound call through ElevenLabs voice agent
2. Extract caller information:
   - Caller name
   - Company
   - Reason for call
3. Verify lead exists in Pipedrive CRM
   - Search by name + company
   - If not found: create new lead on-the-fly
4. Create deal in Pipedrive:
   - Link to existing/new lead
   - Deal stage: "Initial Contact"
   - Deal value: Based on company size/industry
   - Source: "Inbound Call"
5. Log call transcript and summary in CRM
6. Trigger Demo Booking Sub-agent if caller requests demo
7. Send push notification to sales rep
```

**ElevenLabs Voice Agent Script:**
```
"Hello! Thank you for calling [Company Name]. 
I'm your AI assistant. May I have your name and company please?"

[Collect information]

"Great, {name}. I see you're calling from {company}. 
How can I help you today?"

[If demo requested]
"I'd be happy to schedule a demo. Let me check available slots..."
[Trigger Demo Booking Sub-agent]
```

**CRM Deal Fields:**
- Deal name: "{Company} - Inbound Call"
- Stage: "Initial Contact"
- Expected close date: +30 days
- Contact person
- Call recording URL
- Call summary
- Next action: "Demo Scheduled" or "Follow-up Required"

#### 6.2 Demo Booking Sub-agent

**File:** `Account Executive/Demo booking subagent.json`

**Purpose:** Checks calendar availability and books demos during live calls.

**Process Flow:**
```
1. Triggered by Deal Recording Agent or ElevenLabs voice agent
2. Query Google Calendar for available demo slots
   - Time range: Next 14 days
   - Buffer: 30 minutes between meetings
   - Working hours: 9 AM - 5 PM (timezone-aware)
3. Present available slots to caller via voice agent:
   "I have availability on:
    - Monday, January 15th at 2 PM
    - Wednesday, January 17th at 10 AM
    - Friday, January 19th at 3 PM
    Which works best for you?"
4. Caller selects preferred slot
5. Create calendar event:
   - Title: "Demo - {Company Name}"
   - Duration: 45 minutes
   - Attendees: Caller email, sales rep
   - Description: Deal link, company info, call summary
6. Update deal in Pipedrive:
   - Stage: "Demo Scheduled"
   - Next activity: Calendar event link
   - Demo date: Selected slot
7. Send calendar invite to prospect
8. Send push notification to sales rep and Business Dev Manager
```

**Calendar Integration Logic:**
- Blocks out existing meetings
- Respects calendar working hours
- Handles timezone conversions
- Supports rescheduling requests

**Integrations:**
- **ElevenLabs**: Voice AI for natural conversation
- **Google Calendar**: Availability checking and booking
- **Pipedrive CRM**: Deal updates and activity logging
- **Gmail**: Calendar invite sending

---

## 📁 File Structure

```
MAS BD/
├── Account Executive/
│   ├── Deal recording sub-agent.json
│   └── Demo booking subagent.json
│
├── Business Development/
│   ├── Business Development Manager.json    # Main orchestrator
│   ├── RevOps subagent.json                # CRM management
│   └── SDR.json                            # Email drafting
│
└── mcp/
    ├── prospecting mcp.json                # MCP server interface
    └── prospecting sub-agent.json          # Web scraping logic
```

**File Descriptions:**

| File | Purpose | Triggers | Called By |
|------|---------|----------|-----------|
| `Business Development Manager.json` | Main orchestrator | MCP, RevOps, SDR, Account Executive | Manual/Scheduled |
| `prospecting mcp.json` | MCP server interface | Prospecting Sub-agent | Business Dev Manager |
| `prospecting sub-agent.json` | Web scraping & enrichment | RevOps Sub-agent | MCP Server |
| `RevOps subagent.json` | CRM lead creation | SDR Agent (via Manager) | Prospecting Sub-agent |
| `SDR.json` | Email drafting | - | Business Dev Manager |
| `Deal recording sub-agent.json` | Voice call handling | Demo Booking Sub-agent | ElevenLabs Webhook |
| `Demo booking subagent.json` | Calendar booking | - | Deal Recording Sub-agent |

---

## Tech Stack

### Core Platform
- **n8n** (v1.x): Workflow orchestration
- **MCP (Model Context Protocol)**: Standardized agent communication

### Web Scraping & Data Enrichment
- **Firecrawl**: Web scraping for company discovery
- **Apify**: Alternative web scraping platform
- **Hunter.io**: Email finding and verification

### CRM & Sales Tools
- **Pipedrive CRM**: Lead and deal management
- **Gmail API**: Email draft creation and calendar invites
- **Google Calendar**: Demo scheduling

### AI & Voice
- **LLM APIs**: Claude/GPT for email personalization
- **ElevenLabs**: Voice AI for inbound call handling

### Integrations
- **Google Drive**: ICP document storage
- **Push Notification Service**: Real-time alerts
- **Webhooks**: Inter-agent communication

---

## Setup Instructions

### Prerequisites

1. **n8n Installation** (Self-hosted or Cloud)
2. **API Keys Required:**
   - Hunter.io API key
   - Firecrawl/Apify API key
   - Pipedrive API token
   - Google Workspace (Drive, Calendar, Gmail)
   - LLM API key (Claude or OpenAI)
   - ElevenLabs API key
   - Push notification service credentials

### Installation Steps

#### 1. Import Workflows

```bash
# Clone or download workflow files
# Import each JSON file into n8n via:
# Workflows → Import from File → Select JSON
```

**Import Order:**
1. `prospecting mcp.json`
2. `prospecting sub-agent.json`
3. `RevOps subagent.json`
4. `SDR.json`
5. `Deal recording sub-agent.json`
6. `Demo booking subagent.json`
7. `Business Development Manager.json` (last)

#### 2. Configure Credentials

In n8n, add credentials for:

```
- Hunter.io → API Key
- Firecrawl → API Key  
- Pipedrive → API Token
- Google Drive → OAuth2
- Gmail → OAuth2
- Google Calendar → OAuth2
- Claude/OpenAI → API Key
- ElevenLabs → API Key
```

#### 3. Update Webhook URLs

For each workflow, update webhook URLs to match your n8n instance:

**Business Development Manager:**
- MCP prospecting webhook: `https://your-n8n.com/webhook/mcp-prospecting`

**MCP Server:**
- Prospecting sub-agent webhook: `https://your-n8n.com/webhook/prospecting`

**Prospecting Sub-agent:**
- RevOps webhook: `https://your-n8n.com/webhook/revops`

**Deal Recording Sub-agent:**
- ElevenLabs inbound webhook: `https://your-n8n.com/webhook/elevenlabs-call`
- Demo booking webhook: `https://your-n8n.com/webhook/demo-booking`

#### 4. Configure Google Drive

1. Create a folder for ICP definitions
2. Upload ICP template document
3. Update Google Drive node in Business Development Manager with folder ID

**ICP Document Structure:**
```markdown
# Ideal Customer Profile

## Company Characteristics
- Industry: [SaaS, FinTech, E-commerce]
- Size: [10-50, 50-200, 200-1000 employees]
- Revenue: [$1M-$10M, $10M-$50M]
- Location: [North America, Europe, APAC]

## Technology Stack
- Uses: [Salesforce, HubSpot, etc.]
- Pain Points: [Manual processes, data silos]

## Contact Criteria
- Job Titles: [VP of Sales, CRO, Head of RevOps]
- Seniority: [Director+, VP+, C-Level]
```

#### 5. Configure Pipedrive

1. Create custom fields in Pipedrive:
   - Lead Source: "Automated - n8n"
   - Lead Status: "New - Awaiting Outreach"
   - Lead Score: Numeric field
   - Company Size: Dropdown
   - Tech Stack: Text field

2. Update field IDs in RevOps and SDR workflows

#### 6. Set Up ElevenLabs Agent

1. Create conversational agent in ElevenLabs
2. Configure voice and conversation flow
3. Add webhook integration pointing to Deal Recording Sub-agent
4. Test with sample call

#### 7. Test Each Component

**Test Sequence:**
```
1. Manual trigger → Business Development Manager
2. Verify MCP server receives call
3. Check Prospecting Sub-agent scrapes successfully  
4. Confirm RevOps creates leads in Pipedrive
5. Verify SDR drafts emails in Gmail
6. Test ElevenLabs → Deal Recording flow
7. Test Demo Booking calendar integration
```

---

## Configuration

### Environment Variables

Create `.env` file or configure in n8n:

```env
# APIs
HUNTER_API_KEY=your_hunter_key
FIRECRAWL_API_KEY=your_firecrawl_key
PIPEDRIVE_API_TOKEN=your_pipedrive_token
CLAUDE_API_KEY=your_claude_key
ELEVENLABS_API_KEY=your_elevenlabs_key

# Google
GOOGLE_DRIVE_FOLDER_ID=your_folder_id
GOOGLE_CALENDAR_ID=primary

# Webhooks
MCP_WEBHOOK_URL=https://your-n8n.com/webhook/mcp-prospecting
REVOPS_WEBHOOK_URL=https://your-n8n.com/webhook/revops
ELEVENLABS_WEBHOOK_URL=https://your-n8n.com/webhook/elevenlabs-call

# Configuration
MAX_LEADS_PER_RUN=50
EMAIL_DRAFT_LIMIT=20
DEMO_SLOT_DURATION_MINUTES=45
CALENDAR_LOOKAHEAD_DAYS=14
```

### Workflow-Specific Settings

**Business Development Manager:**
- Trigger schedule: Daily at 9 AM
- Max concurrent prospecting runs: 1
- Notification recipients: sales@company.com

**Prospecting Sub-agent:**
- Scraping depth: 3 pages per domain
- Email verification: Enabled
- Rate limit: 100 requests/hour

**RevOps Sub-agent:**
- Duplicate detection: Email + Domain
- Auto-assign: Round-robin to sales reps
- Lead score threshold: 60/100

**SDR Agent:**
- Email length: 120-150 words
- Personalization level: High
- CTA type: Demo booking
- Draft folder: "Sales Outreach"

**Account Executive:**
- Call recording: Enabled
- Transcript storage: Pipedrive notes
- Demo duration: 45 minutes
- Timezone: Auto-detect from caller

---

## Notifications

### Push Notification Events

**Success Notifications:**
- ✅ Leads successfully created in CRM
- ✅ Email drafts ready for review
- ✅ Demo successfully booked
- ✅ Deal created from inbound call

**Failure Notifications:**
- ❌ Prospecting failed (no leads found)
- ❌ RevOps duplicate detection blocked all leads
- ❌ Email generation failed
- ❌ Calendar booking conflict
- ❌ CRM API error

**Notification Payload:**
```json
{
  "event": "leads_created",
  "status": "success",
  "timestamp": "2025-01-15T10:30:00Z",
  "data": {
    "leads_count": 15,
    "duplicates_skipped": 3,
    "emails_drafted": 12,
    "workflow_run_id": "abc123"
  }
}
```

---

## Error Handling

### Retry Logic

| Component | Error Type | Retry Strategy |
|-----------|-----------|----------------|
| MCP Server | Connection timeout | 3 retries, 5s delay |
| Prospecting | Rate limit | Exponential backoff |
| Hunter.io | API limit | Queue for next run |
| Pipedrive | 5xx errors | 3 retries, immediate |
| Gmail API | Auth failure | Re-authenticate |
| ElevenLabs | Webhook timeout | 2 retries, 10s delay |

### Fallback Mechanisms

**Prospecting Failure:**
- Switch from Firecrawl to Apify
- Reduce scraping depth
- Notify admin for manual intervention

**CRM Creation Failure:**
- Log leads to backup CSV
- Queue for retry in 1 hour
- Send notification with failed leads

**Email Drafting Failure:**
- Fall back to template-based emails
- Mark leads for manual review
- Continue workflow execution

### Monitoring & Logs

**Logged Events:**
- All API calls and responses
- Lead creation/skipping decisions
- Email generation prompts and outputs
- Call transcripts and summaries
- Calendar booking confirmations

**Log Retention:**
- Execution logs: 30 days
- Error logs: 90 days
- Call recordings: As per compliance
- Email drafts: Until sent/deleted

---

## Usage Example

### Complete Workflow Execution

```
1. [9:00 AM] Business Development Manager triggered (scheduled)

2. [9:01 AM] Read ICP from Google Drive
   - Target: B2B SaaS companies, 50-200 employees, Series A+

3. [9:02 AM] Call MCP Prospecting Server
   - MCP Server initializes prospecting context
   
4. [9:03 AM] Prospecting Sub-agent scrapes web
   - Firecrawl finds 45 matching companies
   - Hunter.io finds emails for 38 contacts
   
5. [9:08 AM] RevOps Sub-agent processes leads
   - 15 duplicates detected and skipped
   - 23 new leads created in Pipedrive
   - Notification: "23 leads created, 15 duplicates skipped"
   
6. [9:10 AM] SDR Agent drafts emails
   - Generated 23 personalized emails
   - Saved as drafts in Gmail "Sales Outreach" folder
   - Notification: "23 email drafts ready for review"
   
7. [2:30 PM] Inbound call received via ElevenLabs
   - Caller: John Smith from Acme Corp
   - Deal Recording Sub-agent verifies lead in CRM
   - Deal created: "Acme Corp - Inbound Call"
   - Demo requested by caller
   
8. [2:31 PM] Demo Booking Sub-agent checks calendar
   - Available slots presented via voice agent
   - Caller selects: Friday, Jan 19 at 2 PM
   - Calendar event created, invite sent
   - Deal updated: Stage → "Demo Scheduled"
   - Notification: "Demo booked with Acme Corp - Jan 19, 2 PM"
```

---

## Troubleshooting

### Common Issues

**"No leads found"**
- Check ICP criteria (may be too restrictive)
- Verify Firecrawl/Apify API limits not exceeded
- Review scraping target websites (may have changed)

**"All leads marked as duplicates"**
- Review duplicate detection logic in RevOps
- Check if previous run already processed same companies
- Verify domain matching criteria

**"Email drafts not appearing in Gmail"**
- Confirm Gmail API credentials are valid
- Check OAuth token refresh
- Verify "Sales Outreach" folder exists

**"Demo booking fails"**
- Check Google Calendar API permissions
- Verify calendar ID is correct
- Ensure no conflicting events exist

**"ElevenLabs webhook not triggering"**
- Verify webhook URL is publicly accessible
- Check ElevenLabs agent configuration
- Review n8n webhook node settings

---

## Roadmap

### Planned Enhancements

- [ ] Multi-channel outreach (LinkedIn, Twitter DMs)
- [ ] A/B testing for email templates
- [ ] Lead scoring with ML model
- [ ] Automatic follow-up sequences
- [ ] Integration with Slack for team notifications
- [ ] Dashboard for pipeline metrics
- [ ] Voice sentiment analysis on calls
- [ ] Automatic meeting notes generation

---

## Contributing

To extend this workflow:

1. Fork the workflow JSON files
2. Make modifications in your n8n instance
3. Export updated JSON
4. Document changes in this README
5. Test end-to-end before production use

---

## License

This workflow system is provided as-is for internal use.

---

## Support

For issues or questions:
- Check n8n execution logs
- Review API provider status pages
- Consult n8n community forum
- Contact: info@zinnum.com

---

## Credits

Built with:
- n8n workflow automation
- MCP (Model Context Protocol)
- Multiple AI and sales tool integrations

**Author:** Syeda Nawal Fatima
**Version:** 1.0
**Last Updated:** January 2025
- **MCP (Model Context Protocol)**: Standardized agent communication