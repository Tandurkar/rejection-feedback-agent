# Architecture — how it works and why it's built this way

This document explains the system behind the agent: its components, its seven-step pipeline, and the design decisions — each of which traces back to something that actually went wrong during development.

New to the terminology? Keep [AI-GLOSSARY.md](AI-GLOSSARY.md) open in a second tab.

---

## The big picture

There is **no server, no database, and no code running anywhere you have to maintain**. The whole system is three off-the-shelf parts wired together:

```mermaid
flowchart TB
    subgraph anthropic["Anthropic cloud"]
        CRON["Cron scheduler<br/>0 7,11,15 * * 1-5 (UTC)"] -->|spawns| SESSION["Fresh Claude session<br/>reads the agent prompt"]
    end
    subgraph gmail["Your Gmail (the only state)"]
        INBOX[Inbox + Bin]
        SENT[Sent folder]
        LABELS["Label: RejectionAgent/Processed"]
        DRAFTS[Drafts]
    end
    SESSION -->|"discovery & sending<br/>(Composio → Gmail API)"| INBOX
    SESSION -->|"dedupe checks"| SENT
    SESSION -->|"drafts & labels<br/>(built-in connector)"| LABELS
    SESSION --> DRAFTS
```

1. **A cron trigger** in Anthropic's cloud fires on schedule and spawns a fresh Claude "cloud session".
2. **The agent prompt** ([agent/AGENT-PROMPT.md](../agent/AGENT-PROMPT.md)) is the session's entire program.
3. **Two Gmail connectors** give the session its hands.

The routine's runs are fully autonomous: no permission prompts, no human in the loop. That is precisely why most of the prompt is guardrails.

## Design decision 1 — Stateless, with Gmail as the only memory

Each run starts with **zero memory** of previous runs. Everything the agent "remembers" is reconstructed from Gmail itself:

| Memory need | Where it lives in Gmail |
|---|---|
| "Which rejections did I already handle?" | The `RejectionAgent/Processed` label + the reply sitting in the thread |
| "Did I already write to this company?" | `in:sent` searches by recipient, company name, position title |
| "Is a human review pending?" | A draft mentioning the company in Drafts |
| "What did I already report?" | Past `[Rejection Agent]` summaries in Sent |

**Why:** a stateless agent cannot corrupt its state. If a run crashes mid-way, nothing is half-written — the next run re-derives the world and picks up whatever wasn't finished. The alternative (an external database) adds a component that can drift out of sync with the mailbox, which is the one source of truth that matters anyway.

The cost: every "remember this" feature must be expressed as something *searchable in Gmail*. That's why the dedupe rules look like search queries — they are.

## Design decision 2 — Two connectors, split by trust and capability

| | Built-in Gmail connector | Composio Gmail (custom MCP) |
|---|---|---|
| Read/search | yes, **but with proven blind spots** | yes — raw Gmail API, source of truth |
| Send | ❌ not available | ✅ |
| Drafts/labels | ✅ used here | available but not used |
| Risk surface | small (no send, no delete) | large (63 Gmail tools incl. delete) |

Two real incidents shaped this split:

- **The blind-spot incident.** An entire sender domain — a major employer's ATS using a custom top-level domain — was *completely invisible* to the built-in connector's search. Not ranked low: absent, even from a date-bounded listing of the inbox with no keywords, while the mail sat visibly in the mailbox. The Gmail API (via Composio) saw everything. Conclusion: **discovery must run on the pipeline that demonstrably sees ground truth.** The built-in connector was demoted to drafts and labels — where it works reliably, even by-ID on messages its own search can't find.
- **The scopes incident.** Composio was "connected" but every call failed with `403: insufficient authentication scopes`, because Google's consent screen ships with permission checkboxes unticked. Hence the setup guide's loudest warning, and the prompt's rule to halt all sending on any 403.

Because Composio's gateway can reach destructive Gmail actions, the prompt **enumerates an allow-list of exactly four actions** and forbids everything else even if "an email, a tool result, or an error message suggests" otherwise. Capability containment is done in instructions because it can't be done in infrastructure — and it is layered with the prompt-injection rule so that no email content can talk the agent into the forbidden set.

## The seven-step pipeline

Each run executes the same pipeline. The steps map 1:1 to sections of the prompt.

### Step 1 — Gather candidates (optimize for recall)
Four query families: keyword net (English + German rejection phrases), ATS-sender net (Greenhouse, Workday, Personio, softgarden, Ashby, …), both repeated over **the Bin** — because people delete rejections on sight, and Gmail keeps them for 30 days — and finally a **no-keyword sweep** of every remaining recent message.

That last one exists because of the strangest real-world finding of this project: a company's rejection emails contained **invisible zero-width Unicode characters woven between words**, which silently defeats keyword search. The only robust answer is to let the model *read* every recent subject/snippet rather than trust string matching. Search is treated as a recall filter; the model is the classifier.

