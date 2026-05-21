# 🤖 AI Service Desk Copilot
### Copilot Studio + Azure OpenAI GPT-4 + Dynamics 365 Customer Service

An enterprise-grade AI chatbot that automates Tier-1 internal support intake using Microsoft Copilot Studio and Azure OpenAI GPT-4. Deployed at scale to deflect 40% of service desk ticket volume and reduce average handle time by 12 minutes per ticket.

---

## 📌 Overview

Traditional service desks waste analyst capacity on repetitive, low-complexity tickets. This solution uses Copilot Studio to build an intelligent conversational agent that:

- Triages incoming support tickets using NLP
- Searches a GPT-4-backed knowledge base for instant answers
- Automatically routes escalations to the right team
- Logs structured data back into Dynamics 365 Customer Service
- Handles fallback and escalation paths gracefully

---

## 🏗️ Architecture

```
User (Teams / Web Chat)
        │
        ▼
 Copilot Studio Agent
  ├── Topics & Triggers
  ├── Conversation Flows
  ├── Fallback Handling
  └── Escalation Paths
        │
        ▼
 Power Automate Flows
  ├── Ticket Creation in D365
  ├── Knowledge Base Search (Azure OpenAI GPT-4)
  ├── Assignment Routing Logic
  └── Notification & SLA Alerts
        │
        ▼
 Dataverse + Dynamics 365 Customer Service
  ├── Case Records
  ├── SLA Tracking
  └── Queue Management
        │
        ▼
 Microsoft Entra ID (RBAC)
  └── 200+ enterprise users, least-privilege access
```

---

## 🔧 Tech Stack

| Layer | Technology |
|---|---|
| Conversational AI | Microsoft Copilot Studio (Power Virtual Agents) |
| Language Model | Azure OpenAI GPT-4 |
| Automation | Power Automate |
| Data Layer | Microsoft Dataverse |
| CRM | Dynamics 365 Customer Service |
| Identity & Security | Microsoft Entra ID (RBAC) |
| Monitoring | Azure Application Insights |

---

## 📈 Results

| Metric | Result |
|---|---|
| Tier-1 ticket deflection | **40%** |
| First-contact resolution improvement | **+18%** |
| Average handle time reduction | **−12 minutes/ticket** |
| Security review findings | **Zero remediations** |
| Enterprise users secured | **200+** |

---

## 🚀 Key Features

### 1. NLP-Driven Ticket Triage
Copilot Studio uses intent recognition to classify incoming requests — password resets, access requests, software issues, hardware faults — and routes them to the correct resolution path automatically.

### 2. GPT-4 Knowledge Search
Azure OpenAI GPT-4 is connected via Power Automate to search internal knowledge bases and return accurate, context-aware answers in natural language — eliminating the need for agents on common queries.

### 3. Automated D365 Case Creation
On escalation or unresolved queries, the bot auto-creates a structured case in Dynamics 365 with pre-filled fields: category, priority, affected user, and conversation summary.

### 4. Escalation & Fallback Handling
Built-in fallback topics detect low-confidence responses and gracefully hand off to a live agent, preserving full conversation context in the D365 case record.

### 5. Enterprise Security (Entra ID)
All bot interactions are authenticated via Microsoft Entra ID. Role-based access control ensures users only see and action items within their permission scope.

---

## 📁 Project Structure

```
ai-service-desk-copilot/
├── copilot-studio/
│   ├── topics/                  # Conversation topic definitions
│   ├── escalation-flows/        # Escalation path configs
│   └── fallback-handling/       # Fallback topic design
├── power-automate/
│   ├── ticket-creation-flow/    # D365 case creation
│   ├── knowledge-search-flow/   # Azure OpenAI GPT-4 integration
│   └── notification-flow/       # SLA alerts & routing
├── dataverse/
│   ├── entity-schema/           # Custom entity definitions
│   └── security-roles/          # Role & permission configs
├── docs/
│   ├── architecture-diagram.png
│   ├── solution-design.md
│   └── deployment-guide.md
└── README.md
```

---

## ⚙️ Setup & Deployment

### Prerequisites
- Microsoft 365 tenant with Power Platform license
- Azure subscription with Azure OpenAI access
- Dynamics 365 Customer Service environment
- Microsoft Entra ID configured

### Steps

1. **Import Copilot Studio Solution**
   - Navigate to [make.powerapps.com](https://make.powerapps.com)
   - Import the managed solution from `/copilot-studio/`

2. **Configure Azure OpenAI Connection**
   - Deploy a GPT-4 model in Azure OpenAI Studio
   - Add endpoint & API key to Power Automate connection
   - Update the knowledge search flow with your deployment name

3. **Set Up Power Automate Flows**
   - Import flows from `/power-automate/`
   - Update Dataverse environment references
   - Configure D365 connection credentials

4. **Apply Dataverse Schema**
   - Import entity customizations from `/dataverse/entity-schema/`
   - Apply security roles from `/dataverse/security-roles/`

5. **Configure Entra ID Authentication**
   - Register app in Entra ID
   - Assign roles to user groups
   - Link authentication to Copilot Studio channel

6. **Publish & Test**
   - Publish the Copilot Studio agent
   - Run end-to-end test scenarios
   - Monitor via Azure Application Insights

---

## 📄 Documentation

- [Solution Design](docs/solution-design.md)
- [Deployment Guide](docs/deployment-guide.md)
- [Architecture Diagram](docs/architecture-diagram.png)

---

## 👤 Author

**Vishnu Vardhan Reddy V**  
Senior Dynamics 365 & Power Platform Developer  
[LinkedIn](https://linkedin.com/in/vishnu-reddy-v-403b2934b) | [Email](mailto:vishnuvreddy1@outlook.com)

---

## 📜 License

MIT License — feel free to use, adapt, and build on this.
