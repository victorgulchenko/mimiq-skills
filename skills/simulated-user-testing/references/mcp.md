# Hosted Mimiq MCP

Use a configured MCP connection instead of REST when its discovered tools can express the task. Connect your MCP client to `https://mcp.mimiqai.com/mcp` using Streamable HTTP. Configure `Authorization: Bearer` through its secure credential field with the user's Mimiq API key. Keep keys in a secret store, never in prompts or files. Installing this skill does not install the connection.

Before calling, inspect the live tool list and input schemas. These six names are published by the current runtime; your MCP client may prefix or alias them. Never invent an unlisted tool or assume that an older local package has the hosted schema.

| Published tool | Inputs and current limits | What it does |
| --- | --- | --- |
| `mimiq.test_page` | Required `url`; optional `audience`, `count` (1 through 50, default 10), `goal`, `timeout_seconds` (30 through 900, default 300). | Generates a simulated audience and runs a rendered Page review. |
| `mimiq.test_flow` | Required `url`; optional `audience`, `count` (1 through 10, default 5), `goal`, `max_steps` (schema 1 through 100, default 15), `timeout_seconds` (60 through 1800, default 420). | Generates a simulated audience and attempts the arbitrary browser task in `goal`. The backend currently clamps steps to 10 through 60. |
| `mimiq.test_copy` | Required `variant_a`; optional `variant_b`, `audience`, `count` (1 through 50, default 10 per version), `timeout_seconds` (30 through 900, default 180). Each variant is at most 20,000 characters. | General text reactions, or two separate tests reusing one simulated audience. |
| `mimiq.test_text` | Required `text` (at most 20,000 characters); optional `audience`, `count` (1 through 50, default 10), `goal`, `timeout_seconds` (30 through 900, default 180). | General text reactions. The current runtime lists `goal` but does not forward it. |
| `mimiq.test_component` | At least one of `component_html` (at most 50,000 characters) or `component_text` (at most 20,000); optional `audience`, `count` (1 through 50, default 10), `goal`, `timeout_seconds` (30 through 900, default 180). | Packages the component and goal as text for simulated evaluation; does not render HTML. |
| `mimiq.ask_audience` | Required `audience`, `question` (1 through 500 characters), `options` (2 through 10 strings); optional `count` (1 through 50, default 10), `context`, `concurrency` (1 through 20, default 6), `timeout_seconds` (schema 30 through 900, default 300). | Synchronous survey with its own simulated audience. The current runtime does not forward `timeout_seconds`. |

Across these schemas, `audience`, `goal`, and survey `context` are at most 500 characters; a URL is at most 2,000. Always supply a deliberate audience and count within the user's budget rather than inheriting a larger default. A shorter summary may be needed for the tool, but do not remove constraints that change the task.

## Use the tools

1. Choose the tool and inspect its live schema. Carry forward the supplied content, goal, simulated audience, count, and budget. The six tools do not expose usage, audience reuse by ID, or audience-fit acknowledgement inputs. Read live `GET /usage` through an authorized HTTP client before spending, including `flow_credits_per_person`. If a client cannot establish the available allowance, resolve it before running; a server preflight is not a spending authorization.
2. Make one tool call for the intended test. These tools perform recruitment, simulation, and result retrieval themselves, or run the synchronous survey. Do not also run the REST lifecycle for that test. They consume credits despite read-only/idempotent discovery annotations.
3. Parse the returned text content as JSON. Check `isError` first. A normal result includes `tool`, `sample_size`, `counts`, `personas`, `run`, and `notes`. Each entry in `personas` can include `persona_id`, `action`, `monologue`, `objections`, `what_would_help`, and browser journey evidence. Read actual returned fields, not assumed REST row nesting.
4. For two variants, read `variant_a` and `variant_b` envelopes plus `run.simulation_ids`. Match usable `persona_id` values and disclose missing pairs. Separate Page/Flow calls each recruit a new simulated audience; use REST when a comparison must reuse the same IDs for those modes.
5. Report the original question, simulated audience, requested/usable counts, simulation IDs when supplied, findings with evidence, and limitations. `counts.converted`, `engaged`, and `bounced` are generic simulated classifications, not measured conversion or proof of completion. Survey results have no simulation IDs; retain their actual choice and reasoning evidence.

The tools handle waiting internally. If an error or timeout occurs, preserve any simulation or request IDs and inspect existing runs through REST when available. A request ID is not an idempotency key. The published tool schemas do not accept idempotency keys, and repeating a whole tool call can create another simulated audience and charge again. Never blindly retry or assume that a timeout cancelled the run.

Apply the [REST recovery rules](api.md#recovery): stop for authentication or credit issues, obtain actual user review for an audience mismatch, and respect rate limits. Error messages may suggest buying credits; they do not authorize a purchase. If audience fit requires acknowledgement, use the user's reviewed decision and a supported REST continuation rather than inventing an MCP field or silently bypassing it.

## Choose REST when needed

- Feed Copy and inbox Email use REST `media` contracts. MCP `test_copy` and `test_text` use general text and are not equivalent; `test_copy` does not accept `goal` or `context`.
- Open questions, initial interviews, individual follow-ups, room follow-ups, images, and video frames have REST paths but no dedicated published MCP tools. `ask_audience` requires answer options and does not accept an existing audience.
- To apply a goal that `test_text` does not forward, use the supported REST `goal`/`context` fields. Do not claim that a listed but unused input affected a result.
- Browser device selection, selected `persona_id` values, and exact cohort reuse across Page/Flow/Image versions require REST. The published MCP tools have no `devices`, `persona_ids`, or `audience_id` input.
- For component appearance or behavior, use Image, Page, or Flow as appropriate. Textual component evaluation does not produce a rendered interaction test.

For browser tests, use an already authorized public URL. Do not add an unlisted bridge option, open a tunnel, or publish code without explicit authorization. Flow can submit forms, so use a disposable environment that safely accepts the intended task. A dry run or prohibition on paid calls means preparing the tool arguments without executing them.
