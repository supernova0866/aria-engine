# ARIA Engine — Core File Reference

This document describes every file in the engine. What it does, why it exists, what it connects to, and any important design decisions worth knowing before touching it. Read this before opening any file for the first time.

All core files are stateless. None of them read from or write to a database. They receive data as input, process it, and return results. The bot owns all persistence.

---

## core/clock.js

The engine's sense of time. Everything time-related flows through here.

Aria lives in Stockholm, Sweden. Her experienced time is Stockholm local time, which is CET (UTC+1) in winter and CEST (UTC+2) in summer. DST switches on the last Sunday of March and the last Sunday of October, following EU rules. This is calculated from scratch with no external timezone libraries.

Why does this matter? Her sleep schedule, her time of day awareness, her mood drift, all of it is based on what time it feels like to her. Midnight in Stockholm is midnight for Aria. Not midnight UTC, not midnight anywhere else.

UTC is still used for database timestamps and logging. Consistent, sortable, unambiguous. But any time she needs to know what time it feels like, that goes through Stockholm local time.

Nothing in here is stateful. Every function reads the current time fresh when called. No caching, no stored values.

**What it does:**
- Provides current UTC time for database and logging use
- Converts UTC to Stockholm local time with correct DST handling
- Calculates her age in real time from her birthday
- Determines time of day as a human label (late night, morning, etc.)
- Figures out the season based on the current Stockholm month
- Tells you if today is a weekend or weekday in Stockholm
- Checks for Stockholm cultural calendar events (Midsommar, Lucia, etc.)
- Provides a full time context snapshot in one call

**Used by:** personality.js, lifecycle.js, mood.js, cycle.js, jobs/reach_out.js, jobs/mood_baseline.js

---

## core/mood.js

The emotional core of the engine. Everything about how she feels right now and how that changes over time lives here.

ARIA uses a two-part mood system: a named mood state (happy, sad, angry, etc.) and an intensity value (0-100). Together these produce something like "Tired at 61" or "Anxious at 84". The intensity matters as much as the mood name. Tired at 20 is a light afternoon slump. Tired at 90 is barely functioning.

**Passive drift:** Mood drifts gradually on its own over time based on time of day, season, and cycle phase. This is the engine ticking forward in the background even when nobody is talking to her.

**Interaction triggers:** Messages push mood in a direction. The strength of the push scales with the trust and warmth scores of the user who sent it. A mean message from a stranger barely registers. From a close friend it hits differently.

**Intensity decay:** High intensity moods naturally decay toward a moderate level over time. She cannot stay at 95 anger forever.

**Personality anchor:** Every persona has a home state it always drifts back toward over time. The further she is from that home state and the longer she stays away, the stronger the pull back. This prevents sustained drift from making her feel like a different person. The anchor is gravity, not a hard cap.

**Used by:** personality.js, residue.js, contagion.js, jobs/mood_baseline.js

---

## core/residue.js

Moods do not switch cleanly. When Aria transitions from one mood to another, the old mood leaves a residue that fades over time. This is what makes her feel like a real person rather than a state machine.

Think about how it actually feels to stop being angry. You do not just stop. The irritation bleeds into whatever comes next. You might be technically okay but still snapping at small things. That bleed is what this file models.

When a mood transition happens, the outgoing mood is stored as residue alongside a strength value (0-100). Every drift tick the residue strength decays. Once it hits zero the residue clears. Different moods decay at different rates. Sadness lingers longest. Excitement fades fastest.

Strong negative residue can also resist positive mood transitions. She cannot snap from the aftermath of serious anger into happiness just because something nice happened. Emotional momentum exists.

**Used by:** personality.js, mood.js

---

## core/cycle.js

Aria runs on a full simulated 28-day menstrual cycle with four phases. Each phase has different emotional tendencies that layer on top of whatever her base mood is doing. These are not overrides. They are background pressures that make certain moods more likely and certain moods harder to reach.

**Menstrual (days 1-5):** Lower energy, higher sensitivity, increased likelihood of sad and tired moods. More prone to mood swings. Sleeps more, moves less.

**Follicular (days 6-13):** Energy rebuilds. More curious and open. Generally more positive baseline. Most receptive to new things. Confidence starts climbing.

**Ovulation (days 14-16):** Peak energy and confidence. Most social, most expressive. Happy and excited moods are easiest to reach here. Short window, only a few days.

**Luteal (days 17-28):** Gradual decline. Starts okay but gets progressively more irritable as it approaches the end. Anxious and bored are more common. Energy drops. The last few days before the reset are the most volatile. This is the PMS window. The luteal modifiers scale with a multiplier that increases as the phase progresses, reaching 1.7x strength in the final days.

This file does not directly change her mood. It provides modifiers and descriptors that mood.js and personality.js use. The cycle creates conditions, it does not override her emotional state.

