---
name: pick-a-headline
description: Compare two headlines with the same simulated audience using Mimiq and recommend a direction from recorded attention, actions, and objections. Use when choosing between supplied copy variants before a real audience test.
---

# Pick a headline

Compare two supplied headlines as a controlled simulated copy test. Read [API access and lifecycle](references/api.md) for setup, credits, and the current contract.

Use the actual two variants, intended audience, placement, and decision the user cares about. If only one headline was supplied, ask for the alternative or draft alternatives only if requested. Do not invent product claims. Keep the accompanying copy and offer identical across variants. Record their labels locally so that separate simulation IDs cannot be swapped.

1. Check `GET /usage`. Default to 10 simulated people per variant within the authorized budget. Two variants require two tests, and the reused audience's second test also uses credits. If the user prohibits paid calls or asks for a dry run, provide the planned comparison without making generation or simulation requests.
2. Generate one audience through `POST /personas/generate-from-prompt`. Review `audience_fit` and the actual returned size. Use the same audience and persona subset for both variants; do not recruit a different audience for B.
3. Create two `POST /simulations` requests, one for each content value. Use distinct idempotency keys. This example represents A; replace only `content` for B:

   ```json
   {
     "project_id": "mimiq-skills",
     "audience_id": "<same returned audience_id>",
     "type": "TEXT",
     "media": "copy",
     "content": "A simpler way to chase overdue invoices"
   }
   ```

   The current `media: "copy"` path frames content in a feed. It does not render a landing-page layout or emulate a search-results screen. For a layout-dependent headline decision, describe this limitation or test two actual page variants with Page mode when the user has authorized that larger task. Avoid adding a custom `goal_schema`; it selects a different text path. In this copy path `context` does not customize the feed setting.
4. Poll each simulation and collect completed results. Keep failures visible. If either run fails or returns incompatible evidence, do not declare a winner. Prefer comparison on matched usable `persona_id` values, and disclose exclusions.
5. Compare the current copy actions `act`, `read`, `glanced`, and `ignored`; read `monologue`, `gut_reaction`, `objections`, and `what_would_help`. Distinguish simulated attention from positive sentiment. A praised headline can still be ignored.
6. Recommend A, B, or “no clear simulated preference.” Give the usable sample size for each, action counts, the strongest evidence on both sides, and one next real-world test. Counts from a small simulated cohort do not establish significance, measured click-through, numeric lift, or expected revenue. Never turn the API's internal scores into accuracy claims.

An installed MCP tool `mimiq.test_copy` with `variant_a` and `variant_b` may replace the API workflow when its discovered schema supports the same task. Inspect the returned comparison and cohort information rather than assuming it matches the REST `media: "copy"` method. Identify the method used and do not compare unlike outputs as if they were one controlled test.

## Example request

“Pick a headline for a feed post aimed at independent bookkeepers. A: ‘A simpler way to chase overdue invoices’. B: ‘Spend Friday on clients, not payment reminders’. Use 10 simulated people per variant.”

## Illustrative output, not a recorded run

> B is the better next headline to test with real visitors in this illustrative simulated comparison. A and B each have 10 usable reactions from the same audience.
>
> More simulated people read B; A more clearly named the task. The main objection to B was uncertainty about what the product actually does. Keep B's benefit and add a concrete subheading about payment reminders.
>
> This is a simulated preference, not measured click-through or a lift forecast. Run a real test with the same placement and offer before treating it as a conversion improvement.

Use the actual action counts and quotes in a real response. Do not reuse this example's recommendation.
