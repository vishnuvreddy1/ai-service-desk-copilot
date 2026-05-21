# Copilot Studio — Topic Definitions

## Overview
This document describes all conversation topics configured in the Copilot Studio agent, including trigger phrases, conversation flow logic, and action connections.

---

## Topic 1 — Password Reset

### Trigger Phrases
```
- reset my password
- forgot my password
- can't log in
- locked out of my account
- account locked
- password expired
- change my password
- reset password
```

### Conversation Flow

```
Bot: "I can help you reset your password. Are you locked out of your Windows account, 
      Microsoft 365, or a specific application?"

[Quick Replies: Windows Account | Microsoft 365 | Specific App]

→ Windows Account:
    Bot: "To reset your Windows password, you can use the self-service password reset 
          portal at aka.ms/sspr. Do you need me to walk you through the steps?"
    [Yes | No, I can manage]
    → Yes: Guide through SSPR steps
    → No: "Great! Let me know if you need anything else."

→ Microsoft 365:
    Bot: "For Microsoft 365, your password and Windows password are the same. 
          Please use aka.ms/sspr or contact your manager if you're fully locked out."

→ Specific App:
    Bot: "Which application are you having trouble with?"
    [Free text input]
    → Call Knowledge Search Flow with query = "password reset for <app name>"
    → If resolved: Show answer
    → If unresolved: Escalate to ticket
```

### Actions Called
- `Knowledge Search Flow` (for specific app queries)
- `Ticket Creation Flow` (on escalation)

---

## Topic 2 — Access Request

### Trigger Phrases
```
- need access to
- request access
- can't access
- permission denied
- access request
- add me to
- grant me access
- I don't have access to
```

### Conversation Flow

```
Bot: "I can help you request access. What do you need access to?"
[Free text input]

Bot: "What's your business justification for needing this access?"
[Free text input]

Bot: "Is this request urgent?"
[Yes — needed today | No — within a few days]

→ Create ticket with:
   Category: AccessRequest
   Priority: P2 (if urgent) / P3 (if not)
   Description: "Access request for <system> — Justification: <justification>"

Bot: "I've submitted your access request. 
      Case #<number> has been created and assigned to the IT Access team.
      Expected response: <SLA timeframe>"
```

### Actions Called
- `Ticket Creation Flow`
- `Notification Flow` (Teams alert to IT-Access queue)

---

## Topic 3 — Software Install

### Trigger Phrases
```
- install software
- need software
- download application
- setup program
- install Teams
- install Office
- need a tool
- software request
```

### Conversation Flow

```
Bot: "What software do you need installed?"
[Free text input]

→ Call Knowledge Search Flow with query = "how to install <software name>"

→ If resolved (self-install possible):
    Bot: "Here's how to install <software>: <answer>"
    Bot: "Were you able to install it successfully?"
    [Yes — resolved | No — still need help]
    → Yes: Close interaction
    → No: Create ticket

→ If unresolved (requires IT):
    Bot: "Installing <software> requires IT approval and remote assistance. 
          Let me raise a ticket for you."
    → Create ticket (Category: SoftwareInstall, Priority: P3)
```

### Actions Called
- `Knowledge Search Flow`
- `Ticket Creation Flow` (conditional)

---

## Topic 4 — Hardware Issue

### Trigger Phrases
```
- laptop not working
- computer broken
- monitor issue
- keyboard not working
- mouse problem
- printer issue
- hardware problem
- my screen
- device issue
```

### Conversation Flow

```
Bot: "I'm sorry to hear you're having a hardware issue. What device is affected?"
[Quick Replies: Laptop/Desktop | Monitor | Keyboard/Mouse | Printer | Other]

Bot: "Can you describe what's happening?"
[Free text input]

Bot: "Have you tried restarting the device?"
[Yes, still broken | No]

→ No: Guide through restart
→ Yes, still broken:
    → Call Knowledge Search for quick fixes
    → If unresolved: Create ticket (Category: HardwareIssue, Priority: P2)
    
Bot: "I've raised a hardware support ticket.
      Case #<number> — a technician will contact you within <SLA timeframe>."
```

### Actions Called
- `Knowledge Search Flow`
- `Ticket Creation Flow`

---

## Topic 5 — General Query (Fallback)

### Trigger
- System fallback when no other topic matches
- Confidence score below threshold (< 0.6)

### Conversation Flow

```
Bot: "I'm not sure I understood that. Let me search our knowledge base for you."

→ Call Knowledge Search Flow with userQuery = full user message

→ If resolved:
    Bot: "<GPT-4 answer>"
    Bot: "Did that answer your question?"
    [Yes | No, I need more help]
    → Yes: Close
    → No: 
        Bot: "Let me connect you with a support agent."
        → Create ticket (Category: General, Priority: P3)

→ If unresolved:
    Bot: "I wasn't able to find an answer in our knowledge base. 
          Would you like me to raise a support ticket?"
    [Yes | No]
    → Yes: Create ticket
    → No: "No problem! Feel free to come back if you need anything."
```

### Actions Called
- `Knowledge Search Flow`
- `Ticket Creation Flow` (conditional)

---

## Escalation Topic (Global)

### Trigger
- User says: "agent", "human", "speak to someone", "real person", "escalate"
- Or after 2 failed resolution attempts in any topic

### Flow
```
Bot: "I'll connect you with a support agent right away. 
      Can you briefly describe your issue so I can give them context?"
[Free text input]

→ Create ticket with full conversation transcript
→ Notify IT queue via Teams
→ Bot: "Done! A support agent will reach out shortly. 
         Your case number is #<number>."
```
