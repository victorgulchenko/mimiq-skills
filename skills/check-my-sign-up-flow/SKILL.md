---
name: check-my-sign-up-flow
description: Send simulated people through a real multi-step task in a live product with Mimiq (sign-up, onboarding, checkout, booking, upgrade, cancellation, settings, search, or any goal) and find the step where they get stuck or give up. Use when the user asks whether new users can get through sign-up or onboarding, why people drop off in a funnel, whether a checkout or form is confusing, or after you change a flow and want to check it before shipping.
---

# Check my sign-up flow

Answers: can the people this product is for actually finish the task, and where do they stall? Each simulated person uses a real browser, clicking and typing, and Mimiq records their steps, where they stopped and why.

## Run it

1. **Get the entry URL.** It must be public. Use a preview, staging or demo that safely accepts test sign-ups and form submissions: the browser really submits forms. Pages behind a login are not reachable; start from a public page such as the sign-up page.
2. **Write the goal** as a task with a visible finish: "Starting on the home page, create an account with a test email and reach the dashboard. Stop before entering payment details." Name the outcome, not the click path.
3. **Pick the audience** in plain words, or omit it to let Mimiq infer it from the page.
4. **Pick the count.** 3 to 5 simulated people. Flows cost more per person than page tests: read `flow_credits_per_person` from `GET /usage` and stay inside the user's budget.
5. **Call it.**
   - MCP: `mimiq.test_flow` with `url`, `goal`, `audience`, `count` (1 to 10), optional `max_steps` (10 to 60). It waits and returns the result; flows take a few minutes.
   - REST: generate an audience, then `POST /simulations` with `type: "WEB"`, `web_mode: "e2e"`, `content: <url>`, `goal`, `max_steps`; optional `devices: "phone"` for a phone layout. Poll and read results.
6. **If an audience review comes back**, show it to the user and continue only with their decision (`review_id`, `confirm_audience`, `audience_fit_acknowledged`, or `audience_correction`).

## Report

- Lead with: how many of N finished the task, and the step where the others stopped.
- For each blocker: the step or screen, what the simulated person tried, a short exact quote, and the concrete fix.
- Separate real usability friction from things the browser could not do (CAPTCHA, email verification, login walls, step limit). Do not call those user failures.
- Say "simulated". Use counts. A person saying they would finish is not the same as finishing in the browser.
- Offer: fix the top blocker, then run the same flow again on the same kind of people.

## Access

- **MCP (fastest):** add `https://mcp.mimiqai.com/mcp` as a Streamable HTTP server. Without a key, an agent gets one free test. With a key: `claude mcp add --transport http mimiq https://mcp.mimiqai.com/mcp --header "Authorization: Bearer $MIMIQ_API_KEY"`.
- **REST:** base `https://api.mimiqai.com/api`, header `Authorization: Bearer $MIMIQ_API_KEY`. Read `GET /usage` before spending.
- **Key:** free account at https://www.mimiqai.com/sign-up?redirect_url=/app/settings, then Settings, "Use Mimiq from your coding agent", "Create a key". Keep it in the environment or a secret store.

## More

Never use real payment details, real bookings or production accounts without explicit permission. For other kinds of tests use `simulated-user-testing`. Full contracts: [REST](https://github.com/victorgulchenko/mimiq-skills/blob/main/skills/simulated-user-testing/references/api.md), [MCP](https://github.com/victorgulchenko/mimiq-skills/blob/main/skills/simulated-user-testing/references/mcp.md), [flow recipes](https://github.com/victorgulchenko/mimiq-skills/blob/main/skills/simulated-user-testing/references/recipes.md).
