# Copilot Studio — Escalation Flow Design

## Overview
The escalation flow handles graceful handoff from the AI bot to a live human agent. It preserves full conversation context and ensures the agent has everything they need to continue without asking the user to repeat themselves.

---

## Escalation Triggers

| Trigger | Condition |
|---|---|
| User-initiated | User says "agent", "human", "real person", "escalate", "speak to someone" |
| Unresolved after knowledge search | `resolved = false` returned from Knowledge Search Flow |
| Two failed resolution attempts | Counter variable reaches 2 in any topic |
| High-priority keyword detected | User message contains "urgent", "critical", "down", "outage", "emergency" |
| Sentiment threshold | Negative sentiment detected (future enhancement) |

---

## Escalation Flow Steps

### Step 1 — Capture Context
Before escalating, collect any missing context:

```
Bot: "I'll connect you with a support agent. 
      Can you give me a quick summary of your issue so I can brief them?"

[If already known from conversation → skip this step]
[Free text input → store as escalationSummary]
```

### Step 2 — Set Priority
Detect priority keywords in conversation history:

```
If message contains any of: ["urgent", "critical", "outage", "down", "emergency", "asap"]
  → priority = P1 or P2
Else
  → priority = P3
```

### Step 3 — Call Ticket Creation Flow
Pass to Power Automate:
```json
{
  "userId": "<from Entra ID auth>",
  "userName": "<from Entra ID auth>",
  "userEmail": "<from Entra ID auth>",
  "category": "<current topic category or General>",
  "description": "<escalationSummary>",
  "priority": "<detected priority>",
  "conversationTranscript": "<full conversation history>"
}
```

### Step 4 — Call Notification Flow
Trigger Teams notification to the IT queue so an agent picks up immediately.

### Step 5 — Confirm to User
```
Bot: "You're all set! Here's what happens next:

      ✅ Case #<caseNumber> has been created
      👤 A support agent will reach out to you shortly
      📧 You'll receive a confirmation email at <userEmail>
      ⏱️ Expected response: <SLA timeframe based on priority>

      Is there anything else I can help with in the meantime?"
```

---

## Handoff Data Package
The following is passed to the agent at handoff:

```
=== BOT ESCALATION SUMMARY ===
Case #: <caseNumber>
Time: <timestamp>
User: <userName> (<userEmail>)
Department: <from Entra ID>
Priority: <priority>
Category: <category>

Issue Summary:
<escalationSummary>

--- CONVERSATION TRANSCRIPT ---
<full turn-by-turn conversation>

--- ATTEMPTED RESOLUTIONS ---
<list of knowledge articles shown, if any>
<GPT-4 answers provided, if any>
```

---

## Post-Escalation Behavior

After escalation:
- Bot remains available for unrelated new queries
- Bot does NOT attempt to re-resolve the escalated issue
- If user asks about case status → bot retrieves from D365 via Dataverse connector and reports back
- Bot does NOT transfer conversation to agent in real-time (async handoff via D365 case)

---

## Edge Cases

| Scenario | Handling |
|---|---|
| User escalates before any bot interaction | Create ticket with minimal info, agent contacts user to gather details |
| Entra ID auth fails — user identity unknown | Prompt for name and email manually before creating ticket |
| D365 ticket creation fails | Inform user of failure, provide IT helpdesk direct email as fallback |
| User escalates outside business hours | Notify user of business hours, confirm ticket created, set SLA start to next business day |
