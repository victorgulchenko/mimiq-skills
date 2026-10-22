# Recipes: tasks for AI agents

Starting points, not an allowed-use list. Adapt the task to whatever the user names. Counts are small starting runs, subject to the user's budget and `GET /usage` (each agent uses 5 credits).

Every example assumes an authorized, publicly reachable preview, staging or demo with test data, where submissions are safe. A sentence saying "stop before" is not a technical safeguard: do not run against production accounts, real payments, real bookings or messages to third parties without explicit authorization.

Pattern: "Starting at [entry], try to [task] using [test data]. Success is [screen or state]. Stop before [out-of-scope action]."

| # | User ask | Goal for the agents | Who | Agents |
| --- | --- | --- | --- | --- |
| 1 | "Check our sign-up." | "Starting on the home page, create an account and reach the dashboard." | First-time users the product is for | 3 |
| 2 | "Test onboarding to the first useful result." | "Sign up, load the sample data, and reach the first populated report." | Small-retailer operations managers | 3 |
| 3 | "Try checkout with this test coupon." | "Add the cheapest item, apply the code TEST10, and reach order review with the discount shown. Stop before payment." | Occasional online shoppers new to this store | 3 |
| 4 | "Is cancellation discoverable?" | "From the dashboard of the test account, find how to cancel the subscription and reach the final confirmation screen. Stop before confirming." | Subscribers who rarely use the service | 3 |
| 5 | "Can the weekly digest be turned off?" | "Find notification settings and save the weekly digest as off." | Busy team admins who rarely change settings | 3 |
| 6 | "Check appointment booking." | "Find the next free afternoon slot with the demo practitioner and reach the booking review screen. Do not book." | First-time visitors to a clinic site | 3 |
| 7 | "Can search find the right item?" | "Find a waterproof backpack under $80 that fits a 15 inch laptop. Success is a matching product page." | Commuters comparing backpacks | 3 |
| 8 | "Check the upgrade path." | "Find which plan includes exports and reach the upgrade review screen. Stop before purchase." | Free-plan analysts who need an export | 3 |
| 9 | "Does the form explain its errors?" | "Fill in the application with the postcode 0000, understand the error, fix it with 10115, and reach the next step." | First-time applicants | 3 |
| 10 | "Can people find the new feature?" | "Starting on the dashboard, find how to schedule a report and open its settings. Do not schedule a real report." | Returning users who know only the old dashboard | 3 |
| 11 | "Is our pricing page clear?" | "Pick the plan that fits a team of five and get to checkout. Stop before payment." | Team leads at small agencies | 5 |
| 12 | "Check it on phones." | REST `devices: "phone"`, any goal above. | As above | 3 |
| 13 | "Check the preview behind Vercel protection." | Any goal, with `access.vercel_bypass` from a secret. | As above | 3 |
| 14 | "Test the pages behind our login." | "Sign in with the test account, open Settings, and change the company name to Test Co." with `access.login` from the user. | As above | 3 |
| 15 | "Check your own change before handing it over." | "On the preview, start at the empty reports screen, find the demo-data action, and reach a populated report." | First-time analysts | 3 |

For recipe 12, confirm from the returned screenshots that the run used a phone layout before calling it a mobile result. For 13 and 14, pass secrets only through `access`, never in `goal`, and ask the user before passing any password.
