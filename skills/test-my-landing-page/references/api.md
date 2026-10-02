# Mimiq API access and lifecycle

## Account and client setup

Create or sign in to an account at https://mimiqai.com/sign-up?redirect_url=/app/settings. Open https://mimiqai.com/app/settings, find **API keys**, click **Create a key**, then **Copy key**. Store it in the HTTP client's credential store or `MIMIQ_API_KEY`; do not put it in chat or checked-in files. The raw key is shown once.

API base: `https://api.mimiqai.com/api`. Authenticate every request with `X-API-Key: <mq_sk_...>` or `Authorization: Bearer <mq_sk_...>`. Do not send the key to the tested site. Use JSON request bodies and `Content-Type: application/json`. All paths below are relative to that base. A configured, user-approved API base takes precedence for private installations.

Use an existing authorized HTTP client. These skills do not install tooling, open tunnels, publish sites, or authorize messages to third parties. Runtime API calls consume account credits even when no money is charged at the moment of the call. Honor the user's limits before the first generation request.

## Request sequence

1. `GET /usage` returns `remaining_personas`, `max_personas_per_sim`, and `flow_credits_per_person`. Read the live values. Page/copy tests use a base credit per simulated person. Audience generation reserves base credits and supplies an entitlement for that audience's first test; subsequent tests charge again. Flow mode adds a browser premium. Do not advertise a fixed multiplier or assume generation is free.
2. `POST /personas/generate-from-prompt` creates the audience synchronously. Use a fresh, stable `Idempotency-Key` for this operation:

   ```json
   {
     "name": "Landing page review",
     "prompt": "Independent bookkeepers serving small businesses",
     "count": 5
   }
   ```

   The response contains `audience_id`, `size`, `personas`, and `audience_fit`. Optional `page_url` provides page context. Use the returned ID and actual count. If `audience_fit.requires_acknowledgement` is true, show the mismatch and stop for the user's decision. Only set `audience_fit_acknowledged: true` on a simulation after that actual acknowledgement, never to bypass a rejection.
3. `POST /simulations` accepts the body in the skill. `project_id`, `audience_id`, `type`, and `content` are required. Set `web_mode: "visual_journey"` for Page, `web_mode: "e2e"` for Flow, or `type: "TEXT", media: "copy"` for feed copy. Use a different stable `Idempotency-Key` for each intended test. The response includes `simulation_id` and `status`; creation is not completion.
4. `GET /simulations/{simulation_id}` returns status. Poll every 15 seconds, with a maximum of 20 minutes for a run. `COMPLETED` permits fetching results. `FAILED`, `CANCELLED`, and `CANCELED` are terminal failures. `PENDING`, `RUNNING`, `PROCESSING`, and `CANCELLING` need another poll. Unexpected statuses need inspection, not a guessed success. A client timeout does not cancel a server run.
5. `GET /simulations/{simulation_id}/results` returns:

   ```json
   {
     "summary": { "total": 1 },
     "results": [
       {
         "simulation_id": "example-id",
         "persona_id": "example-persona",
         "persona": { "first_name": "Example" },
         "result": {
           "action": "left",
           "monologue": "Illustrative simulated response only.",
           "objections": ["Illustrative objection only."],
           "what_would_help": "Illustrative suggestion only."
         }
       }
     ]
   }
   ```

   The array lives in `results`; fields such as `action` usually live inside each row's `result`. Not every mode returns every field. Exclude rows with errors from findings, disclose missing/failed reactions, and do not invent a quote, browser event, or metric. Keep the original IDs for follow-up.

## Recovery and output

- HTTP 401: stop and fix the key. HTTP 402: stop and explain the available-credit problem; never buy credits automatically. HTTP 409 with `audience_fit_review_required`: obtain the actual user's review. Other 409 responses can indicate an in-progress or closed operation; inspect the response.
- Do not blindly retry generation or simulation creation after a timeout. Preserve the operation's idempotency key and request body; inspect an already returned ID or the account's simulations before deciding to retry. Reuse an idempotency key only for the exact same operation, never for changed content.
- For a transient read failure, retry at most three times with a delay of at least 15 seconds. Respect a longer `Retry-After`. On persistent failure or the 20-minute deadline, report the known ID and last status. Do not start another charged run as a polling workaround.
- `POST /simulations/{id}/cancel` exists for a user-requested cancellation. Cancellation can be asynchronous; do not promise an immediate refund.
- Default guarded mutation rate is 60 requests per minute per authenticated account, configurable on the server. Pace parallel tests; a 429 response requires waiting, not additional workarounds.
- Always call the participants and feedback simulated. Report counts instead of percentages. Recommend real-world validation for a decision; do not claim accuracy, significance, or a numerical lift forecast from this workflow.

## Optional MCP connection

Connect a Streamable HTTP MCP client to `https://mcp.mimiqai.com/mcp` and configure `Authorization: Bearer <your key>` securely. Read its tool list and schemas before use. The public Mimiq workflows include `mimiq.test_page`, `mimiq.test_flow`, and `mimiq.test_copy`; a client can prefix or alias those names. Never invent an unlisted tool or assume an older local package has the hosted schema. If the available tool cannot express the task, use the API when configured or explain the missing capability.

Public connection reference: https://github.com/victorgulchenko/mimiq-mcp.
