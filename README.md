# Zoho CRM MCP Workflows

An n8n-based AI workflow for managing Zoho CRM leads through an MCP server and an AI agent.

The system allows users to create, update, retrieve, list, and delete Zoho CRM leads using natural-language requests. The AI agent also extracts structured lead information, validates required fields, generates follow-up tasks, and assigns an initial lead score.

---

## Architecture

The workflow currently contains three main responsibilities:

```text
User
 │
 ▼
Chat Trigger
 │
 ▼
AI Agent
 │
 ├── OpenAI Chat Model
 ├── Simple Memory
 └── MCP Client
          │
          │ MCP
          ▼
   MCP Server Trigger
          │
          ├── Create Lead
          ├── Update Lead
          ├── Get Lead
          ├── Get All Leads
          └── Delete Lead
                  │
                  ▼
              Zoho CRM


Errors
  │
  ▼
Error Trigger
  │
  ▼
Format Error Log
  │
  ▼
Save Error Log
```

---

## Features

- Create leads in Zoho CRM
- Update existing leads
- Retrieve a specific lead
- Retrieve all leads
- Delete leads
- Natural-language interaction through an AI Agent
- MCP-based communication between the AI Agent and Zoho CRM tools
- Conversation memory using Simple Memory
- Structured lead extraction and validation
- Required-field detection
- Follow-up task generation
- Initial lead scoring
- Centralized error logging

---

## Zoho CRM MCP Server

The MCP Server exposes Zoho CRM operations as tools that can be used by an MCP client.

Available tools:

| Tool | Description |
|---|---|
| Create a lead | Creates a new lead in Zoho CRM |
| Update a lead | Updates an existing lead |
| Get a lead | Retrieves a specific lead |
| Get All leads | Retrieves all available leads |
| Delete a lead | Deletes a lead |

The MCP server is exposed through the following local endpoint:

```text
http://localhost:5678/mcp/zoho-crm-mcp
```

---

## AI Agent

The AI Agent receives natural-language requests and decides which Zoho CRM operation should be performed.

For example:

```text
Create a lead for John Smith from ABC Company.
His email is john@example.com and his phone is 09123456789.
```

The agent can extract the relevant information and use the appropriate MCP tool to interact with Zoho CRM.

The agent is configured with:

- OpenAI Chat Model
- Simple Memory
- MCP Client Tool
- Structured Output Parser

The conversation memory uses the session ID when available.

---

## Structured Output

The AI Agent uses a Structured Output Parser to produce a predictable JSON structure.

The output contains five main sections:

```json
{
  "lead": {},
  "followUpTask": {},
  "validity": {},
  "missing_fields": [],
  "scoring": {}
}
```

### Lead

Contains the extracted customer information:

```json
{
  "fullName": "John Smith",
  "company": "ABC Company",
  "email": "john@example.com",
  "phone": "09123456789",
  "leadSource": "Website",
  "message_summary_en": "Customer is interested in the company's services."
}
```

The required lead fields are:

- `fullName`
- `email`
- `phone`
- `message_summary_en`

Email and phone are validated using the configured schema.

### Follow-up Task

The agent generates a suggested follow-up task:

```json
{
  "taskType": "call",
  "dueInDays": 2,
  "notes": "Contact the customer to discuss their requirements."
}
```

Supported task types:

```text
call
email
```

### Validity

The agent determines whether the required information is available:

```json
{
  "is_valid": true,
  "reasons": "All required lead fields are available."
}
```

### Missing Fields

Missing required information is returned as an array:

```json
{
  "missing_fields": [
    "email",
    "phone"
  ]
}
```

### Lead Scoring

The agent also generates an initial lead score:

```json
{
  "lead_score": 80,
  "lead_quality": "high"
}
```

The score ranges from `0` to `100`.

Supported quality levels:

```text
very_low
low
medium
high
very_high
```

---

## Error Handling

The workflow includes an error-handling flow:

```text
Error Trigger
     │
     ▼
Format Error Log
     │
     ▼
Save Error Log
     │
     ▼
Error Notification
```

The error log records information such as:

- Timestamp
- Workflow name and ID
- Execution ID
- Execution mode
- Failed node
- Error message
- Error name
- Error stack
- Retry information
- Last executed node

Error logs are currently written to:

```text
/tmp/n8n-errors/
```

---

## Requirements

- n8n
- Zoho CRM account
- Zoho CRM credentials configured in n8n
- OpenAI credentials configured in n8n
- An OpenAI chat model
- MCP Server and MCP Client support in n8n

---

## Setup

### 1. Import the workflow

Import the workflow JSON file into n8n.

### 2. Configure Zoho CRM credentials

Configure the Zoho CRM credentials used by the Zoho CRM tool nodes.

### 3. Configure OpenAI credentials

Configure the OpenAI credentials used by the AI Agent's chat model.

### 4. Verify the MCP endpoint

The MCP Server uses:

```text
http://localhost:5678/mcp/zoho-crm-mcp
```

Make sure the n8n instance is running and the MCP endpoint is accessible.

### 5. Configure the MCP Client

The MCP Client connects to the MCP Server using the endpoint above.

### 6. Configure Error Workflow

The workflow containing the `Error Trigger` should be configured as the Error Workflow for the workflows that need centralized error handling.

---

## Example Requests

### Create a lead

```text
Create a lead for Ali Ahmadi from ABC Company.
His email is ali@example.com and his phone is 09123456789.
```

### Get a lead

```text
Get the lead with ID 123456789.
```

### Update a lead

```text
Update lead 123456789 and change the company to XYZ Company.
```

### List leads

```text
Show me all leads in Zoho CRM.
```

### Delete a lead

```text
Delete lead 123456789.
```

The agent should handle destructive operations carefully and request confirmation when the user's intent is ambiguous.

---

## Workflow Structure

The current n8n workflow contains:

```text
Zoho-CRM-MCP-Workflows
│
├── MCP Server Trigger
│   ├── Create a lead in Zoho CRM
│   ├── Update a lead in Zoho CRM
│   ├── Get a lead in Zoho CRM
│   ├── Get All leads in Zoho CRM
│   └── Delete a lead in Zoho CRM
│
├── When chat message received
│   └── AI Agent
│       ├── OpenAI Chat Model
│       ├── Simple Memory
│       ├── MCP Client
│       └── Structured Output Parser
│
└── Error Handling
    ├── Error Trigger
    ├── Format Error Log
    ├── Save Error Log
    └── Error Notification
```

---

## Future Improvements

Possible improvements include:

- Separate the MCP Server, AI Agent, and Error Handler into independent workflows
- Add more Zoho CRM modules and operations
- Add stronger lead validation
- Connect lead scoring to explicit business rules or a trained model
- Add persistent logging and monitoring
- Add notifications for workflow failures
- Add automated tests for MCP tools and structured outputs
