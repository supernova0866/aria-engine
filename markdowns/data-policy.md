# ARIA Data Policy

This document defines how ARIA collects, stores, uses, and protects data generated through interactions with the bot. It covers everything from what gets logged in real time to how training data is prepared, used, and eventually retired. This is an internal policy document and a binding reference for every contributor and phase of the project.

If you are working on any part of this project that touches user data, conversation logs, or the training pipeline, read this document fully before writing a single line of code.

---

## Core Principles

**Users are not the product.** ARIA exists to simulate a personality, not to harvest data. Every data decision should pass a simple test: would a user feel uncomfortable if they knew this was happening. If yes, it does not happen.

**Minimalism.** Collect only what is needed. Store only what is used. Delete what has served its purpose. Data that sits around unused is a liability, not an asset.

**Honesty.** Users in every phase are told that their interactions contribute to improving the bot. Not buried in fine print. Actually told, clearly, in plain language.

**Separation of concerns.** What gets stored for memory and relationship purposes is different from what gets stored for training purposes. They live in different places, follow different rules, and have different retention windows.

**Ownership.** All data generated through ARIA interactions and all models trained on that data are internal project assets. Nothing gets shared, sold, uploaded publicly, or used outside the context of this project.

---

## What Data ARIA Collects

### Real Time Per Interaction

Every time a message is processed, the following gets logged:

**In the User DB (aria-users):**
- A behavioral summary of the interaction. What the general nature of the exchange was, not the verbatim message. Example: "user asked about music, seemed friendly, introduced a new slang term."
- Her mood and intensity at the time of the interaction
- A sentiment score for the interaction (positive, neutral, negative)
- The channel the interaction happened in
- A timestamp

**In the Discord logging server:**
- The full assembled prompt that was sent to the AI
- The raw AI response
- Her mood, intensity, cycle phase, and energy at the time
- The closeness tier of the user she was talking to (not their identity)
- The response behavior type (message, reaction, silence)
- Any flags that fired (unknown word, memorable moment, block trigger, etc.)

**What is never logged verbatim anywhere:**
- The user's actual message text
- Any names, personal details, or sensitive information a user shared
- Anything a user said that was clearly private or distressing in nature

### Nightly Jobs

Every midnight UTC the nightly pipeline runs and generates updated state. The outputs of each job get written to the Core DB and logged to the Discord logging server. No new user data is collected during this process. It only processes data that was already captured during the day.

### Vocabulary and Knowledge

When she encounters an unknown word and asks about it, the definition a user provides gets stored in the vocabulary table alongside the user ID of who taught her. This is the only place a user's direct contribution is stored with attribution. It is used to track expertise trust, not for training data.

When facts or knowledge get added to the knowledge DB, the source user IDs are stored in a JSON array for expertise trust purposes. The actual content stored is her interpretation of what she learned, not a verbatim quote.

---

## Data Storage Locations

### aria-core (Turso)

Her internal state. Mood history, cycle, schedule, vocabulary, memories, life events, fatigue, atmosphere. This database contains no user-identifiable information beyond the user IDs stored in vocabulary and memorable moment entries for attribution purposes.

**Retention:** Indefinite for core state. Mood history and interaction-adjacent logs rotate on a 90 day window. Daily schedules older than 30 days are deleted. Memorable moments and life events are permanent unless manually reviewed and removed.

### aria-users (Turso)

Everything about the people she knows. Relationship scores, thoughts, blocks, social graph, expertise trust, interaction logs. This is the most sensitive database in the project.

**Retention:**
- Active user profiles: retained as long as the user has interacted in the past 12 months
- Interaction logs: rolling 12 month window, older entries deleted automatically
- Block records: retained for 24 months after the block is lifted, then deleted
- Social graph entries: retained as long as both users are active, pruned 6 months after either user goes inactive
- Thoughts and relationship scores: retained while the user is active, deleted 12 months after last interaction

### aria-knowledge (Turso)

General world knowledge, topic opinions, preferences and aversions. No user-identifiable information except source user ID arrays for expertise tracking.

**Retention:**
- Shallow knowledge: TTL of 14 days from last reference, auto-deleted by nightly job
- Deep knowledge: retained indefinitely, reviewed manually if it becomes outdated or incorrect
- Topic opinions and preferences: retained indefinitely, updated as new exposure accumulates

### Discord Logging Server

Structured logs of engine decisions, prompt and response pairs, mood shifts, flag events, nightly job outputs, and errors. This is the primary source of raw training data.

