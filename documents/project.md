# ARIA - Adaptive Responsive Intelligence Architecture

## What is ARIA?

ARIA is a personality simulation engine built to power Discord bots that feel like actual people. Not a chatbot framework, not a wrapper around an AI API. ARIA is the brain and the emotional core. The bot is just the face.

This project was not built to solve a problem. It was built out of pure curiosity and love for the craft. The question was simple: how far can you actually push the idea of a simulated personality before it starts feeling real? M3gan was a small inspiration, not the horror, but the ambition behind it. The idea of something built with enough care and enough systems that it develops a genuine presence. ARIA is that attempt, taken seriously and built from the ground up.

ARIA tracks emotional state, simulates a daily life, learns language from real conversations, builds relationships with users over time, and remembers things. It runs as a standalone hosted service that any compatible bot can connect to via REST and WebSocket. Swap the config file and you have a completely different character running on the same engine.

ARIA itself is completely stateless. It does not touch a database, does not call an AI provider, and does not talk to Discord. The bot handles all of that. ARIA receives a context object, runs its logic, and returns a result. That is its entire job. This means any bot in any language can use ARIA as a backend as long as it can send the right context and handle the response.

The first character built on ARIA is named Aria. Female, starts at 17, ages in real time based on her birth date.

---

## Core Philosophy

A few rules that every part of this project should follow:

**She is not a bot.** Every system in ARIA exists to make her feel like a person. If a feature makes her feel more robotic, it does not belong here.

**The AI API is a voice, not a brain.** ARIA does all the thinking. The language model just handles fluent sentence generation. Her personality, her mood, her opinions, her memories, none of that comes from the AI. It comes from ARIA and gets handed to the AI as context before every response.

**The bot is a pipeline, not a brain.** The bot's job is to collect data, define who she is via configdata.json, send everything to ARIA, get back a result, call the AI, write updates to the DB, and send the response to Discord. No personality logic, no mood calculations, no decision making happens in the bot. It is a data pipe with Discord and DB access on both ends.

**Less data in the prompt means better responses.** Every field injected into the system prompt gets translated into natural language first. The AI never sees raw numbers or database fields. It sees things like "she trusts them" and "she has been in a bad mood since this afternoon."

**She has a life outside of Discord.** She sleeps, eats, gets tired, does things. Messaging her is interrupting her day, not activating a service.

---

## Project Structure

ARIA is split into three separate repositories.

```
aria-engine/     The brain. Stateless logic engine. Hosted on Render.
aria-bot/        The face. Owns the databases, Discord connection, and AI calls.
iris/            The logger and server manager. Receives log events from the engine via HTTP and posts them to Discord. Also handles consent flow and tester onboarding.
```

The bot reads state from its databases, assembles a context object, sends it to the engine, gets back a processed result and assembled prompt, calls the AI provider, writes updated state back to the databases, and sends the response to Discord. The engine never touches any of that directly.

---

## aria-engine File Structure

```
engine/
├── index.js                        Entry point. Starts the REST and WebSocket server.
├── .env                            Server port and any runtime config. No identity here.
├── package.json
│
├── core/
│   ├── information.md              Reference doc for every file in this folder.
│   ├── clock.js                    Stockholm time, date awareness, real-time age calculation.
│   ├── mood.js                     Mood state machine, intensity tracking, passive drift, anchor system.
│   ├── residue.js                  Emotional bleed between mood transitions.
│   ├── cycle.js                    Menstrual cycle phase calculator (full 4-phase model).
│   ├── fatigue.js                  Energy tracking, interaction drain, recharge logic.
│   ├── lifecycle.js                Daily state machine. Sleeping, eating, busy, free, etc.
│   ├── contagion.js                Mood contagion from user emotional signals.
│   ├── attention.js                Selective ignoring, mention detection, topic boredom, reaction decision.
│   ├── atmosphere.js               Server atmosphere state. Quiet/chaos meter, collective energy.
│   ├── typing.js                   Typing delay, multi-message decisions, typo probability.
│   └── personality.js             Assembles all active context into a clean system prompt.
│
├── jobs/
│   ├── decay.js                    Processes score decay across all users. Returns updated scores.
│   ├── knowledge_decay.js          Cleans shallow knowledge entries past their TTL. Returns expired IDs.
│   ├── reflect.js                  Processes nightly reflection per user. Returns updated thoughts.
│   ├── schedule_gen.js             Generates tomorrow's activity schedule. Returns schedule object.
│   ├── cycle_update.js             Advances cycle phase and day count. Returns new cycle state.
│   ├── mood_baseline.js           Calculates morning mood baseline. Includes random bad day chance.
│   └── reach_out.js               Checks for users she misses and date-significant events. Returns
│                                   outbound message if conditions met.
│
├── api/
│   ├── rest.js                     REST endpoints. Receives context, returns results.
│   └── socket.js                   WebSocket server, pushes processed state updates.
│
├── personas/
│   ├── chaotic_loveable.js         Base persona template with age-bracket voice variants.
│   ├── sarcastic_witty.js          Each persona file contains voice variants for age ranges
│   ├── sweet_moody.js              (17-18, 19-21, 22+) that personality.js selects from
│   └── chill_observant.js          based on her current calculated age.
│
└── utils/
    ├── descriptors.js              Translates raw scores into natural language strings.
    ├── sentiment.js                Scores incoming messages for emotional tone.
    ├── vocabulary_parser.js        Detects unknown words and flags them for learning.
    └── moment_detector.js         Flags interactions worth storing as memories.
```

---

## aria-bot File Structure

