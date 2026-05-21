# Solution Design — AI Service Desk Copilot

## 1. Business Problem

Service desk analysts were spending 60%+ of their time on repetitive, low-complexity Tier-1 tickets (password resets, access requests, software installs, basic troubleshooting). This left limited capacity for high-severity compliance cases and strategic work.

### Pain Points
- High Tier-1 ticket volume consuming analyst bandwidth
- Inconsistent first-response times across shifts and regions
- Manual ticket triage prone to misrouting
- No self-service option for end users
- SLA breaches due to delayed assignment

---

## 2. Solution Overview

An enterprise Copilot Studio agent integrated with Azure OpenAI GPT-4, Power Automate, and Dynamics 365 Customer Service to:

- Automatically handle Tier-1 support intake via conversational AI
- Triage and classify tickets using NLP intent recognition
- Search internal knowledge bases using GPT-4 for instant resolution
- Auto-create structured cases in Dynamics 365 on escalation
- Route cases to the correct queue with priority and SLA metadata pre-filled

---

## 3. Architecture Decisions

### 3.1 Why Copilot Studio?
- Native integration with Power Platform and Dynamics 365
- No-code/low-code topic authoring for business users
- Built-in channel support (Teams, Web Chat, telephony)
- Entra ID authentication out of the box

### 3.2 Why Azure OpenAI GPT-4 for Knowledge Search?
- Superior semantic understanding vs keyword-based search
- Handles varied phrasing of the same question
- Can synthesize answers from multiple knowledge articles
- Controlled via prompt engineering for safe, scoped responses

### 3.3 Why Power Automate for Orchestration?
- Tight Dataverse and D365 connector ecosystem
- Enables complex multi-step flows without custom code
- Built-in retry and error handling
- Auditable flow run history

### 3.4 Dataverse vs SQL
- Dataverse chosen for native D365 integration
- Eliminates dual-write complexity
- Row-level security aligns with Entra ID groups
- Rollup and calculated fields reduce report query load

---

## 4. Conversation Design

### 4.1 Topic Categories

| Topic | Trigger Phrases | Resolution Path |
|---|---|---|
| Password Reset | "reset password", "locked out", "can't login" | Self-service guided flow |
| Access Request | "need access to", "request permission" | Auto-ticket to IT Access queue |
| Software Install | "install", "download", "setup" | Knowledge article → escalate if unresolved |
| Hardware Issue | "laptop", "monitor", "keyboard", "mouse" | Ticket to Hardware queue |
| General Query | (fallback) | GPT-4 knowledge search |

### 4.2 Conversation Flow

```
User Message
    │
    ▼
Intent Recognition (Copilot Studio NLP)
    │
    ├── High Confidence → Topic Flow
    │       │
    │       ├── Resolved → Log interaction, close
    │       └── Unresolved → GPT-4 Knowledge Search
    │               │
    │               ├── Resolved → Log interaction, close
    │               └── Unresolved → Escalate to Agent
    │
    └── Low Confidence → Clarification Prompt (max 2 attempts)
            │
            └── Still unclear → Escalate to Agent
```

### 4.3 Escalation Handling
- Full conversation transcript passed to D365 case
- User details pre-populated from Entra ID profile
- Priority set based on keyword detection (e.g. "urgent", "critical", "down")
- Assigned to correct queue based on topic classification

---

## 5. Security Design

### 5.1 Authentication
- All channels secured with Entra ID OAuth 2.0
- Bot validates token on every session start
- No anonymous access permitted

### 5.2 Authorization
- RBAC enforced via Entra ID security groups
- Agents see only their assigned queue cases
- Admins have full Dataverse access
- End users can only view their own submitted cases

### 5.3 Data Handling
- No PII stored in conversation logs beyond session
- Dataverse row-level security segments data by business unit
- All API calls use managed identity (no stored secrets)
- Azure Key Vault used for OpenAI API key management

---

## 6. SLA Configuration

| Priority | First Response | Resolution Target |
|---|---|---|
| Critical (P1) | 15 minutes | 4 hours |
| High (P2) | 1 hour | 8 hours |
| Medium (P3) | 4 hours | 2 business days |
| Low (P4) | 1 business day | 5 business days |

SLA timers are configured in Dynamics 365 Customer Service and pause automatically on "Waiting for Customer" status.

---

## 7. Known Limitations

- GPT-4 knowledge search limited to indexed knowledge articles (no real-time system queries)
- Copilot Studio does not support voice channel natively in this configuration
- Maximum conversation history passed to GPT-4 is 10 turns (token limit management)
- Multi-language support not implemented in v1 (English only)

---

## 8. Future Enhancements

- [ ] Proactive notifications (SLA breach warnings pushed to users via Teams)
- [ ] Voice channel integration via Azure Communication Services
- [ ] Multi-language support (Spanish, French priority)
- [ ] Sentiment analysis to auto-escalate frustrated users
- [ ] Power BI embedded analytics within the bot for self-service reporting
