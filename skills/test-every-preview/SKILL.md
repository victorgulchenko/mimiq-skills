---
name: test-every-preview
description: Set up the Mimiq preview check GitHub Action in a repository so AI agents use every Vercel, Netlify or Cloudflare Pages preview deploy like users and keep one pull request comment with what broke and where they got stuck. Use when the user wants user testing, QA or a usability check on every pull request or deploy, in CI, or without remembering to run it.
---

# Test every preview

Adds one workflow file. When a preview deploy succeeds, agents try a task on it and the pull request gets one comment, updated on every push: how many finished, what the browser caught (errors, failed and slow requests), where the rest got stuck, and the report link.

## Set it up

1. **Key:** the user creates a free Mimiq account, then Settings, "Use Mimiq from your coding agent", "Create a key", and stores it as the repository secret `MIMIQ_API_KEY`. Never write the key into a file.
2. **Workflow:** create `.github/workflows/mimiq.yml`:

```yaml
name: Mimiq preview check
on: deployment_status
permissions:
  pull-requests: write
jobs:
  mimiq:
    if: github.event.deployment_status.state == 'success'
    runs-on: ubuntu-latest
    steps:
      - uses: victorgulchenko/mimiq-preview-check@v1
        with:
          api-key: ${{ secrets.MIMIQ_API_KEY }}
          goal: Sign up and reach the dashboard
          people: 3
```

3. **Set the goal** to the task that matters most for this app, in plain words.
4. **Protected previews:** Vercel Deployment Protection needs `vercel-bypass-secret: ${{ secrets.VERCEL_AUTOMATION_BYPASS_SECRET }}`; a password prompt needs `basic-auth-username` and `basic-auth-password`; pages behind a sign-in need `login-email` and `login-password` for a test account. All from secrets.
5. **Gating:** `fail-on` is `never` by default. After a week of comments, the user can choose `gave-up` (fail when an agent could not continue) or `stuck`.

## Cost

Each agent uses 5 credits, so 3 agents are 15 credits a deploy. When credits run out, the comment says so and links the top-up; the build does not fail for that.

Inputs and outputs: https://github.com/victorgulchenko/mimiq-preview-check. Any other CI can call the MCP endpoint over HTTP: see `simulated-user-testing`.
