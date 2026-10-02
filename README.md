# Mimiq skills

Three agent skills for simulated feedback before you launch:

| Skill | Use it to |
| --- | --- |
| [test-my-landing-page](skills/test-my-landing-page/SKILL.md) | Find confusing copy and objections on a public landing page. |
| [check-my-sign-up-flow](skills/check-my-sign-up-flow/SKILL.md) | See where simulated visitors get stuck in a sign-up journey. |
| [pick-a-headline](skills/pick-a-headline/SKILL.md) | Compare two headlines on the same simulated audience. |

Each skill runs a test through [Mimiq](https://www.mimiqai.com), where simulated customers use your live page or flow and say what stopped them. The skills need a client that can make authenticated HTTP requests, or an installed Mimiq MCP connection. A skill is a set of instructions, not an API key or a credit allowance.

## Install

```sh
npx skills add victorgulchenko/mimiq-skills --list
npx skills add victorgulchenko/mimiq-skills
```

Pick the skills and your client when the installer asks. To install one skill only:

```sh
npx skills add victorgulchenko/mimiq-skills --skill test-my-landing-page
```

## Set up

1. Create a free account at [mimiqai.com](https://www.mimiqai.com/sign-up?redirect_url=/app/settings). The first test is free.
2. Open [Settings](https://www.mimiqai.com/app/settings). Under **API keys**, click **Create a key**, then **Copy key**. Store it in your client's secret store or local environment as `MIMIQ_API_KEY`. Do not paste it into a prompt or commit it.
3. Check your credits under **Plan and usage** before a run. Audiences and tests use credits; a flow costs more per simulated person than a page, and `GET /api/usage` gives the current numbers.

## Use

Ask a concrete question, for example:

> Test my landing page at https://example.com for independent bookkeepers. Use 5 simulated people and look for objections to starting a trial.

The agent creates a simulated audience, runs the test, and returns findings tied to the actual result rows and simulation IDs. Sample outputs inside the skills are illustrative, not recorded runs.

## MCP instead of the API

Add `https://mcp.mimiqai.com/mcp` as a Streamable HTTP server in your client's MCP settings, and set its `Authorization` header to `Bearer <your Mimiq API key>` through the client's secure credential field. Reload the client and check its tool list. Installing a skill does not install the MCP server.

## What the results are

Everyone in a Mimiq test is simulated. Use the findings as hypotheses to check with real visitors, not as measured conversion or a forecast of lift.

## License

MIT. See [LICENSE](LICENSE).
