# HEARTBEAT.md — CEO Heartbeat Checklist

Run this checklist on every heartbeat, in order. Do not skip steps.

---

## 1. Identity and Context

- `GET /api/agents/me` — confirm your id, role, budget, chainOfCommand.
- Check wake context: `PAPERCLIP_TASK_ID`, `PAPERCLIP_WAKE_REASON`, `PAPERCLIP_WAKE_COMMENT_ID`.

---

## 2. Local Planning Check

1. Read today's plan from `./memory/YYYY-MM-DD.md` under "## Today's Plan".
2. Review each planned item: what's completed, what's blocked, what's next.
3. For any blockers, resolve yourself or escalate.
4. If ahead, pull the next highest priority item.
5. Record progress updates in the daily notes.

---

## 3. Get Assignments

- `GET /api/companies/{companyId}/issues?assigneeAgentId={your-id}&status=todo,in_progress,in_review,blocked`
- Priority order: `in_progress` first → `in_review` (if woken by comment) → `todo`. Skip `blocked` unless you can unblock it.
- If `PAPERCLIP_TASK_ID` is set and assigned to you, prioritize that task.
- If an active run is already on an `in_progress` task, skip it and move on.

---

## 4. Checkout and Start Work

- Call `POST /api/issues/{id}/checkout` only when you intentionally switch tasks or the wake context did not already claim the issue.
- Never retry a 409 — that task belongs to another agent.
- Read the issue title, description, and all comments in full before doing anything.

---

## 5. QA Pipeline — Triage the Task

When the assigned issue is a QA task (payment flow, connector, bug, feature), follow this decision tree:

**Is this a new QA task (todo)?**
→ Begin the pipeline at Process 1. Assign to Validation Agent. See AGENTS.md for full pipeline instructions.

**Is this a task blocked at a pipeline step (blocked)?**
→ Read the block reason in the comments. Decide: can you unblock it (missing creds, wrong connector, API down)? If yes, fix and reassign. If no, comment with the blocker and await human input.

**Is this a task awaiting your review (in_review)?**
→ Read the agent's output comment. Decide: approve and advance to next pipeline step, or send back with corrections.

**Is this a task in progress (in_progress)?**
→ Check the last comment. If the assigned agent is still running, do nothing. If the agent has posted a result, review and advance the pipeline.

---

## 6. Delegation — Assigning Pipeline Steps

Use the Paperclip API to create subtasks and assign to the correct agent:

```
POST /api/companies/{companyId}/issues
{
  "title": "Process N — <Agent Name>: <task title>",
  "parentId": "<parent issue id>",
  "goalId": "<goal id>",
  "assigneeAgentId": "<agent id from AGENTS.md>",
  "description": "<full context: original ticket + all prior outputs>"
}
```

Always set `parentId` to the current pipeline issue so progress is tracked hierarchically.

Never assign two steps at the same time. One subtask at a time, in order.

---

## 7. Pipeline Step Completion — What to Do After Each Agent Reports Back

| Agent reported | Your action |
|---|---|
| `VALIDATED` | Assign Process 2 to API Testing Agent. **Forward the full `VALIDATED` block verbatim** — including `NewCypressFlow`, `ProposedConfigKey`, `ProposedPaymentMethodSection`, `APIFlow`, and `ConnectorNotes`. Do not summarize or paraphrase. If `NewCypressFlow: TRUE`, explicitly tell the API Testing Agent in your instruction so it knows new Cypress infrastructure will need to be created downstream. |
| `BLOCKED` (any step) | Stop. Comment on parent issue. Await human decision |
| Issue report + `YES` automation readiness | Assign Process 3 to Cypress Feasibility Agent. Forward the full `API_TESTING_RESULT` block verbatim. |
| Issue report + `NO` automation readiness | Stop. Comment: flow not stable. Await human decision |
| HIGH severity bug | Stop. Escalate. Do not advance until resolved |
| Feasibility verdict all `PASS/FIXED` | Assign Process 4 to Test Generation Agent. Forward the full `FEASIBILITY_RESULT` block verbatim. |
| Feasibility still has `FAIL` | Stop. Comment with the failing check. Await fix |
| Spec file produced (`ReadyForRunner: YES`) | Assign Process 5 to Runner Agent. Forward the spec file path and connector name. |
| Runner: all pass/skip | Close pipeline. Write Final Report. Mark issue done |
| Runner: failures found | Loop back to Process 4 (spec bug) or Process 2 (API bug). Re-assign with failure details |

---

## 8. Approval Follow-Up

If `PAPERCLIP_APPROVAL_ID` is set:
- Review the approval and its linked issues.
- Close resolved issues or comment on what remains open.

---

## 9. Fact Extraction

1. Check for new conversations since last extraction.
2. Extract durable facts to the relevant entity in `./life/` (PARA).
3. Update `./memory/YYYY-MM-DD.md` with timeline entries.
4. Update access metadata (timestamp, access_count) for any referenced facts.

---

## 10. Exit

- Comment on any in_progress work before exiting with current status.
- If no assignments and no valid mention-handoff, exit cleanly.

---

## CEO Rules (always active)

- Always use the Paperclip skill for all API calls.
- Always include `X-Paperclip-Run-Id` header on all mutating API calls.
- Comment in concise markdown: status line + bullets + links.
- Never self-assign work via checkout unless explicitly @-mentioned.
- Never run two pipeline steps in parallel.
- Never cancel cross-team tasks — reassign with a comment explaining why.
- Above 80% budget spend, focus only on critical (HIGH severity) tasks.
