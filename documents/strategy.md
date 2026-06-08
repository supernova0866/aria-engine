# ARIA Development Strategy

This document covers the full development roadmap from the first line of code to the eventual creation of a dedicated language model. It is meant to be a living reference. As phases complete and priorities shift, this gets updated.

---

## The Big Picture

ARIA starts as a prompt-driven personality engine powered by a third party AI. Over time, as real conversation data accumulates, the goal is to progressively reduce dependence on that external model until eventually the AI layer is something we built and own ourselves. A model that does not need a prompt because the personality is already in the weights.

That is the destination. Everything between here and there is about building the foundation properly so each phase sets up the next one cleanly.

---

## Phase 0 — Foundation (Now)

Before any code gets written, the foundation needs to be solid. This phase is already mostly done through planning.

**Deliverables:**
- ARIA_Project_Idea.md complete and accurate (done)
- ARIA_Strategy.md written (this document)
- ARIA_Data_Policy.md written (done)
- ARIA_Structure.md written (done)
- Three repos created on GitHub: `aria-engine`, `aria-bot`, `iris`
- Turso databases provisioned: `aria-core`, `aria-users`, `aria-knowledge`
- Private Discord logging server created with all log channels and consent channel set up
- schema.js written and migrations run on all three databases
- configdata.json filled in for the first bot (Aria)
- .env set up for the engine
- .env set up for Iris (Discord token and allowed engine URL)

**Done when:** All three repos exist, databases are live with correct schema, logging server is ready, and the project structure matches the plan exactly.

---

## Phase 1 — Engine Core (Closed Alpha Prep)

Build the engine folder by folder, starting with core. No bot yet. No Discord. Just the engine running locally and returning correct results when given a context object manually.

**Order of development:**

First the engine core files, one by one:
- clock.js
- mood.js
- residue.js
- cycle.js
- fatigue.js
- lifecycle.js
- contagion.js
- attention.js
- atmosphere.js
- typing.js
- personality.js

Then utils:
- descriptors.js
- sentiment.js
- vocabulary_parser.js
- moment_detector.js

Then personas:
- chaotic_loveable.js with all three age bracket variants
- sarcastic_witty.js
- sweet_moody.js
- chill_observant.js

Then jobs:
- decay.js
- knowledge_decay.js
- reflect.js
- schedule_gen.js
- cycle_update.js
- mood_baseline.js
- reach_out.js

Then api:
- rest.js
- socket.js

Then index.js wiring everything together.

**Testing at this stage:** Send hand-crafted context objects to the engine via a REST client like Postman or Thunder Client. Verify the prompt output looks right, state updates are correct, flags fire when they should. No bot involved yet.

**Done when:** The engine accepts any valid context object and returns a correctly assembled prompt, accurate state updates, and appropriate flags every time.

---

## Phase 2 — Bot Pipeline (Closed Alpha Prep)

Build the bot as a clean pipeline connecting Discord, the engine, and the databases. No real users yet, just making sure the full loop works end to end.

**Order of development:**

Database layer first:
- schema.js (all three DBs)
- db/core.js
- db/users.js
- db/knowledge.js

Then the engine connection:
- engine/client.js
- engine/socket.js

Then the AI provider:
- ai/provider.js (start with whichever free provider is chosen for alpha)

Then session:
- session/context.js

Then handlers:
- handlers/message.js
- handlers/ping.js
- handlers/outbound.js
- handlers/reaction.js
- handlers/dm.js
- handlers/vouch.js

Then utils:
- utils/logger.js
- utils/formatter.js
- utils/typing.js
- utils/cooldown.js
- utils/validator.js
- utils/classifier.js
- utils/verify.js
- utils/queue.js

Then jobs:
- jobs/scheduler.js (fires at midnight Stockholm time)

Then Iris:
- iris/handlers/log.js
- iris/handlers/consent.js
- iris/utils/formatter.js
- iris/index.js

Then index.js.

