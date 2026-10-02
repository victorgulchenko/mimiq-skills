# Mimiq REST contracts

Checked against the current request schemas, routes, authorization rules, and published MCP runtime on 2026-10-02. This is a source-checked contract, not a record of live test runs.

Contents: [access and credits](#access-and-credits), [audiences](#audiences), [simulation requests](#simulation-requests), [page and flow](#page-and-flow), [text and email](#text-and-email), [images and video](#images-and-video), [components](#components), [comparisons](#comparisons), [questions and interviews](#questions-and-interviews), [polling and results](#polling-and-results), [recovery](#recovery).

## Available modes

| Mode | Public REST contract | Dedicated published MCP tool |
| --- | --- | --- |
| Page | `POST /simulations`, `type: "WEB"`, `web_mode: "visual_journey"` | `mimiq.test_page` |
| Any browser task | Same route, `type: "WEB"`, `web_mode: "e2e"` | `mimiq.test_flow` |
| General text or idea | Same route, `type: "TEXT"`, optional `context` and `goal` | `mimiq.test_text`, `mimiq.test_copy` |
| Feed copy | Same route, `type: "TEXT"`, `media: "copy"` | No exact equivalent; MCP copy uses general text |
| Email | Same route, `type: "TEXT"`, `media: "email"` | None for inbox mode |
| A/B variants | Separate simulations with the same audience and `persona_id` values | `mimiq.test_copy` for two text variants |
| Ad, image, or screenshots | Same route, `type: "IMAGE"`, `images` | None |
| Video frames | Same route, `type: "IMAGE"`, `media: "video"`, `images`, `frame_times`, `duration` | None |
| Open audience question or initial interview | Same route, `type: "TEXT"`, `goal_schema: {"goal_type":"interview"}` | None for this open-answer mode |
| Question with answer choices | `POST /surveys/simulate` | `mimiq.ask_audience` |
| Follow up with one simulated person | `POST /personas/chat` | None |
| Follow up with an existing simulated audience | `POST /rooms/ask` | None |
| Component described as HTML/text | `POST /simulations`, `type: "TEXT"`, component goal below | `mimiq.test_component` |

All testing modes above are available through API-key REST access. None is app-only. App labels and input controls are not additional request types: do not send `type: "AD"`, `"EMAIL"`, `"SURVEY"`, or `"COMPONENT"` to `/simulations`.

The app's file picker and video frame extraction are client-side preparation, not public raw-file testing endpoints. `/content/upload` and `/content/upload-multiple` require an internal credential and are unavailable to normal API keys. Supply image bytes in `images` instead. A video URL, raw clip, or audio track is not a supported simulation body. There is no browser-rendered HTML component mode, arbitrary browser viewport field, or dedicated measured card-sort, tree-test, or first-click study contract here. Do not invent one from an app label or a returned field.

## Access and credits

API base: `https://api.mimiqai.com/api`. All paths below are relative to it. A configured, user-approved base takes precedence for private installations.

Authenticate with `X-API-Key` or `Authorization: Bearer` using the user's Mimiq key. Use JSON bodies and `Content-Type: application/json`. Obtain a key from **API keys** in [Settings](https://www.mimiqai.com/app/settings), after creating or signing in to an account. Keep it in a secret store, optionally injected into the HTTP client's environment as `MIMIQ_API_KEY`. Never put keys in prompts, files, URLs, logs, or requests to the tested site.

Use `GET /usage` before any new credit-consuming operation. Read `remaining_personas`, `max_personas_per_sim`, and `flow_credits_per_person`. Other returned fields include `total_remaining`, `free_remaining`, `paid_remaining`, `monthly_remaining`, `monthly_credits`, `purchased_remaining`, `used_personas`, `total_used`, `total_personas`, `plan`, `plan_status`, and any applicable `guest_flow_max_people`, `guest_pair_max_people`, or `device_capped` limits.

Audience generation reserves credits and grants an entitlement for that simulated audience's first test. The first test consumes that entitlement; a Flow adds the browser premium. Reusing the audience for another version or rerun consumes more credits. Estimate the full Flow allowance as the actual count of simulated people times the live `flow_credits_per_person`, accounting for any already reserved entitlement. Do not hardcode a price or per-person cost, assume recruitment is free, or double-count the first-test entitlement. `/usage` does not expose a separate rate field for every medium; do not invent fields or pricing. If the allowance for the full plan cannot be established, resolve the budget before starting it.

Use only authorized calls and stay within the user's budget, including variants and follow-ups. A dry run or prohibition on paid calls means no generation, simulations, surveys, interviews, or comparison synthesis. Never buy credits, open a tunnel, publish code, or message third parties without explicit authorization.

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

## Simulation requests

All asynchronous modes use `POST /simulations`. Required fields:

| Field | Type and meaning |
| --- | --- |
| `project_id` | String identifying the project. An ad hoc label is accepted; an existing project must be accessible to the caller. |
| `audience_id` | String, returned audience ID owned by the account or accessible workspace. |
| `type` | Exactly `"WEB"`, `"TEXT"`, or `"IMAGE"`. |
| `content` | Nonempty string: entry URL, actual text, image caption, or question as appropriate. |

Supported optional fields:

| Field | Contract |
| --- | --- |
| `audience_fit_acknowledged` | Boolean, default false. Only after actual user review. |
| `persona_ids` | Array of strings selecting simulated people from this audience. Omitted or empty means the full audience, not zero. |
| `web_mode` | String, default `"visual_journey"`. Use `"e2e"` for interaction. `"quick"` is also supported for page-content analysis. |
| `context` | String providing situational background where the selected mode uses it. |
| `goal` | String: task for Flow; research focus for other modes. Can change non-Flow result taxonomy. |
| `metric` | Optional string; a fallback research focus for text. It does not create a measured metric. |
| `goal_schema` | Object for the evaluation taxonomy; see text and question modes. Omit unless deliberately selecting that path. |
| `max_steps` | Integer for Flow. Default 40; server clamps to 10 through 60. Inspect the accepted value. |
| `devices` | String for Flow: `"phone"`, `"laptop"`, or `"mix"`. Only explicit phone/laptop overrides are retained; mix uses the runtime default. |
| `images` | Array of base64-encoded image strings for `IMAGE`; at most the first 10 are retained. Ignored for other types. |
| `media` | `"copy"`, `"email"`, or `"video"`; other values are discarded. Use only with the corresponding type below. |
| `frame_times` | Array of numbers, frame timestamps in seconds for video, first 10 retained. |
| `duration` | Number, clip duration in seconds for video. |
| `nodes`, `edges`, `viewport` | Optional arrays of objects, arrays of objects, and object respectively, for saved canvas state. `viewport` is canvas pan/zoom, not browser dimensions. |
| `activation_id` | Optional string used for correlation; unnecessary for these workflows. |

A simulation uses the actual selected IDs, capped by `max_personas_per_sim`; Flow has a further maximum of 50 simulated people through REST. Keep the returned `persona_ids` from the simulation record for evidence and comparisons. MCP has smaller count limits in some tools.

Use a different stable `Idempotency-Key` for each intended simulation. Creation returns `simulation_id` and `status`, normally `PENDING`. An exact replay may return `idempotent_replay: true`. Creation is not completion.

## Page and Flow

Page accepts any publicly reachable website page, including pricing, product details, documentation, dashboards, error states, and previews:

```json
{
  "project_id": "mimiq-skills",
  "audience_id": "<returned audience_id>",
  "type": "WEB",
  "content": "https://demo.example.com/pricing",
  "web_mode": "visual_journey",
  "goal": "Understand which plan fits a solo practice and what makes the limits unclear"
}
```

Page uses rendered page evidence and simulated attention/reactions. It does not establish completion across screens. `web_mode: "quick"` uses page-content analysis; label it accordingly rather than reporting a rendered journey.

Flow expresses any task as an entry URL plus an observable goal:

```json
{
  "project_id": "mimiq-skills",
  "audience_id": "<returned audience_id>",
  "type": "WEB",
  "content": "https://demo.example.com/dashboard",
  "web_mode": "e2e",
  "goal": "Find notification settings and disable the weekly digest in this disposable test account. Success is a saved disabled state.",
  "max_steps": 15
}
```

The browser clicks, types, navigates, and can submit forms. Use an authorized disposable environment and test data. Goal text is not an enforcement boundary for payments, bookings, invitations, or production account changes. There are no public request fields for importing cookies, a saved login session, or browser credentials. Use an appropriately prepared demo or state the access limitation; never put secrets in `goal` or `context`.

For a narrow browser layout, REST accepts `devices: "phone"`; `"laptop"` requests a desktop layout. The current browser path can use a 390 by 844 phone layout and a 1440 by 900 laptop layout. Effective device behavior depends on the deployed browser path and configuration, so verify returned browser evidence before labeling a run mobile. `"mix"` does not guarantee a mix. There is no arbitrary width/height override or Page device selector. Supplied narrow-layout screenshots can be reviewed with Image mode, with no claim that they were interacted with.

## Text and Email

For instructions, positioning, product ideas, notices, or other text in a particular setting:

```json
{
  "project_id": "mimiq-skills",
  "audience_id": "<returned audience_id>",
  "type": "TEXT",
  "content": "No reports yet. Connect a demo data source to build your first report.",
  "context": "The empty reports screen during first-time setup",
  "goal": "Explain what to do next and what information seems missing"
}
```

This general text path accepts `context` and `goal`; the default context is a feed if omitted. A non-Flow `goal` can automatically select a taxonomy. A custom `goal_schema` can specify `goal_type`, `metric_name`, and `positive_actions`, `neutral_actions`, `negative_actions` arrays. Those labels classify simulated answers, not measured outcomes. Use the actual returned fields. There is no top-level `test_mode` field in the public request.

For feed copy, use this body without `goal` or `goal_schema`:

```json
{
  "project_id": "mimiq-skills",
  "audience_id": "<returned audience_id>",
  "type": "TEXT",
  "media": "copy",
  "content": "Spend Friday on clients, not payment reminders. See how the reminder workflow works."
}
```

For inbox reactions, use `media: "email"` and put the supplied sender context, subject, and body together in `content`:

```json
{
  "project_id": "mimiq-skills",
  "audience_id": "<returned audience_id>",
  "type": "TEXT",
  "media": "email",
  "content": "From: Example Operations\nSubject: Fewer manual handoffs\n\nCould a shared handoff checklist help your team? Reply if you would like to see the example."
}
```

Copy and Email frame the content in a feed and inbox respectively and currently read only the first 4,000 content characters. Do not silently test a truncated long message. This path ignores custom `context`; there are no separate `subject`, `sender`, or `html` fields. It returns simulated reactions, not delivery, rendering, or an actually sent email.

These media-specific paths apply only when no `goal_schema` is present. Setting `goal` can generate a schema and switch the request to general text. To keep feed/inbox conditions stable, omit both fields, record the research question as the analysis goal, and use an authorized follow-up for a direct question. Do not add a schema to a feed comparison casually.

Typical Copy/Email result fields include `action` (`act`, `read`, `glanced`, `ignored`), `outcome`, `category`, `gut_reaction`, `first_impression`, `monologue`, `objections`, `what_would_help`, `sentiment`, and sometimes `answer`. Frequency judgments such as `stop_per_100` and `click_per_100` are simulated inputs, not observed rates. General text can return a different taxonomy. MCP text/copy uses general text, not these media-specific contracts.

## Images and Video

Send image bytes, not a local path or an image URL:

```json
{
  "project_id": "mimiq-skills",
  "audience_id": "<returned audience_id>",
  "type": "IMAGE",
  "content": "A weatherproof pannier for the daily commute. See the supplied product details.",
  "images": ["<base64-encoded PNG or JPEG bytes>"]
}
```

One image without a `goal` is evaluated in a feed context. Supply the actual caption; if there is none, `"Uploaded image"` is the recognized placeholder. Typical actions are `like`, `reply`, `click`, `scroll`, `scrolled_past`, and `ignore`, with `gut_reaction`, `monologue`, `objections`, and `metrics` when present. Do not treat `metrics` as measured performance.

For a focused single-image question, put the question in `content` and its evaluation context in `goal`. With the example project label above, this uses `yes`, `maybe`, or `no` reactions. The app's `project_id: "mimiq-v3"` instead preserves feed framing for a single image with a goal. Record the method used and keep it identical across variants.

Multiple images in one request are treated as a sequence of prototype screens reviewed together, not independent A/B versions. Use ordered screenshots, a description in `content`, and an optional focus in `goal`. Possible fields include `flow_assessment`, `favorite_screen`, `worst_screen`, and actions such as `would_use`, `interested`, or `confused`. Screenshots are not clickable. For a controlled image comparison, use separate simulations on the same IDs. If `images` is missing, the service falls back to text; never describe that as visual evidence.

Video is also supported through `/simulations`: use `type: "IMAGE"`, `media: "video"`, `images` containing up to 10 ordered frames, matching numeric `frame_times` in seconds, and numeric `duration` in seconds. Put the supplied transcript/caption in `content`, or `"Uploaded video"` when absent; `goal` can carry a question. It is a frame-based simulated review, not raw video playback or audio analysis. Prepare frames with authorized local tooling or use the app's preparation controls. Never invent a video-upload endpoint.

## Components

Components are supported as textual evaluation of supplied HTML and/or a description. REST equivalent of the published MCP component workflow:

```json
{
  "project_id": "mimiq-skills",
  "audience_id": "<returned audience_id>",
  "type": "TEXT",
  "content": "Evaluate this UI component: Is the action clear?\nComponent HTML:\n<button>Save notification preferences</button>\nComponent description: Below a set of notification toggles on a settings screen.",
  "context": "evaluating a UI component in an application",
  "goal": "__component_evaluation__"
}
```

The exact goal marker selects component evaluation. Put the actual component question in `content`. It can produce an intended click target and rationale; it does not render HTML or click the control. Use Image for supplied appearance, Page for a rendered page, or Flow for actual interaction. There is no `component_html` field on `/simulations`; that field belongs to the MCP tool, which packages it into `content`.

## Comparisons

There is no `variant_a`/`variant_b` body on `/simulations`. Recruit once, then submit one request per version with the same `audience_id` and explicit identical `persona_ids`. Preserve mode, goal, context, media, device setting, and other conditions. Change only the intended variable and use distinct idempotency keys. Reusing a cohort does not make later tests free.

Poll each ID and match usable rows by `persona_id`. Report both IDs, per-version action counts, exclusions, disagreements, and whether any simulated preference is clear. Two failed or incomparable runs do not make a winner. These are simulated comparisons, not production A/B experiments or evidence of numerical lift.

For existing app sessions, an additional authenticated endpoint is available: `POST /compare/call` with `{"a":"<owned session ID A>","b":"<owned session ID B>"}`. These are session IDs, not simulation IDs. It returns `pick`, `votes`, `reason`, `version`, and, for a new available result, `created_at`; `unavailable: true` with no pick is returned for unsupported pairs such as video or mismatched kinds. It requires accessible existing sessions, may reuse a cached response, and is rate/daily limited. This optional synthesis neither starts the two tests nor proves cohort matching. The raw-result comparison above does not require it. Do not publish sessions to obtain a comparison.

## Questions and Interviews

### Open question or initial interview

Use a generated or existing simulated audience and the usual asynchronous lifecycle:

```json
{
  "project_id": "mimiq-skills",
  "audience_id": "<returned audience_id>",
  "type": "TEXT",
  "content": "How do you handle image permissions when preparing a museum exhibition?",
  "goal": "How do you handle image permissions when preparing a museum exhibition?",
  "context": "A research conversation about the current workflow, without a product pitch",
  "goal_schema": {"goal_type": "interview"}
}
```

This requests open simulated answers rather than scoring a question as feed copy. Use it for niche audience questions, needs, product ideas, or the start of an interview. Read the mode-specific answer fields in each `result`; preserve the complete row for follow-ups.

### Question with choices

`POST /surveys/simulate` is synchronous and recruits its own simulated audience. Do not generate another audience first. Required fields: `audience_prompt` (nonempty string), `question` (nonempty string, at most 500 characters), `options` (2 through 10 choices, at least two nonempty). Optional: `project_id` (string, default `"mcp"`), `count` (integer, default 40, valid 1 through `max_personas_per_sim`), `context` (string, default `"Product validation survey"`), `concurrency` (integer, default 6, effective range 1 through 20).

```json
{
  "project_id": "mimiq-skills",
  "audience_prompt": "Museum archivists at small regional museums",
  "count": 5,
  "question": "Which part of preparing an exhibition takes the most avoidable work?",
  "options": ["Finding records", "Checking permissions", "Coordinating loans", "Something else"],
  "context": "Prioritizing workflow research",
  "concurrency": 6
}
```

Use a stable `Idempotency-Key`. The response has `summary.total`, `summary.distribution`, `summary.extraction_stats` (`success`, `parse_errors`, `timeout_errors`, `option_match_failures`), and `results`. Survey rows have `persona_id`, `persona_name`, `persona`, `selected_option`, `response_text`, `thought_process`, `action`, and `trust_score` directly, not inside a nested `result`.

There is no returned simulation ID, polling route, audience ID, or audience-fit review step in this response. Distribution excludes unmatched choices; count usable choices and disclose extraction failures. A replay can return 409 rather than replaying answers. This endpoint does not accept an existing audience or acknowledgement field. If cohort reuse or fit review is essential, use `/simulations` with a reviewed audience instead. Do not run separate surveys and describe them as the same simulated people.

### One simulated person, follow-up

`POST /personas/chat` accepts:

| Field | Contract |
| --- | --- |
| `persona` | Required object: the returned simulated person's profile. |
| `simulation_result` | Required object: that simulated person's actual nested `result`. |
| `simulation_content` | Optional string with the tested content or question. |
| `message` | Required string with the follow-up question. |
| `chat_history` | Optional array, default empty, of objects with `role` (`"user"` or `"assistant"`) and `content` strings. Only the last six turns are used. |

Response: `{"response":"<simulated answer>"}`. Send the recorded profile and result rather than inventing a new identity or passing only an ID. Keep supplied history tied to that simulated person. This is synchronous and subject to rate and daily limits, not simulation polling. It is currently not deducted from persona credits, but still requires an authorized call; do not promise unlimited interviews.

### Existing simulated audience, follow-up

`POST /rooms/ask` accepts `simulation_id` (required string), `question` (required string, 3 through 400 characters after whitespace normalization), optional `persona_ids` (array of strings), and optional `options` (array of strings). Provide 2 through 6 nonempty options of at most 80 characters for a choice question; the server truncates extras and treats fewer than two as no options.

```json
{
  "simulation_id": "<completed simulation ID>",
  "question": "What information was missing when you tried to choose a plan?"
}
```

It uses existing recorded reactions, up to 60 simulated people; no reactions yields 409. Prefer a completed test. Response: `question`, `options`, `answers` (each with `persona_id`, `answer`, `stance`, `gist`), `tally`, `estimate`, `themes`, `asked`. Open questions can have themes and no tally; choice questions can include a simulated estimate. Use actual answers and counts, not `estimate.shares` as observed behavior. This call is synchronous, rate/daily limited, and currently not deducted from persona credits. Neither follow-up route offers a guaranteed idempotent answer replay; do not blindly retry.

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
        "action": "left",
        "monologue": "Illustrative simulated reaction only.",
        "objections": ["Illustrative objection only."],
        "what_would_help": "Illustrative suggestion only."
      }
    }
  ]
}
```

Fields differ by mode. Read each nested `result`, not invented top-level actions. Useful browser evidence may include `journey_steps`, `journey`, outcomes, final state, URLs, and screenshots. Authenticate to `GET /simulations/{id}/evidence/{filename}` only for evidence filenames actually returned. Never invent an evidence URL or public report link.

Exclude error rows from findings and disclose missing/failed reactions. `summary.total` counts returned rows, not automatically usable reactions or the original requested count. A narrative claim of completion is not a recorded successful browser task. Separate usability friction from a login wall, CAPTCHA, inaccessible site, browser error, or step limit. Quotes must be exact and attributed to actual IDs.

## Recovery

- `401`: stop and fix authentication through the secret store. Do not expose the key or try an unauthenticated identity as a workaround.
- `402`: stop and explain the available-credit or account-limit problem. Do not purchase credits without explicit user authorization.
- `409` with `audience_fit_review_required`: show `detail.audience_fit` and obtain actual user review. Other 409 responses can mean `in_progress`, `closed`, a repeated survey, or absent reactions. Inspect the detail before acting.
- `429`: respect `Retry-After` and the operation's rate/daily limit. Pace work; do not change identities or submit parallel replacements to bypass it. The default guarded mutation rate is 60 requests per minute per account, but the deployment can change it.
- `400`/`422`: correct the body against the contract. `403`/`404`: check permission and IDs; never infer that an unavailable resource belongs to the caller.
- Transient read failure: retry at most three times, waiting at least 15 seconds or a longer `Retry-After`. On persistent failure or the waiting deadline, report the ID and last known state.

Recruitment and simulation creation support `Idempotency-Key` (also `X-Idempotency-Key`), at most 200 characters with no control characters. Keep a stable key and exact body for the intended operation. Reuse a key only for that same operation; never for changed content, counts, audience, or acknowledgement. A request trace ID is not an idempotency key.

After an uncertain mutation outcome, preserve the key and body and inspect returned IDs, `GET /simulations`, or `GET /audiences` before deciding on a replay. Never assume a timeout means nothing was created. Do not blindly retry any generation, test, survey, interview, or comparison. Start a replacement only after resolving the original state and confirming that the existing authorization and budget cover it.

`POST /simulations/{id}/cancel` is available for user-requested cancellation. It returns `simulation_id`, `status`, `cancelled`, and sometimes `message`; a Flow may first become `CANCELLING`. Do not promise an immediate refund. Recheck `/usage` when credit accounting matters.

Always report simulated evidence as hypotheses. Use counts, disclose limitations, and keep returned scores, predicted shares, and action labels separate from measured conversion or real-world outcomes.
