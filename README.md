# Mimiq skills

Let AI agents use your app before your users do. These skills teach a coding agent to send [Mimiq](https://www.mimiqai.com)'s agents through a web app: each agent acts as one of your users in a real browser, tries a task in plain words, and reports what broke, what the browser caught (JavaScript errors, failed requests, requests slower than 3 seconds), and where it got stuck, with steps and screenshots.

A skill is a set of instructions, not an API key or a credit allowance. The skills need a Mimiq MCP connection or a client that can make HTTP requests.

## The skills

| Skill | Use it to |
| --- | --- |
| [simulated-user-testing](skills/simulated-user-testing/SKILL.md) | Test any web app or task with AI agents acting as users: sign-up, onboarding, checkout, settings, booking, search, a new feature. Start here if unsure. |
| [test-before-done](skills/test-before-done/SKILL.md) | After building a user-facing change, run agents on the preview, fix what they hit, and run again before calling it done. |
| [check-my-sign-up-flow](skills/check-my-sign-up-flow/SKILL.md) | Find what breaks and where new users stall in sign-up, onboarding, checkout or any multi-step task. |
| [test-my-landing-page](skills/test-my-landing-page/SKILL.md) | Have agents land on a page and try to do what it asks: start a trial, pick a plan, book a demo. |
| [test-every-preview](skills/test-every-preview/SKILL.md) | Set up the GitHub Action that runs agents on every Vercel, Netlify or Cloudflare Pages preview and comments on the pull request. |

## Install

In Claude Code, add the plugin. It brings the skills and the Mimiq MCP server in one step:

```
/plugin marketplace add victorgulchenko/mimiq-skills
/plugin install mimiq@mimiq
```

Set `MIMIQ_API_KEY` in your environment to use your account; without it, the MCP server gives one free run of up to 3 agents.

For other clients, install the skills on their own:

```sh
npx skills add victorgulchenko/mimiq-skills
```

## Set up

The fastest start is the hosted MCP server: add `https://mcp.mimiqai.com/mcp` as a Streamable HTTP server.

For more runs:

1. Create a free account at [mimiqai.com](https://www.mimiqai.com/sign-up?redirect_url=/app/settings) (50 credits; each agent uses 5).
2. Open [Settings](https://www.mimiqai.com/app/settings). Under **Use Mimiq from your coding agent**, choose **Create a key**. Store it in your client's secret store, optionally as `MIMIQ_API_KEY`. Never put keys in prompts or files.
3. Send it as `Authorization: Bearer $MIMIQ_API_KEY`, for MCP for example `claude mcp add --transport http mimiq https://mcp.mimiqai.com/mcp --header "Authorization: Bearer $MIMIQ_API_KEY"`, or to the REST API at `https://api.mimiqai.com/api`.

Use a publicly reachable preview or staging environment that safely accepts test sign-ups and form submissions: agents really click and submit. For protected previews and pages behind a sign-in, pass `access` with a bypass secret or a test account. Buying credits, opening tunnels, publishing code and messaging third parties each need your explicit authorization.

## Make it part of every change

Add these lines to your project's `CLAUDE.md` or `AGENTS.md`, and your coding agent checks a user-facing change before it calls it done:

```markdown
## Before calling a user-facing change done
After changing anything a user goes through (signup, onboarding, checkout,
settings, a new feature), run mimiq.test_flow on the preview URL with the task
a user would try. Fix what the agents hit, then run it again. Ask me before
using more than 25 credits.
```

## Use

- "Have three agents sign up on this preview and tell me what breaks."
- "Check the new checkout on the preview before I merge. Apply the test coupon and stop before payment."
- "Can three agents find cancellation in our test account? Stop at the confirmation screen."
- "Run Mimiq on staging with parents shopping for school shoes, then fix what they hit."
- "Sign in with this test account on the protected preview and change the company name."
- "Set up Mimiq on every preview deploy for this repo."

## What the results are

The agents are simulated users. The browser's records (errors, failed and slow requests) are observed facts; the agents' words show where a newcomer gets confused or gives up. Agents find problems; they do not forecast conversion, and they are not a deterministic regression suite.

## License

MIT. See [LICENSE](LICENSE).
