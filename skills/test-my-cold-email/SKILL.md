---
name: test-my-cold-email
description: Put a cold email, outreach message, newsletter or launch announcement in front of simulated recipients with Mimiq and hear whether they would open, read, reply or delete it, and why. Use when the user asks if an email or DM will get replies, how to improve a subject line or ask, or which of two email versions to send. Nothing is actually sent.
---

# Test my cold email

Answers: would the people this email is for open it, read it, and do what it asks? Each simulated recipient sees it in an inbox and reacts in their own words. Nothing is sent to anyone.

## Run it

1. **Get the email** with its subject: first line `Subject: ...`, then the body. Keep the real sender context if it matters ("From: a founder at a 5-person agency").
2. **Pick the audience** in plain words: the actual recipients, as specific as the user's list ("operations managers at small logistics firms").
3. **Pick the count.** 5 to 10 simulated people.
4. **Call it.**
   - **One version, REST (needs a key):** generate an audience, then `POST /simulations` with `type: "TEXT"`, `media: "email"`, `content: <sender, subject and body together>`. Omit `goal` and `goal_schema` so it stays an inbox test. The first 4,000 characters are read.
   - **Two versions, MCP:** `mimiq.compare_copy` with `version_a`, `version_b`, `format: "email"`, `audience`, `count` (at least 3). The same simulated people see both. Without a key, one free A/B on up to 10 per version is available while the daily free allowance lasts.
   - **One version, MCP only:** `mimiq.test_text` with the email as `text` reads it as general text, not an inbox. Say so in the report.
5. Never send the email, and never put real recipient addresses or personal data into the test.

## Report

- Lead with counts: how many of N would open, read, reply, or ignore, using the returned actions.
- Quote the line that lost them and the line that worked. Name the objection to the ask itself.
- Rewrite: a sharper subject, a shorter first line, a smaller ask. Offer to test the rewrite against the original with `compare_copy`.
- Say "simulated". These are not open or reply rates.

## Access

- **MCP:** add `https://mcp.mimiqai.com/mcp` as a Streamable HTTP server. With a key: `claude mcp add --transport http mimiq https://mcp.mimiqai.com/mcp --header "Authorization: Bearer $MIMIQ_API_KEY"`.
- **REST:** base `https://api.mimiqai.com/api`, header `Authorization: Bearer $MIMIQ_API_KEY`. Read `GET /usage` before spending.
- **Key:** free account at https://www.mimiqai.com/sign-up?redirect_url=/app/settings, then Settings, "Use Mimiq from your coding agent", "Create a key".

## More

For other tests use `simulated-user-testing`. Full contracts: [REST, text and email](https://github.com/victorgulchenko/mimiq-skills/blob/main/skills/simulated-user-testing/references/api.md#text-and-email), [MCP](https://github.com/victorgulchenko/mimiq-skills/blob/main/skills/simulated-user-testing/references/mcp.md).
