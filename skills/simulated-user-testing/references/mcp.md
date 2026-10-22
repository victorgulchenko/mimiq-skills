# Hosted Mimiq MCP

Connect your MCP client to `https://mcp.mimiqai.com/mcp` using Streamable HTTP. Without a key, the server answers as a guest: one free run of up to 3 agents while the daily free allowance lasts. With a key, configure `Authorization: Bearer` through the client's secure credential field, for example `claude mcp add --transport http mimiq https://mcp.mimiqai.com/mcp --header "Authorization: Bearer $MIMIQ_API_KEY"`. Keep keys in a secret store, never in prompts or files. Installing this skill does not install the connection.

Inspect the live tool list and input schemas before calling. Coding-agent clients are offered two tools (server 0.5, checked 2026-10-22); your client may prefix or alias them.

| Tool | Inputs and limits | What it does |
| --- | --- | --- |
| `mimiq.test_flow` | Required `url`; optional `goal` (the task, at most 500 characters), `audience` (who the agents act as, at most 500), `count` (1 to 10, default 5; without a key up to 3), `max_steps` (10 to 60, default 40), `timeout_seconds` (60 to 1800, default 1200), `access`, and the audience review inputs below. | AI agents, each acting as one of the app's users, open the URL in real browsers and try the task. Returns each agent's steps, where it stopped and why, the problems ranked by how many agents hit them, `browser_caught`, counts, a `summary_card` and, with a key, `report_url`. |
| `mimiq.show_report` | Required `session_id`. | Shows a saved run from the account again. Spends no credits. |

## Access for protected previews and sign-ins

`access` is an object with any of: `vercel_bypass` (a Vercel "Protection Bypass for Automation" secret), `basic_auth` (`username`, `password`), `headers` (up to 4 headers sent only to the site), `login` (`email`, `password`, optional `note`: a test account the agents sign in with). It is sealed apart from the test, used for that run only, filled in by the browser while agents type placeholders, and deleted when the run ends; no result or report shows a value. Ask the user before passing a password, use a test account and never a real one. Access needs paid credits.

## What comes back

- `counts` with `counts_meaning` (finished, got partway, gave up; or reached the stop point the goal named).
- `personas[]`: each agent's `journey_steps` (steps where something changed, with the agent's thinking, and `tech` lines when the browser recorded something on that step), `objections`, `monologue`.
- `browser_caught[]`: what the browser itself recorded on the site under test, grouped and ranked: `{ kind, text, agents, hits: [{ name, step }] }`. Kinds: `http_error` (a request answered 4xx or 5xx), `request_failed`, `js_error` (uncaught), `console_error`, `slow` (a request slower than 3 seconds). Third-party scripts are left out. These are observed facts; report them first.
- `summary_card`: plain text ready to paste, ending with what the browser caught.
- `report_url` (with a key): the full report, with screenshots of every step. Put it in your summary.

## Audience review

`test_flow` starts at once unless the recruited agents are wrong (more than 1 in 5 not the requested audience). Then the result is `status: "audience_review"` instead of a run. Show the warning to the user. Continue only with their decision: `review_id` with `confirm_audience: true` and `audience_fit_acknowledged: true`, or `review_id` with `audience_correction` (at most 1,500 characters) to recruit again. Never set the acknowledgement on your own.

## Use the tools

1. Read `GET /usage` through an authorized HTTP client before spending when the budget matters (`flow_credits_per_person`, normally 5).
2. Make one `test_flow` call for the intended run; it recruits, runs and waits. Do not also run the REST lifecycle for the same test. It spends credits.
3. Check `isError` first. On `INSUFFICIENT_CREDITS`, stop and tell the user; an error message that mentions buying credits does not authorize a purchase.
4. A timeout does not cancel the run; it keeps going on Mimiq's side. Use `show_report` or the report link later. Never blindly retry: a repeated call recruits and charges again.

Mimiq needs a public URL. For a local dev server, the agent may open a temporary tunnel only with the user's OK (`cloudflared tunnel --url http://localhost:PORT`), pass the https tunnel URL, and close it when done.

The server also accepts some older tools for existing scripts (page, copy and audience tests). They are no longer listed and are not documented here; Mimiq tests web apps by having agents use them.
