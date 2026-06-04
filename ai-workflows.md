# AI Workflows at Axonius

Axonius deployed a private, enterprise-licensed instance of Google Gemini. Because it was isolated from Google's public infrastructure, I could attach real customer data -- tickets, logs, adapter files -- without data handling concerns. That made it genuinely useful in ways a public AI tool wouldn't have been.

I built a set of Gems (Gemini's saved prompt feature) to handle the high-friction, repeatable parts of my workflow. The prompts were short on purpose. My experience, here and everywhere else, is that clarity scales better than complexity.

---

## The Gems

### Ticket Triage

**Prompt:** `Provide the TL;DR and next steps from the provided file.`

**Workflow:** Download the Zendesk ticket as a PDF, attach it to the Gem.

At 50+ active cases, the hardest part of context switching isn't reading a ticket -- it's re-orienting to where things stood. This Gem eliminated that. Nine words, applied consistently across a full queue.

Before going on vacation, I used it one more way: I ran every ticket I owned through the Gem and posted the output as a comment, so anyone picking up a case would know exactly where things stood.

The day I got back, multiple coworkers mentioned it in standup. They hadn't been asked. One had pulled up a ticket to help a customer who reached out while I was out -- the TL;DR showed not just that the issue had been escalated to engineering, but what engineering was actively working on and when a fix was expected. Separately, Keren -- our director -- had looked at one of my other tickets and came away satisfied with what she found there too. Nobody had to track me down. Nobody had to reconstruct context from a thread of 40 replies.

A nine-word prompt. Written once. Did its job while I was on a beach somewhere.

---

### Log Analysis

**Prompt (v1):** `Scan the provided file for HTTP errors.`

**Prompt (v2):** `Scan the provided file for errors.`

**Workflow:** Export Coralogix output to PDF, attach it to the Gem.

The first version targeted HTTP errors specifically. As Gemini's training on the Axonius platform improved, I updated the prompt to cast a wider net -- covering HTTP errors and platform-specific errors in a single pass. One word removed. Meaningfully better output.

---

### Curl Statement

**Prompt:** `Scan the provided files for the API endpoints. Select the most basic endpoint to use in a curl statement. Also provide the curl needed to obtain the access token.`

**Workflow:** Download the adapter source files, attach them to the Gem.

This one required knowing what to ask for. The access token step isn't obvious if you don't already understand how API authentication works. The Gem read the adapter code, identified the right endpoint to start with, and returned both calls ready to run.

---

### Escalation Forms

**Prompt:** A field-by-field list of what each engineering team's escalation form required.

**Workflow:** Download the Zendesk ticket as a PDF, attach it to the Gem. Copy the extracted data into the Workado form in Slack.

Each engineering team used a different form and asked for a different set of data points. Rather than re-reading every ticket to hunt for the right fields, I built a Gem for each team. The Gem did the extraction. I did the copy-paste. Not full automation, but a meaningful reduction in the back-and-forth.

---

### Improve Communication

**Prompt (v1):** `Make the provided input sound friendly, but professional.`

**Prompt (v2):** `Make the provided input sound friendly, but professional. No emojis. No em dashes.`

After years of technical writing, my customer communication had gotten precise but dry. Some customers read it as aloof, or even scolding. I used this Gem to soften the edges before sending.

I used it for four or five months. Then I stopped needing it.

Watching the rewrites consistently, I started to internalize what the adjustments had in common -- what warmth actually looks like at the sentence level without becoming vague or performative. I came out the other side a more empathetic writer. That wasn't the goal when I built the Gem. It was a side effect of paying attention.

---

## The Throughput Numbers

While colleagues averaged 35 to 40 active tickets, I consistently managed 55 to 60. These workflows were part of how that was possible -- not the whole story, but a real part of it.

[← Back to Home](https://github.com/mjburak/aboutme/blob/main/README.md)