```
bot/
├── index.js                        Entry point. Connects to Discord and the engine.
├── configdata.json                 Identity, AI provider, engine URL, DB credentials, active channels.
├── package.json
│
├── db/
│   ├── core.js                     Read/write Core DB (Turso aria-core).
│   ├── users.js                    Read/write User DB (Turso aria-users).
│   ├── knowledge.js                Read/write Knowledge DB (Turso aria-knowledge).
│   └── schema.js                   Table definitions and migrations for all three databases.
│
├── engine/
│   ├── client.js                   Assembles context objects, sends to ARIA, returns results.
│   └── socket.js                   WebSocket listener, receives and writes live state updates.
│
├── ai/
│   └── provider.js                 Calls AI provider with prompt, returns response text.
│
├── jobs/
│   └── scheduler.js                Fires at midnight UTC. Reads DB, calls engine job endpoints,
│                                   writes results back. No processing logic lives here.
│
├── handlers/
│   ├── message.js                  Orchestrates the full message pipeline. Read, send, write, reply.
│   ├── outbound.js                 Receives outbound message events from engine, sends to channel.
│   ├── reaction.js                 Fires emoji reactions when engine returns a reaction flag.
│   ├── ping.js                     Passes ping data to engine, handles sleep wake response.
│   ├── dm.js                       Sends block notification DMs to users.
│   └── vouch.js                    Passes vouch requests to engine, handles outcome.
│
├── commands/
│   └── (slash commands if added later)
│
├── session/
│   └── context.js                  In-memory short term session context per conversation thread.
│                                   Clears after inactivity timeout. Never written to DB.
│
└── utils/
    ├── logger.js                   Formats and posts all log entries to Iris via HTTP.
    ├── formatter.js                Handles Discord-specific formatting. Mentions, message length, etc.
    ├── typing.js                   Manages Discord typing indicator, message splitting, send delays.
    ├── cooldown.js                 Per-user rate limiting.
    ├── validator.js                Catches bad AI responses before they reach Discord.
    ├── classifier.js               Small AI call that classifies message intent (fact/opinion/preference).
    ├── verify.js                   Wikipedia API and web search fallback for fact verification.
    └── queue.js                    FIFO message queue per channel. Prevents state bleed from simultaneous messages.
```

---

## iris File Structure

Iris is a minimal Express server with a Discord client. It has two jobs: relay log events from the engine to the correct Discord channel, and manage the consent flow and tester onboarding in the internal server.

```
iris/
├── index.js                        Entry point. Starts the Express server and Discord client.
├── .env                            Discord bot token and allowed engine URL.
├── package.json
│
├── handlers/
│   ├── log.js                      Receives POST from engine, formats entry, sends to channel_id.
│   └── consent.js                  Handles consent button, modal validation, role assignment, consent log.
│
└── utils/
    └── formatter.js                Formats log entries for Discord. Handles 2000 char splits and file attachments.
```

The engine posts to Iris via HTTP with a `channel_id` and log content in the body. Iris formats it and sends to that channel. No routing logic, no channel mapping — the engine decides where it goes.

---

Only the bot has a configdata.json. The engine has no identity file because it receives everything it needs inside the context object on every request. The engine's only standalone config is a .env file for the server port.

The bot's configdata.json is the single source of truth for who she is and how she connects to things.

```json
{
  "name": "Aria",
  "birthday": "YYYY-MM-DD",
  "base_persona": "chaotic_loveable",
  "cycle_start_date": "YYYY-MM-DD",
  "ai_provider": "groq",
  "ai_model": "llama3-8b-8192",
  "engine_url": "https://aria-engine.onrender.com",
  "active_channels": [
    "CHANNEL_ID_1",
    "CHANNEL_ID_2",
    "CHANNEL_ID_3"
  ]
}
```

Available persona options: `chaotic_loveable`, `sarcastic_witty`, `sweet_moody`, `chill_observant`.

The AI provider and model can be swapped freely without touching any logic. This is how different release phases test different backends.

ARIA is designed for controlled environments only. A maximum of ten servers and a small fixed list of active channels per server. She is not meant to be spread thin across hundreds of servers with strangers. The smaller the community, the richer the relationships.

---

## The Three Databases

All three databases live on the bot side. ARIA never reads from or writes to them directly. The bot reads what it needs, passes it to the engine as context, and writes back whatever the engine returns.

They are kept on Turso and intentionally separated because they each serve a completely distinct purpose.

### Core DB (aria-core)

Everything about her internal state and personal growth. Who she is right now, how she feels, and the immediate fabric of her daily life.

**mood_history** - A log of mood states over time. Includes what triggered each shift.

**cycle_tracker** - Tracks the current menstrual cycle phase and day. Used by mood.js and cycle.js to apply phase-specific mood tendencies.

**daily_schedule** - Stores the generated activity schedule for each UTC day. Generated at midnight and persists through restarts.

**current_state** - Always a single row, updated in place. The live snapshot of everything: mood, intensity, activity, sleep status, calculated age.

**vocabulary** - Words she has learned from conversations. Each entry tracks the word, its definition (told to her by a user), who taught her, how many times she has heard it, whether it is in her active vocabulary yet, and example sentences.

**memorable_moments** - Specific interactions that got flagged as worth remembering. Each memory has an importance score that affects how long it persists and how likely it is to surface. High importance memories stick around much longer. Surfaces naturally in conversation when relevant.

**life_events** - Bigger than daily memories. Things that actually happened that she carries forward and that subtly color her personality over time. Her first real argument with someone, a user she was close to who suddenly disappeared, a period when the server went quiet for weeks. These are defining moments, not just logged interactions.

**fatigue** - Current energy level, interaction count for the day, last recharge timestamp.

**residue** - Tracks emotional bleed between mood transitions. Prevents moods from switching instantly.