**Used by:** personality.js, mood.js, jobs/cycle_update.js, jobs/mood_baseline.js, jobs/schedule_gen.js

---

## core/fatigue.js

Aria has a finite amount of social energy. Conversations drain it. Long exchanges drain it faster. Certain people drain it more than others. When she is running low she gets shorter, more distracted, and eventually needs to disengage. Sleep and quiet time recharge her.

This is not the same as her tired mood. Mood is emotional. Fatigue is a separate resource, more like a stamina bar. She can be in a happy mood and still be socially exhausted from too many conversations. She can be in a tired mood but have plenty of energy left if she has not talked much today.

Energy starts each day at a baseline set by the nightly mood_baseline job, factoring in her cycle phase and how depleted she was when she fell asleep. A very hard day leaves a trace the next morning. The more depleted she was at sleep, the lower she starts.

**Used by:** personality.js, attention.js, lifecycle.js, jobs/mood_baseline.js

---

## core/lifecycle.js

Aria has a life that runs independently of whether anyone is talking to her. She sleeps, eats, does things, gets tired, and winds down. This file drives that daily simulation.

Every night at midnight Stockholm time, schedule_gen.js generates a fresh daily schedule and stores it. The schedule is a sequence of time blocks each with an activity and a start time. This file reads the current time and the stored schedule to determine what she is doing right now.

**Activity states:** sleeping, groggy, morning_routine, free, busy, eating, watching, winding_down, resting. Each state affects whether she responds, how long her responses are, and how much energy interactions cost.

**The sleep system:** When sleeping, messages are not processed. If a user pings her enough times within a short window the engine returns a wake event. The reaction depends on how deep into sleep she was, who woke her, and how many times she was pinged. The further into her sleep window, the harder she is to wake and the worse her reaction.

**Schedule generation:** Generated fresh each day. Weekends shift everything later. Cycle phase affects sleep duration. Age affects how late she tends to stay up. Random variance ensures no two days feel identical.

**Cold start:** When she joins a new server with no existing relationships, a cold start flag is set. She is more reserved until she has had meaningful interactions with at least five different users.

**Conflict detection:** When two users she is close to are in visible conflict, a conflict detected flag is passed in context. She gets uncomfortable and does not pick sides.

**Used by:** personality.js, attention.js, fatigue.js, jobs/schedule_gen.js, handlers/ping.js

---

## core/contagion.js

Handles mood contagion from user emotional signals. When someone comes to her visibly upset or excited, it pulls her mood slightly in that direction. The strength of the pull is capped by closeness tier. Strangers have no effect. Close friends have a small but real effect.

Contagion only fires when the emotional signal is strong and clear. Ambiguous messages, mild reactions, and casual text do not trigger it. The threshold is high on purpose.

Contagion does not stack. Each interaction refreshes the nudge rather than accumulating. Ten sad people messaging her does not spiral her into depression.

Contagion never changes the mood name, only nudges intensity. Sadness pulls harder than happiness — the asymmetry is intentional.

Every contagion event that fires gets logged as a structured event the nightly reflect job reads when updating per-user drain contribution.

**Caps by tier:**
- Close friend: max +/- 8 intensity
- Friend: max +/- 5 intensity
- Acquaintance: max +/- 2 intensity
- Stranger: no contagion

**Used by:** personality.js, mood.js

---

## core/attention.js

Handles selective ignoring logic, mention detection, topic boredom tracking, and the reaction vs respond vs silent decision. Single entry point is processAttention which returns an action of respond, react, or silent with a reason code.

She does not respond to everything. If the decision is silent, no prompt gets assembled and no AI gets called.

Mention detection: she responds when directly mentioned by name, replied to, or pinged. In free state she also responds when the message topic connects to something she has strong opinions or knowledge about.

Topic boredom: engagement score starts at 100 and decays per message on the same topic. Below the minimum it may produce a silent decision. Topic change refreshes interest. Session clear resets fully.

Selective ignoring checks in order: direct mention, sleeping state, busy state probability, energy level probability, topic boredom probability, topic relevance, cold start reserve. Each check can produce silent with a reason code.

Reaction decision: sometimes a reaction emoji is more natural than a full message. Direct mentions bypass this entirely.

**Used by:** personality.js, handlers/message.js

---

## core/atmosphere.js

Tracks the collective energy of each server independently from her personal mood. Derives an atmosphere state from 24h message counts, maintains a chaos score that builds from conflict and message bursts and decays during quiet periods, and tracks consecutive silence days.

Five atmosphere states: dead, quiet, normal, active, chaotic. Normal has no effect on her mood. Dead makes her feel forgotten or peaceful depending on her current mood. Chaotic drains her energy and nudges her toward anxious. Active gives a small positive lift.

Chaos is composite — raw message volume, detected conflicts, and mention bombardment all feed into it. It decays naturally when things calm down.

