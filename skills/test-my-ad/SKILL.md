---
name: test-my-ad
description: Show an ad, social post image, banner, thumbnail, product shot or a set of screenshots to simulated people with Mimiq and hear what they notice, what they think it offers, and what makes them scroll past or doubt it. Use when the user has an ad creative or image (with or without a caption) and asks whether it works, which creative to run, or what to change before spending on it.
---

# Test my ad

Answers: what does the target audience see in this creative, what do they think it offers, and would they stop, click or scroll past? Each simulated person sees the image in a feed and reacts in their own words.

## Run it

1. **Get the image(s)** as PNG or JPEG files and the real caption or post text. Several images in one request are read as a sequence of screens reviewed together, not as A/B versions.
2. **Pick the audience** in plain words: who would see this ad in their feed.
3. **Pick the count.** 5 to 10 simulated people.
4. **Call it (REST, needs a key):** generate an audience with `POST /personas/generate-from-prompt`, then `POST /simulations` with `type: "IMAGE"`, `content: <caption, or "Uploaded image">`, and `images: [<base64 bytes>]` (not a path or URL; up to 10). For a focused question ("Is the offer clear?") put it in `content` and the setting in `goal`. Poll `GET /simulations/{id}` and read `/results`.
5. **To compare two creatives:** one audience, two simulations with the same `audience_id` and identical `persona_ids`, one image each. Compare matched rows.
6. **No key and only MCP?** The hosted MCP has no image tool yet. Describe the creative and its text with `mimiq.test_text`, and say clearly that the people read a description, not the image.

## Report

- Lead with what most simulated people noticed first and what they thought was being offered. If that differs from the intended message, say so plainly.
- Count stops, clicks and scroll-pasts from the returned actions. Quote the strongest objection.
- Give concrete changes: the headline on the image, the visual focus, the offer, or the caption.
- Say "simulated". Returned metrics are simulated signals, not ad performance.

## Access

- **REST:** base `https://api.mimiqai.com/api`, header `Authorization: Bearer $MIMIQ_API_KEY`. Read `GET /usage` before spending.
- **Key:** free account at https://www.mimiqai.com/sign-up?redirect_url=/app/settings, then Settings, "Use Mimiq from your coding agent", "Create a key". Keep it in the environment or a secret store.
- The web app at https://www.mimiqai.com/app also takes an image by drag and drop, if the user prefers to run it themselves.

## More

Video ads work as ordered frames (`media: "video"`, `frame_times`, `duration`). See `simulated-user-testing` and the [REST contract](https://github.com/victorgulchenko/mimiq-skills/blob/main/skills/simulated-user-testing/references/api.md#images-and-video).