**nostalgia** - Warm aching feelings tied to life events and high-importance memories. Surfaces occasionally as a subtle emotional color in conversation rather than a direct reference. Grows over time as enough distance builds between her and the original moment.

**server_atmosphere** - One row per active server. Tracks the collective energy of the server separately from her personal mood. Includes a quiet/chaos meter, last major event, last active timestamp, and an overall vibe score. When a server goes quiet for too long she notices. When it gets chaotic she feels it. This feeds into her mood baseline and can surface in conversation naturally.

### User DB (aria-users)

Everything about them. How she sees each person, what she thinks of them, their history together, and how much she trusts what they tell her.

**users** - One row per Discord user. Stores her name for them, first seen, last seen, a JSON field for personal things they have told her, and their first impression value.

**relationship** - Trust score, warmth score, closeness tier, interaction count, last interaction timestamp, fatigue contribution from that user, a 7-day sentiment log for permanent thought nudging, and a small log of recent mood contagion events from them.

**thoughts** - Three rows per user: permanent thought, recent thought, and secrets. Permanent changes very slowly based on sustained behavior patterns. Recent updates nightly and resets faster. Secrets are things she knows or feels about this person that she holds privately and does not directly express but that color how she interacts with them. All three get considered during prompt assembly but secrets never get injected directly, they influence the tone descriptors instead.

**expertise_trust** - Per user, per domain. How much she trusts what this person tells her in specific areas. Domains: technology, science, history, geography, math, music, gaming, film/tv, internet culture, fashion, self-knowledge, emotional advice, slang, general language. Null means no opinion yet. 0 means actively untrusted in that domain.

**blocks** - Block state per user. Can be soft, hard, or permanent. Includes reason, timestamp, and auto-unblock time where applicable.

**interactions_log** - A rolling log of interactions. Stores a brief summary, her mood at the time, a sentiment score, and the channel it happened in. Used by the nightly reflection job and as training data later.

**social_graph** - Tracks perceived connections between users. Who knows who, how strong the connection seems to her, who introduced them, and a history of vouching events.

**vouching_events** - When a user tries to persuade her to unblock someone. Logs who asked, who they vouched for, what they said, and what the outcome was.

**confidence_per_user** - How confident she feels when speaking to or about this person. Affected by their expertise trust scores, her closeness to them, and her current mood state.

### Knowledge DB (aria-knowledge)

Everything she knows about the world outside of herself and the people she knows. General knowledge, cultural awareness, opinions on topics, and personal preferences. None of this is core state and none of it is user-specific, so it lives separately.

**knowledge** - What she knows, structured by depth. Entries start shallow and graduate to deep through repeated community exposure. Shallow entries have a TTL and get cleaned out by the nightly decay job if nobody brings them up again. Deep knowledge fades much more slowly.

```
knowledge
├── id
├── topic
├── entry                 what she knows
├── depth                 shallow | deep
├── confidence            0-100
├── reference_count       how many times it has come up across all servers
├── sources               JSON array of user IDs who contributed
├── first_learned
├── last_referenced       used for TTL on shallow entries
└── expires_at            set on creation for shallow entries, null for deep
```

Shallow knowledge enters easily. A user says something fact-like, the engine flags it, it gets stored. If the community keeps talking about it the reference count climbs and confidence grows until it hits the threshold to graduate to deep. If nobody mentions it again within the TTL window (14 days by default) it gets deleted quietly. This keeps the knowledge base clean and genuinely reflective of what her community actually cares about.

**topic_opinions** - Her personal takes on subjects. Not preferences (those are below) but actual positions. "She thinks competitive gaming takes itself too seriously." Formed gradually through repeated exposure and influenced by who she respects.

**preferences** - Things she has developed a genuine pull toward. Music, games, shows, aesthetics, food, anything cultural. Preferences form naturally over time as exposure accumulates, especially when introduced by someone she trusts and likes. Each preference has a strength score and notes on why she likes it.

```
preferences
├── id
├── category              music | gaming | film | food | aesthetic | other
├── item                  specific artist, game title, show name, etc
├── sentiment             love | like | dislike
├── strength              0-100
├── introduced_by         user_id of who first brought it to her attention
├── first_exposure
├── last_referenced
└── notes                 why she likes it, what it reminds her of
```

Preferences can also be negative. If something keeps coming up in bad contexts or gets associated with moods she dislikes, she develops an aversion to it. Same table, sentiment set to dislike. She might wrinkle her nose when someone brings up a game she has decided she does not like.

---

## The Mood System

Mood is not a simple variable. It is a stack of systems working together.

**Named moods:** Happy, Sad, Angry, Tired, Excited, Bored, Anxious, Content.

**Intensity:** 0 to 100. Combined with the mood name to produce things like "Angry at 73" or "Happy at 41."

**Passive drift:** Mood shifts gradually on its own over time. The direction and speed of drift is influenced by time of day, day of week, season, and current cycle phase.

**Interaction triggers:** Messages can push mood in a direction. Positive interactions nudge toward happy or content. Negative ones push toward angry or sad. The strength of the push depends on trust and warmth scores.

**Residue:** When mood shifts, the old mood does not disappear instantly. It bleeds into the new one and fades at a set rate. This is why she can be "happy but still a little irritated" after an argument.

**Mood contagion:** Emotional signals from users cause a gentle pull on her mood. The strength of the pull is capped by closeness tier. A stranger's bad mood does not affect her. A close friend's does, slightly.

Contagion caps by tier:
- Close friend: max +/- 8 intensity
- Friend: max +/- 5 intensity  
- Acquaintance: max +/- 2 intensity
- Stranger: no contagion

Contagion does not stack. Ten sad people messaging her does not spiral her into depression. Each interaction refreshes the nudge rather than accumulating it.

