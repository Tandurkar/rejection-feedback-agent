# Troubleshooting

Every entry in this file is a **real failure that happened while building this project**, with the diagnosis path that found it and the fix that worked. Start with the quick table, then read the matching story.

| Symptom | Most likely cause | Fix |
|---|---|---|
| Runs show green, but nothing ever appears in your inbox | Connectors not attached **to the routine** | §1 |
| Every send/read fails with `403: insufficient authentication scopes` | Google consent checkboxes were left unticked in Composio | §2 |
| A rejection you can see in Gmail was never picked up | Search blind spot or invisible characters | §3 |
| Agent replied to nothing although rejections exist | They're all unreachable (no-reply) or already handled — read the summary | §4 |
| You fear it emailed someone twice | It almost certainly didn't — verify with the label + thread | §5 |
| No summary emails for days | No news. Verify health once, then trust the quiet | §6 |
| A reply is stuck as a draft | That's the warm-rejection feature, not a bug | §7 |

---

## §1 — The silent no-op: runs fire, nothing happens

**Symptom:** the routine's run list fills with green checkmarks on schedule; your mailbox shows no replies, no labels, no summaries. Days can pass like this.

**Cause:** connecting Gmail/Composio to your *claude.ai account* does not attach them to your *routine*. A routine without connectors wakes up with no email tools, and the agent's own startup check then ends the run quietly (by design — better silent than half-working). **A green run only means the session ran, not that it could do anything.**

**Diagnosis:** open the routine's page → the **Connectors** line must list your Gmail and Composio connectors, not just an internal `Claude_Code_Remote` entry. Alternatively click any run and read its transcript — you'll see the agent finding no tools and stopping.

**Fix:** routine page → edit (pencil) → **Connectors → Add connector** → add both → Save → press **Run now** and confirm a `[Rejection Agent]` summary reaches your inbox. This exact bug cost this project three days, because every dashboard signal looked healthy.

## §2 — `403: insufficient authentication scopes`

**Symptom:** the Composio connection shows as ACTIVE, yet every Gmail action fails with HTTP 403 "insufficient authentication scopes" — even plain reads.

**Cause:** Google's OAuth consent screen lists each Gmail permission as an **individually untickable checkbox, all unticked by default**. It is entirely possible to "successfully connect" an account that can do nothing.

**Fix:** Composio dashboard → disconnect the Gmail account → reconnect → on Google's consent screen **tick every checkbox** → re-test with a harmless read. The agent's prompt treats any 403 during a run as a full stop — no retries, no workarounds — so fixing the grant is the only path.

## §3 — A visible rejection was never found

**Symptom:** a rejection email sits in your inbox; the agent's summaries never mention it.

**Two real causes, both encountered:**

1. **Search-pipeline blind spot.** One connector's search returned *nothing* from a particular sender domain — not even in a no-keyword, date-bounded inbox listing — while the Gmail API saw the mail fine. That's why discovery runs through Composio (raw Gmail API). If you ever suspect this again: compare "what the run's transcript searched and saw" against your actual inbox for the same window.
2. **Invisible zero-width characters.** Some companies' rejection text is laced with invisible Unicode (zero-width spaces between words), which silently defeats every keyword query — `Absage` in the text doesn't match `Absage` in the query. Defense: Step 1(d) of the prompt lists *all* recent mail and reads subjects/snippets with the model instead of trusting string matching.

**Also check the boring causes first:** is the rejection older than the 7-day window? Does its thread carry the `RejectionAgent/Processed` label (already handled)? Did you ever reply in that thread yourself (permanently disqualifying)?

## §4 — "It found rejections but sent nothing"

Read the summary email — every skip carries its reason. The common ones, all correct behavior:

- **"mailbox unmonitored, no alternate contact"** — the rejection came from a no-reply pipeline with no `Reply-To` and no designated contact. There is literally no address that receives mail. Sending anyway would be theater; the agent refuses to fake progress. If a human contact was visible in the signature, the summary hands it to you for a personal note.
- **"reason already given"** — the company already stated a concrete reason (e.g. a named missing requirement). Asking "why?" after they told you why makes the sender look like a bot.
- **"already handled"** / one-note-per-company — a combined note went out earlier; additional rejections from the same employer don't generate more emails.
- **"no record of application (possible spam/phishing)"** — no confirmation, no sent application, no interview thread anywhere (including trash). Genuine rejections almost always leave a paper trail; mails without one are treated as bait.

## §5 — "Did it double-email someone?"

Three independent layers make this the hardest failure to produce: any message from you in a thread disqualifies it forever; `in:sent` queries by recipient/company/position catch cross-thread duplicates; the `RejectionAgent/Processed` label catches the rest — and the checks re-run immediately before each individual send.

**To verify a specific case:** open the company's thread — you'll find exactly one reply from your address — and search `in:sent to:<their-address>`. If you *ever* find a genuine double-send, please open an issue with the (redacted) timeline.

## §6 — No summaries for days

No new rejections + nothing new to skip = no email, by design. The agent doesn't send "nothing happened" mail.

To distinguish healthy quiet from silent failure (§1) once: press **Run now** *after temporarily* widening `newer_than:7d` to `newer_than:30d` in the prompt — old, already-handled items will be re-listed as "already handled" only in the run transcript, but any never-reported skip in the window will produce a summary. Change it back afterwards. Or simply check that the routine page's Connectors line still lists both connectors — the only thing that has ever actually broken.

## §7 — "A reply is sitting in my Drafts instead of being sent"

Feature, not bug: the rejection was classified **warm** (personal — references your interviews, offers a call, invites staying in touch). The agent drafts but never sends those, because templated automation aimed at the one recruiter who liked you destroys more value than every other reply creates. Read the draft, edit if you like, press Send.

---

## Debugging toolbox

- **Run transcripts:** routine page → click any run → the agent's full reasoning, searches, and tool calls. The fastest way to answer "what did it actually see?"
- **Gmail as audit log:** `label:RejectionAgent-Processed` (everything handled), `in:sent subject:"[Rejection Agent]"` (every report), `in:sent "specific reason"` (every feedback request ever sent).
- **SCHEDULED vs MANUAL:** the run list labels which runs came from cron and which from the Run-now button — useful when you're wondering whether the schedule itself is alive.
- **When in doubt, ask Claude:** paste the summary or symptom into a Claude chat with the routine link. The whole system is instructions + mailbox state, and both are inspectable.
