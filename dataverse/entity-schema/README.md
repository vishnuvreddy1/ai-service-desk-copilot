# Dataverse — Entity Schema

## Overview
This document defines the custom Dataverse entities created to support the AI Service Desk Copilot beyond the standard Dynamics 365 Customer Service tables.

---

## Standard D365 Tables Used (No Modification)

| Table | Display Name | Purpose |
|---|---|---|
| `incident` | Case | Core support ticket record |
| `contact` | Contact | End user records |
| `queue` | Queue | IT support queues |
| `queueitem` | Queue Item | Case-to-queue assignment |
| `knowledgearticle` | Knowledge Article | Standard KB articles |
| `sla` | SLA | SLA definitions |
| `slakpiinstance` | SLA KPI Instance | Per-case SLA tracking |

---

## Custom Tables

### Table 1 — Bot Interaction Log (`cr_botinteraction`)

**Purpose:** Audit trail of every bot conversation session. Used for analytics, debugging, and improving bot topics.

| Column | Type | Description |
|---|---|---|
| `cr_botinteractionid` | Unique Identifier (PK) | Auto-generated GUID |
| `cr_name` | Single Line of Text | Auto-name: "Bot Session — {date} — {user}" |
| `cr_userid` | Single Line of Text | Entra ID Object ID of the user |
| `cr_username` | Single Line of Text | Display name of the user |
| `cr_useremail` | Single Line of Text | Email address |
| `cr_query` | Multiple Lines of Text | User's original query |
| `cr_response` | Multiple Lines of Text | Bot's final response |
| `cr_category` | Option Set | PasswordReset, AccessRequest, SoftwareInstall, HardwareIssue, General |
| `cr_resolved` | Two Options (Yes/No) | Was the query resolved without escalation? |
| `cr_escalated` | Two Options (Yes/No) | Was a ticket created? |
| `cr_caseid` | Lookup → incident | Related D365 case (if escalated) |
| `cr_fallbacklevel` | Whole Number | 0 = direct topic, 1-4 = fallback level reached |
| `cr_sessionduration` | Whole Number | Conversation duration in seconds |
| `cr_knowledgesearched` | Two Options (Yes/No) | Was GPT-4 knowledge search called? |
| `cr_createdon` | Date and Time | Timestamp (system-managed) |

**Relationships:**
- Many-to-one with `incident` (via `cr_caseid`)
- Many-to-one with `contact` (via `cr_userid` lookup, optional)

---

### Table 2 — Knowledge Article Extended (`cr_knowledgearticle_ext`)

**Purpose:** Extends the standard Knowledge Article table with metadata for AI Search indexing and bot-specific tagging.

| Column | Type | Description |
|---|---|---|
| `cr_knowledgearticle_extid` | Unique Identifier (PK) | Auto-generated GUID |
| `cr_name` | Single Line of Text | Article title (mirrors knowledgearticle.title) |
| `cr_knowledgearticleid` | Lookup → knowledgearticle | Parent standard KB article |
| `cr_category` | Option Set | PasswordReset, AccessRequest, SoftwareInstall, HardwareIssue, General |
| `cr_aiindexed` | Two Options (Yes/No) | Has been indexed in Azure AI Search |
| `cr_lastindexedon` | Date and Time | When the article was last synced to AI Search |
| `cr_searchhitcount` | Whole Number | How many times returned by GPT-4 search |
| `cr_resolvedcount` | Whole Number | Times this article led to a resolved interaction |
| `cr_confidence` | Decimal Number | Average confidence score when returned (0–1) |

**Relationships:**
- One-to-one with `knowledgearticle`

---

### Table 3 — Escalation Log (`cr_escalationlog`)

**Purpose:** Detailed record of every escalation from bot to human agent, including the reason and resolution outcome.

| Column | Type | Description |
|---|---|---|
| `cr_escalationlogid` | Unique Identifier (PK) | Auto-generated GUID |
| `cr_name` | Single Line of Text | Auto-name: "Escalation — {CaseNumber} — {date}" |
| `cr_caseid` | Lookup → incident | Related D365 case |
| `cr_botinteractionid` | Lookup → cr_botinteraction | Source bot session |
| `cr_escalationreason` | Option Set | UserRequested, UnresolvedQuery, HighPriorityKeyword, MaxAttemptsReached |
| `cr_escalationlevel` | Whole Number | Fallback level at time of escalation (1–4) |
| `cr_transcript` | Multiple Lines of Text | Full conversation transcript |
| `cr_attempteddresolutions` | Multiple Lines of Text | KB articles / GPT-4 answers shown before escalation |
| `cr_agentid` | Lookup → systemuser | Agent who handled the case |
| `cr_agentresolutiontime` | Whole Number | Minutes taken by agent to resolve |
| `cr_caseresolved` | Two Options (Yes/No) | Was the case ultimately resolved? |
| `cr_createdon` | Date and Time | Escalation timestamp (system-managed) |

**Relationships:**
- Many-to-one with `incident`
- Many-to-one with `cr_botinteraction`
- Many-to-one with `systemuser`

---

## Calculated & Rollup Fields

### On `incident` (standard table — added columns)

| Column | Type | Calculation |
|---|---|---|
| `new_source` | Option Set | Populated by bot: "Copilot Studio Bot" vs "Manual" |
| `new_transcript` | Multiple Lines of Text | Conversation transcript from bot escalation |
| `new_sla_first_response_deadline` | Date and Time | Calculated from SLA KPI + case creation time |
| `new_first_response_sent` | Two Options | Flipped to true when first agent response is logged |

---

## Option Set Definitions

### Category (`cr_category_optionset`)
| Value | Label |
|---|---|
| 100000000 | Password Reset |
| 100000001 | Access Request |
| 100000002 | Software Install |
| 100000003 | Hardware Issue |
| 100000004 | General |

### Escalation Reason (`cr_escalationreason_optionset`)
| Value | Label |
|---|---|
| 100000000 | User Requested |
| 100000001 | Unresolved Query |
| 100000002 | High Priority Keyword |
| 100000003 | Max Attempts Reached |

---

## Indexing & Performance

- `cr_botinteraction`: Index on `cr_userid`, `cr_createdon`, `cr_resolved`
- `cr_escalationlog`: Index on `cr_caseid`, `cr_createdon`
- `cr_knowledgearticle_ext`: Index on `cr_category`, `cr_aiindexed`

All custom tables use **Dataverse row-level security** scoped to the user's business unit. Agents see interactions from their queue only. Admins have full read access.