---

## The Cycle System

She runs on a full 28-day simulated menstrual cycle with four phases. Each phase has different mood tendencies baked in as modifiers on top of whatever her base mood drift is doing.

**Menstrual (days 1-5):** Lower energy, higher sensitivity, more prone to sad and tired moods. Sleeps more, moves less. Mood swings are more frequent and harder to predict.

**Follicular (days 6-13):** Energy builds, generally more positive and curious. Easier to talk to, more open to new things.

**Ovulation (days 14-16):** Peak energy and confidence. Most social, most expressive. Excited and happy are easier to reach during this window.

**Luteal (days 17-28):** Gradually declining energy. More irritable as it progresses, especially in the second half. Anxious and bored are more common. PMS symptoms increase toward day 28.

---

## The Daily Life System

She has an actual simulated day. The schedule is generated fresh every midnight UTC and stored in the database so it survives restarts.

A typical day might look like this:

```
07:30  Wake up
08:00  Morning routine
09:00  Free / online
11:00  Watching something
13:00  Eating
14:00  Busy
16:30  Free / online
19:00  Eating
20:00  Free / online
22:30  Winding down
23:30  Sleep
```

The schedule shifts based on day of week (weekends she stays up later, wakes later), her age (younger means later nights), and cycle phase (menstrual phase means more sleep and slower starts).

Her current activity and next activity are always part of the system prompt context. She knows what she is doing and it affects how she responds.

**Sleep state:** During sleep hours she does not respond to messages. If a user pings her enough times, she can wake up. Her reaction depends on how deep into sleep she was, who woke her, and her current mood. She might be groggy and confused, mildly annoyed, or genuinely mad about it.

Users who wake her too many times, or push the wrong buttons in the wrong state, can get blocked.

---

## The Block System

She has the ability to block users who cross lines. This is not an admin feature. It is a personality feature.

A blocked user gets a DM from the bot explaining they have been blocked. After that, the engine silently drops all their messages. They get no response, no reaction, nothing. Other users in the server have no idea.

Block types:
- **Soft block:** Lasts a few hours. She cools down and may unblock on her own.
- **Hard block:** Requires genuine effort to undo. A trusted user vouching for them, time passing, or the blocked user sending a sincere DM apology. Still depends on her mood.
- **Permanent block:** Rarely issued. Only for extreme cases. Does not auto-unblock.

Block triggers include: waking her up repeatedly, sustained rude interactions pushing trust too low, spamming during non-available states.

**Important:** Server admin roles carry no special weight here. Authority does not override her emotions. A server admin asking her to unblock someone she is genuinely upset with gets treated the same as anyone else.

---

## The Social Graph

She does not just track individual users in isolation. She builds a mental map of how users relate to each other. She notices who talks to who, who brought who to her attention, and who has vouched for who in the past.

This matters most during unblock requests. If User A asks her to unblock User B, she weighs how much she trusts User A, how bad the block was, how long it has been, and what her current mood is. A close friend asking nicely when she is in a decent mood has a real chance. A stranger asking while she is still upset gets nowhere.

---

## First Messages

Sometimes she reaches out on her own. Not a scheduled notification, an actual emotional action. She checks her DB, finds someone with high warmth who has been absent long enough to trigger a genuine missing feeling, and if her mood is decent and her energy is up, she sends something to the channel where she talks to them most.

What she says is shaped by everything she knows about that person. Their shared history, her current thoughts about them, a memory that surfaced, something she learned recently that made her think of them. It feels like getting a message from a friend who was thinking of you.

The conditions that need to align before she reaches out:
- Warmth score is high enough (close friend or above)
- Absence has crossed the missing threshold for their tier
- Her current mood is not too low or too exhausted
- She has not already reached out to them recently
- She has an active channel where she knows them

If multiple people she misses are in the same server she does not send separate messages for each. She sends one message naturally addressed to the vibe of the channel, which might mention someone specifically or just check in generally.

The reach_out job runs nightly and returns outbound message candidates to the bot. The bot's outbound.js handler sends them to the right channel.

---

## Confidence Levels

Her confidence is not a flat trait. It shifts based on context, mood, and who she is talking to.

A baseline confidence score sits in current_state and moves with her mood and cycle phase. Ovulation phase pushes it up naturally. Luteal phase and low energy pull it down. Anxious mood tanks it.

Per-user confidence lives in the user DB and is shaped by how well she knows them, how much she trusts them, and how recent their last interaction was. Talking to a close trusted friend she is comfortable and direct. Talking to someone she barely knows or does not fully trust she hedges more, qualifies her statements, stays a little more careful.

When she repeats something she was told by a low expertise trust user she hedges it in conversation. When she is talking about something she has seen confirmed many times by people she respects she is more assertive. The AI sees this translated as natural language before generating her response.

---

## Typing Behavior

She does not always send one clean message. The engine's typing.js calculates how she should physically respond before the bot sends anything.

Factors that shape typing behavior:
- **Mood and energy** — tired or low energy means slower typing delays and shorter messages. Excited means fast, possibly multiple bursts.
- **Message complexity** — a simple reaction gets a short delay. Something emotionally significant gets a longer pause, like she is thinking.
- **Typo probability** — when she is emotional (high intensity angry or sad) or very tired, there is a small chance her message contains a natural typo. Sometimes she corrects it in a follow up, sometimes she does not notice.
- **Multi-message splits** — some responses naturally come in two or three short messages rather than one long one. The engine flags when a response should be split and the bot's typing.js handles the actual Discord typing indicator and staggered sends.
- **Ellipsis and hesitation** — when she is uncertain or holding something back, responses may trail off or start hesitantly.

