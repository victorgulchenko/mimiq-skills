---
name: would-they-pay
description: Ask a simulated target audience whether they would pay for a product, plan or feature, at what price, and why not, with Mimiq. Use when the user asks if people would pay, what to charge, which plan or package to offer, which feature to build next, whether an idea is worth building, or wants quick customer-discovery answers from a specific audience before talking to real people.
---

# Would they pay?

Answers: would the people this is for pay, which option would they pick, and what stops them? Simulated people answer on their own, with their reasoning.

## Run it

1. **Write the question** the way you would ask a real customer, with the price and what they get: "Would you pay $12 a month for an app that plans your weekly meals and orders the groceries?"
2. **Pick the audience** in plain words, as specific as possible: "parents of two or more kids in US suburbs who cook most nights", not "consumers".
3. **Pick the shape.**
   - **Choices** (price points, plans, features to build): MCP `mimiq.ask_audience` with `audience`, `question` (up to 500 characters), `options` (2 to 10, include an honest "No, I would not pay" or "Something else"), `count`, and `context`. REST: `POST /surveys/simulate` with `audience_prompt`, `question`, `options`, `count`.
   - **Open answers** (why or why not, what they use today): REST `POST /simulations` with `type: "TEXT"`, `content` and `goal` set to the question, and `goal_schema: {"goal_type": "interview"}` on a generated audience. With MCP only, `mimiq.test_text` with the question as `text`.
4. **Pick the count.** 10 to 20 simulated people for choices, 5 for open answers. Stay inside the user's budget.
5. **Go deeper** on an interesting answer: REST `POST /personas/chat` with that person's recorded profile and result, or `POST /rooms/ask` to ask the whole simulated audience a follow-up.

## Report

- Lead with the distribution as counts ("7 of 15 picked $12, 5 would not pay"), then the main reasons in quotes.
- Separate "would pay" from what they say would change their mind (price, proof, trial, a missing feature).
- Suggest the next real-world step: the price or plan to test with real customers, and the question to ask them.
- Say "simulated" every time. Stated willingness to pay is a hypothesis, not demand or revenue. Never present it as a market size.

## Access

- **MCP:** add `https://mcp.mimiqai.com/mcp` as a Streamable HTTP server. Without a key, an agent gets one free test. With a key: `claude mcp add --transport http mimiq https://mcp.mimiqai.com/mcp --header "Authorization: Bearer $MIMIQ_API_KEY"`.
- **REST:** base `https://api.mimiqai.com/api`, header `Authorization: Bearer $MIMIQ_API_KEY`. Read `GET /usage` before spending.
- **Key:** free account at https://www.mimiqai.com/sign-up?redirect_url=/app/settings, then Settings, "Use Mimiq from your coding agent", "Create a key".

## More

For other tests use `simulated-user-testing`. Full contracts: [questions and interviews](https://github.com/victorgulchenko/mimiq-skills/blob/main/skills/simulated-user-testing/references/api.md#questions-and-interviews), [MCP](https://github.com/victorgulchenko/mimiq-skills/blob/main/skills/simulated-user-testing/references/mcp.md).