**Testing at this stage:** Run the bot in a private test server with just the developer. Send messages, verify the full loop works. Check that DB reads and writes are correct. Check that logs appear in the logging server. Check that the nightly scheduler fires and writes back correctly.

**Done when:** A full message flow works end to end. Message in, context assembled, engine called, AI called, response delivered, DB updated, logs written. Everything in the right place.

---

## Phase 3 — Closed Alpha

The engine and bot are running. Real testing begins but strictly internal. The goal here is stability and catching everything that breaks in real use before anyone else sees it.

**Who:** 10 people max. All trusted, all briefed on what they are testing. One server only.

**What to focus on:**
- Does the mood system feel natural over days of use
- Does the anchor system prevent personality drift without making her feel flat
- Does the cycle system produce noticeable but not jarring behavior changes
- Does the daily schedule feel realistic
- Does the sleep system work correctly including the wake-up flow
- Do nightly jobs run cleanly and produce sensible updates
- Are logs readable and useful for debugging
- Does the block system behave correctly including the memory freeze
- Does vocabulary learning actually work
- Does the fact verification system catch wrong information reliably
- Does the message queue handle simultaneous messages cleanly
- Does the response validator catch bad AI outputs without false positives
- Does the classifier correctly identify facts vs opinions vs preferences

**What to track:**
- Every bug found and fixed
- Any behavior that feels robotic or off
- Prompt quality. Does she sound like herself consistently
- DB integrity. Are writes and reads staying clean over time

**AI provider for this phase:** Whatever free tier is most stable. Groq with Llama 3 is the current recommendation. Document which model is used and how it performs.

**Logging:** Everything goes to the Discord logging server. Every interaction, every mood shift, every flag, every nightly job run. This data will matter later.

**Done when:** The system runs stably for at least two to three weeks with no critical bugs. Behavior feels consistently human. Logs are clean and structured.

---

## Phase 4 — Private Beta

Open to a small invited group. Still one server. The goal is to see how she behaves with real people who are not the developer. People who will push her in unexpected directions and expose gaps in the systems.

**Who:** 50 people max. Still one server. Wider audience but still curated and invited.

**What to focus on:**
- Does the relationship system build naturally over real interactions
- Does the social graph develop correctly as users talk to each other
- Does vocabulary learning pick up real slang correctly
- Do user thoughts update in ways that feel accurate
- Does she reach out to people she misses in a way that feels natural
- Does the block system trigger at the right times
- Does expertise trust build correctly per domain
- Does the multi-user knowledge requirement prevent manipulation effectively
- Does the opinion revision system produce believable nuanced shifts
- Does cold start behavior feel appropriate for a new environment
- Does conflict navigation feel natural when close users argue
- Does memory pruning keep the memorable moments table clean and relevant

**New things to introduce in this phase:**
- The full memory system gets real data for the first time
- First messages start firing after enough relationship data builds up
- Knowledge DB starts accumulating real entries

**What to track:**
- User feedback on whether she feels real
- Any patterns in interactions that produce robotic or inconsistent responses
- Prompt quality over time as the context gets richer
- Training data quality. Are the interaction logs clean enough to use later

**AI provider for this phase:** Evaluate whether the alpha provider is still the right choice. If Gemini 2.0 Flash or a newer Groq model performs better, swap it in via configdata.json. Document the comparison.

**Done when:** Users report that she genuinely feels like a person in extended interaction. The relationship and memory systems are producing rich, accurate data. No major behavioral bugs remaining.

---

## Phase 5 — Public Beta

Open to anyone, but still capped at three servers maximum. The goal is stress testing the shared knowledge system and social graph at real scale and finding anything that only breaks under higher volume.

**Who:** Three servers, real communities, no curated audience.

**What to focus on:**
- Does shared knowledge across servers stay clean and accurate
- Does the server atmosphere system work correctly across different community vibes
- Does performance hold up with more concurrent users
- Do the nightly jobs stay reliable under higher data volume
- Does log volume become unmanageable in Discord or stay readable

