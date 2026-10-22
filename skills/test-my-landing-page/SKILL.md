---
name: test-my-landing-page
description: Have AI agents land on a landing, pricing or product page with Mimiq and try to do what it asks (start a trial, pick a plan, book a demo, join a waitlist), then report what broke, what confused them and where they stopped. Use when the user asks whether visitors can figure out the page and act on it, why a page gets no sign-ups, or before launching or changing a page.
---

# Test my landing page

Answers: can a newcomer work out what this is and do what the page asks, and what stops them? Agents act as the page's intended visitors, each in a real browser, and try the page's main action.

## Run it

1. **Get the public URL** of the page (a preview is fine).
2. **Write the goal** as the page's main action: "Work out what this product does and start the free trial. Stop at the first screen that asks for payment." For a pricing page: "Pick the plan that fits a team of five and get to checkout. Stop before payment."
3. **Pick who** in plain words (the visitors the page is for), or leave it out.
4. **Pick the count:** 3 to 5 agents, 5 credits each, within the user's budget.
5. **Run** `mimiq.test_flow` (MCP) with `url`, `goal`, `audience`, `count`, or the REST flow lifecycle.

## Report

- How many reached the action, and where the rest stopped.
- What the browser caught on the page (errors, failed or slow requests).
- What they misunderstood, in short exact quotes, tied to the screen: the headline, the pricing table, the call to action.
- Concrete changes, most-hit first. Say "simulated", use counts, and do not predict conversion rates.

Access, keys and contracts: see `simulated-user-testing`.
