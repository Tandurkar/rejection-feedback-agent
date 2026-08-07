# Setup Guide — from zero to a working agent

This guide assumes **nothing**. Follow it top to bottom and you will end with an autonomous agent that replies to your job-rejection emails three times every weekday. Budget about **30 minutes**, most of it clicking through permission screens.

Every ⚠️ box marks a step where real setups have silently failed. Do not skip them.

---

## What you need before starting

| Thing | Why | Cost |
|---|---|---|
| A **claude.ai** account with Claude Code cloud routines (Pro/Max/Team plan) | The agent runs as a scheduled "routine" in Anthropic's cloud | Your existing subscription |
| A **Gmail** account | The mailbox your job applications and rejections flow through | Free |
| A **Composio** account | Provides the connector that can actually *send* Gmail on your behalf (the built-in one can't) | Free tier is plenty |

---

## Step 1 — Connect the built-in Gmail connector

1. Go to **[claude.ai/customize/connectors](https://claude.ai/customize/connectors)**.
2. Find **Gmail** in the list of connectors and click **Connect**.
3. A Google sign-in window opens. Choose the Gmail account you use for job applications and approve.

This connector lets the agent read mail, create drafts, and manage labels. It **cannot send email** — that's what Step 2 is for.

## Step 2 — Set up Composio (the sending arm)

1. Create a free account at **[composio.dev](https://composio.dev)** (Google login works).
2. In the Composio dashboard, find **Connect Apps** (or "Connected accounts") and connect **Gmail**.
3. Google will show a consent screen with **individual permission checkboxes**.

> ⚠️ **TICK EVERY CHECKBOX on Google's consent screen.** Google leaves them unchecked by default. If you miss the send/modify boxes, everything will *look* connected but every action will fail with a silent `403: insufficient authentication scopes` — the single most confusing failure this project ever hit. If you already connected with missing boxes: disconnect the Gmail account in Composio and reconnect, ticking everything.

4. Back on **[claude.ai/customize/connectors](https://claude.ai/customize/connectors)**, click **Add custom connector**:
   - **Name:** `Composio Gmail` (any name works)
   - **URL:** `https://connect.composio.dev/mcp`
   - Leave advanced/auth fields empty, click **Add**, and complete the one-time browser authorization that follows.

## Step 3 — Create the routine (the agent itself)

1. Open [agent/AGENT-PROMPT.md](../agent/AGENT-PROMPT.md) from this repository. Copy the entire prompt block and replace the placeholders at the top: your name, your Gmail address, and one line describing what kind of roles you're applying for.
2. Go to **[claude.ai/code/routines](https://claude.ai/code/routines)** and create a new routine:
   - **Name:** `Rejection feedback requester` (or anything you like)
   - **Instructions:** paste your filled-in prompt
   - **Schedule (cron):** `0 7,11,15 * * 1-5` runs at 07:00, 11:00 and 15:00 **UTC**, Monday–Friday — that's 9:00/13:00/17:00 in Berlin (summer). Shift the hours to fit your timezone; see [CUSTOMIZATION.md](CUSTOMIZATION.md).
   - **Model:** `claude-sonnet-5` is the tested default.
   - No repository is needed — leave repo/source fields empty.

   *Alternative:* if you use Claude Code as an app or CLI, you can just ask Claude to set this up: "create a routine with this prompt, weekdays at 9/13/17 my time" and paste the prompt.

## Step 4 — Attach the connectors to the routine ⚠️ THE step people miss

Connecting connectors to your *account* (Steps 1–2) does **not** automatically give this *routine* access to them.

1. Open your routine's page at **claude.ai/code/routines** and click the **edit (pencil)** icon.
2. Find the **Connectors** section.
3. Click **Add connector** and select **both**: your `Composio Gmail` connector **and** the built-in `Gmail` connector.
4. **Save.** The routine's page should now list both connectors (alongside an internal `Claude_Code_Remote` entry — leave that one alone).

> ⚠️ **If you skip this, everything looks fine and nothing works.** Runs fire on schedule, show green checkmarks — and do absolutely nothing, because the agent wakes up with no email tools and (by its own safety rules) ends quietly rather than half-working. This exact failure cost this project three days. Green run ≠ working run; the proof is always in your inbox.

## Step 5 — Verify with a live test

1. On the routine's page, click **Run now**.
2. Wait ~5 minutes.
3. Check your Gmail inbox for an email titled **`[Rejection Agent] …`**.

**What you should see, depending on your mailbox:**

- *You have recent unanswered rejections* → replies appear inside those threads, plus a summary listing them.
- *Your recent rejections are all from no-reply pipelines* → a summary listing them as skips, with reasons.
- *No rejections in the last 7 days at all* → silence. That's correct behavior — to force a visible test, wait for your next rejection (sadly, they come) or temporarily widen `newer_than:7d` in the prompt to `newer_than:30d`, run once, and change it back.

4. Optional deeper check: click the run entry on the routine page to read the agent's full working transcript.

## Step 6 — First-week habits

- **Read the first few summaries.** They quote the decisive line of every rejection and explain every decision. If a reply's wording isn't *you*, change the template — see [CUSTOMIZATION.md](CUSTOMIZATION.md).
- **Check your Drafts folder when a summary says "draft ready for your review"** — warm, personal rejections are deliberately left for you to send with one click.
- **Don't delete the summary emails for at least a week** — the agent reads its own past summaries to avoid re-reporting the same skips.

## You're done

From here the agent runs unattended: rejection arrives → reply goes out in your name → report lands in your inbox. On quiet days you hear nothing. If anything ever seems off, go straight to [TROUBLESHOOTING.md](TROUBLESHOOTING.md) — it is written from the actual failures that happened while building this.
