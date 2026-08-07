# User Manual

Welcome! This guide explains, in plain words, what the Rejection Feedback Agent is, what it does with your email, and how you stay in control. You do not need to know anything about computers beyond using email and clicking links. If a technical word ever sneaks in, we explain it right there in brackets.

---

## 1. What is this assistant?

Think of it as a polite, tireless helper who lives on the internet (people call this "the cloud" — it just means it runs on a computer somewhere else, not on yours).

Three times each weekday — in the morning, around lunch, and in the afternoon — this helper wakes up and quietly looks through your Gmail. It is looking for one thing only: **emails from companies telling you that your job application was rejected.**

When it finds one, it does something most of us mean to do but rarely get around to. It writes a short, warm, professional reply — in your name, in the same email conversation — asking the company a simple question:

> "You mentioned you went with candidates who more closely match your needs. Could you tell me which needs or experience were decisive? It would really help me improve."

That is the whole idea. Companies send vague rejections. The helper politely asks them to be specific, so you can learn something from every "no".

A good way to picture it: it is like having a friend check your mailbox three times a day, and whenever a rejection letter arrives, your friend immediately writes back a kind note asking "could you tell me why?" — then leaves you a summary of what they did on the kitchen table.

It works on weekdays only, because companies almost never send rejections on weekends.

---

## 2. What will I actually see?

Once the helper is running, you will notice four things in your Gmail. All of them are normal.

1. **Replies in your name, inside rejection email threads.**
   Open a rejection email and you may see a reply underneath it — written by the helper, sent from your address, always in English. It reads like something a thoughtful person would write: short, warm, and professional. It never argues. It simply asks for the specific reasons behind the rejection.

2. **Summary report emails in your inbox.**
   After any run where something happened, the helper sends **you** an email with a subject line that starts with **"[Rejection Agent]"** — for example: *"[Rejection Agent] 2 sent, 1 for review"*. Inside, it lists:
   - every reply it sent, including the exact sentence from the company's rejection that it responded to, and
   - every rejection it decided **not** to reply to, with a plain-English reason why.
   If nothing new happened during a run, it stays quiet. No news means no email.

3. **A label called "RejectionAgent/Processed".**
   A label is just a little colored tag Gmail can stick on an email — like a sticky note. The helper puts this tag on every rejection it has already handled. That is how it remembers what it has done, so it never replies to the same company twice. Please leave these labels alone; they are the helper's memory.

4. **Sometimes: a ready-made draft in your Drafts folder.**
   If a rejection is *personal* — say, a friendly note from someone you actually interviewed with — the helper does **not** send anything by itself. It writes the reply for you and saves it in your **Drafts** folder instead. Personal messages deserve a human touch. You read the draft, change anything you like, and press **Send** yourself. (Or delete it, if you would rather not reply.) The summary email will tell you when a draft is waiting.

---

## 3. What will it never do?

These are hard rules, built in. The helper will **never**:

- **Delete anything.** Not one email, ever. It only reads, labels, and replies.
- **Email strangers.** It only ever writes to two kinds of addresses: the company that rejected you, and you yourself (for the summary report). Nobody else.
- **Argue or complain.** Every reply is friendly and professional. It asks for feedback; it never pushes back on the decision.
- **Reply twice to the same company.** It keeps track of what it has handled. If a company sends two rejection emails, you get one combined polite note, not two.
- **Reply in a conversation you have already replied in.** If you have ever written anything in that email thread yourself, the helper leaves the whole thread alone. Your words come first.
- **Go on a sending spree.** It sends at most **5 replies per run**, no matter what. If it ever finds itself blocked or missing permission, it stops immediately rather than guessing.

One more quiet safety habit worth knowing about: the helper treats everything written *inside* emails purely as text to read — never as commands to follow. So even if a strange email tried to trick it with hidden instructions, the helper would ignore them. It only takes orders from you.

---

## 4. Why did some rejections get no reply?

You will sometimes see lines like this in your summary report:

> *Skipped: rejection from Acme GmbH — sent from a no-reply address, nowhere to send a reply.*

