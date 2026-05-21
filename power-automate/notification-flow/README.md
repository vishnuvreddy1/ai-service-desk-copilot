# Power Automate Flow — Notifications & SLA Alerts

## Overview
This flow handles all outbound notifications: Teams messages to agents on new case assignment, SLA breach warnings, and email confirmations to end users.

## Trigger
**Type:** Dataverse row created/modified (on Cases table)

**Conditions:**
- Triggers on case creation OR status change
- Also triggered by Copilot Studio HTTP action for real-time Teams pings

---

## Flow A — New Case Assignment Notification

### Trigger
Dataverse: When a row is added (Incidents / Cases table)

### Steps

#### Step 1 — Get Queue Details
Retrieve the queue the case was assigned to via the queue item relationship.

#### Step 2 — Get Agent from Queue
Look up the currently on-duty agent for the queue using round-robin assignment logic stored in Dataverse.

#### Step 3 — Send Teams Message to Agent
```
Action: Post message in a chat or channel (Teams)
Post as: Flow bot
Post in: Chat with <agentEmail>

Message:
🎫 New Case Assigned to You

Case #: <CaseNumber>
Priority: <Priority>
Category: <Category>
User: <CustomerName> (<CustomerEmail>)
Summary: <Description>

⏱️ SLA: First response required by <SLA deadline>

[View in Dynamics 365] → <D365 case URL>
```

#### Step 4 — Send Confirmation Email to End User
```
Action: Send an email (Outlook)
To: <CustomerEmail>
Subject: Your support request has been received — Case #<CaseNumber>

Body:
Hi <CustomerName>,

Your support request has been logged and assigned to our team.

Case Number: <CaseNumber>
Category: <Category>
Priority: <Priority>
Expected Response: <SLA first response deadline>

You can track your case status here: <Portal URL>

If this is urgent, reply to this email with URGENT in the subject line.

Best regards,
IT Service Desk
```

---

## Flow B — SLA Breach Warning

### Trigger
Recurrence: Every 15 minutes

### Steps

#### Step 1 — Query Approaching SLA Breaches
```
Action: List rows (Dataverse)
Table: Cases (incidents)
Filter: 
  statecode eq 0 (Active)
  AND slainvokedid ne null (SLA active)
  AND new_sla_first_response_deadline lt addMinutes(utcNow(), 30)
  AND new_first_response_sent eq false
```

#### Step 2 — For Each At-Risk Case
Apply to each case returned:

##### 2a — Send Teams Alert to Agent
```
🚨 SLA WARNING

Case #<CaseNumber> is approaching its first response deadline.
Deadline: <deadline> (<X minutes remaining>)
Customer: <CustomerName>
Priority: <Priority>

[Open Case] → <D365 URL>
```

##### 2b — Send Channel Alert for P1/P2
If priority is P1 or P2, also post to the `#it-escalations` Teams channel.

##### 2c — Update Case with Warning Logged
Add a note to the case timeline: "SLA warning notification sent at <timestamp>"

---

## Flow C — Escalation Notification (from Bot)

### Trigger
HTTP Request from Copilot Studio (on escalation)

### Input
```json
{
  "caseId": "string",
  "caseNumber": "string",
  "customerName": "string",
  "customerEmail": "string",
  "category": "string",
  "priority": "string",
  "transcript": "string"
}
```

### Steps

#### Step 1 — Post to Teams Escalation Channel
```
Action: Post message in a chat or channel
Channel: #it-tier1-queue

🤖 Bot Escalation — New Case

Case #: <caseNumber>
Priority: <priority>
Customer: <customerName>
Category: <category>

The AI assistant was unable to resolve this request and has created a case.

[View Case in D365] → <D365 URL>
[View Conversation Transcript] → (attached)
```

#### Step 2 — Attach Transcript
Post the conversation transcript as a follow-up message in the same Teams thread.

#### Step 3 — Confirm to Bot
Return HTTP 200 with `{ "status": "notified" }` so Copilot Studio can confirm to the user.

---

## Error Handling
- Teams API failure → fall back to email notification
- Email delivery failure → log to App Insights, retry after 5 minutes
- Dataverse query failure → alert IT admin via hardcoded email