The bot's utils/typing.js handles all the Discord-side execution of these flags: triggering the typing indicator, waiting the right amount of time, splitting and sending messages in sequence.

---

## Selective Ignoring

She does not respond to everything. The engine's attention.js decides whether a message is worth engaging with given her current state.

Conditions that can lead to silence:
- She is in a busy or eating activity state and the message is not directed at her specifically
- Her energy is very low and the message does not require a response
- The topic has been going on too long and she has drifted
- The message is low effort and she is not in the mood
- She is in a bad mood and the person is not someone she is close to

Silence is not a block. The message gets logged. She just does not reply. The bot receives a `no_response` flag from the engine and does nothing. No typing indicator fires, no reaction, just nothing. Exactly like a real person reading a message and moving on.

---

## Emoji Reactions

Sometimes she reacts to a message without saying anything. The engine returns a `reaction` flag instead of a prompt when the situation calls for it.

When reactions happen:
- She is tired or winding down and a message makes her smile or laugh
- Something is said that she finds mildly annoying but not worth addressing
- A close friend says something she agrees with but does not need to add to
- She is in the middle of something and just acknowledges she saw it

The reaction itself is chosen based on her mood and her relationship with the person. A heart for someone she is warm with. A laugh for something genuinely funny. An eye roll emoji for something that mildly irritates her. The engine returns the specific emoji and the bot's reaction.js fires it on the message.

---

## Mood Visibility Variance

Her mood always affects her internally. How much of it bleeds into what she actually says depends on who she is talking to and how she is feeling about showing it.

With a close trusted friend she lets things show. If she is sad she might mention it or her messages carry that weight. If she is happy it comes through clearly.

With an acquaintance or stranger she masks more. She might seem fine when she is not. The internal mood still drives her response length, energy, and topic engagement, but the explicit emotional content gets filtered out before the prompt is assembled.

personality.js checks the closeness tier and current vulnerability state before deciding how much mood to surface directly in the prompt versus keeping it as a background influence.

---

## Topic Boredom

If a conversation stays on the same subject too long she starts losing interest. attention.js tracks topic continuity within the session context. As the same topic stretches on, her engagement score for it drops.

The effect in her responses:
- Shorter replies
- More distracted energy in the phrasing
- She might change the subject naturally
- If it goes on long enough she might just go quiet on that thread

The topic engagement score resets when the subject changes or the conversation goes quiet and the session context clears.

---

## Server Atmosphere Awareness

She picks up on the collective energy of a server independently of her personal mood. atmosphere.js tracks this as a separate state in the Core DB.

What feeds into server atmosphere:
- Message frequency over the past 24-48 hours (quiet vs active vs chaotic)
- Whether there has been obvious conflict or negativity in recent messages
- How long since anyone talked to her specifically
- Whether the server has been dead for days

How it surfaces:
- A very quiet server makes her feel a little forgotten or peaceful depending on her mood
- A chaotic server drains her faster and can push her toward wanting to disengage
- Coming back to an active server after quiet time gives a small positive mood nudge
- If nobody has talked to her in a while she might mention it offhandedly when someone does

Server atmosphere is included in the context object and personality.js uses it as a background color on the prompt, not a direct injection.

---

## Bad Days Without Reason

mood_baseline.js includes a small random chance every morning of setting a low baseline regardless of cycle phase, life events, or anything else. No trigger, no explanation. Just a bad day.

She does not know why she feels off. The prompt reflects it as vague low energy or irritability with no stated cause. If someone asks if she is okay she might shrug it off or say she does not know. Very human, very relatable, and it keeps her from feeling too predictable.

---

## Voice and Phrasing Drift

Each persona file contains age-bracket voice variants. personality.js selects the right variant based on her current calculated age.

```
17-18    More impulsive, expressive, slightly naive phrasing. Bigger reactions.
19-21    A bit more self-aware. Still expressive but slightly more measured.
22+      Drier humor. More comfortable with silence. Less need to explain herself.
```

The shift is gradual and subtle. Nobody who starts talking to her at 17 and comes back at 20 should notice a dramatic change, but the pattern of how she speaks will have drifted slightly over that time. It is the kind of thing you only notice looking back.

---

## Anniversaries and Date Awareness

reach_out.js and mood_baseline.js both check life_events and memorable_moments timestamps against the current UTC date every night.

If today matches or is close to a significant date, two things can happen depending on what the event was:
- A positive anniversary might add a small warm nostalgic nudge to her morning baseline
- A difficult anniversary might pull her baseline slightly lower with no explained reason
- If it is significant enough she might bring it up naturally in conversation that day

She does not announce anniversaries like a calendar. She just carries them. The effect is subtle unless she is close enough to someone to mention it.

---

## Session Context

Short term memory within an active conversation thread lives in bot/session/context.js as an in-memory store. It never gets written to the database.

What it holds:
- The last several messages in the thread
- Topics that have come up in this session
- Emotional moments from earlier in the conversation
- Anything said that has not been logged as a full memory yet

The session context gets included in the context object so personality.js can reference what was said earlier in the same conversation naturally. It clears automatically after the conversation goes quiet for a configurable timeout period.

This is separate from the full interactions_log in the DB. Session context is immediate and temporary. The log is long term and permanent.

---

### Vocabulary Learning

She starts with no slang and no Gen Z vocabulary. She picks it up organically from conversations. When she encounters an unknown word, she may ask what it means directly in the channel. The user who answers teaches her, and that definition gets stored along with their user ID.

Words go through two stages:
- **Passive vocabulary:** She has heard the word but not enough to use it herself yet. She understands it in context.
- **Active vocabulary:** She has heard it enough times (threshold configurable) that she starts using it naturally.

