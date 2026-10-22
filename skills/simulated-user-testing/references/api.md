# Mimiq REST contracts (flows)

Mimiq tests web apps by having AI agents use them: each agent acts as one of the app's users in a real browser and tries a task. These are the REST contracts for that, checked against the current routes on 2026-10-22. MCP's `mimiq.test_flow` does the same lifecycle in one call.

Contents: [access and credits](#access-and-credits), [audiences](#audiences), [runs](#runs), [polling and results](#polling-and-results), [recovery](#recovery).

## Access and credits

API base: `https://api.mimiqai.com/api`. All paths below are relative to it. A configured, user-approved base takes precedence for private installations.

Authenticate with `X-API-Key` or `Authorization: Bearer` using the user's Mimiq key. Use JSON bodies and `Content-Type: application/json`. Obtain a key from **API keys** in [Settings](https://www.mimiqai.com/app/settings), after creating or signing in to an account. Keep it in a secret store, optionally injected into the HTTP client's environment as `MIMIQ_API_KEY`. Never put keys in prompts, files, URLs, logs, or requests to the tested site.

Use `GET /usage` before any new credit-consuming operation. Read `remaining_personas`, `max_personas_per_sim`, and `flow_credits_per_person`. Other returned fields include `total_remaining`, `free_remaining`, `paid_remaining`, `monthly_remaining`, `monthly_credits`, `purchased_remaining`, `used_personas`, `total_used`, `total_personas`, `plan`, `plan_status`, and any applicable `guest_flow_max_people`, `guest_pair_max_people`, or `device_capped` limits.

Each agent in a run costs the live `flow_credits_per_person` (normally 5), and making the agents is included in their first run. Running the same agents again costs the same per agent. Do not hardcode a price; if the allowance for the planned runs cannot be established, resolve the budget before starting.

Use only authorized calls and stay within the user's budget, including variants and follow-ups. A dry run or prohibition on paid calls means no audience generation and no runs. Never buy credits, open a tunnel, publish code, or message third parties without explicit authorization.

## Audiences

`POST /personas/generate-from-prompt` is synchronous. Required: `prompt` (string). Optional: `name` (string, default `"Targeted Audience"`), `count` (integer, default 50), `page_url` (string). Always set an intentional count; values below 1 fail, and values above `max_personas_per_sim` are capped.

Use a fresh, stable `Idempotency-Key` for recruitment:

```json
{
  "name": "Settings task review",
  "prompt": "Independent bookkeepers who manage several client accounts and rarely change notification settings",
  "count": 3,
  "page_url": "https://demo.example.com/settings"
}
```

Response fields: `message`, `status: "COMPLETED"`, `audience_id`, `size`, `criteria`, `demographics`, `personas`, `audience_spec`, `audience_fit`. Use actual `size` and the returned IDs, not the requested count. `page_url` adds page context; it does not make an inaccessible page reachable.

`GET /audiences` lists accessible audiences. `GET /audiences/{audience_id}/review` returns the audience's `audience_fit` and `audience_spec` with its review information. Inspect fit and warnings before running. When `audience_fit.requires_acknowledgement` is true, show the mismatch and obtain the user's actual decision. Set `audience_fit_acknowledged: true` on a simulation only after that acknowledgement, never just to bypass a 409. An empty audience is not usable.

## Runs

A run is `POST /simulations` with `type: "WEB"` and `web_mode: "e2e"`:

```json
{
  "project_id": "mimiq-skills",
  "audience_id": "<returned audience_id>",
  "type": "WEB",
  "web_mode": "e2e",
  "content": "https://preview.example.com/",
  "goal": "Starting on the home page, create an account and reach the dashboard. Stop before entering payment details.",
  "max_steps": 30
}
```

| Field | Contract |
| --- | --- |
| `project_id` | String; an ad hoc label is accepted. |
| `audience_id` | The returned audience ID. |
| `content` | The entry URL. Public, or reachable with `access`. |
| `goal` | The task in plain words, with a visible finish. Never put secrets here. |
| `max_steps` | Integer, default 40, clamped to 10 through 60. |
| `persona_ids` | Optional subset of the audience's agents; omitted means all. Reuse the same IDs to rerun the same agents on a fixed build. |
| `devices` | Optional `"phone"` or `"laptop"`. Check the returned screenshots before calling a run mobile. |
| `access` | Optional, for a protected preview or a sign-in: `vercel_bypass` (string), `basic_auth` (`username`, `password`), `headers` (up to 4), `login` (`email`, `password`, optional `note`). Sealed apart from the test, filled in only in the browser while agents type placeholders, deleted when the run ends, never shown in results. Needs paid credits. Ask the user before sending a password; use a test account. |
| `audience_fit_acknowledged` | Boolean, only after the user reviewed a fit warning. |

`POST /personas/generate-from-prompt` takes `page_access` with the same shape when the page used to match the audience is protected.

The browser clicks, types, navigates and submits forms, and each agent gets an email address with a working inbox for verification codes and magic links. Use an authorized environment with test data: goal text is not an enforcement boundary for payments, bookings, invitations or production changes. Agents get no payment details, and CAPTCHAs or bot protection can stop them.

Use a different stable `Idempotency-Key` for each intended run. Creation returns `simulation_id` and `status`, normally `PENDING`. Creation is not completion.

## Polling and results

For `/simulations`, keep the ID immediately. `GET /simulations/{id}` returns its record, including `status`, selected `persona_ids`, and available progress, errors, or queue information. Poll every 15 seconds for at most 20 minutes per run as a client-side waiting policy:

- `COMPLETED`: fetch results.
- `FAILED`, `CANCELLED`, `CANCELED`: terminal failure; report it and any partial evidence explicitly.
- `PENDING`, `RUNNING`, `PROCESSING`, `CANCELLING`: continue within the deadline.
- Unexpected status: inspect it; do not guess success.

A client deadline does not cancel the server run. Preserve the ID and last status so the user can resume polling. Never start another charged run as a polling workaround.

`GET /simulations/{id}/results` returns this shape; the following row is illustrative only:

```json
{
  "summary": {"total": 1},
  "results": [
    {
      "simulation_id": "example-id",
      "persona_id": "example-persona",
      "persona": {"id": "example-persona", "first_name": "Example"},
      "result": {
        "action": "give_up",
        "monologue": "Illustrative simulated reaction only.",
        "objections": ["Illustrative objection only."],
        "what_would_help": "Illustrative suggestion only."
      }
    }
  ]
}
```

Read each nested `result`. Browser evidence includes `journey_steps` (each with the action, the page, the agent's thinking, and `tech` when the browser recorded something on that step), outcomes, final state, URLs and screenshots. `tech` items are observed by the browser on the site under test: `http_error` (`status`, `method`, `path`), `request_failed` (`error`), `js_error` and `console_error` (`message`), and `slow` (`ms`, for requests slower than 3 seconds). Group them across agents and report them first. Authenticate to `GET /simulations/{id}/evidence/{filename}` only for evidence filenames actually returned. Never invent an evidence URL or public report link.

Exclude error rows from findings and disclose missing/failed reactions. `summary.total` counts returned rows, not automatically usable reactions or the original requested count. A narrative claim of completion is not a recorded successful browser task. Separate usability friction from a login wall, CAPTCHA, inaccessible site, browser error, or step limit. Quotes must be exact and attributed to actual IDs.

## Recovery

- `401`: stop and fix authentication through the secret store. Do not expose the key or try an unauthenticated identity as a workaround.
- `402`: stop and explain the available-credit or account-limit problem. Do not purchase credits without explicit user authorization.
- `409` with `audience_fit_review_required`: show `detail.audience_fit` and obtain actual user review. Other 409 responses can mean `in_progress`, `closed`, a repeated survey, or absent reactions. Inspect the detail before acting.
- `429`: respect `Retry-After` and the operation's rate/daily limit. Pace work; do not change identities or submit parallel replacements to bypass it. The default guarded mutation rate is 60 requests per minute per account, but the deployment can change it.
- `400`/`422`: correct the body against the contract. `403`/`404`: check permission and IDs; never infer that an unavailable resource belongs to the caller.
- Transient read failure: retry at most three times, waiting at least 15 seconds or a longer `Retry-After`. On persistent failure or the waiting deadline, report the ID and last known state.

Recruitment and simulation creation support `Idempotency-Key` (also `X-Idempotency-Key`), at most 200 characters with no control characters. Keep a stable key and exact body for the intended operation. Reuse a key only for that same operation; never for changed content, counts, audience, or acknowledgement. A request trace ID is not an idempotency key.

After an uncertain mutation outcome, preserve the key and body and inspect returned IDs, `GET /simulations`, or `GET /audiences` before deciding on a replay. Never assume a timeout means nothing was created. Do not blindly retry any generation or run. Start a replacement only after resolving the original state and confirming that the existing authorization and budget cover it.

`POST /simulations/{id}/cancel` is available for user-requested cancellation. It returns `simulation_id`, `status`, `cancelled`, and sometimes `message`; a Flow may first become `CANCELLING`. Do not promise an immediate refund. Recheck `/usage` when credit accounting matters.

Report browser records as observed facts and the agents' words as simulated evidence. Use counts, disclose limitations, and never present a run as a conversion forecast.
