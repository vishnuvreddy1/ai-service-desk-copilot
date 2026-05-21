# Dataverse — Security Roles

## Overview
Three custom security roles control access to the AI Service Desk Copilot solution. All roles are scoped to the **Business Unit** level and mapped to Entra ID security groups for automated assignment.

---

## Role 1 — Copilot End User (`CopilotEndUser_Role`)

**Mapped Entra ID Group:** `SG-CopilotEndUsers`  
**Assigned to:** All employees who interact with the bot

### Permissions

| Table | Create | Read | Write | Delete | Append | Append To |
|---|---|---|---|---|---|---|
| `incident` (Case) | ✅ (own) | ✅ (own) | ❌ | ❌ | ❌ | ❌ |
| `cr_botinteraction` | ✅ (own) | ✅ (own) | ❌ | ❌ | ❌ | ❌ |
| `knowledgearticle` | ❌ | ✅ (org) | ❌ | ❌ | ❌ | ❌ |
| `contact` | ❌ | ✅ (own) | ❌ | ❌ | ❌ | ❌ |
| `cr_escalationlog` | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |

**Miscellaneous Privileges:**
- Can read their own case status via Power Pages portal
- Cannot access Dynamics 365 model-driven app directly
- Cannot see other users' cases, contacts, or bot interactions

---

## Role 2 — Copilot Agent (`CopilotAgent_Role`)

**Mapped Entra ID Group:** `SG-CopilotAgents`  
**Assigned to:** IT service desk agents handling escalated cases

### Permissions

| Table | Create | Read | Write | Delete | Append | Append To |
|---|---|---|---|---|---|---|
| `incident` (Case) | ✅ (BU) | ✅ (BU) | ✅ (BU) | ❌ | ✅ | ✅ |
| `cr_botinteraction` | ❌ | ✅ (BU) | ❌ | ❌ | ❌ | ❌ |
| `cr_escalationlog` | ❌ | ✅ (BU) | ✅ (BU) | ❌ | ✅ | ✅ |
| `knowledgearticle` | ✅ | ✅ (org) | ✅ (own) | ❌ | ✅ | ✅ |
| `contact` | ✅ | ✅ (BU) | ✅ (BU) | ❌ | ✅ | ✅ |
| `queue` | ❌ | ✅ (BU) | ❌ | ❌ | ❌ | ❌ |
| `queueitem` | ✅ | ✅ (BU) | ✅ | ❌ | ✅ | ✅ |
| `sla` | ❌ | ✅ (org) | ❌ | ❌ | ❌ | ❌ |
| `slakpiinstance` | ❌ | ✅ (BU) | ❌ | ❌ | ❌ | ❌ |
| `cr_knowledgearticle_ext` | ❌ | ✅ (org) | ❌ | ❌ | ❌ | ❌ |

**Miscellaneous Privileges:**
- Access to Customer Service Hub model-driven app
- Can view conversation transcripts in escalation log
- Can resolve and close cases
- Cannot delete cases (supervisor-only action)
- Cannot modify SLA definitions or security roles

---

## Role 3 — Copilot Admin (`CopilotAdmin_Role`)

**Mapped Entra ID Group:** `SG-CopilotAdmins`  
**Assigned to:** IT Platform team and Power Platform admins

### Permissions

| Table | Create | Read | Write | Delete | Append | Append To |
|---|---|---|---|---|---|---|
| All custom tables | ✅ (org) | ✅ (org) | ✅ (org) | ✅ (org) | ✅ | ✅ |
| `incident` | ✅ (org) | ✅ (org) | ✅ (org) | ✅ (org) | ✅ | ✅ |
| `knowledgearticle` | ✅ | ✅ (org) | ✅ (org) | ✅ (org) | ✅ | ✅ |
| `queue` | ✅ | ✅ (org) | ✅ (org) | ✅ | ✅ | ✅ |
| `sla` | ✅ | ✅ (org) | ✅ (org) | ✅ | ✅ | ✅ |
| `systemuser` | ❌ | ✅ (org) | ❌ | ❌ | ✅ | ✅ |
| `businessunit` | ❌ | ✅ (org) | ❌ | ❌ | ❌ | ❌ |

**Miscellaneous Privileges:**
- Full access to Power Platform admin center
- Can publish/update Copilot Studio solutions
- Can modify Power Automate flows
- Can manage Dataverse entity schema
- Can assign and revoke security roles
- Can view all analytics and audit logs

---

## Field-Level Security

The following sensitive fields use Field Security Profiles to restrict access:

| Table | Field | Restriction |
|---|---|---|
| `cr_botinteraction` | `cr_transcript` | Agents + Admins only (not End Users) |
| `cr_escalationlog` | `cr_transcript` | Agents + Admins only |
| `contact` | `emailaddress1` | Agents + Admins only |
| `incident` | `new_transcript` | Agents + Admins only |

---

## Row-Level Security (Business Unit Scoping)

Cases, bot interactions, and escalation logs are scoped to the **agent's business unit**. This ensures:
- APAC agents see APAC tickets only
- EMEA agents see EMEA tickets only
- Global admins have cross-BU visibility

BU assignment is inherited from the user's Entra ID department attribute, synced to Dataverse via Azure AD group membership.

---

## DLP Policy Alignment

This solution complies with the corporate Power Platform DLP policy:

| Connector Group | Connectors |
|---|---|
| Business (allowed) | Dataverse, Dynamics 365, SharePoint, Teams, Outlook, Azure OpenAI |
| Non-Business (blocked) | All consumer connectors (Twitter/X, Gmail, Dropbox, etc.) |
| Blocked | No connectors in blocked list used by this solution |

DLP policy enforced at the **environment level** — no tenant-wide policy override required.

---

## Audit Logging

All Create, Update, Delete operations on the following tables are captured in Dataverse audit history:
- `incident`
- `cr_botinteraction`
- `cr_escalationlog`
- `cr_knowledgearticle_ext`

Audit logs retained for **90 days** per corporate data retention policy.
