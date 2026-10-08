---
name: compare-two-pages
description: Compare two live web pages (a redesign against the current page, a preview deployment against production, or two landing page variants) on the same simulated people with Mimiq and get a call on which to ship. Use when the user asks which version of a page is better, whether a redesign or change is an improvement, or wants an A/B test of two URLs before real traffic.
---

# Compare two pages

Answers: which of two pages should ship? The same simulated people see page A and page B, and Mimiq returns a call (clear, leaning, or too close to call), how many moved toward each page, and each page's top objections.

## Run it

1. **Get both URLs.** Both must be public: production plus a preview deployment works well. Keep everything except the change under test the same when you can.
2. **Pick the audience** in plain words: the visitors these pages are for. Omit it to let Mimiq infer it.
3. **Pick the count.** 10 simulated people per page is a good default (minimum 3).
4. **Call it.**
   - MCP: `mimiq.compare_urls` with `url_a`, `url_b`, `audience`, `count`. Without a key, one free A/B on up to 10 simulated people per page is available while the daily free allowance lasts.
   - REST: generate one audience, then two `POST /simulations` with the same `audience_id` and identical `persona_ids`, `type: "WEB"`, `web_mode: "visual_journey"`, one per URL. Compare matched `persona_id` rows.
5. **To compare how well people complete a task on each version**, run `mimiq.test_flow` on each URL with the same goal and audience description, and compare completions and stopping points. Separate flow calls recruit separate simulated people; say so.

## Report

- Lead with the call exactly as returned. "Too close to call" is a valid answer, not a tie to break.
- Give the reason, how many simulated people moved toward each page, and each page's top objection in their words.
- If the change did not move people, say what did bother them on both pages: that is usually the next thing to fix.
- Say "simulated". On whole web pages the forecast is a hint, not a prediction of conversion; never promise lift.

## Access

- **MCP (fastest):** add `https://mcp.mimiqai.com/mcp` as a Streamable HTTP server. With a key: `claude mcp add --transport http mimiq https://mcp.mimiqai.com/mcp --header "Authorization: Bearer $MIMIQ_API_KEY"`.
- **REST:** base `https://api.mimiqai.com/api`, header `Authorization: Bearer $MIMIQ_API_KEY`. Read `GET /usage` before spending.
- **Key:** free account at https://www.mimiqai.com/sign-up?redirect_url=/app/settings, then Settings, "Use Mimiq from your coding agent", "Create a key".

## More

For other tests use `simulated-user-testing`. Full contracts: [comparisons](https://github.com/victorgulchenko/mimiq-skills/blob/main/skills/simulated-user-testing/references/api.md#comparisons), [MCP](https://github.com/victorgulchenko/mimiq-skills/blob/main/skills/simulated-user-testing/references/mcp.md).
