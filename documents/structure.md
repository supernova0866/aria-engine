# ARIA — Project Structure

Status legend: ✅ Done · 🔲 Not started

---

## aria-engine

```
aria-engine/
├── 🔲 index.js
├── 🔲 .env
├── 🔲 package.json
│
├── core/
│   ├── ✅ clock.js
│   ├── ✅ mood.js
│   ├── ✅ residue.js
│   ├── ✅ cycle.js
│   ├── ✅ fatigue.js
│   ├── ✅ lifecycle.js
│   ├── ✅ contagion.js
│   ├── ✅ attention.js
│   ├── ✅ atmosphere.js
│   ├── ✅ typing.js
│   └── 🔲 personality.js
│
├── jobs/
│   ├── 🔲 decay.js
│   ├── 🔲 knowledge_decay.js
│   ├── 🔲 reflect.js
│   ├── 🔲 schedule_gen.js
│   ├── 🔲 cycle_update.js
│   ├── 🔲 mood_baseline.js
│   └── 🔲 reach_out.js
│
├── api/
│   ├── 🔲 rest.js
│   └── 🔲 socket.js
│
├── personas/
│   ├── 🔲 chaotic_loveable.js
│   ├── 🔲 sarcastic_witty.js
│   ├── 🔲 sweet_moody.js
│   └── 🔲 chill_observant.js
│
└── utils/
    ├── 🔲 descriptors.js
    ├── 🔲 sentiment.js
    ├── 🔲 vocabulary_parser.js
    └── 🔲 moment_detector.js
```

---

## aria-bot

```
aria-bot/
├── 🔲 index.js
├── 🔲 configdata.json
├── 🔲 package.json
│
├── db/
│   ├── 🔲 core.js
│   ├── 🔲 users.js
│   ├── 🔲 knowledge.js
│   └── 🔲 schema.js
│
├── engine/
│   ├── 🔲 client.js
│   └── 🔲 socket.js
│
├── ai/
│   └── 🔲 provider.js
│
├── jobs/
│   └── 🔲 scheduler.js
│
├── handlers/
│   ├── 🔲 message.js
│   ├── 🔲 outbound.js
│   ├── 🔲 reaction.js
│   ├── 🔲 ping.js
│   ├── 🔲 dm.js
│   └── 🔲 vouch.js
│
├── commands/
│   └── 🔲 (slash commands, added later)
│
├── session/
│   └── 🔲 context.js
│
└── utils/
    ├── 🔲 logger.js
    ├── 🔲 formatter.js
    ├── 🔲 typing.js
    ├── 🔲 cooldown.js
    ├── 🔲 validator.js
    ├── 🔲 classifier.js
    ├── 🔲 verify.js
    └── 🔲 queue.js
```

---

## iris

```
iris/
├── 🔲 index.js
├── 🔲 .env
├── 🔲 package.json
│
├── handlers/
│   ├── 🔲 log.js
│   └── 🔲 consent.js
│
└── utils/
    └── 🔲 formatter.js
```

---

## Documents

```
docs/
├── ✅ ARIA_Project_Idea.md
├── ✅ ARIA_Strategy.md
├── ✅ ARIA_Data_Policy.md
├── ✅ ARIA_Structure.md
└── ✅ core_information.md
```
