---
name: test-my-landing-page
description: Show a landing page, homepage, pricing page or any public web page to simulated visitors with Mimiq and hear who would stay, who would leave, and why. Use when the user asks why a page is not converting, whether visitors understand what the product does, what makes them leave or doubt, how to improve a hero, pricing or call to action, or after you build or change a page and want an outside read before shipping.
---

# Test my landing page

Answers: would the people this page is for understand it, believe it, and take the next step? Each simulated visitor scrolls the rendered page and says what confused them, what they doubted, and what would change their mind.

## Run it

1. **Get the URL.** It must be public (a deployed preview is fine). For localhost, ask the user before opening any tunnel.
2. **Pick the audience** in plain words, from the user or the page: "owners of small dental practices in the US", not "everyone". If unsure, omit it and Mimiq infers the likely visitors from the page.
3. **Pick the count.** 5 to 10 simulated people is a good first read. Stay inside the user's budget.
4. **Call it.**
   - MCP (connected to `https://mcp.mimiqai.com/mcp`): `mimiq.test_page` with `url`, `audience`, `count`, and `goal` for what the user wants to learn ("Is the pricing clear?").
   - REST: generate an audience with `POST /personas/generate-from-prompt`, then `POST /simulations` with `type: "WEB"`, `web_mode: "visual_journey"`, `content: <url>`, `goal`. Poll `GET /simulations/{id}` and read `GET /simulations/{id}/results`.
5. **If an audience review comes back** (the recruited crowd does not match), show the warning to the user. Continue only with their decision: `review_id` + `confirm_audience` + `audience_fit_acknowledged`, or `audience_correction` to recruit again.

## Report

- Lead with the answer: how many of N would stay, leave or act, and the top two or three reasons, each backed by a short exact quote from a named simulated person.
- Then concrete changes: the exact headline, section or claim to change and how.
- Say "simulated" every time. Use counts, not percentages. Do not promise conversion lift.
- Offer the next step: fix the top issue, then run the same test again to compare.

## Access

- **MCP (fastest):** add `https://mcp.mimiqai.com/mcp` as a Streamable HTTP server. Without a key, an agent gets one free test. With a key: `claude mcp add --transport http mimiq https://mcp.mimiqai.com/mcp --header "Authorization: Bearer $MIMIQ_API_KEY"`.
- **REST:** base `https://api.mimiqai.com/api`, header `Authorization: Bearer $MIMIQ_API_KEY`. Read `GET /usage` before spending.
- **Key:** free account at https://www.mimiqai.com/sign-up?redirect_url=/app/settings, then Settings, "Use Mimiq from your coding agent", "Create a key". Keep it in the environment or a secret store, never in prompts or committed files.

## More

For anything beyond one page (flows, copy, ads, comparisons, interviews), use the general skill `simulated-user-testing` from this repo. Full contracts: [REST](https://github.com/victorgulchenko/mimiq-skills/blob/main/skills/simulated-user-testing/references/api.md), [MCP](https://github.com/victorgulchenko/mimiq-skills/blob/main/skills/simulated-user-testing/references/mcp.md).
