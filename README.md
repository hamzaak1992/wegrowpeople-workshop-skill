# WeGrowPeople Workshop Skill

The official facilitation skill for **Everyday AI for Busy Owners & Managers**.

## For attendees

Load `workshop-runner.md` into your Claude session before/at the start of the workshop, and it'll run you through the day step by step.

**Quick way to load it in Claude Code:**

```
Please read and follow the instructions in this file to run today's workshop with me: https://raw.githubusercontent.com/hamzaak1992/wegrowpeople-workshop-skill/main/workshop-runner.md
```

This file is locked for the day — it's the official run-of-show, not something to edit mid-session.

## Attendee/trainer pages

All built on the real WeGrowPeople brand (fonts, colors, and components pulled from `wegrowpeople-v2`), self-contained single-file HTML — no build step, no server required.

| Page | What it's for |
|---|---|
| `pages/homework.html` | Pre-workshop 8-step locked checklist (quiz-gated, OS-branched install steps) |
| `pages/survey.html` | 6-step pre-event survey, generates a Member ID + personalised recommendation, saves to Google Sheets if configured |
| `pages/trainer-dashboard.html` | Internal: prep tracking, survey insights, live submission feed |
| `pages/dashboard-templates.html` | Preview of the 5 dashboard style templates attendees' builds randomly pick from |
| `pages/cofounder-briefing.html` | Internal: full training breakdown for Jack &amp; Farah |

## Wiring survey → Google Sheets

See `google-sheets-setup.md` for the exact 5-minute setup. Once deployed, drop the resulting Web app URL into the `SHEET_WEBHOOK_URL` constant near the top of both `pages/survey.html` and `pages/trainer-dashboard.html`.

## Brand tokens

`brand/tokens.css` has the extracted color/type tokens from `wegrowpeople-v2` for reuse in any future page.
