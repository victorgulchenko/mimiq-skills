---
name: check-my-sign-up-flow
description: Send AI agents through a sign-up, onboarding, checkout, booking, upgrade, cancellation, settings change or any multi-step task in a live web app with Mimiq, and find what breaks and where they get stuck. Use when the user asks whether new users can get through sign-up or onboarding, why people drop off, whether a checkout or form works and makes sense, or after a flow changed and should be checked before shipping.
---

# Check my sign-up flow

Answers: can the people this app is for actually finish the task, what breaks on the way, and where do they stall? Each agent acts as one of those users in a real browser, clicking and typing, with an inbox of its own for verification codes.

## Run it

1. **Get the entry URL.** Public: a preview, staging or demo that safely accepts test sign-ups and form submissions, because the browser really submits forms. For a protected preview or pages behind a sign-in, use `access` (paid credits): a Vercel bypass secret, basic auth, or a test account the user gives you.
2. **Write the goal** as a task with a visible finish: "Starting on the home page, create an account and reach the dashboard. Stop before entering payment details." Name the outcome, not the click path.
3. **Pick who** in plain words, or omit it to let Mimiq match the page.
4. **Pick the count.** 3 to 5 agents, 5 credits each. Read `GET /usage` and stay in the user's budget.
5. **Call it.**
   - MCP: `mimiq.test_flow` with `url`, `goal`, `audience`, `count` (1 to 10), optional `max_steps` (10 to 60) and `access`. It waits and returns the result.
   - REST: generate an audience, then `POST /simulations` with `type: "WEB"`, `web_mode: "e2e"`, `content: <url>`, `goal`, optional `max_steps`, `devices: "phone"` and `access`. Poll and read results.
6. **If an audience review comes back**, show it to the user and continue only with their decision.

## Report

- Lead with how many of N finished, and the step where the others stopped.
- Then what the browser caught: failed requests, 4xx/5xx answers, JavaScript errors and requests slower than 3 seconds, with how many agents hit each.
- For each blocker: the step, what the agent tried, a short exact quote, and the concrete fix.
- Separate the app's problems from what the test could not do (CAPTCHA, a real payment, the step limit).
- Say "simulated" and use counts. Offer to fix the top problem and run the same task again.

## Access

- **MCP:** `https://mcp.mimiqai.com/mcp` (Streamable HTTP). One free run of up to 3 agents without a key. With a key: `claude mcp add --transport http mimiq https://mcp.mimiqai.com/mcp --header "Authorization: Bearer $MIMIQ_API_KEY"`.
- **REST:** base `https://api.mimiqai.com/api`, header `Authorization: Bearer $MIMIQ_API_KEY`.
- **Key:** free account at https://www.mimiqai.com/sign-up?redirect_url=/app/settings, then Settings, "Use Mimiq from your coding agent", "Create a key".

Never use real payment details, real bookings or production accounts without explicit permission. Full contracts: [MCP](https://github.com/victorgulchenko/mimiq-skills/blob/main/skills/simulated-user-testing/references/mcp.md), [REST](https://github.com/victorgulchenko/mimiq-skills/blob/main/skills/simulated-user-testing/references/api.md), [recipes](https://github.com/victorgulchenko/mimiq-skills/blob/main/skills/simulated-user-testing/references/recipes.md).
