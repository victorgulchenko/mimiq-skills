---
name: simulated-user-testing
description: Use Mimiq to have AI agents use a web app the way its users would, in real browsers, and find what is broken, slow or confusing. Invoke when the user wants their app, site, preview deploy or a change you just built tested by someone who uses it; asks whether new users can sign up, onboard, check out, book, upgrade, cancel, change settings, find a feature or finish any task; asks why people drop off or get stuck; wants QA, exploratory or end-to-end testing without writing scripts; or wants a check on every preview deploy or pull request. Agents act as different users (optionally built from the user's own customers), click, type, fill in forms and read their own verification emails, and the report shows what broke, what the browser recorded (JavaScript errors, failed and slow requests), where each agent got stuck and why, with steps and screenshots.
---

# Simulated user testing (AI agents use the app)

Mimiq sends AI agents through a web app. Each agent acts as one of the app's users in its own real browser, tries a task given in plain words, and reports what happened. Use [the hosted MCP workflow](references/mcp.md) for an installed connection or [the REST contracts](references/api.md) for HTTP. [Recipes](references/recipes.md) has goal wording for common tasks.

## Get access

- **Hosted MCP, fastest:** connect `https://mcp.mimiqai.com/mcp` (Streamable HTTP). Without a key, an agent gets one free run of up to 3 agents. With a key, for example in Claude Code: `claude mcp add --transport http mimiq https://mcp.mimiqai.com/mcp --header "Authorization: Bearer $MIMIQ_API_KEY"`.
- **REST:** base `https://api.mimiqai.com/api` with `Authorization: Bearer $MIMIQ_API_KEY`.
- **Key:** a free account at https://www.mimiqai.com/sign-up?redirect_url=/app/settings (50 credits), then Settings, "Use Mimiq from your coding agent", "Create a key". Keep it in the environment or a secret store, never in prompts, URLs or committed files.

Short skills for common jobs: `check-my-sign-up-flow`, `test-my-landing-page`, `test-before-done` (after you build a change), `test-every-preview` (set up the GitHub Action). This skill covers all of them.

## One procedure

1. **Get a public URL.** A preview deployment, staging or the live site. Agents browse from Mimiq's servers, so localhost needs a tunnel; do not open one or publish code without the user's explicit OK. Use an environment that safely accepts test sign-ups and form submissions.
2. **Write the task** with a visible finish: "Starting at [entry], try to [task] using [test data if any]. Success is [screen or state]. Stop before [out-of-scope action]." Name the outcome, not the click path, when discovery is what you are testing.
3. **Choose who.** Describe the users in plain words (roles, experience, situation), or leave it out and Mimiq matches the page. Agents built from the user's own customers are created in the app from a CSV.
4. **Choose how many.** 3 agents catch what everyone hits; 5 to 10 show which problems are common. Each agent uses 5 credits. Read `GET /usage` before spending and stay inside the budget the user gave.
5. **Access, if needed.** For a protected preview or pages behind a sign-in, pass `access` (a Vercel bypass secret, basic auth, or a test account). Ask the user before passing any password, and use a test account, never a real one. This needs paid credits.
6. **Run** `mimiq.test_flow` (MCP) or the REST lifecycle. A run takes a few minutes. If an `audience_review` comes back, show it and continue only with the user's decision.
7. **Report** with the rules below, fix what the agents hit if that is in scope, and run the same task again on the new build.

## Report evidence

- Lead with how many of N agents finished, and the step where the others stopped.
- List what the browser caught first (`browser_caught` in MCP results, or `tech` on journey steps): these are observed facts, such as "POST /api/signup answered 500 (3 of 5 agents)". Then the blockers from the agents' steps: the screen, what the agent tried, a short exact quote, and the fix.
- Separate the app's problems from things the test could not do (CAPTCHA, bot protection, a real payment, the step limit). Do not call those user failures.
- Say "simulated" and use counts, not percentages. Agents find problems; they do not forecast conversion. Put the report link in your summary.

## When not to use it

Not for native mobile apps, load testing, or behavior that must be checked exactly the same way on every build (keep a scripted test for that). Not a substitute for real users' opinions or measured experiments. Skip it when calls are prohibited, credits or authorization are missing, or the environment cannot safely accept the task; explain the limitation or prepare the run instead.