**Retention:**
- Interaction logs (prompt and response pairs): 18 months rolling window
- Engine decision logs: 6 months rolling window
- Error logs: 3 months rolling window
- Nightly job logs: 30 days rolling window

After the retention window closes, entries are either archived into the cleaned training dataset or deleted. Nothing sits in the logging server indefinitely.

---

## Consent by Phase

### Closed Alpha (10 people, 1 server)

Every participant is personally invited and personally briefed before the bot is in their server. Participation in closed alpha is conditional on agreeing to full data collection. There is no opt out available during this phase.

This is not a hidden condition. Every participant is told explicitly before they join:

- ARIA is in early testing
- Their interactions are essential for building the dataset that makes the system work
- Full data collection is a requirement of participation, not optional
- Interaction summaries and her responses are logged for development purposes
- Their actual messages are never stored verbatim
- If they are uncomfortable with full data collection, closed alpha is not the right fit for them and that is completely fine

Consent at this phase is direct and conversational. The briefing is documented for every participant before they join.

### Private Beta (50 people, 1 server)

Every participant is still invited. There is no opt out available during private beta. Like closed alpha, full data collection is a condition of participation.

Before the bot is added to the server, a clear message is pinned in the channel she will be active in covering the same points as the closed alpha briefing in plain readable language. Participants who are not comfortable with full data collection should not join the private beta. This is communicated clearly before access is granted.

The data generated in these two phases is the foundation of the training pipeline. Gaps caused by opt outs during the most critical data collection window would directly undermine the quality of everything built on top of it. Transparency upfront is the tradeoff for not offering opt out during these phases.

### Public Beta (3 servers)

When the bot joins a server, her first message in each active channel includes a brief honest disclosure. Something like: "Hey, just so you know, I learn from conversations and my responses help train future versions of me. Your actual messages are never stored, just how I respond to them. You can type [command] if you want to opt out of that."

The opt out mechanism is the same as private beta. Opt out users are flagged in the User DB and excluded from all training data pipelines automatically.

### Full Release (10 servers, 10k users)

Same disclosure model as public beta. The opt out mechanism is permanent and survives bot restarts. Opted out users are never included in any training data regardless of how long they interact with the bot.

A public-facing version of this data policy document is maintained and linked from the bot's disclosure message so anyone who wants the full picture can read it.

---

## Opt Out Mechanics

Opt out is available from public beta onward. It is not available during closed alpha or private beta. Participation in those phases requires full data collection as a condition of joining.

From public beta onward the opt out flag lives in the users table in aria-users:

```
users
├── user_id
├── ...
├── data_opt_out          boolean, default false
├── opt_out_timestamp     when they opted out
```

When data_opt_out is true:
- The interaction still gets processed so the bot responds normally
- Nothing gets written to the interaction log
- The entry is excluded from any training data export scripts
- The opt out flag persists forever and is never automatically cleared

Users can opt back in at any time. The opt-in is logged with a timestamp. Interactions before the opt-in are not retroactively included in training data.

---

## What Gets Used for Training

Not everything logged is appropriate for training. The data cleaning process that runs before any fine-tuning or model training work begins filters aggressively.

### What Qualifies as a Training Example

A valid training pair consists of:
- A full assembled prompt representing her state at the time
- Her response to the message
- Metadata: mood, intensity, cycle phase, closeness tier, response type

For an interaction to qualify it must meet all of these conditions:
- The user was not opted out
- The response was a genuine message (not a reaction or silence event)
- No engine error or API failure flags were raised during that interaction
- The response was not flagged as off-character during any manual review
- The interaction happened more than 30 days ago (gives time for manual review before training)

### What Gets Excluded

The following are automatically excluded from training data regardless of anything else:

- Any interaction where the user's message (not stored but inferrable from context) appeared to be a test, a jailbreak attempt, or deliberately adversarial
- Any interaction during a known engine bug window
- Any interaction where her response was extremely short (under 10 words) with no clear reason, which usually indicates something went wrong
- Any interaction flagged during manual review as unrepresentative, broken, or out of character
- All interactions from opted out users
- Any interaction where the assembled prompt contained a known error or malformed field

### Sensitive Content Exclusion

Any interaction log entry that contains any of the following in the assembled prompt or response gets excluded automatically and flagged for manual review:

- Mental health related content (detected via keyword matching)
- Any content that appears to involve a user in genuine distress
- Anything that triggered a hard or permanent block
- Any interaction where the bot's response was later manually corrected

---

## Anonymization Standard

By the time any data enters the training pipeline it must be fully anonymized. No exceptions.

**What gets replaced or removed:**

