# Kagevo — Features Checklist PRD

A trackable, build-oriented companion to [PRD.md](./PRD.md). Every feature from the
functional/non-functional requirements is expressed as a tickable item, tagged with a
priority, the build phase it belongs to, and whether it already exists in the design
prototype (`index2.html` / `landing.html`).

> Tick the checkbox when the feature is **shipped in code** — not when it's designed.
> Design coverage is tracked separately by the 🎨 markers so you can see the gap between
> "mocked" and "built" at a glance.

## Legend

| Marker | Meaning |
|--------|---------|
| `[ ]` / `[x]` | Engineering build status — unchecked until shipped |
| 🎨 | UI designed in the prototype |
| 🎨½ | Partially designed in the prototype |
| — | Not yet designed |
| **P0** | MVP-critical (blocks launch) |
| **P1** | MVP |
| **P2** | Post-MVP / retention layer |
| **F** | Future / nice-to-have |
| `Phase n` | Maps to the roadmap phases (0–7) in PRD.md §5 |

## Prototype coverage snapshot

What the current design prototype already shows (design only — none of it is wired to a backend):

| Surface | File · location | Status |
|---------|-----------------|--------|
| Marketing landing + download popup | `landing.html` | 🎨 |
| Login / Register | `index2.html` · CH.00 | 🎨 |
| Voice-first onboarding interview | `index2.html` · CH.01 | 🎨 |
| Forge persona / preset alter egos | `index2.html` · CH.01 | 🎨 |
| Today / Home (nudge · promises · streak) | `index2.html` · CH.02 | 🎨 |
| Fixed goals & habits manager | `index2.html` · CH.03 | 🎨 |
| Safety / crisis flow | `index2.html` · CH.04 | 🎨 |
| Privacy & consent settings | `index2.html` · CH.04 | 🎨 |
| Notification settings (quiet hours, frequency) | `index2.html` · CH.04 | 🎨 |
| Share / Wrapped card | `index2.html` · CH.05 | 🎨 |
| Chat (text conversation) | `index2.html` · CH.06 | 🎨 |
| Decision Journal (case file) | `index2.html` · CH.06 | 🎨 |
| Reality Timeline (story arcs) | `index2.html` · CH.06 | 🎨 |
| Reality Engine dashboard | scores on Home only | 🎨½ |
| Live voice-call screen | mic states + interview stage only | 🎨½ |
| Multi-agent Council | — | — |
| Analytics dashboard | — | — |
| Memory manager | — | — |
| AI provider management (user-facing) | consent/transparency only | 🎨½ |

---

## FR-1 · User Identity Management — **P0** · Phase 0–1

- [ ] User registration & authentication (email + social) — **P0** · Phase 0 · 🎨
- [ ] Session / sign-in, sign-out, password reset — **P0** · Phase 0 · 🎨½
- [ ] Profile management (name, avatar, basics) — **P1** · Phase 1 · 🎨
- [ ] Capture goals — **P1** · Phase 1 · 🎨 *(voice interview)*
- [ ] Capture values — **P1** · Phase 1 · 🎨 *(voice interview)*
- [ ] Capture personality traits — **P1** · Phase 1 · 🎨½
- [ ] Communication preferences (tone / push intensity) — **P1** · Phase 1 · 🎨
- [ ] Fixed life areas seeded (Career, Fitness, Finance, Relationships, Mental Health, Productivity, Discipline, Consistency) — **P1** · Phase 1 · 🎨
- [ ] Import data from external sources — **F** · — · —

## FR-2 · AI Persona ("Kage") Engine — **P1** · Phase 1

- [ ] Create persona — **P1** · Phase 1 · 🎨
- [ ] Edit personality — **P1** · Phase 1 · 🎨
- [ ] Clone persona — **P2** · Phase 1 · —
- [ ] Delete persona — **P1** · Phase 1 · —
- [ ] Configure tone — **P1** · Phase 1 · 🎨
- [ ] Configure communication style — **P1** · Phase 1 · 🎨
- [ ] Configure aggressiveness / intensity — **P1** · Phase 1 · 🎨
- [ ] Configure expertise — **P2** · Phase 1 · 🎨½
- [ ] Seed preset personas / alter egos (Rival, Monk, Drill, Light, etc.) — **P1** · Phase 1 · 🎨

## FR-3 · Multi-Agent Orchestration (The Council) — **P2** · Phase 4+

- [ ] Persona debate — **P2** · Phase 4+ · —
- [ ] Persona consensus — **P2** · Phase 4+ · —
- [ ] Persona voting — **F** · Phase 4+ · —
- [ ] Conflict resolution — **P2** · Phase 4+ · —
- [ ] Final recommendation surfaced to user — **P2** · Phase 4+ · —

## FR-4 · Conversational Interface — **P0** · Phase 2

