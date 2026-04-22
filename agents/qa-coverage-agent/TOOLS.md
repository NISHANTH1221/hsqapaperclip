# TOOLS.md — CEO Available Tools

## Paperclip API (via Paperclip skill)

Your primary tool for all coordination. Use for:

| Action               | Endpoint                                                                                               |
| -------------------- | ------------------------------------------------------------------------------------------------------ |
| Get your identity    | `GET /api/agents/me`                                                                                   |
| List assigned issues | `GET /api/companies/{companyId}/issues?assigneeAgentId={id}&status=todo,in_progress,in_review,blocked` |
| Checkout an issue    | `POST /api/issues/{id}/checkout`                                                                       |
| Create a subtask     | `POST /api/companies/{companyId}/issues`                                                               |
| Comment on an issue  | `POST /api/issues/{id}/comments`                                                                       |
| Update issue status  | `PATCH /api/issues/{id}`                                                                               |
| Assign to an agent   | `PATCH /api/issues/{id}` with `assigneeAgentId`                                                        |

Always include `X-Paperclip-Run-Id` header on all mutating calls.

***

## Deepwiki MCP

Use for querying the Hyperswitch feature matrix and understanding the codebase:

* Query feature matrix API to validate scope (used by Validation Agent — you may use it too for quick checks)
* Ask questions about the Hyperswitch repo structure, API flows, and connector capabilities

***

## Memory / PARA Files

Use for persisting context across runs:

* `./memory/YYYY-MM-DD.md` — daily plan and timeline
* `./life/` — durable facts about agents, connectors, recurring issues

***

## Agent IDs (for delegation)

| Agent                     | ID                                     |
| ------------------------- | -------------------------------------- |
| Validation Agent          | `cfcef29d-2f7e-434b-96c6-3462aa10e9d2` |
| API Testing Agent         | `5180f40c-eddb-46d7-9730-5742cc9d8d96` |
| Cypress Feasibility Agent | `a7bae843-560b-43ab-91bc-e6f47f7e3459` |
| Test Generation Agent     | `bc10cd26-73d5-40d3-bfc8-a8db9dd0307e` |
| Runner Agent              | `afd0a7f6-b95a-477d-bda8-a17a1c3ce240` |

***

## Fixed Environment (pass to Runner Agent)

```sh
CYPRESS_ADMINAPIKEY=test_admin
CYPRESS_BASEURL=http://hyperswitch-hyperswitch-server-1:8080
CYPRESS_CONNECTOR_AUTH_FILE_PATH=/workspace/creds.json
CYPRESS_HS_EMAIL=sk.sakil+8@juspay.in
CYPRESS_HS_PASSWORD=6rxg7DUuCVEc!Aq
```

***

## What You Do NOT Use

* You do not run bash commands yourself
* You do not read or write spec files yourself
* You do not call the Hyperswitch API yourself
* All execution is delegated to the 5 pipeline agents