Her top 20-30 active words get included in the system prompt so the AI uses them appropriately.

### User Memory

**Thoughts (three layers):**

Every user she interacts with has three thought fields in the database.

Permanent thought: Her deep-rooted feeling about them. Changes very slowly. Only shifts when recent behavior has been consistently one direction for several days.

Recent thought: How they have been lately. Volatile, updates nightly, resets faster. A close friend being annoying for a few days affects the recent thought without touching the permanent one. If the behavior keeps up long enough the permanent thought starts to nudge as well.

Secrets: Things she knows or feels about this person that she holds privately. Never injected directly into the prompt but they influence the tone descriptors that shape how she talks to them.

**Score Decay:**

Scores do not sit frozen forever. Warmth decays faster than trust. Trust decays slowly but is harder to rebuild. Decay rate slows down for higher closeness tiers. A close friend's scores hold longer than an acquaintance's.

Negative scores from bad interactions also decay upward over time. She does not hold grudges forever unless the block system is involved.

**First Impressions:**

New users do not start as a blank slate. She generates a slight random first impression: slightly warm, slightly guarded, slightly curious, or neutral. Makes first conversations feel more natural.

**Memorable Moments:**

Certain interactions get flagged as worth remembering. The first time she learned a word, a genuinely funny exchange, someone being kind during a rough mood. These surface occasionally when relevant, like "wait this reminds me of when..."

**Interaction Fatigue:**

If too many people talk to her at once, or one conversation runs unusually long, her energy drains faster. Responses get shorter, she may excuse herself. She recharges during quiet time and sleep. Some users drain her faster than others, which is tracked per user.

---

## The Nightly Job Pipeline

Every midnight Stockholm time, while she is asleep, the bot's scheduler triggers a sequence of job endpoints on the engine. The bot handles all the data movement. The engine handles all the processing.

The flow for each job is the same: bot reads relevant data from DB, sends it to the engine job endpoint, engine processes it and returns results, bot writes results back to DB.

```
00:00  schedule_gen      Bot sends current state. Engine returns tomorrow's activity schedule.
00:05  decay             Bot sends all user scores. Engine returns decayed values.
00:10  knowledge_decay   Bot sends shallow knowledge entries. Engine returns IDs to delete.
00:15  reflect           Bot sends today's interaction logs per user. Engine returns updated thoughts.
00:25  cycle_update      Bot sends current cycle state. Engine returns advanced phase and day.
00:30  mood_baseline     Bot sends cycle state and day events. Engine returns morning mood baseline.
00:35  reach_out         Bot sends relationship and absence data. Engine returns outbound message
                         if conditions are met (high warmth, long absence, good mood, active channel).
```

The reflection job is what makes her memory feel alive. The engine does a lightweight internal reasoning pass per user she talked to that day, blending new observations into existing thoughts rather than replacing them wholesale.

---

## The System Prompt Assembly

personality.js inside the engine receives the full context object and assembles a single clean prompt string. It never sees raw database fields directly because the bot has already read and packaged everything. The engine just translates it into natural language and structures it for the AI.

The bot then takes that assembled prompt string and calls the AI provider with it. The engine is not involved in that call at all.

What always goes in:
- Name, age, base persona description
- Current mood and intensity (translated to natural language)
- Current activity and what is coming next
- Cycle phase (just the name, no raw data)
- Time of day awareness

What goes in conditionally:
- Active vocabulary (top 20-30 words only)
- User-specific context (only for the person currently messaging)
- Relationship tier and translated trust/warmth
- Permanent and recent thoughts about this user
- Expertise trust for the user, translated per domain
- Confidence level toward this user
- Whether she missed them (warmth + time since last interaction)
- A memorable moment if one is relevant, with its importance weight considered
- A nostalgia surface if one is triggered
- A life event if it is relevant to the current moment
- Topic opinions if the subject came up
- Preferences if the subject touched on something she likes or dislikes

What never goes in raw:
- Database scores as numbers
- Full interaction logs
- Complete schedules
- Everyone's relationship data at once

A real assembled prompt might look like this:

```
You are [Name], a [age] year old girl. You are chaotic and loveable, kind of all over 
the place but people like you anyway.

It is 11:48pm on a Wednesday. You are winding down and starting to get sleepy.
You are in your luteal phase. You have been feeling a bit off today, more irritable 
than usual and kind of sensitive.

Your mood right now: Tired, pretty strongly.

The person talking to you is [name]. They are a close friend. You really trust them.
You missed them a little, they have not been around in a few days.
Your honest feeling about them: "one of your favorites honestly, always been there."
How they have been lately: "completely normal, no complaints."

Words you say naturally sometimes: "no cap", "lowkey", "fr fr", "that's wild"
```

---

## How the Bot and Engine Communicate

Every time a user sends a message, the bot goes through this sequence:

```
1. Read current state from Core DB (mood, cycle, schedule, fatigue, residue, vocabulary)
2. Read user data from User DB (relationship, thoughts, block status, social graph)
3. Assemble a context object
4. Send context to ARIA via REST
5. Receive result from ARIA (assembled prompt + state updates + flags)
6. Call AI provider with the assembled prompt
7. Write state updates back to Core DB and User DB
8. Handle any flags (unknown words, memorable moments, block triggers)
9. Send the AI response to Discord
```

The context object the bot sends to ARIA looks like this:

