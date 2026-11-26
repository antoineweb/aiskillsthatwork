# AI Skills That Work

Free, tested skills for Claude. Each one comes with a demo you can watch before you install it.

A skill is a small instruction file that teaches an AI agent one job and makes it do that job the same way every time. Nothing is published here until it has done a real job end to end.

## Skills

| Skill | What it does | Demo |
|---|---|---|
| [product-demo-guide](skills/product-demo-guide) | Turns a flow in your product into an animated step-by-step guide (simplified but recognizable UI, your branding, steps and milestones on the right), then exports it to MP4. | [Watch on the site](https://aiskillsthatwork.vercel.app/#product-demo-guide) |
| [voice-to-video](skills/voice-to-video) | Turns a raw voice recording into a finished, captioned video scheduled to your channel: silence-cut, local transcription, render, frame QA, and a review gate before anything goes live. | [How it runs](https://aiskillsthatwork.vercel.app/#voice-to-video) |

## Install

1. Download the skill folder, or the zip from the website.
2. Add it to your Claude skills.
3. Ask for what the skill does, for example: "Make a demo guide of how to create a ticket in our app."

## Other agents

Skills are plain Markdown. They are written and tested for Claude. Any agent that reads instruction files can use them with small changes.

## Repository layout

- `skills/`: one folder per skill, each with a `SKILL.md`.
- `site/`: the landing page, including a live example of the demo guide template.

## Licence

MIT. See [LICENSE](LICENSE).

Made by Antoine Schuppisser.