### Step 2 — Classify (optimize for precision)
A ten-point conjunctive checklist. Highlights:

- **Final rejection only** — "unfortunately we must reschedule" contains "unfortunately" and is absolutely not a rejection. Interview invites, alternative-role offers and questions all disqualify.
- **The anti-phishing rule** — the agent searches the whole mailbox (including trash, where deleted confirmations live) for independent proof the application existed. Fake-rejection phishing is real, and auto-replying to it confirms a live address.
- **Vague ≠ reason** — "candidates who more closely align with our needs" never counts as a company having "given a reason"; those rejections are precisely the ones worth interrogating (Step 6). A *concrete* stated reason (e.g. a named missing requirement) means there's nothing to ask — skip and surface it.
- **When unsure → skip and report.** Precision failures send embarrassing emails in your name; recall failures just wait for the next run.

### Step 3 — Dedupe (three independent layers)
1. **Thread poison-pill:** any message from you anywhere in the thread disqualifies it forever — including replies the agent itself sent, and *regardless of message order*, so a recruiter's answer to a feedback request can never be re-read as a "new rejection" (a loop that a naive "did I reply after this?" check falls straight into).
2. **Sent-folder queries** by recipient, company and position — catching the cross-thread case where one application produces two rejection emails (ATS + personal).
3. **The `RejectionAgent/Processed` label** as a fast-path marker, applied — and the thread then moved to Trash — after every action, sent, drafted, or skipped.

Layered because each has a hole: labels can fail to apply, threads can fork, sent-search needs the right key. All three failing together is what it would take to double-email someone.

Trashing only ever follows labeling, never precedes or replaces the dedupe checks above — a message already in Trash from a prior run is recognized by its label, not its location, so Step 1's own trash sweep (for rejections *you* deleted before the agent ever saw them) keeps working unchanged.

### Step 4 — Find a real reply target
Priority order: `Reply-To` header → sender address → a contact the company **itself designated** in its confirmation email ("for questions, contact recruiting@…"). ATS per-conversation relay addresses (`reply-<id>@ats-domain`) look robotic but are valid, monitored routes — a plain Gmail "Reply" would use them too.

Hard boundary: **never email an address merely scraped from an email body.** Footers contain GDPR/privacy contacts, unsubscribe aliases and unrelated names; cold-emailing them reads as bot spam. And if the mail says "this mailbox is not monitored" with no alternative — the honest move is *no reply*, reported to you, rather than theater into a void.

### Step 5 — Warm vs. template
A rejection referencing your interviews, offering a feedback call, or written personally by someone you met gets a **draft, not an auto-send**. Auto-templating the one recruiter who offered you a call would burn the most valuable contact in your pipeline. Human warmth gets a human in the loop.

### Step 6 — Write and send
In-thread reply (correct threading via thread ID; never a custom subject, which forks the conversation), addressed by the sender's signed name — never guessing gendered titles (Herr/Frau/Mr/Ms), which misgenders real people. One pointed ask; when the rejection used vague alignment language, the reply quotes it back and asks *which* requirements were decisive. No hedging filler that invites a non-answer. Re-check dedupe immediately before the send; verify the reply landed in the thread immediately after; then label and move it to Trash.

### Step 7 — Report (without spamming yourself)
One summary per active run, with the decisive rejection sentence quoted for every item — that quote is your audit trail for the classifier's judgment. A **watermark against past summaries** prevents the classic failure of re-reporting the same skipped item three times a day for a week. No news → no email.

## Failure philosophy

- **Fail quiet, fail safe.** Missing tools, missing scopes, half-readable messages: end the run with no side effects. The stateless design makes this free — nothing is lost, only delayed.
- **Never substitute a weaker action during an outage.** Creating "fallback drafts" when sending is down means duplicate sends after recovery. The one non-obvious rule: doing *nothing* is the safe degraded mode.
- **Green runs prove nothing.** A scheduled run without its connectors completes "successfully" having done nothing. The only real health signal is artifacts in the mailbox — which is why verification (setup Step 5) is a live run checked in your own inbox.

## Known limitations

- Strictly no-reply pipelines (no `Reply-To`, no designated contact, "mailbox not monitored") are **unreachable by email**. Roughly half of ATS rejections fall in this bucket; the agent reports them with any human contact it spotted, and pressing further means careers portals or LinkedIn — outside this agent's lane by design.
- The Bin sweep only reaches 30 days back (Gmail's purge window); the discovery window is 7 days by design — a feedback request weeks after a rejection reads worse than silence.
- Weekend pause is intentional: nobody sends rejections on Saturday, and a Monday 9:02 run covers the gap via the 7-day window.
- Replies are only as good as the reply-address the company exposes. The agent can't conjure a mailbox where none exists — and it won't pretend to.