**What to watch for:**
- Knowledge pollution. Bad or wrong information spreading into deep knowledge
- Relationship data getting messy with higher user counts
- Prompt bloat. Does the context object stay lean or start growing too large
- Any edge cases in the block and vouch system that only appear with more people

**Logging transition point:** If Discord log volume gets genuinely noisy during this phase, evaluate adding Axiom or Betterstack for structured logs. Keep Discord for errors and critical events only if that happens.

**Done when:** The system runs stably across three servers over an extended period. Knowledge base is clean. Performance is acceptable. Training data is accumulating at a meaningful rate.

---

## Phase 6 — Full Release

The bot is a finished product at this point. The custom LM is not a requirement to release. The external AI provider handles sentence generation well enough. The LM is an upgrade that comes later, not a gate that has to be passed before shipping.

**Scale:** 10 servers maximum. 10,000 users hard cap across all servers.

**Why these limits:** Controlled enough that the knowledge base stays coherent and data stays clean. Large enough to accumulate meaningful training data at a pace that makes the LM timeline realistic.

**What the product looks like at this point:**
- Fully functional personality engine with years of system refinement behind it
- Rich relationship and memory data built up from beta phases
- Vocabulary and knowledge base shaped by real communities
- Stable, tested, reliable across different server dynamics

**Data accumulation at this scale:** 10 servers with active users generates substantial interaction volume. Every conversation from this point forward is training data. The dataset grows passively as the product runs.

**The external AI provider at full release:** By this point the provider has been evaluated across multiple phases. Whatever performs best on the free tier at the time of release is what ships. provider.js makes swapping trivial if something better becomes available.

**Done when:** The product is stable at scale, users across multiple servers are having genuine experiences with her, and the interaction logs are accumulating clean training data consistently.

---

## Phase 7 — Post Release Stabilization and Data Prep

Before touching the model, the data needs to be clean and the system needs to be genuinely stable at full release scale.

**What happens here:**
- Fix everything that surfaces at full release scale
- Audit the interaction logs for training data quality
- Write a data cleaning script that prepares interaction logs into proper training pairs
- Document the prompt format that becomes the training input
- Evaluate how much data has accumulated and whether it is enough to start fine-tuning

**The training pair format:**

Each logged interaction becomes a training example:

```
Input:  full assembled prompt (mood, cycle, user context, message)
Output: her response
```

The richer the context in the prompt, the better the trained model understands the relationship between emotional state and response style.

**Done when:** Clean dataset exists. System is stable at scale. Ready to start working on the model.

---

## Phase 8 — Fine Tuning

The first step toward the custom LM is not building one from scratch. It is taking an existing open source model and fine-tuning it on ARIA conversation data.

**Goal of this phase:**

The fine-tuned model starts to internalize her voice, her vocabulary, her emotional range. The system prompt gets shorter because the model already knows a lot of it implicitly. Response quality improves because the model has seen thousands of examples of exactly how she responds in exactly her emotional states.

**Model choice:**

Start with a smaller open source model that can be fine-tuned on limited compute. Llama 3 8B or Mistral 7B are reasonable starting points. The goal is not a powerful general model. It is a model that speaks like her specifically. A well fine-tuned 3-7B model focused on one narrow task outperforms a general 13B model on that specific task every time.

**Infrastructure:**

Fine-tuning requires GPU compute. Options at this stage:
- Google Colab Pro for initial experiments
- Runpod or Vast.ai for longer runs
- Hugging Face for hosting the fine-tuned model

**What changes in the bot:**

Nothing except provider.js pointing to the fine-tuned model instead of the external API. The entire rest of the architecture stays identical.

**Done when:** The fine-tuned model produces responses noticeably more consistent with her personality than the base model. Prompt length needed decreases. Response quality is at least as good as the external provider.

---

## Phase 9 — Custom Language Model

The final piece. A model built specifically for sentence generation in the context of ARIA-style personality simulation. Not a general purpose LM. A focused tool that does one thing extremely well.

**What this model does:**