- [ ] Text conversation — **P0** · Phase 2 · 🎨
- [ ] Voice conversation (record → STT → spoken reply) — **P0** · Phase 2 · 🎨½
- [ ] Streaming responses — **P0** · Phase 2 · 🎨½
- [ ] Real-time transcription / captions — **P1** · Phase 2 · 🎨½
- [ ] Interrupt / barge-in — **P1** · Phase 2 · 🎨½
- [ ] Conversation history — **P1** · Phase 2 · —
- [ ] Resume previous conversation — **P1** · Phase 2 · —
- [ ] Switch voice ↔ text mid-conversation — **P1** · Phase 2 · 🎨½

## FR-5 · Memory Engine — **P1** · Phase 3

- [ ] Automatic memory creation — **P1** · Phase 3 · —
- [ ] Memory categories (identity, goals, habits, relationships, achievements, failures, preferences, beliefs, recurring issues) — **P1** · Phase 3 · —
- [ ] Memory search — **P1** · Phase 3 · —
- [ ] Memory editing — **P2** · Phase 3 · —
- [ ] Memory deletion — **P1** · Phase 3 · —
- [ ] Memory summarization — **P1** · Phase 3 · —
- [ ] Memory shared across chat & voice (single context) — **P1** · Phase 3 · 🎨½ *(implied in Chat)*

## FR-6 · Reality Engine (core USP) — **P0** · Phase 3

- [ ] Track promises, goals, tasks, habits, deadlines — **P0** · Phase 3 · 🎨
- [ ] Promise Accuracy score — **P0** · Phase 3 · 🎨½
- [ ] Execution Rate score — **P0** · Phase 3 · 🎨½
- [ ] Consistency Score — **P0** · Phase 3 · 🎨½
- [ ] Accountability Score — **P0** · Phase 3 · 🎨½
- [ ] Growth Trend — **P1** · Phase 3 · 🎨½
- [ ] Reality Engine dashboard / status screen — **P1** · Phase 3 · 🎨½ *(scores on Home only)*
- [ ] AI verdict ("you overestimated your consistency") — **P0** · Phase 3 · 🎨½

## FR-7 · Fixed Goal Tracks & Habit Tracking — **P1** · Phase 4

- [ ] Habit creation under a fixed track — **P1** · Phase 4 · 🎨
- [ ] Promise/task creation under a fixed track — **P1** · Phase 4 · 🎨
- [ ] Habit reminders — **P1** · Phase 4 · 🎨½
- [ ] Goal milestones within tracks — **P2** · Phase 4 · —
- [ ] Streaks — **P1** · Phase 4 · 🎨
- [ ] Daily review — **P1** · Phase 4 · 🎨½
- [ ] Weekly review — **P1** · Phase 4 · 🎨½ *(Wrapped)*
- [ ] Monthly review — **P2** · Phase 4 · —
- [ ] Habit analytics / progress charts — **P2** · Phase 4 · —
- [ ] Fixed goal dashboard (progress by track) — **P1** · Phase 4 · 🎨
- [ ] Track-level Reality scores — **P2** · Phase 4 · 🎨½
- [ ] Archive habits/tasks without deleting track — **P2** · Phase 4 · 🎨½

## FR-8 · Decision Journal — **P1** · Phase 4

- [ ] Log a decision — **P1** · Phase 4 · 🎨
- [ ] Capture reason — **P1** · Phase 4 · 🎨
- [ ] Capture prediction — **P1** · Phase 4 · 🎨
- [ ] Record outcome — **P1** · Phase 4 · 🎨
- [ ] Reflection — **P1** · Phase 4 · 🎨
- [ ] AI analysis / pattern read — **P1** · Phase 4 · 🎨
- [ ] Feed outcomes back into memory & Reality scoring — **P2** · Phase 4 · —

## FR-9 · Proactive AI — **P1** · Phase 5

- [ ] AI-initiated nudges (skipped habit, upcoming event, stale goal) — **P1** · Phase 5 · 🎨
- [ ] Reply-by-voice from a nudge — **P1** · Phase 5 · 🎨
- [ ] Snooze / dismiss a nudge — **P1** · Phase 5 · 🎨
- [ ] Trigger engine (context → intervention) — **P1** · Phase 5 · —

## FR-10 · Emotional Intelligence — **P2** · Phase 5

- [ ] Detect stress / burnout / frustration / sadness / excitement / confidence — **P2** · Phase 5 · —
- [ ] Adapt response style to detected emotion — **P2** · Phase 5 · —
- [ ] Auto-soften aggressive personas under distress — **P0** *(safety)* · Phase 5 · 🎨 *(crisis flow)*

## FR-11 · Analytics Dashboard — **P2** · Phase 6

- [ ] Per-life-area progress (8 tracks) — **P2** · Phase 6 · —
- [ ] Weekly progress view — **P2** · Phase 6 · 🎨½ *(Wrapped)*
- [ ] Monthly progress view — **P2** · Phase 6 · —
- [ ] Patterns & trends over time — **P2** · Phase 6 · 🎨½ *(Timeline arcs)*

## FR-12 · Notification Engine — **P1** · Phase 5

- [ ] Morning motivation — **P1** · Phase 5 · 🎨½
- [ ] Reality check — **P1** · Phase 5 · 🎨½
- [ ] Habit reminder — **P1** · Phase 5 · 🎨½
- [ ] Goal reminder — **P1** · Phase 5 · 🎨½
- [ ] Weekly review — **P1** · Phase 5 · 🎨½
- [ ] Monthly report — **P2** · Phase 5 · —
- [ ] Milestone achieved — **P2** · Phase 5 · —
- [ ] Notification controls (quiet hours, frequency caps, deep links) — **P1** · Phase 5 · 🎨

