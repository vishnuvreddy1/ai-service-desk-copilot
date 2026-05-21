# Copilot Studio — Fallback Handling Design

## Overview
Fallback handling ensures users never hit a dead end. When the bot cannot confidently interpret a user's message, a structured fallback strategy attempts recovery before escalating.

---

## Fallback Hierarchy

```
Level 1 → Clarification Prompt (attempt 1)
Level 2 → Clarification Prompt (attempt 2)
Level 3 → GPT-4 Knowledge Search (open-ended)
Level 4 → Graceful Escalation to Human Agent
```

---

## Level 1 — First Clarification Attempt

**Trigger:** Topic confidence score < 0.6 on initial message

```
Bot: "I want to make sure I help you with the right thing. 
      Could you tell me a bit more about what you need?"

[Show topic quick replies as hints:]
[🔐 Password / Login | 🔑 Access Request | 💻 Software | 🖥️ Hardware | ❓ Something Else]
```

**Logic:**
- If user selects a quick reply → route to that topic
- If user types a message → re-evaluate NLP, increment clarification counter

---

## Level 2 — Second Clarification Attempt

**Trigger:** Clarification counter = 1, confidence still < 0.6

```
Bot: "I'm having a little trouble understanding your request. 
      Can you try rephrasing it, or choose from one of these options?"

[Same quick replies as Level 1]
```

**Logic:**
- If user selects a quick reply → route to that topic
- If user types again → proceed to Level 3 regardless of confidence score

---

## Level 3 — GPT-4 Open Knowledge Search

**Trigger:** Clarification counter = 2 OR user message is long/complex

```
Bot: "Let me search our knowledge base for you..."
[Show typing indicator]

→ Call Knowledge Search Flow with full user message as query

→ If answer found (resolved = true):
    Bot: "Here's what I found:"
    Bot: "<GPT-4 answer>"
    Bot: "Did that help?"
    [Yes ✅ | No, I still need help 🙋]
    → Yes → close, log resolved
    → No → Level 4

→ If no answer (resolved = false):
    → Proceed directly to Level 4
```

---

## Level 4 — Graceful Human Escalation

**Trigger:** All recovery levels exhausted OR `resolved = false` from Level 3

```
Bot: "I wasn't able to find what you're looking for in our knowledge base. 
      Let me connect you with a support agent who can help directly.

      Would you like me to raise a support ticket for you?"

[Yes, please | No thanks]

→ Yes:
    Bot: "Can you briefly describe your issue for the agent?"
    [Free text]
    → Call Ticket Creation Flow
    → Call Notification Flow
    Bot: "Done! Case #<number> has been created. 
          An agent will reach out to you at <email> shortly."

→ No:
    Bot: "No problem! If you change your mind, just type 'help' 
          and I'll be here. You can also email the helpdesk at 
          itsupport@<company>.com"
```

---

## Fallback Logging

Every fallback event is logged to Dataverse (`cr_botinteraction`) with:

| Field | Value |
|---|---|
| `cr_fallbacklevel` | 1, 2, 3, or 4 |
| `cr_originalquery` | User's original message |
| `cr_resolved` | true / false |
| `cr_escalated` | true / false |
| `cr_createdon` | Timestamp |

This data feeds the Copilot Studio Analytics dashboard and Power BI reports to track:
- Most common unrecognized intents (used to create new topics)
- Fallback-to-escalation conversion rate
- GPT-4 knowledge search resolution rate

---

## Fallback Prevention Strategy

To minimize fallback triggers over time:
1. **Weekly review** of `cr_originalquery` values where `cr_fallbacklevel = 4`
2. **Add new trigger phrases** to existing topics based on patterns
3. **Expand knowledge base** for top unresolved GPT-4 queries
4. **Create new topics** for recurring unrecognized intents (threshold: 10+ instances/week)