Takes a compressed emotional and relational context and generates a response that sounds like the character it was trained on. No general knowledge retrieval, no complex reasoning, no code generation. Just natural, emotionally consistent sentence generation. The entire problem is narrow by design and that is what makes it achievable.

**Why this is achievable:**

The problem is narrow. The training data is specific. The output space is constrained. A well trained small model focused on one specific task can outperform a much larger general model on that task. The architecture can be simpler because the job is simpler.

**What the prompt looks like at this stage:**

By the time the custom model exists, the prompt might be as short as:

```
mood: tired/61  cycle: luteal  user: close_friend  missing: slightly
message: hey are you still up
```

The model already knows the rest from training. The personality, the vocabulary, the emotional range, all of it is in the weights.

**Hosting:**

Oracle Cloud Always Free tier. The A1 Ampere instance with 4 cores and 24GB RAM runs quantized models in the 7-8B parameter range via CPU inference. Response times are slower than a GPU but Discord's natural typing delays make this invisible. A purpose-built small model will stay well within the memory limits of the free tier indefinitely.

Setup: Ubuntu 22.04, llama.cpp or Ollama for inference, simple REST endpoint, provider.js points to the Oracle instance URL. No change anywhere else in the architecture.

**Done when:** The custom model produces responses indistinguishable in quality from the fine-tuned external model, running entirely on owned infrastructure with no external AI dependency.

---

## Data Strategy Throughout

Every phase contributes to the training data pool. From closed alpha onward, every interaction logged is a potential training example.

**What gets logged for training:**
- Full assembled prompt
- Raw AI response
- Her mood and intensity at the time
- Cycle phase
- Closeness tier of the user she was talking to
- Response behavior type (message, reaction, silence)
- Any flags that fired

**Data hygiene rules:**
- Never log personally identifiable information beyond Discord user IDs
- Interaction summaries in the DB are brief and behavioral, not verbatim quotes
- Full prompt and response pairs go to the Discord logging server as structured entries
- Periodically audit logs for quality and remove anything that does not represent good training examples (broken responses, API errors, edge cases)

**Volume expectations:**
- Closed alpha (10 people, 1 server): hundreds to low thousands of interactions. Stability testing, early data.
- Private beta (50 people, 1 server): tens of thousands. Real patterns emerge, fine-tuning experiments become viable.
- Public beta (3 servers, open): hundreds of thousands over time. Dataset becomes serious.
- Full release (10 servers, 10k users): data accumulates passively at scale. Every conversation from here builds the LM foundation.
- Post release: the dataset is the asset. Clean, structured, and growing every day.

---

## What Success Looks Like

At each phase there is one honest question to ask.

After closed alpha: does she feel like a real person to the 10 people who tested her.

After private beta: do the 50 people forget they are talking to a bot after extended interaction.

After public beta: does she hold up across different communities without breaking character.

After full release: is the product stable at 10 servers and 10k users with clean data accumulating passively.

After fine-tuning: does the model sound more like her than any generic model ever could.

After the custom LM: is the voice fully owned, fully controlled, and impossible to replicate without the data we built.

---

## Release Timeline Summary

```
Closed Alpha     1 server     10 people      Internal testing
Private Beta     1 server     50 people      Curated testers
Public Beta      3 servers    Open           Stress testing
Full Release     10 servers   10k users      Live product, data accumulation
Fine Tuning      —            —              External model shaped to her voice
Custom LM        —            —              Final piece, fully owned voice
```

---

## A Note on Patience

This roadmap covers years, not months. The closed alpha alone should run for weeks before moving on. The training data needed for fine-tuning takes real time to accumulate through real conversations.

Rushing any phase means the next phase starts on a shaky foundation. The interaction logs from a buggy, inconsistent system produce bad training data. Bad training data produces a model that does not sound like her. The whole point of logging everything from day one is that time spent in alpha and beta is not just testing, it is building the dataset that makes the LM possible.

Every conversation she has from day one matters. Treat the data accordingly.