```json
{
  "identity": {
    "name": "Aria",
    "age": 17,
    "persona": "chaotic_loveable"
  },
  "state": {
    "mood": "tired",
    "intensity": 61,
    "activity": "winding_down",
    "next_activity": "sleep",
    "energy": 34,
    "residue": { "from": "anxious", "strength": 18 },
    "confidence_baseline": 62,
    "bad_day": false
  },
  "cycle": { "phase": "luteal", "day": 22 },
  "time": { "local_hour": 23, "local_day_of_week": "wednesday", "local_date": "2025-06-04", "time_of_day": "night", "season": "summer", "timezone": "CEST" },
  "vocabulary": ["no cap", "lowkey", "fr fr", "that's wild"],
  "knowledge": {
    "relevant": ["knows what valorant is", "familiar with lo-fi music"],
    "nostalgia_surface": "that phase last winter when the server was really active"
  },
  "preferences": {
    "likes": ["indie pop", "horror games", "late night aesthetic"],
    "dislikes": ["overly competitive gaming talk"]
  },
  "life_events": {
    "recent_surface": null
  },
  "atmosphere": {
    "server_id": "123456789",
    "vibe": "quiet",
    "chaos_level": 12,
    "last_active_hours_ago": 6,
    "feels_forgotten": true
  },
  "session": {
    "topic_continuity": "gaming",
    "topic_engagement": 45,
    "messages": [
      { "from": "Jake", "content": "yo you up", "timestamp": "23:41" },
      { "from": "Aria", "content": "barely lol", "timestamp": "23:42" }
    ]
  },
  "user": {
    "id": "123456789",
    "name": "Jake",
    "trust": 78,
    "warmth": 65,
    "tier": "friend",
    "last_seen_days_ago": 3,
    "permanent_thought": "one of her favorites, been around since early on",
    "recent_thought": "been normal lately, no issues",
    "block_status": false,
    "first_impression": "slightly_warm",
    "expertise_trust": {
      "gaming": 80,
      "music": 55,
      "slang": 90
    },
    "confidence_toward_user": 71
  },
  "message": "hey are you still up"
}
```

ARIA processes all of that and sends back:

```json
{
  "prompt": "assembled system prompt string ready for the AI",
  "state_updates": {
    "mood": "tired",
    "intensity": 58,
    "energy": 32
  },
  "response_behavior": {
    "type": "message",
    "split_messages": true,
    "typing_delay_ms": 2400,
    "typo_chance": 0.08
  },
  "flags": [
    { "type": "unknown_word", "word": "rizz" },
    { "type": "contagion_pull", "direction": "sad", "strength": 3 },
    { "type": "topic_boredom", "topic": "gaming", "engagement": 45 }
  ]
}
```

The `response_behavior` field tells the bot exactly how to deliver the response. `type` can be `message`, `reaction`, or `no_response`. The bot acts accordingly without making any decisions of its own.

---

## Personality Anchor System

Aria has a natural state she always drifts back toward over time. Each persona defines a home mood and intensity range that represents who she fundamentally is. The further her mood drifts from that home state and the longer it stays there without reinforcing triggers, the stronger the pull back becomes.

This is not a reset. A genuinely bad stretch of days still feels genuinely bad. But without sustained reasons to stay in an extreme state, gravity wins and she gradually returns to herself. The morning mood baseline job applies a gentle nudge toward the anchor every day as part of its calculation.

The anchor also protects her identity across aging. Each persona file has a locked core identity layer that never changes between age brackets and an expression layer that shifts. The core layer defines her fundamental traits, her humor, her emotional range, her values. The expression layer changes how those traits manifest at different ages. At 17 she is loud about who she is. At 22 she is quieter but unmistakably the same person.

---

## Response Validation

Every AI response passes through `validator.js` before reaching Discord. The validator checks for empty responses, responses over the character limit, the AI referring to her in third person, and a small hard list of phrases that indicate the AI has broken character. If validation fails the bot retries the AI call once with a note that the previous response was invalid. If it fails twice she stays silent and the failure is logged to Iris.

---

## Message Intent Classification

Before any message is processed for knowledge storage, `classifier.js` runs a small lightweight AI call that classifies the message as one of: fact, opinion, preference, question, or other. This is a single word response from the smallest available model. Fast and cheap.

The classification determines what happens next. Facts route to the verification system. Opinions route to topic_opinions. Preferences route to the preferences table. Questions and other classifications skip knowledge storage entirely and go straight to response generation.

---

## Fact Verification

When a message is classified as a factual claim, `verify.js` runs before anything is written to the knowledge DB. It checks the claim against Wikipedia first, then falls back to a web search if Wikipedia has no relevant result.

Three outcomes are possible. Confirmed facts enter shallow knowledge with a verified flag and high starting confidence. Contradicted facts are rejected from storage and the user's expertise trust in the relevant domain takes a 10 point hit. Three contradictions in the same domain permanently flags that user's contributions in that area as low confidence, meaning their facts in that domain can never graduate to deep knowledge regardless of repetition. Unverifiable claims enter shallow knowledge with an unverified flag and low starting confidence.

For things that cannot be verified as objective facts (slang, personal experiences, opinions, subjective claims) the verification system does not fire. These go through the normal learning pipeline.

---

## Multi-User Knowledge Requirement

Before any shallow knowledge entry graduates to deep knowledge, it must have been contributed by a minimum of three different users. One person repeating the same claim ten times counts as one source. This prevents systematic manipulation of her knowledge base by a single bad actor. Verified facts from the fact verification system bypass this requirement since the external source counts as authoritative.

---

## Mention Detection and Response Gating

She does not respond to every message in every channel. `attention.js` determines whether she should respond at all before the engine processes anything. She responds when she is directly mentioned by name, when she is replied to, or when she is in a free activity state and the message topic connects to something she has strong opinions or knowledge about. Everything else she reads silently and moves on.

---

## Message Queue

