# Deployment Guide — AI Service Desk Copilot

## Prerequisites

Before deploying, ensure you have:

| Requirement | Details |
|---|---|
| Microsoft 365 Tenant | With Power Platform license (Premium recommended) |
| Azure Subscription | With Azure OpenAI access approved |
| Dynamics 365 License | Customer Service Enterprise or above |
| Entra ID Access | Global Admin or Application Admin role |
| Power Platform Admin | Environment Admin rights |

---

## Environment Strategy

Deploy across three environments following ALM best practices:

```
DEV  →  TEST  →  PROD
```

- **DEV**: Active development, unmanaged solutions
- **TEST**: UAT and integration testing, managed solutions
- **PROD**: Live environment, managed solutions only

---

## Step 1 — Azure OpenAI Setup

### 1.1 Deploy GPT-4 Model
1. Navigate to [Azure OpenAI Studio](https://oai.azure.com)
2. Create a new deployment → select `gpt-4`
3. Set deployment name: `copilot-gpt4-prod`
4. Note the **Endpoint URL** and **API Key**

### 1.2 Store Credentials in Key Vault
```bash
# Create Key Vault
az keyvault create --name kv-copilot-prod --resource-group rg-copilot

# Store OpenAI key
az keyvault secret set --vault-name kv-copilot-prod \
  --name "openai-api-key" \
  --value "<your-api-key>"
```

### 1.3 Configure Knowledge Base
1. Upload internal knowledge articles as `.txt` or `.pdf` to Azure Blob Storage
2. Create an Azure AI Search index pointing to the blob container
3. Connect AI Search to OpenAI via the "Add your data" feature in Azure OpenAI Studio

---

## Step 2 — Dataverse & D365 Configuration

### 2.1 Import Entity Schema
1. Go to [make.powerapps.com](https://make.powerapps.com) → select DEV environment
2. **Solutions → Import** → upload `dataverse/entity-schema/CustomEntities_managed.zip`
3. Verify the following custom tables exist after import:
   - `cr_botinteraction` — stores conversation logs
   - `cr_knowledgearticle_ext` — extended knowledge article metadata
   - `cr_escalationlog` — escalation audit trail

### 2.2 Apply Security Roles
1. Go to **Settings → Security → Security Roles**
2. Import roles from `dataverse/security-roles/`
   - `CopilotAgent_Role.xml` — for service desk agents
   - `CopilotAdmin_Role.xml` — for administrators
   - `CopilotEndUser_Role.xml` — for end users (read-only own records)
3. Assign roles to Entra ID security groups (see Step 4)

### 2.3 Configure D365 Customer Service
1. Enable **Case Management** in Customer Service Hub
2. Create Queues: `IT-Tier1-Bot`, `IT-Access`, `IT-Hardware`, `IT-Software`, `IT-Escalated`
3. Configure SLA KPIs per the table in `docs/solution-design.md`
4. Set up routing rules to map case categories to queues

---

## Step 3 — Power Automate Flows

### 3.1 Import Flows
1. Navigate to [make.powerautomate.com](https://make.powerautomate.com)
2. Import each flow from the `power-automate/` directory:

| Flow | File | Purpose |
|---|---|---|
| Ticket Creation | `ticket-creation-flow/flow.zip` | Creates D365 case from bot conversation |
| Knowledge Search | `knowledge-search-flow/flow.zip` | Calls Azure OpenAI GPT-4 |
| Notification | `notification-flow/flow.zip` | Sends Teams/email alerts |

### 3.2 Update Connection References
After import, update connection references for each flow:
- **Dataverse** → connect with service account credentials
- **Dynamics 365** → connect to your D365 org URL
- **HTTP (Azure OpenAI)** → update endpoint URL from Step 1.1
- **Teams** → connect with bot service account

### 3.3 Configure Knowledge Search Flow Variables
Open `knowledge-search-flow` and update these variables:

```
OPENAI_ENDPOINT     = https://<your-resource>.openai.azure.com/
OPENAI_DEPLOYMENT   = copilot-gpt4-prod
OPENAI_API_VERSION  = 2024-02-15-preview
SEARCH_ENDPOINT     = https://<your-search>.search.windows.net
SEARCH_INDEX        = knowledge-articles
```

---

## Step 4 — Entra ID Configuration

### 4.1 Register Application
1. Go to **Entra ID → App Registrations → New Registration**
2. Name: `AI-ServiceDesk-Copilot`
3. Supported account types: **Single tenant**
4. Note the **Application (client) ID** and **Directory (tenant) ID**

### 4.2 Configure API Permissions
Add the following delegated permissions:
- `User.Read` (Microsoft Graph)
- `offline_access`
- Dynamics 365 → `user_impersonation`

### 4.3 Create Security Groups
Create these groups in Entra ID and assign members:

| Group Name | Role Assigned |
|---|---|
| `SG-CopilotAgents` | CopilotAgent_Role |
| `SG-CopilotAdmins` | CopilotAdmin_Role |
| `SG-CopilotEndUsers` | CopilotEndUser_Role |

---

## Step 5 — Copilot Studio Setup

### 5.1 Import Solution
1. Go to [copilotstudio.microsoft.com](https://copilotstudio.microsoft.com)
2. Import the solution from `copilot-studio/CopilotSolution_managed.zip`
3. Verify all topics imported: Password Reset, Access Request, Software Install, Hardware Issue, Fallback

### 5.2 Configure Authentication
1. Open the bot → **Settings → Security → Authentication**
2. Select **Azure Active Directory v2**
3. Enter the **Client ID** and **Tenant ID** from Step 4.1
4. Set token exchange URL to your Power Automate HTTP trigger endpoint

### 5.3 Connect Power Automate Actions
For each topic that calls Power Automate:
1. Open the topic in the authoring canvas
2. Select the **Call an action** node
3. Re-link to the imported flow (connections reset on import)

### 5.4 Publish the Bot
1. Click **Publish** in Copilot Studio
2. Add channels:
   - **Microsoft Teams** → follow the Teams app manifest setup
   - **Custom Website** → copy the embed script for your internal portal

---

## Step 6 — ALM & CI/CD (Azure DevOps)

### 6.1 Export Solution for Pipeline
```yaml
# azure-pipelines.yml
trigger:
  branches:
    include: [main]

stages:
  - stage: ExportDev
    jobs:
      - job: ExportSolution
        steps:
          - task: PowerPlatformExportSolution@2
            inputs:
              authenticationType: PowerPlatformSPN
              PowerPlatformSPN: $(DEV_CONNECTION)
              SolutionName: AICopilotServiceDesk
              SolutionOutputFile: $(Build.ArtifactStagingDirectory)/solution.zip
              Managed: false

  - stage: DeployTest
    dependsOn: ExportDev
    jobs:
      - job: ImportToTest
        steps:
          - task: PowerPlatformImportSolution@2
            inputs:
              authenticationType: PowerPlatformSPN
              PowerPlatformSPN: $(TEST_CONNECTION)
              SolutionInputFile: $(Pipeline.Workspace)/solution.zip
```

### 6.2 Environment Variables
Store these as pipeline secrets in Azure DevOps:
- `DEV_CONNECTION` — Service principal for DEV environment
- `TEST_CONNECTION` — Service principal for TEST environment
- `PROD_CONNECTION` — Service principal for PROD environment

---

## Step 7 — Testing & Validation

### 7.1 Test Scenarios

| Scenario | Expected Outcome |
|---|---|
| "I forgot my password" | Triggers Password Reset topic |
| "I need access to SharePoint" | Creates Access Request ticket in D365 |
| Unknown query | GPT-4 knowledge search returns answer |
| GPT-4 returns no answer | Graceful escalation to live agent |
| Unauthenticated user | Redirected to Entra ID login |

### 7.2 UAT Checklist
- [ ] All 5 topics trigger correctly on sample phrases
- [ ] D365 cases created with correct queue assignment
- [ ] SLA timers start on case creation
- [ ] GPT-4 search returns relevant answers
- [ ] Escalation passes full conversation transcript to agent
- [ ] Teams notifications fire on P1/P2 cases
- [ ] RBAC prevents unauthorized data access
- [ ] Flow run history shows no errors after 50 test interactions

---

## Monitoring

After go-live, monitor via:
- **Azure Application Insights** — bot telemetry and latency
- **Power Automate** — flow run success/failure rates
- **Copilot Studio Analytics** — topic trigger rates, escalation rates, CSAT
- **D365 Customer Service Hub** — SLA attainment, queue depth, resolution time
