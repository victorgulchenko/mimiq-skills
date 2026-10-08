# Mimiq skills

Test anything your audience sees or uses with simulated people, then turn their reactions into useful next changes.

The skills run through [Mimiq](https://www.mimiqai.com). They need a client that can make HTTP requests or a Mimiq MCP connection. A skill is a set of instructions, not an API key or a credit allowance.

## The skills

| Skill | Use it to |
| --- | --- |
| [simulated-user-testing](skills/simulated-user-testing/SKILL.md) | Test anything: any page, any task in a live product, copy, images, video frames, email, ideas, pricing, A/B variants, questions and interviews with a target audience, or your own UI work before handing it over. Start here if unsure. |
| [test-my-landing-page](skills/test-my-landing-page/SKILL.md) | Hear who would stay on a page, who would leave, and why. |
| [check-my-sign-up-flow](skills/check-my-sign-up-flow/SKILL.md) | Send simulated people through sign-up, onboarding, checkout or any task in a real browser and find where they get stuck. |
| [pick-a-headline](skills/pick-a-headline/SKILL.md) | Compare two headlines or short messages on the same simulated people and get a call on which to ship. |
| [compare-two-pages](skills/compare-two-pages/SKILL.md) | Compare two live pages, such as a preview against production, and get a call on which to ship. |
| [test-my-ad](skills/test-my-ad/SKILL.md) | See what people notice in an ad or image and whether they would stop or scroll past. |
| [test-my-cold-email](skills/test-my-cold-email/SKILL.md) | Hear whether recipients would open, read or reply to an email, without sending it. |
| [would-they-pay](skills/would-they-pay/SKILL.md) | Ask a target audience whether they would pay, which plan or price they would pick, and why not. |

The general skill covers everything the short ones do; the short ones are quicker to find and to follow for one job.

## Install

```sh
npx skills add victorgulchenko/mimiq-skills
```

Pick the skills and your client when the installer asks. To install one skill directly, or to see the list first:

```sh
npx skills add victorgulchenko/mimiq-skills --skill simulated-user-testing
npx skills add victorgulchenko/mimiq-skills --list
```

## Set up

The fastest start is the hosted MCP server: add `https://mcp.mimiqai.com/mcp` as a Streamable HTTP server. Without a key, an agent gets one free test.

For more, and for every mode over REST:

1. Create a free account at [mimiqai.com](https://www.mimiqai.com/sign-up?redirect_url=/app/settings).
2. Open [Settings](https://www.mimiqai.com/app/settings). Under **Use Mimiq from your coding agent**, choose **Create a key**. Store it in your client's secret store, optionally exposed as `MIMIQ_API_KEY`. Never put API keys in prompts or files.
3. Send it as `Authorization: Bearer $MIMIQ_API_KEY`, for MCP for example `claude mcp add --transport http mimiq https://mcp.mimiqai.com/mcp --header "Authorization: Bearer $MIMIQ_API_KEY"`, or to the REST API at `https://api.mimiqai.com/api`.
4. Check the balance before a run with `GET /usage`. Tests use credits. Set a count and credit budget the agent may use.

Use an already authorized, publicly reachable preview for browser tests. Flow tests click and type and can submit forms, so use disposable test data in an environment that safely accepts those actions. Buying credits, opening tunnels, publishing code, and messaging third parties each need your explicit authorization.

## Use

- “Review our pricing page for 5 simulated independent accountants. What makes the plans hard to choose?”
- “Check our disposable checkout demo with 3 simulated shoppers. Apply the supplied test coupon and reach the order review screen.”
- “Can 3 simulated subscribers find cancellation in our test account? Stop at the confirmation screen.”
- “Compare these two hero headlines with the same 10 simulated small-business owners. Which should we ship?”
- “Compare our preview deployment with production on 10 simulated first-time visitors.”
- “Test this ad image and caption with 5 simulated cycling commuters. What do they notice or misunderstand?”
- “Review this cold email with 5 simulated operations managers. What would make them reply or delete it?”
- “Would 15 simulated parents of young kids pay $12 a month for this? Which plan would they pick?”
- “Ask 5 simulated museum archivists how they handle missing image permissions, then interview one about the biggest obstacle.”
- “Before handing over your UI change, check the empty state and route to first value on the authorized preview with 3 simulated first-time subscribers.”

The agent chooses the mode, defines the simulated audience, runs within your budget, and ties findings to the returned evidence. [Recipes](skills/simulated-user-testing/references/recipes.md) show more starting points. The task can be any goal you name.

## MCP or REST

The [MCP reference](skills/simulated-user-testing/references/mcp.md) covers the nine published tools, including page and copy comparisons and audience review. The [REST reference](skills/simulated-user-testing/references/api.md) also covers images, video frames, inbox email, interviews, follow-ups and other capabilities without a dedicated MCP tool.

## What the results are

All Mimiq reactions and journeys are simulated. They help form hypotheses about comprehension, objections, and usability. They are not measured conversion, proof of task reliability, or a substitute for real-world research and functional testing.

## License

MIT. See [LICENSE](LICENSE).
