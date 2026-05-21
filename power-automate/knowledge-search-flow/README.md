# Power Automate Flow — Knowledge Search (Azure OpenAI GPT-4)

## Overview
This flow is triggered by Copilot Studio when a user's query requires a knowledge base search. It calls Azure OpenAI GPT-4 with retrieval-augmented generation (RAG) using Azure AI Search as the knowledge source.

## Trigger
**Type:** HTTP Request (from Copilot Studio action)

**Input Schema:**
```json
{
  "type": "object",
  "properties": {
    "userQuery": { "type": "string", "description": "The user's question or issue description" },
    "category": { "type": "string", "description": "Topic category for scoping the search" },
    "conversationHistory": {
      "type": "array",
      "description": "Last 5 turns of conversation for context",
      "items": {
        "type": "object",
        "properties": {
          "role": { "type": "string" },
          "content": { "type": "string" }
        }
      }
    }
  },
  "required": ["userQuery"]
}
```

## Flow Steps

### Step 1 — Initialize Variables
```
searchEndpoint    = https://<your-search>.search.windows.net
searchIndex       = knowledge-articles
openAiEndpoint    = https://<your-resource>.openai.azure.com/
openAiDeployment  = copilot-gpt4-prod
apiVersion        = 2024-02-15-preview
maxTokens         = 800
temperature       = 0.2
```

### Step 2 — Search Knowledge Base (Azure AI Search)
```
Action: HTTP
Method: POST
URI: @{variables('searchEndpoint')}/indexes/@{variables('searchIndex')}/docs/search?api-version=2023-11-01

Headers:
  Content-Type: application/json
  api-key: @{parameters('AzureSearchApiKey')}

Body:
{
  "search": "@{triggerBody()?['userQuery']}",
  "queryType": "semantic",
  "semanticConfiguration": "default",
  "top": 3,
  "select": "title,content,category",
  "filter": "category eq '@{triggerBody()?['category']}'"
}
```

### Step 3 — Extract Search Results
Parse the AI Search response and extract the top 3 knowledge article chunks into a formatted context string:

```
Apply to each (value in search results):
  Append to string variable:
    "Article: <title>\n<content>\n\n"
```

### Step 4 — Call Azure OpenAI GPT-4
```
Action: HTTP
Method: POST
URI: @{variables('openAiEndpoint')}/openai/deployments/@{variables('openAiDeployment')}/chat/completions?api-version=@{variables('apiVersion')}

Headers:
  Content-Type: application/json
  api-key: @{parameters('OpenAiApiKey')}

Body:
{
  "messages": [
    {
      "role": "system",
      "content": "You are a helpful IT support assistant. Answer the user's question using ONLY the knowledge articles provided. If the answer is not in the articles, say 'I couldn't find an answer in our knowledge base' — do not make up information. Be concise and clear."
    },
    {
      "role": "user",
      "content": "Knowledge Articles:\n\n<context from Step 3>\n\nUser Question: <userQuery>"
    }
  ],
  "max_tokens": 800,
  "temperature": 0.2
}
```

### Step 5 — Parse GPT-4 Response
Extract the answer text from `choices[0].message.content`.

### Step 6 — Confidence Check
If the response contains "couldn't find an answer" → set `resolved = false`
Otherwise → set `resolved = true`

### Step 7 — Log Interaction
```
Action: Add a new row (Dataverse)
Table: cr_botinteraction

Fields:
  cr_query      = <userQuery>
  cr_response   = <GPT-4 answer>
  cr_resolved   = <resolved flag>
  cr_createdon  = utcNow()
```

### Step 8 — Return Response
```json
{
  "answer": "<GPT-4 response text>",
  "resolved": true,
  "sourcesUsed": ["<article title 1>", "<article title 2>"]
}
```

## Prompt Engineering Notes

The system prompt is designed to:
- Scope GPT-4 strictly to provided knowledge articles (prevents hallucination)
- Use low temperature (0.2) for factual, consistent responses
- Return a clear "not found" signal to trigger escalation when needed

## Error Handling
- Azure AI Search timeout (>10s) → return `resolved: false` to trigger escalation
- OpenAI API rate limit → retry with exponential backoff (3 attempts)
- OpenAI content filter triggered → return safe fallback message, log to App Insights

## Security Notes
- API keys stored in Azure Key Vault, retrieved via managed identity at runtime
- No user PII sent to OpenAI — only the query text and knowledge article content
- All interactions logged to Dataverse for audit purposes