- Discord user IDs replaced with anonymous session tokens generated per training export. The same user gets the same token within one export but different tokens across exports.
- Closeness tier is kept (stranger, acquaintance, friend, close) but no identifying relationship details
- Any name a user shared that ended up referenced in her response gets replaced with a generic placeholder
- Server names and channel names removed entirely
- Specific timestamps replaced with generalized time context (late night weekday, weekend afternoon, etc.)
- Cycle phase kept as-is since it carries no user data
- Mood and intensity kept as-is

**What is explicitly not kept in training data:**

- Any actual user message content, verbatim or paraphrased
- Any user ID, username, or Discord handle
- Any server or channel identifier
- Any real name mentioned in conversation
- Any personal detail a user shared about themselves

The anonymization script runs before any training export and produces a clean dataset with none of the above. The raw logs that fed the export are retained for the remainder of their retention window and then deleted according to the schedule above.

---

## Model Ownership and Usage

### Ownership

All models trained on ARIA data are internal project assets. This includes:

- Fine-tuned versions of open source base models
- Any custom model architecture trained from scratch
- The weights, configurations, and training checkpoints of all of the above

None of these are uploaded to public model repositories like Hugging Face under any circumstances. None are shared with third parties. None are used to power any product or service outside of ARIA.

### Usage Restrictions

Models trained on ARIA data may only be used:

- As the AI provider layer in the ARIA bot via provider.js
- For internal evaluation and benchmarking during development
- For continued fine-tuning on updated datasets

Models trained on ARIA data may not be used:

- To power any other bot or product
- As a base for training models with different purposes
- For any commercial application outside of ARIA itself
- To generate content outside of the ARIA system

### Third Party Base Models

When fine-tuning begins on an existing open source model, the license of that base model must be reviewed before training starts. Most open source models (Llama 3, Mistral) permit fine-tuning for non-commercial use. The specific license terms of whatever base model is chosen at that time govern what can and cannot be done with the resulting fine-tuned weights.

This needs to be documented at the start of Phase 8. If the chosen base model has a license that conflicts with the intended usage, a different base model must be selected.

---

## Data Breach Protocol

If at any point there is reason to believe that user data has been exposed, accessed without authorization, or compromised in any way, the following steps happen immediately:

1. The bot goes offline. No new data is collected until the breach is understood and contained.
2. The scope of the breach is assessed. What data was exposed, how, and to whom.
3. If any user-identifiable data was involved, affected users are notified directly via Discord DM within 48 hours.
4. The logging server access is audited and any unauthorized access is revoked.
5. A post-mortem is written documenting what happened, what data was affected, and what changes are being made to prevent recurrence.
6. The bot does not come back online until the vulnerability is fixed and the post-mortem is complete.

Users are never left in the dark about a breach that involves their data. The project's credibility depends on handling these situations with transparency and speed.

---

## Manual Review Process

Periodically, before any training export, a manual review of a sample of interaction logs is conducted. The purpose is to catch anything the automated exclusion filters missed and to verify that the data quality is genuinely representative of good behavior.

**Review sample size:** At minimum 5% of the interactions from the period being exported, randomly selected.

**What reviewers look for:**
- Off-character responses that slipped through automated filters
- Responses that feel robotic or inconsistent with her persona
- Any sensitive content that automated filters did not catch
- Interactions that look like they were produced during an engine error

Anything flagged during manual review is excluded from the training export and tagged in the logging server for reference.

Manual review logs are kept separately and retained for 24 months so there is always a record of what was reviewed and what decisions were made.

---

## Contributor Responsibilities

Anyone with access to the Discord logging server, the Turso databases, or the training data pipeline has a responsibility to:

- Never share log contents outside the team
- Never use individual interaction data to identify or profile specific users
- Report anything that looks like a privacy violation or data quality issue immediately
- Follow the anonymization standard without exception before working with training data
- Never attempt to reverse-engineer user identities from anonymized training pairs

Access to the logging server and databases is granted on a need-to-know basis. Contributors working on the engine or bot core do not automatically get access to the full interaction log archive. That access is separate and deliberate.

---

## Policy Updates

This document is a living reference. As the project moves through phases, new situations will arise that require new rules. When the policy is updated:

- The change is documented with a date and a brief note on what changed and why
- All active contributors are notified of the change
- If the change affects users (new data collection, changed retention, etc.) the disclosure message in active servers is updated within 7 days

No policy change that reduces user protections goes into effect without at least 30 days notice to active server communities.

---

## Version History

```
v1.0   Initial policy written during planning phase. Covers alpha through full release 
       and the training pipeline up to the custom LM.
```