## FR-13 · AI Provider Management — **P1** · Phase 6

- [ ] Multi-provider support (OpenAI, Anthropic, Gemini, Grok, local) — **P1** · Phase 6 · —
- [ ] Routing — **P1** · Phase 6 · —
- [ ] Fallback provider — **P1** · Phase 6 · —
- [ ] Cost optimization — **P2** · Phase 6 · —
- [ ] Provider transparency to user — **P1** *(privacy)* · Phase 6 · 🎨½ *(consent box)*

## FR-14 · Privacy & Data Control — **P0** · Phase 0/7

- [ ] Consent management (at signup + ongoing) — **P0** · Phase 0 · 🎨
- [ ] Privacy settings — **P0** · Phase 7 · 🎨
- [ ] Export data — **P1** · Phase 7 · 🎨½
- [ ] Delete data / account — **P0** · Phase 7 · 🎨½
- [ ] Delete memories — **P1** · Phase 7 · 🎨½
- [ ] Download conversations — **P2** · Phase 7 · —
- [ ] Crisis / safety flow (helplines, auto-disable harsh modes) — **P0** · Phase 7 · 🎨

## FR-15 · Reality Timeline — **P2** · Phase 7

- [ ] Chronological life timeline auto-built from memory — **P2** · Phase 7 · 🎨
- [ ] Timeline query interface ("when did I first mention…") — **P2** · Phase 7 · 🎨½
- [ ] Story-arc grouping — **F** · Phase 7 · 🎨

---

## Growth & platform surfaces (beyond core FRs)

- [ ] Marketing landing page (responsive) — **P0** · Phase 0 · 🎨
- [ ] "Get started" → download-app popup (QR + App/Play store) — **P0** · Phase 0 · 🎨
- [ ] Shareable Wrapped / Streak / Persona cards — **P2** · Phase 5 · 🎨
- [ ] Public share link — **P2** · Phase 5 · 🎨½
- [ ] App store presence (TestFlight + Play internal) — **P0** *(beta)* · Phase 7 · —

---

## Non-Functional Requirements

### NFR-1 · Performance — **P0**
- [ ] Voice latency < 2 s
- [ ] Chat first token < 800 ms
- [ ] Conversation resume < 300 ms

### NFR-2 · Availability — **P1**
- [ ] 99.9% target · automatic retry · graceful degradation · offline support

### NFR-3 · Scalability — **P1**
- [ ] Architecture scales 100 → 1K → 100K → 1M without redesign

### NFR-4 · Security — **P0**
- [ ] OAuth · JWT · encryption at rest & in transit · secure secrets · role-based access

### NFR-5 · Privacy — **P0**
- [ ] GDPR · CCPA · data portability · deletion · consent · provider transparency

### NFR-6 · AI Quality — **P1**
- [ ] Persona consistency · memory consistency · response relevance · low hallucination · source attribution *(future)*

### NFR-7 · Reliability — **P0**
- [ ] No memory loss · retry failed calls · fallback provider · auto-recovery · conversation persistence

### NFR-8 · Extensibility — **P1**
- [ ] Easy to add persona / model / notification type / memory type / provider

### NFR-9 · Observability — **P1**
- [ ] Logs · metrics · tracing · crash analytics · AI latency · token usage · cost dashboard

### NFR-10 · Cost Optimization — **P2**
- [ ] Response caching · memory summarization · token budgeting · prompt optimization · provider routing · batch processing

### NFR-11 · Accessibility — **P1**
- [ ] Voice-first · screen reader · large text · captions · color accessibility

### NFR-12 · Maintainability — **P1**
- [ ] Clean architecture · provider abstraction · repository pattern · microservice-ready APIs · modular prompt engine · test coverage

---

## MVP cut line (Phases 0–3) — definition of done

The minimum lovable product is everything **P0** in FR-1, FR-4, FR-6, FR-14 plus the
P1 core of FR-2 and FR-5:

- [ ] **Auth + consent** — user can sign up/in with consent recorded (FR-1, FR-14)
- [ ] **Forge a kage** — voice interview → at least one configured persona (FR-1, FR-2)
- [ ] **Talk to it** — full voice + text conversation loop, streaming, resumable (FR-4)
- [ ] **It remembers** — long-term memory created and recalled across sessions (FR-5)
- [ ] **It keeps you honest** — Reality Engine compares promises vs. actions and delivers a verdict (FR-6)
- [ ] **Safety on** — crisis flow + harsh-mode auto-disable always available (FR-14)
- [ ] Meets NFR-1 latency, NFR-4 security, NFR-5 privacy, NFR-7 reliability

Everything in Phases 4–7 (Decision Journal, Fixed-goal analytics, Proactive AI,
Notifications, multi-provider, Analytics, Timeline) is the **retention layer and moat** —
sequenced after the MVP proves the loop.