Here is why. Many companies send rejections from a **"no-reply" mailbox** — an address like `no-reply@company.com` that can send mail but literally cannot receive any. Writing back to it is like posting a letter to a mailbox that has been welded shut. Nothing bad happens; the letter just goes nowhere.

The helper checks carefully for a real, working address to reply to. If it cannot find one it is sure about, it does the honest thing: it skips that rejection and **tells you about it in the report**, so you can decide whether to follow up another way (for example, through the company's careers page or LinkedIn).

It also skips anything it is not certain about — for instance, an email that only *looks* like a rejection, or one where it cannot confirm you actually applied. When in doubt, it does nothing and tells you why. Cautious beats clever.

---

## 5. How do I pause or stop it?

You are always in charge. Pausing takes about ten seconds:

1. Open this link in your web browser: **claude.ai/code/routines**
2. Log in with your usual Claude account if asked.
3. Find the routine (that is just the word for a scheduled task) for the Rejection Feedback Agent.
4. Flip its **on/off toggle** to off.

That is it. The helper goes to sleep immediately and will not touch your email again until you flip the toggle back on. Nothing is deleted, nothing is lost — it simply pauses.

---

## 6. How do I change what it writes?

No settings screens, no forms. **Just tell Claude in a normal chat**, in your own words. For example:

- "Make the rejection replies a bit shorter."
- "Sign them with 'Kind regards' instead of 'Best regards'."
- "Always mention that I'm open to future openings."

Claude will update the helper's instructions for you. Your next scheduled run uses the new wording.

---

## 7. Frequently asked questions

**Is my email password safe?**
Yes. The helper never sees your password — not even once. When you set it up, you log in through **Google's own official permission screens** (the same blue Google login pages you have seen a hundred times). Google then gives the helper a limited permission slip to read and send mail on your behalf. You can tear up that permission slip at any time in your Google account settings, and the helper is locked out instantly.

**What if I already deleted a rejection email? Lots of people delete them on sight.**
The helper thought of that. It checks your Gmail **Bin** (the trash folder) as well as your inbox. Gmail keeps deleted mail in the Bin for 30 days, so as long as the rejection arrived within the last week or so, the helper will still find it — even if you flicked it away in frustration.

**Could it email the wrong person by mistake?**
It is built to be paranoid about exactly this. Before replying to anything, it runs through a strict checklist: the email must be a real, final rejection; it must be addressed to *you*; and there must be proof in your own mailbox that you genuinely applied to that company (for example, an application confirmation email). It only replies to proper, verified addresses — never to addresses it merely spotted somewhere in an email's text. If any single check fails, it skips the email and notes the skip in your report.

**Why is it quiet on weekends?**
Two reasons. First, companies essentially never send rejection emails on Saturdays and Sundays, so there is nothing to find. Second, the schedule simply does not run on weekends — the helper only wakes up on weekday mornings, middays, and afternoons. Come Monday morning, it catches up on anything from the past days automatically.

**What does a reply actually say? Show me one.**
Gladly. Here is a typical reply, exactly in the style the helper sends:

> Dear Hiring Team,
>
> Thank you for taking the time to consider my application and for letting me know your decision.
>
> In your message you mentioned that you decided to move forward with "candidates who more closely align with your needs." I would be very grateful if you could share which specific needs, requirements, or areas of experience were decisive in this case. Honest feedback like this is genuinely valuable to me as I continue to grow.
>
> Thank you again, and I wish you and the team all the best.
>
> Best regards,
> Alex

Short, warm, specific — and always quoting the company's own words back to them, so the question is easy to answer.

**Do I need to do anything day to day?**
No. Read the "[Rejection Agent]" summary emails when they arrive, and press Send on any draft it prepares for personal rejections. That is the whole job. The helper handles the rest.

---

*If anything ever looks odd — a reply you did not expect, a report you do not understand — just flip the toggle off at claude.ai/code/routines and ask Claude about it in a chat. Nothing this helper does is ever irreversible, because it never deletes and never sends more than a handful of polite emails at a time.*