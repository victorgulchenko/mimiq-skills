# Mimiq skills

Test anything your audience sees or uses with simulated people, then turn their reactions into useful next changes.

One skill, [simulated-user-testing](skills/simulated-user-testing/SKILL.md), for any page, any task in a live product, copy, images, email, product ideas, variant comparisons, and questions or interviews with a simulated target audience. An agent can also use it to check its own UI work before handing it over.

The skill runs through [Mimiq](https://www.mimiqai.com). It needs a client that can make authenticated HTTP requests or a configured Mimiq MCP connection. A skill is a set of instructions, not an API key or a credit allowance.

## Install

```sh
npx skills add victorgulchenko/mimiq-skills
```

Pick `simulated-user-testing` and your client when the installer asks. To see the available skill first:

```sh
npx skills add victorgulchenko/mimiq-skills --list
```

## Set up

1. Create or sign in to an account at [mimiqai.com](https://www.mimiqai.com/sign-up?redirect_url=/app/settings).
2. Open [Settings](https://www.mimiqai.com/app/settings). Under **API keys**, create and copy a key. Store it in your client's secret store, optionally exposed to the HTTP client as `MIMIQ_API_KEY`. Never put API keys in prompts or files.
3. Check **Plan and usage** before a run. Read `GET /usage` at the API base `https://api.mimiqai.com/api` for the current balance, limits, and `flow_credits_per_person`. Audience generation and tests use credits. Set a count and credit budget the agent may use.

Use an already authorized, publicly reachable preview for browser tests. Flow tests click and type and can submit forms, so use disposable test data in an environment that safely accepts those actions. Buying credits, opening tunnels, publishing code, and messaging third parties each need your explicit authorization.

## Use

- “Review our pricing page for 5 simulated independent accountants. What makes the plans hard to choose?”
- “Check our disposable checkout demo with 3 simulated shoppers. Apply the supplied test coupon and reach the order review screen.”
- “Can 3 simulated subscribers find cancellation in our test account? Stop at the confirmation screen.”
- “Compare these two hero headlines with the same 5 simulated small-business owners. Which makes the offer clearer?”
- “Test this ad image and caption with 5 simulated cycling commuters. What do they notice or misunderstand?”
- “Review this cold email with 5 simulated operations managers. What would make them read or dismiss it?”
- “Ask 5 simulated museum archivists how they handle missing image permissions, then interview one about the biggest obstacle.”
- “Before handing over your UI change, check the empty state and route to first value on the authorized preview with 3 simulated first-time subscribers.”

The agent chooses the mode, defines the simulated audience, runs within your budget, and ties findings to the returned evidence. [Recipes](skills/simulated-user-testing/references/recipes.md) show more starting points. The task can be any goal you name.

## MCP instead of REST

Add `https://mcp.mimiqai.com/mcp` as a Streamable HTTP server in your MCP client. Configure its `Authorization` header through the client's secure credential field using a Bearer API key. Reload and inspect the live tool list and schemas. Installing the skill does not install the MCP connection.

The [MCP reference](skills/simulated-user-testing/references/mcp.md) covers the six published tools and their limits. The [REST reference](skills/simulated-user-testing/references/api.md) also covers images, email, interviews, and other capabilities without a dedicated MCP tool.

## What the results are

All Mimiq reactions and journeys are simulated. They help form hypotheses about comprehension, objections, and usability. They are not measured conversion, proof of task reliability, or a substitute for real-world research and functional testing.

## License

MIT. See [LICENSE](LICENSE).
