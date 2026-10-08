# Hosted Mimiq MCP

Use a configured MCP connection instead of REST when its discovered tools can express the task. Connect your MCP client to `https://mcp.mimiqai.com/mcp` using Streamable HTTP. Without a key, the server answers as a guest: one free test, plus one free A/B on up to 10 simulated people per version while the daily free allowance lasts. With a key, configure `Authorization: Bearer` through the client's secure credential field, for example `claude mcp add --transport http mimiq https://mcp.mimiqai.com/mcp --header "Authorization: Bearer $MIMIQ_API_KEY"`. Keep keys in a secret store, never in prompts or files. Installing this skill does not install the connection.

Before calling, inspect the live tool list and input schemas. These nine names are published by the current runtime (server 0.4.1, checked 2026-10-08); your MCP client may prefix or alias them. Never invent an unlisted tool or assume that an older local package has the hosted schema.

| Published tool | Inputs and current limits | What it does |
| --- | --- | --- |
| `mimiq.test_page` | Required `url`; optional `audience`, `count` (1 through 50, default 10), `goal`, `timeout_seconds` (30 through 900, default 300), plus the audience review inputs below. | Recruits a simulated audience and runs a rendered Page review. |
| `mimiq.test_flow` | Required `url`; optional `audience`, `count` (1 through 10, default 5), `goal`, `max_steps` (10 through 60), `timeout_seconds` (60 through 1800, default 1200), plus the audience review inputs. | Recruits a simulated audience and attempts the browser task in `goal`, clicking and typing in a real browser. |
| `mimiq.compare_urls` | Required `url_a`, `url_b`; optional `audience`, `count` (3 through 50, default 10), `timeout_seconds` (60 through 900, default 600). | The same simulated people see both pages; returns a call (clear, leaning, or too close to call), movement toward each page, and each page's top objections. |
| `mimiq.compare_copy` | Required `version_a`, `version_b` (each at most 20,000 characters); optional `format` (`"copy"` default, or `"email"`, where a first line `Subject: ...` is the subject), `audience`, `count` (3 through 50, default 10), `timeout_seconds` (60 through 900, default 480). | The same simulated people see both versions; returns a call, movement, and top objections per version. |
| `mimiq.test_copy` | Required `variant_a`; optional `variant_b`, `audience`, `count` (1 through 50, default 10 per version), `timeout_seconds` (30 through 900, default 180). Each variant is at most 20,000 characters. | General text reactions, or two separate tests reusing one simulated audience. Prefer `compare_copy` when the user wants a call on which to ship. |
| `mimiq.test_text` | Required `text` (at most 20,000 characters); optional `audience`, `count` (1 through 50, default 10), `goal`, `timeout_seconds` (30 through 900, default 180). | General text reactions. Do not claim `goal` changed the result unless the returned evidence shows it. |
| `mimiq.test_component` | At least one of `component_html` (at most 50,000 characters) or `component_text` (at most 20,000); optional `audience`, `count` (1 through 50, default 10), `goal`, `timeout_seconds` (30 through 900, default 180). | Packages the component and goal as text for simulated evaluation; does not render HTML. |
| `mimiq.ask_audience` | Required `audience`, `question` (at most 500 characters), `options` (2 through 10 strings); optional `count` (1 through 50, default 10), `context`, `concurrency` (1 through 20, default 6), `timeout_seconds` (30 through 900, default 300). | Synchronous multiple-choice survey with its own simulated audience. |
| `mimiq.show_report` | Required `session_id`; optional `compare_session_id`. | Shows a saved test, or two compared, from the account: the call, counts, objections and the report link. Spends no credits. |

Across these schemas, `audience`, `goal`, and survey `context` are at most 500 characters; a URL is at most 2,000. Always supply a deliberate audience and count within the user's budget rather than inheriting a larger default. A shorter summary may be needed for the tool, but do not remove constraints that change the task.

## Audience review

`test_page` and `test_flow` start at once unless the recruited crowd is wrong (more than 1 in 5 not the requested audience). Then the result is an `audience_review` instead of a test. Show the warning to the user. Continue only with their decision: `review_id` with `confirm_audience: true` and `audience_fit_acknowledged: true` to run with the reviewed people, or `review_id` with `audience_correction` (at most 1,500 characters) to recruit again. Never set the acknowledgement on your own.

## Use the tools

1. Choose the tool and inspect its live schema. Carry forward the supplied content, goal, simulated audience, count, and budget. Read live `GET /usage` through an authorized HTTP client before spending, including `flow_credits_per_person`. If a client cannot establish the available allowance, resolve it before running; a server preflight is not a spending authorization.
2. Make one tool call for the intended test. These tools perform recruitment, simulation, and result retrieval themselves, or run the synchronous survey. Do not also run the REST lifecycle for that test. They consume credits despite read-only or idempotent discovery annotations.
3. Parse the returned content. Check `isError` first. Read the actual returned fields: counts, per-person reactions and objections, browser journey evidence for flows, the call and movement for comparisons, and the report link when the call carries a key. Put the report link in your summary.
4. Report the original question, simulated audience, requested and usable counts, findings with evidence, and limitations. Action counts are simulated classifications, not measured conversion or proof of completion.

The tools handle waiting internally. If an error or timeout occurs, preserve any IDs and inspect existing runs through REST or `mimiq.show_report` when available. The published tool schemas do not accept idempotency keys, and repeating a whole tool call can recruit another simulated audience and charge again. Never blindly retry or assume that a timeout cancelled the run.

Apply the [REST recovery rules](api.md#recovery): stop for authentication or credit issues, obtain actual user review for an audience mismatch, and respect rate limits. Error messages may suggest buying credits or getting a key; they do not authorize a purchase.

## Choose REST when needed

- Feed Copy and inbox Email single tests use REST `media` contracts. MCP `test_copy` and `test_text` use general text. For two email versions, `compare_copy` with `format: "email"` is the MCP path.
- Open questions, initial interviews, individual follow-ups, room follow-ups, images, and video frames have REST paths but no dedicated published MCP tools. `ask_audience` requires answer options and does not accept an existing audience.
- Browser device selection, selected `persona_id` values, and exact cohort reuse across separate Page, Flow or Image tests require REST. The published MCP tools have no `devices`, `persona_ids`, or `audience_id` input.
- For component appearance or behavior, use Image, Page, or Flow as appropriate. Textual component evaluation does not produce a rendered interaction test.

For browser tests, use an already authorized public URL. Do not open a tunnel or publish code without explicit authorization. Flow can submit forms, so use a disposable environment that safely accepts the intended task. A dry run or prohibition on paid calls means preparing the tool arguments without executing them.
