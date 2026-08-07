# Customization

The agent is a prompt — customizing it means editing text, not code. Open your routine at [claude.ai/code/routines](https://claude.ai/code/routines) → edit (pencil) → change the Instructions → Save. The next run uses the new version. (Or just tell Claude in a chat what you want changed and let it edit the routine for you.)

**One rule above all: read any reply wording before you deploy it.** These emails go to real recruiters in your name. The original author of this project made wording review a hard requirement after nearly auto-sending a template they'd never seen — don't repeat that.

---

## Change the reply language(s)

Default: replies always in **English**; rejections understood in **English and German**; other languages are skipped and surfaced.

- *Reply in the rejection's language instead:* in Step 6, replace "Write in English even when the rejection is in German" with "Write in the same language as the rejection. For German, mirror the sender's register (du if they wrote du, otherwise Sie) and never guess Herr/Frau — use 'Guten Tag <full name>' when unsure."
- *Add a language:* extend Step 2.6 (e.g. "English, German or French") and add that language's rejection phrases to the Step 1(a) keyword list (e.g. `"malheureusement"`, `"candidature n'a pas été retenue"`).

## Change the tone or template

Step 6 defines the reply's shape (thank → accept → one pointed ask → well-wishes → sign-off). Edit freely; keep three properties that protect you: **never ask to reconsider** (reads as arguing), **one question only** (two questions get zero answers), **no hedging filler** ("I totally understand if you can't…" invites a non-answer).

## Change the schedule

The cron expression is in **UTC**: `0 7,11,15 * * 1-5` = 07:00/11:00/15:00 UTC, Mon–Fri.

| You want | Cron (UTC) |
|---|---|
| 3×/weekday, Berlin (summer) | `0 7,11,15 * * 1-5` |
| 3×/weekday, India (IST) | `30 3,7,11 * * 1-5` |
| 3×/weekday, US East (EDT) | `0 13,17,21 * * 1-5` |
| Once per weekday, morning | `0 7 * * 1-5` |
| Include weekends | `0 7,11,15 * * *` |

Fewer runs is fine — the 7-day window means nothing is missed, only answered a little later. (Minimum spacing on routines is 1 hour.)

## Draft-only mode (human approves every send)

Don't trust auto-send yet? Two edits make the agent a pure drafter:

1. In Step 6, replace the send instruction with: "Do NOT send. Create a Gmail DRAFT reply with the built-in connector in the rejection's thread, addressed to the Step 4 reply target, and list it in the summary as 'draft ready to send'."
2. In the ground rules, remove GMAIL_REPLY_TO_THREAD from the allowed actions (keep GMAIL_SEND_EMAIL for the summary).

You then review Drafts and press Send yourself. Great as a first-week trust-building mode; switch back by restoring the original Step 6.

## Add ATS / employer domains to the discovery net

Step 1(b) lists sender domains of common recruiting systems. Add any ATS or employer mail domain you encounter (look at the From/Reply-To of confirmations you receive, e.g. `jobs@yourdreamcompany.com` → add `yourdreamcompany.com`). This only improves *recall* — classification still decides what's really a rejection.

## Widen or narrow the time window

`newer_than:7d` appears throughout Step 1; change all occurrences consistently (e.g. `14d`). Keep Step 2.7 (per-message age check) aligned. Remember: replying weeks late reads worse than not replying.

## Change caps

Ground rules: `at most 5 feedback replies per run` and one note per company. Raise cautiously — several near-identical mails from one address in minutes is exactly what spam filters and recruiters notice.

## Change the model

`claude-sonnet-5` is the tested default and handles the checklist reliably. A larger model (e.g. Opus) buys slightly better judgment on ambiguous classification at higher cost per run — worth it only if you see misclassifications in the summaries.

## Point it at a different kind of email (the fun one)

Nothing in the architecture is rejection-specific: the pattern is *"find emails of type X → verify → reply per template → report"*. People have adapted this skeleton for follow-ups on unanswered applications, invoice chasing, and newsletter triage. Keep Steps 1/3/4/7 (discovery, dedupe, reply-target, reporting) and rewrite Steps 2/5/6 (what qualifies, what's automatic vs. reviewed, what to say). The guardrails transfer unchanged — they're about sending email as a human, not about rejections.