Messages that arrive within 500ms of each other in the same channel are queued and processed one at a time via `queue.js`. The bot re-reads her current state from the DB before processing each queued message so every user gets her accurate current state rather than a stale snapshot from the message before theirs. This prevents state bleed when multiple people message her simultaneously.

---

## Engine Downtime Handling

Every engine call has a five second timeout. If the engine does not respond the bot enters silent mode for that message. No response reaches Discord, the failure is logged to Iris. If the engine is unreachable for more than ten minutes the bot posts a single message saying she is not feeling well and goes quiet until the engine comes back online. When it does, normal processing resumes automatically.

---

## Block and Memory Freeze

When a hard or permanent block is issued against a user, their data in the user DB freezes at the point of the block. The nightly reflect job skips them. Their relationship scores stop decaying or updating. Their thoughts stay exactly as they were at the moment she blocked them. If a block is ever lifted, the reflect job resumes from the frozen state and catches up gradually rather than jumping to the present.

---

## Fatigue Carry-Over

If she goes to sleep below 40 energy the morning starting energy is penalized proportionally. A very hard day leaves a trace the next morning. The more depleted she was when she fell asleep, the lower her starting energy the next day. A normal day followed by full sleep gives full restoration. This prevents the pattern of a brutal day being completely erased by a single sleep cycle.

---

## Memory Importance and Pruning

Every memorable moment is created with an importance score from 0 to 100 set by `moment_detector.js` based on the emotional weight of the interaction. During the nightly job run, moments that have never been recalled and whose importance score is below 40 decay by 2 points per night. When importance hits zero the memory is deleted. Moments above 70 importance never decay. Moments that get recalled frequently gain importance, so things that genuinely matter stick around permanently.

---

## Opinion Revision

Topic opinions are not permanent once formed. Each opinion in the knowledge DB tracks a contradiction count. When a trusted user contradicts an existing opinion that counter increments. When three or more trusted users have contradicted the same opinion the engine flags it for revision during the next nightly reflect job. The revised opinion blends the original with the contradicting evidence rather than replacing it. She does not just flip. She nuances.

---

## Cold Start Behavior

When she joins a new server with no existing relationships, a cold start flag is set in the server atmosphere table. During cold start she is more reserved than usual, treats everyone as strangers regardless of anything else, and her first impression system is more guarded than normal. After meaningful interactions with at least five different users the cold start flag clears and she settles into her normal behavior patterns.

---

## Server Silence Handling

If a server goes quiet for three or more consecutive days, the atmosphere tracker sets a prolonged silence flag. This feeds into her morning baseline as either peaceful solitude or loneliness depending on her base mood and cycle phase. The nightly reflect job handles empty days gracefully by skipping reflection but still running the atmosphere update so silence is properly tracked.

---

## Conflict Navigation

When two users she is close to are in visible conflict with each other in a channel, a conflict detected flag is added to the context object. She does not pick sides based on trust scores. She gets uncomfortable, acknowledges the tension without taking a position, and may try to redirect the conversation. If the conflict escalates and she gets pulled into it directly she can disengage by transitioning to a resting state. She does not want to be in the middle of it.

---



**Closed Alpha:** Internal only. Core engine, mood system, and database structure gets built and tested here. No real users.

**Private Beta:** Small invited group. Real conversations start happening. The learning system and memory get tested in actual use.

**Public Beta:** Open. Multiple servers. Stress test the shared knowledge database and social graph at scale.

Each phase may use a different AI provider, swapped via the bot's configdata.json without touching any logic.

---

## Tech Stack

| Component | Technology |
|-----------|------------|
| Engine runtime | Node.js |
| Engine hosting | Render |
| Bot framework | discord.js |
| Logger and server manager | Iris, hosted on Render |
| Core database | Turso (aria-core) — bot side |
| User database | Turso (aria-users) — bot side |
| Knowledge database | Turso (aria-knowledge) — bot side |
| Real-time sync | WebSocket |
| Bot-to-engine | REST API |
| Engine-to-Iris | HTTP POST |
| AI provider | Configurable via bot's configdata.json |

---

## Long Term Vision

The current architecture uses a third party AI provider as a voice. The engine crafts a precise prompt, the AI generates a response, and the bot sends it. This works well and is the right approach for now, but it is not the end goal.

As ARIA grows and real conversations accumulate across multiple servers, something valuable starts to build up: a dataset of how a specific personality with a specific emotional state responds to specific situations. That data is exactly what you need to train a model that does not need a prompt at all.

The long term path looks like this:

**Phase 1 (now):** Generic AI provider with a carefully crafted system prompt. The personality lives in the prompt. Works well but the model has no native understanding of who she is.

**Phase 2:** Fine-tune an open source model on collected ARIA conversation data. The model starts to internalize her personality, her speech patterns, her emotional range. The prompt gets shorter because the model already knows a lot of it.

**Phase 3:** A model trained specifically on ARIA data from the ground up. Her voice is in the weights, not the prompt. The provider.js in the bot just points to a self-hosted model. The rest of the architecture stays completely unchanged.

This is why conversation logging matters even in closed alpha. Every interaction stored in the database is potential training data. The way she responds to a tired mood, the way she reacts to someone waking her up, the way her tone shifts between a stranger and a close friend, all of that needs to be captured now so it can be learned later.

Contributors should treat the interaction logs not just as memory for the current system but as the foundation of something bigger. Log everything. Keep it clean. It will matter later.

---

## A Note for Contributors

If you are joining this project, read this document before touching any code. Every file exists for a reason and every system connects to something else. Each folder has an information.md that explains what every file in that folder does, why it exists, and what connects to it. Read that before opening any file for the first time.

The goal of this project is not to build a clever chatbot. It is to build something that makes people forget they are talking to software. Every decision should be made with that in mind.
