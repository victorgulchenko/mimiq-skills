---
name: test-before-done
description: After you build or change anything a user goes through (sign-up, onboarding, checkout, settings, a new feature, a form), run Mimiq on the preview so AI agents use the change like users before you call it done; read what broke, fix it, and run again. Use at the end of any user-facing change, when the user asks you to verify your work in the browser, or when a CLAUDE.md or AGENTS.md rule asks for it.
---

# Test before done

You built the change, so you cannot see it like a stranger. Before you say it is done, let agents who can try it.

## The loop

1. **Find the preview URL.** The deployment preview for the branch (Vercel, Netlify, Cloudflare Pages), staging, or the published app. For a local dev server you need a temporary public tunnel: tell the user and get their OK before opening one, and close it afterwards.
2. **Phrase the task the change affects,** from where a user would start: "Starting on the home page, sign up and create your first project." Not the click path you implemented.
3. **Run** `mimiq.test_flow` with `url`, `goal` and `count: 3`. Add `access` only with test credentials the user gave you for a protected preview or a sign-in.
4. **Read the result.** Browser records first (`browser_caught`: errors, failed and slow requests), then where agents stopped and why.
5. **Fix what is yours.** If the problem is in the change, fix it, redeploy the preview, and run the same task again. If it is outside the change, report it instead of widening the work.
6. **Report** in your summary: how many of 3 finished before and after, what you fixed, anything left, and the report link.

## Budget and limits

Each agent uses 5 credits, so a check is 15 credits. Without a key there is one free run. Stay within the budget the user gave; ask before going over. Agents cannot pay with real cards and can be stopped by CAPTCHAs; say so rather than calling it a bug in the change.

## Make it automatic

The user can add this to CLAUDE.md or AGENTS.md:

```markdown
## Before calling a user-facing change done
After changing anything a user goes through (signup, onboarding, checkout,
settings, a new feature), run mimiq.test_flow on the preview URL with the task
a user would try. Fix what the agents hit, then run it again. Ask me before
using more than 25 credits.
```

Setup and contracts: see `simulated-user-testing`.
