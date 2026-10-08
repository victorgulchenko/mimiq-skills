---
name: pick-a-headline
description: Compare two headlines, taglines, hero lines, subject lines, button labels or short messages on the same simulated people with Mimiq and get a call on which to ship (clear, leaning, or too close to call). Use when the user has two versions of copy and asks which is better, clearer or more likely to make people act, or wants to A/B test wording before launch.
---

# Pick a headline

Answers: which of two versions should ship? The same simulated people see version A and version B. Mimiq's call combines a forecast of how people like them behave (asked in both orders) with how those same people moved, and says clear, leaning, or too close to call.

## Run it

1. **Get both versions** exactly as they would appear. Change one thing between them when possible, so the result says something about that change.
2. **Pick the audience** in plain words: who sees this copy in real life. Omit it to let Mimiq infer it.
3. **Pick the count.** 10 simulated people per version is a good default (minimum 3).
4. **Call it.**
   - MCP: `mimiq.compare_copy` with `version_a`, `version_b`, `audience`, `count`, and `format: "copy"` (headline, tagline, short message) or `format: "email"` (an email; a first line `Subject: ...` is read as the subject). Without a key, one free A/B on up to 10 simulated people per version is available while the daily free allowance lasts.
   - REST: generate one audience, then run two `POST /simulations` with the same `audience_id` and identical `persona_ids`, `type: "TEXT"`, `media: "copy"`, one per version. Compare matched `persona_id` rows.
5. If a headline only makes sense on its page, compare the two pages instead (see `compare-two-pages`).

## Report

- Lead with the call exactly as returned: which version, or "too close to call". Never turn "too close to call" into a winner.
- Give the one-sentence reason, how many simulated people moved toward each version, and each version's top objection in their own words.
- Suggest a stronger third version if both share the same objection.
- Say "simulated". The forecast has held up best on headlines; still present it as a strong hint, not a measured result, and do not promise lift.

## Access

- **MCP (fastest):** add `https://mcp.mimiqai.com/mcp` as a Streamable HTTP server. With a key: `claude mcp add --transport http mimiq https://mcp.mimiqai.com/mcp --header "Authorization: Bearer $MIMIQ_API_KEY"`.
- **REST:** base `https://api.mimiqai.com/api`, header `Authorization: Bearer $MIMIQ_API_KEY`. Read `GET /usage` before spending.
- **Key:** free account at https://www.mimiqai.com/sign-up?redirect_url=/app/settings, then Settings, "Use Mimiq from your coding agent", "Create a key". Keep it in the environment or a secret store.

## More

For anything else use `simulated-user-testing`. Full contracts: [REST](https://github.com/victorgulchenko/mimiq-skills/blob/main/skills/simulated-user-testing/references/api.md), [MCP](https://github.com/victorgulchenko/mimiq-skills/blob/main/skills/simulated-user-testing/references/mcp.md).