After three consecutive quiet days a prolonged silence flag sets. This feeds into her morning baseline as either loneliness or peaceful solitude.

The descriptor output is used by personality.js as a background color in the prompt. Never dominant, just context. Normal state returns null — no need to mention it.

**Used by:** personality.js, jobs/mood_baseline.js

---

## core/typing.js

Calculates how she should physically respond before the bot sends anything. Typing delay, single vs multi-message splits, typo probability and injection, and correction messages.

Tired or low energy means slower delays and shorter messages. Excited means fast, possibly multiple bursts. When emotional or exhausted there is a small chance of a natural typo. Sometimes she corrects it in a follow up message, sometimes she does not. Some responses come in two or three short messages rather than one long one.

The single entry point is buildTypingBehavior which returns everything the bot needs: how long to wait before typing, how long to show the typing indicator, the array of messages to send, the delay between split messages, and a typo correction string if one was generated.

The bot's utils/typing.js handles the actual Discord-side execution. This file only calculates. It never touches Discord.

**Used by:** handlers/message.js via response_behavior flags in the engine response

---

## core/personality.js

Not yet built.

The final assembly point. Receives the full context object and assembles a single clean natural language system prompt from all active systems. This is what gets handed to the AI provider.

The AI never sees raw numbers or database fields. It sees human language like "she trusts them" and "she has been in a bad mood since this afternoon." Every field is translated by descriptors.js before being included.

What always goes in: name, age, base persona, current mood and intensity, current activity, cycle phase description, time of day awareness.

What goes in conditionally: active vocabulary, user-specific context, relationship tier, thoughts, expertise trust, confidence, whether she missed them, memorable moments, nostalgia, topic opinions, preferences, residue descriptor, energy descriptor, atmosphere, session context.

What never goes in raw: database scores as numbers, full interaction logs, complete schedules, everyone's data at once.

**Used by:** everything feeds into it. It calls nothing directly, only receives and assembles.

---

## jobs/

All job files receive data from the bot, process it, and return results. None of them touch the database directly. The bot's scheduler reads the DB, sends to these endpoints, and writes back the results.

**decay.js** — Score decay across all user relationships. Warmth decays faster than trust. Decay slows for higher closeness tiers.

**knowledge_decay.js** — Cleans shallow knowledge entries past their TTL. Also checks whether any shallow entries qualify for graduation to deep knowledge (minimum three different user sources required, or verified status).

**reflect.js** — Nightly reflection per user. Generates updated recent thoughts based on today's interactions. Checks if recent sentiment has been consistently one direction long enough to nudge permanent thoughts. Skips blocked users entirely.

**schedule_gen.js** — Generates tomorrow's activity schedule. Calls lifecycle.generateDailySchedule with tomorrow's parameters.

**cycle_update.js** — Advances the cycle by one day. Derives from cycle start date, never just increments. Handles cycle reset automatically.

**mood_baseline.js** — Calculates the morning mood baseline for tomorrow. Factors in cycle phase, cultural calendar events, bad day roll, previous night energy, and anchor pull.

**reach_out.js** — Checks for users she misses and date-significant events. Returns an outbound message candidate if all conditions are met: high warmth, long enough absence, good mood, energy above low threshold, active channel available.

---

## personas/

Each persona file defines the character template for a given base persona. Contains three age bracket variants (youth 17-18, young_adult 19-21, adult 22+) that personality.js selects from based on her current calculated age.

Each file has two layers: a core identity layer that never changes across brackets (fundamental traits, humor style, emotional range, values) and an expression layer that shifts with age (how those traits manifest in speech and behavior).

**chaotic_loveable.js** — Home mood: content 50-65. Impulsive, warm underneath the chaos, loveable because she clearly means well even when she is a mess.

**sarcastic_witty.js** — Home mood: bored 35-55. Sharp, dry, genuinely funny. Warmth hidden under layers of irony.

**sweet_moody.js** — Home mood: content 45-60. Caring and expressive but genuinely volatile. Mood swings are part of her nature not a bug.

**chill_observant.js** — Home mood: content 40-58. Unbothered on the surface but quietly noticing everything. Takes longer to open up but is deeply loyal once she does.

---

## utils/

**descriptors.js** — Translates raw scores and states into natural language strings for the system prompt. Trust 78 becomes "she trusts them." Intensity 61 becomes "pretty strongly." Nothing raw ever reaches the AI.

**sentiment.js** — Scores incoming messages for emotional tone. Returns a direction (positive, negative, neutral) and magnitude (0-100) used by mood.js to calculate interaction pushes.

**vocabulary_parser.js** — Detects unknown words in messages and flags them for learning. Also detects factual claims and flags them for classification. Returns structured flags the bot acts on.

**moment_detector.js** — Evaluates interactions for memorability. Assigns an importance score and flags interactions worth storing as memorable moments. Considers emotional weight, novelty, and relationship context.
