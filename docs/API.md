# Kagevo API Specification

REST + streaming API for the **mvp.html** life-RPG design. Implemented on the Hono gateway ([TECH_STACK.md](./TECH_STACK.md)); auth via Supabase JWT.

**Design reference:** [mvp.html](../mvp.html) (final UI)  
**Product requirements:** [PRD.md](./PRD.md)

---

## Conventions

| Item | Value |
|------|--------|
| Base URL | `https://api.kagevo.app/v1` |
| Auth | `Authorization: Bearer <supabase_jwt>` |
| Content-Type | `application/json` (unless noted) |
| Timestamps | ISO 8601 UTC (`2026-06-25T07:41:00Z`) |
| IDs | UUID v4 strings |
| Pagination | `?cursor=` + `?limit=` (default 20, max 100) |

### Standard success envelope

```json
{
  "data": { },
  "meta": {
    "request_id": "req_01J..."
  }
}
```

### Standard error envelope

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Human-readable summary.",
    "details": [
      { "field": "goals", "message": "Maximum 3 season goals allowed." }
    ],
    "request_id": "req_01J..."
  }
}
```

### Global error codes

| HTTP | Code | When |
|------|------|------|
| 400 | `VALIDATION_ERROR` | Invalid body, query, or business rule |
| 401 | `UNAUTHORIZED` | Missing or invalid JWT |
| 403 | `FORBIDDEN` | Valid auth but not allowed (e.g. not guild member) |
| 404 | `NOT_FOUND` | Resource does not exist |
| 409 | `CONFLICT` | State conflict (e.g. day already closed) |
| 422 | `UNPROCESSABLE` | Semantically invalid (e.g. skip all MITs with active streak) |
| 429 | `RATE_LIMITED` | Too many AI or auth attempts |
| 503 | `AI_UNAVAILABLE` | LLM/STT/TTS provider down; retry later |

---

## Domain model (mvp)

```text
Legacy → Saga → Season (90d) → Chapter (~30d) → Weekly Quest → Daily Plan
                                                      ↓
                              Priority (MIT ×3) · Side Quest (×5) · Quick Task
                                                      ↓
                                                    XP → Character (level, attributes, titles)
```

### Enums

**Sensei character** (`sensei_id`): `naruto` | `vegeta` | `goku` | `levi` | `tanjiro` | `luffy`

**Identity** (`identity_id`): `builder` | `athlete` | `learner` | `creator` | `professional` | `better` | `custom`

**Season goal** (`goal_id`): `health` | `discipline` | `career` | `learn` | `relationships` | `finance` | `project` | `balance`

**Challenge** (`challenge_id`): `procrastinate` | `motivation` | `distracted` | `notime` | `lost` | `unfinished`

**Daily time budget** (`time_budget_minutes`): `15` | `30` | `60` | `120` | `180`

**Mission type**: `priority` | `side_quest` | `quick_task`

**Mission status**: `pending` | `in_progress` | `done` | `skipped` | `missed`

**Difficulty / XP**: `easy` (10) | `medium` (30) | `hard` (75) | `boss` (300)

**Story node status**: `locked` | `active` | `done`

---

## 1. Authentication

Maps to **PAGE 00a / 00b** — sign in, register, OAuth.

### `POST /auth/register`

Create account and empty user profile.

**Request**

```json
{
  "name": "Kanishk",
  "email": "hero@kagevo.app",
  "password": "secure-password-min-8"
}
```

**Response `201`**

```json
{
  "data": {
    "user": {
      "id": "usr_...",
      "name": "Kanishk",
      "email": "hero@kagevo.app",
      "onboarding_completed": false
    },
    "session": {
      "access_token": "eyJ...",
      "refresh_token": "...",
      "expires_in": 3600
    }
  }
}
```

**Errors**

| Code | HTTP | Condition |
|------|------|-----------|
| `VALIDATION_ERROR` | 400 | Password &lt; 8 chars, invalid email |
| `CONFLICT` | 409 | Email already registered |

---

### `POST /auth/login`

**Request**

```json
{
  "email": "hero@kagevo.app",
  "password": "········",
  "remember_me": true
}
```

**Response `200`**

```json
{
  "data": {
    "user": {
      "id": "usr_...",
      "name": "Kanishk",
      "email": "hero@kagevo.app",
      "onboarding_completed": true
    },
    "session": {
      "access_token": "eyJ...",
      "refresh_token": "...",
      "expires_in": 3600
    }
  }
}
```

**Errors**

| Code | HTTP | Condition |
|------|------|-----------|
| `UNAUTHORIZED` | 401 | Wrong email or password |

---

### `POST /auth/oauth`

Apple or Google sign-in.

**Request**

```json
{
  "provider": "apple",
  "id_token": "provider-issued-jwt"
}
```

`provider`: `apple` | `google`

**Response `200`** — same shape as login.

**Errors**

| Code | HTTP | Condition |
|------|------|-----------|
| `VALIDATION_ERROR` | 400 | Unknown provider |
| `UNAUTHORIZED` | 401 | Invalid provider token |

---

### `POST /auth/forgot-password`

**Request:** `{ "email": "hero@kagevo.app" }`  
**Response `202`:** `{ "data": { "sent": true } }` (always success to prevent email enumeration)

---

## 2. Onboarding

Maps to **CH.01** — 8-screen flow ending in AI journey generation.

### `GET /onboarding/options`

Static option lists for all onboarding steps.

**Response `200`**

```json
{
  "data": {
    "sensei_characters": [
      {
        "id": "vegeta",
        "name": "Vegeta",
        "glyph": "王",
        "modes": ["roast", "brutal"],
        "notification_style": "Roast messages when you make excuses.",
        "sample_quote": "A warrior doesn't skip leg day. Pathetic."
      }
    ],
    "identities": [
      { "id": "builder", "label": "Builder", "glyph": "造", "description": "Create, ship, make things happen." }
    ],
    "challenges": [
      { "id": "procrastinate", "label": "I procrastinate", "glyph": "遅" }
    ],
    "season_goals": [
      { "id": "discipline", "label": "Build discipline & consistency", "glyph": "律" }
    ],
    "time_budgets": [
      { "minutes": 30, "label": "30 min", "title": "Steady momentum", "hint": "2 MITs · daily progress" }
    ],
    "limits": {
      "max_identities": 3,
      "max_season_goals": 3
    }
  }
}
```

---

### `POST /onboarding/submit`

Submit answers after screens 1–6 (before loading/reveal).

**Request**

```json
{
  "sensei_id": "vegeta",
  "identity_ids": ["builder", "better"],
  "custom_identity": null,
  "challenge_id": "procrastinate",
  "goal_ids": ["discipline", "career", "health"],
  "time_budget_minutes": 30
}
```

**Response `200`**

```json
{
  "data": {
    "onboarding_session_id": "obs_...",
    "valid": true
  }
}
```

**Errors**

| Code | HTTP | Condition |
|------|------|-----------|
| `VALIDATION_ERROR` | 400 | &gt; 3 identities/goals; `custom` identity without `custom_identity` name |
| `VALIDATION_ERROR` | 400 | Invalid enum id |

---

### `POST /onboarding/generate-journey`

AI creates Season I, Chapter 1, Week 1, and today's priorities. Maps to loading + reveal screens.

**Request**

```json
{
  "onboarding_session_id": "obs_..."
}
```

**Response `201`** (may take 3–15s; client polls or uses SSE — see streaming section)

```json
{
  "data": {
    "journey": {
      "character_title": "The Builder",
      "season": {
        "id": "sea_...",
        "name": "Season I",
        "theme": "The Awakening",
        "duration_days": 90,
        "day_number": 1
      },
      "chapter": {
        "id": "chp_...",
        "number": 1,
        "name": "Foundation — show up & ship",
        "duration_days": 30
      },
      "week": {
        "id": "wk_...",
        "name": "Complete your first weekly Quest",
        "quest_target": "Ship Something"
      },
      "mentor": {
        "sensei_id": "vegeta",
        "name": "Vegeta"
      },
      "today_priorities": [
        { "rank": 1, "title": "Complete your #1 MIT first", "xp_reward": 30 },
        { "rank": 2, "title": "One career-focused deep block", "xp_reward": 30 },
        { "rank": 3, "title": "Move your body today", "xp_reward": 25 }
      ],
      "time_budget_minutes": 30
    },
    "user": {
      "onboarding_completed": true
    }
  }
}
```

**Errors**

| Code | HTTP | Condition |
|------|------|-----------|
| `NOT_FOUND` | 404 | Invalid `onboarding_session_id` |
| `AI_UNAVAILABLE` | 503 | Generation failed after retries |
| `CONFLICT` | 409 | User already completed onboarding |

---

### `POST /onboarding/complete`

Mark onboarding done and start Season I (idempotent if journey already created).

**Request:** `{}`  
**Response `200`:** `{ "data": { "redirect": "journey/home" } }`

---

## 3. Journey — Daily dashboard

Maps to **CH.02** — home tab, victory/skipped states, day navigation.

### `GET /journey/today`

Primary dashboard payload. Target: &lt; 300ms (NFR).

**Query:** `?date=2026-06-25` (optional; default today in user timezone)

**Response `200`**

```json
{
  "data": {
    "date": "2026-06-25",
    "day_number": 18,
    "season_total_days": 90,
    "streak_days": 18,
    "xp_today": 320,
    "xp_missed_today": 0,
    "day_progress_percent": 67,
    "day_status": "in_progress",
    "season": {
      "id": "sea_...",
      "name": "Season I",
      "chapter_name": "Building momentum",
      "chapter_day": 18,
      "chapter_progress_percent": 60,
      "momentum_score": 74
    },
    "mentor_strip": {
      "sensei_id": "naruto",
      "name": "NARUTO",
      "glyph": "渦",
      "message": "Every small step today gets you closer to your dream."
    },
    "priorities": [
      {
        "id": "mis_...",
        "rank": 1,
        "title": "Deep Work",
        "icon": "⚡",
        "status": "pending",
        "xp_reward": 30,
        "difficulty": "medium"
      }
    ],
    "side_quests": [
      {
        "id": "mis_...",
        "title": "Hydrate",
        "icon": "💧",
        "status": "done",
        "xp_reward": 10
      }
    ],
    "quick_tasks": [
      { "id": "mis_...", "title": "Buy groceries", "status": "pending" }
    ],
    "weekly_card": {
      "goals_completed": 2,
      "goals_total": 3,
      "week_quest_progress": "3/5 sessions"
    },
    "insight": {
      "sensei_name": "NARUTO",
      "message": "Two of three priorities done. Finish Read before midnight to keep momentum."
    },
    "filters": {
      "mits_pending": 1,
      "done": 2,
      "skipped": 0
    }
  }
}
```

`day_status`: `in_progress` | `complete` | `skipped_heavy` | `empty`

**Errors**

| Code | HTTP | Condition |
|------|------|-----------|
| `NOT_FOUND` | 404 | No active season for user |

---

### `GET /journey/days/{date}`

Historical day view (prev/next day navigation).

**Response `200`** — same shape as `/journey/today`.

**Errors**

| Code | HTTP | Condition |
|------|------|-----------|
| `NOT_FOUND` | 404 | Date before journey start or invalid format |

---

### `POST /journey/replan`

Maps to **Ask Sensei sheet** (PAGE 02f).

**Request**

```json
{
  "reason": "less_time",
  "message": "I only have 20 minutes today"
}
```

`reason`: `replan` | `less_time` | `tired` | `traveling` | `finished_early`

**Response `200`**

```json
{
  "data": {
    "day_plan_id": "dpl_...",
    "priorities": [
      { "rank": 1, "title": "Ship investor deck — 20 min", "xp_reward": 30 }
    ],
    "sensei_message": "Fine. One mission. Twenty minutes. No excuses.",
    "side_quests_adjusted": true
  }
}
```

**Errors**

| Code | HTTP | Condition |
|------|------|-----------|
| `CONFLICT` | 409 | Day already marked complete |
| `AI_UNAVAILABLE` | 503 | Re-plan generation failed |

---

## 4. Missions

Daily priorities (MITs), side quests, and quick tasks.

### Limits (enforced server-side)

| Type | Max per day | XP |
|------|-------------|-----|
| `priority` | 3 | 10–75 by difficulty |
| `side_quest` | 5 | 10–15 |
| `quick_task` | unlimited | 0 |

---

### `POST /missions`

Add mission (Adjust Today flows).

**Request**

```json
{
  "date": "2026-06-25",
  "type": "side_quest",
  "title": "Stretch",
  "icon": "🤸",
  "difficulty": "easy"
}
```

**Response `201`**

```json
{
  "data": {
    "mission": {
      "id": "mis_...",
      "type": "side_quest",
      "title": "Stretch",
      "status": "pending",
      "xp_reward": 10
    }
  }
}
```

**Errors**

| Code | HTTP | Condition |
|------|------|-----------|
| `VALIDATION_ERROR` | 400 | Exceeds daily slot limits |
| `CONFLICT` | 409 | Day closed |

---

### `PATCH /missions/{mission_id}`

Update title, reorder rank, or edit (not after day close).

**Request**

```json
{
  "title": "90 min deep work",
  "rank": 1
}
```

**Response `200`:** `{ "data": { "mission": { ... } } }`

**Errors**

| Code | HTTP | Condition |
|------|------|-----------|
| `FORBIDDEN` | 403 | Mission belongs to another user |
| `CONFLICT` | 409 | Cannot edit completed/skipped mission |

---

### `POST /missions/{mission_id}/complete`

Mark done; awards XP, updates streak, chapter progress.

**Request**

```json
{
  "completed_at": "2026-06-25T14:02:00Z",
  "focus_session_id": "foc_..."
}
```

**Response `200`**

```json
{
  "data": {
    "mission": {
      "id": "mis_...",
      "status": "done",
      "xp_earned": 30
    },
    "xp_today": 350,
    "day_progress_percent": 100,
    "day_complete": true,
    "streak_days": 19,
    "character_xp_total": 8450,
    "level_up": null
  }
}
```

**Errors**

| Code | HTTP | Condition |
|------|------|-----------|
| `CONFLICT` | 409 | Already completed or skipped |
| `NOT_FOUND` | 404 | Invalid mission id |

---

### `POST /missions/{mission_id}/skip`

Maps to **PAGE 02d** skip confirmation.

**Request**

```json
{
  "action": "skip",
  "reschedule_to": null
}
```

`action`: `skip` | `reschedule`  
If `reschedule`: `{ "reschedule_to": "2026-06-26" }`

**Response `200`**

```json
{
  "data": {
    "mission": {
      "id": "mis_...",
      "status": "skipped",
      "xp_earned": 0
    },
    "xp_missed_today": 135,
    "streak_at_risk": true,
    "sensei_message": "Running away from the hard mission? How predictable.",
    "story_progress_paused": true
  }
}
```

**Errors**

| Code | HTTP | Condition |
|------|------|-----------|
| `UNPROCESSABLE` | 422 | All 3 MITs skipped — streak break warning acknowledged required |
| `CONFLICT` | 409 | Mission already done |

---

### `DELETE /missions/{mission_id}`

Remove a pending mission from today's plan.

**Response `204`**

**Errors:** `NOT_FOUND`, `CONFLICT` (cannot delete completed)

---

## 5. Focus sessions

Maps to **PAGE 02c**.

### `POST /focus/sessions`

Start focus timer on a priority.

**Request**

```json
{
  "mission_id": "mis_...",
  "planned_duration_seconds": 1500
}
```

**Response `201`**

```json
{
  "data": {
    "session": {
      "id": "foc_...",
      "mission_id": "mis_...",
      "status": "active",
      "started_at": "2026-06-25T07:52:00Z",
      "planned_duration_seconds": 1500,
      "xp_on_complete": 30,
      "sensei_hint": "Believe it — but actually do the work."
    }
  }
}
```

---

### `PATCH /focus/sessions/{session_id}`

Pause, resume, or complete.

**Request**

```json
{
  "action": "complete"
}
```

`action`: `pause` | `resume` | `complete` | `cancel`

**Response `200`**

```json
{
  "data": {
    "session": {
      "id": "foc_...",
      "status": "completed",
      "elapsed_seconds": 1423
    },
    "mission_completed": true,
    "xp_earned": 30
  }
}
```

**Errors**

| Code | HTTP | Condition |
|------|------|-----------|
| `CONFLICT` | 409 | Session not active |
| `NOT_FOUND` | 404 | Invalid session |

---

## 6. Story

Maps to **CH.03** — Season, chapter, quest, boss, timeline, edit/regenerate.

### `GET /story/current`

Full story tab for active season.

**Response `200`**

```json
{
  "data": {
    "season": {
      "id": "sea_...",
      "name": "Season I",
      "theme": "The Awakening",
      "day_number": 18,
      "duration_days": 90,
      "progress_percent": 42,
      "days_remaining": 72,
      "status": "active"
    },
    "story_path": [
      { "node": "season", "icon": "🌅", "label": "Season I — The Awakening", "status": "done" },
      { "node": "chapter", "chapter_number": 1, "label": "Foundation", "status": "done" },
      { "node": "chapter", "chapter_number": 2, "label": "Building momentum", "status": "active" },
      { "node": "chapter", "chapter_number": 3, "label": "Mastery", "status": "locked" },
      { "node": "boss", "label": "Destroy procrastination", "status": "locked" },
      { "node": "season_end", "label": "Season complete", "status": "locked" }
    ],
    "current_chapter": {
      "id": "chp_...",
      "number": 2,
      "name": "Building momentum",
      "days_remaining": 8,
      "missions_completed": 12,
      "missions_total": 20,
      "progress_percent": 60
    },
    "weekly_quest": {
      "id": "que_...",
      "name": "Complete 5 deep work sessions",
      "progress": 3,
      "target": 5,
      "days_remaining": 2,
      "xp_reward": 300
    },
    "boss_battle": {
      "id": "bos_...",
      "name": "Destroy procrastination",
      "status": "unlocked",
      "requirement": "Complete all MITs for 7 consecutive days",
      "days_until": 12,
      "xp_reward": 500,
      "badge": "Builder badge"
    },
    "timeline": [
      { "date": "2026-01-14", "event": "Started journey", "status": "done" },
      { "date": "2026-02-02", "event": "Completed chapter 1 · Foundation", "status": "done" },
      { "date": null, "event": "Win first boss battle", "status": "pending" }
    ],
    "narrative_summary": {
      "title": "Your story this month",
      "body": "You struggled during the first week. Instead of quitting, you came back...",
      "generated_at": "2026-06-25T10:00:00Z"
    },
    "achievements_recent": [
      { "id": "ach_...", "name": "No zero days", "earned_at": "2026-02-12" }
    ],
    "future_journey": [
      { "step": "Next chapter", "label": "Mastery" },
      { "step": "Next boss", "label": "Destroy procrastination" },
      { "step": "Next season", "label": "Season II · The Forge" }
    ]
  }
}
```

**Errors:** `NOT_FOUND` if no season (return empty state payload with `has_story: false`)

---

### `GET /story/arcs`

Life arcs timeline (PAGE 02l).

**Response `200`**

```json
{
  "data": {
    "arcs": [
      {
        "id": "arc_...",
        "volume": "壱",
        "title": "Foundation",
        "period": "Day 1–30",
        "status": "complete",
        "summary": "First habits locked. Streak began."
      },
      {
        "id": "arc_...",
        "volume": "弐",
        "title": "Building Momentum",
        "period": "Day 31 · NOW",
        "status": "active",
        "summary": "12/20 missions done. Boss in 12 days."
      }
    ]
  }
}
```

---

### `POST /story/season/edit`

Maps to **PAGE 03d** — Edit Goals / Change Timeline / Regenerate Plan.

**Request**

```json
{
  "action": "regenerate_plan",
  "goal_ids": ["discipline", "career", "health"],
  "chapter_end_date": "2026-07-15"
}
```

`action`: `edit_goals` | `change_timeline` | `regenerate_plan`

**Response `200`**

```json
{
  "data": {
    "season_id": "sea_...",
    "updated": true,
    "narrative_note": "Regenerating your plan keeps the arc — Sensei only reshuffles daily missions.",
    "today_plan_regenerated": true
  }
}
```

**Errors**

| Code | HTTP | Condition |
|------|------|-----------|
| `VALIDATION_ERROR` | 400 | &gt; 3 goals; timeline in past |
| `CONFLICT` | 409 | Season already completed |

---

### `POST /story/boss/{boss_id}/attempt`

Record boss battle progress (auto-evaluated from MIT streak).

**Response `200`**

```json
{
  "data": {
    "boss": {
      "id": "bos_...",
      "status": "defeated",
      "xp_earned": 500
    },
    "achievement_unlocked": {
      "id": "ach_...",
      "name": "Boss slayer"
    }
  }
}
```

**Errors:** `UNPROCESSABLE` if requirements not met

---

## 7. Sensei

Maps to **CH.04** — voice-first mentor, chat, weekly review.

### `GET /sensei/home`

Morning brief + today's focus line.

**Response `200`**

```json
{
  "data": {
    "sensei": {
      "id": "vegeta",
      "name": "Vegeta",
      "glyph": "王",
      "modes": ["roast", "brutal"]
    },
    "context": {
      "chapter": "Ch. 2",
      "day_number": 18,
      "boss_days_remaining": 12,
      "streak_days": 7
    },
    "morning_brief": {
      "delivered_at": "2026-06-25T07:30:00Z",
      "text": "You cleared 2 of 3 MITs yesterday — Confidence grew. Today: ship the deck, deep work 90 min, run 5K.",
      "audio_url": null
    },
    "today_focus": ["① Ship investor deck", "② Deep work", "③ Run 5K"]
  }
}
```

---

### `POST /sensei/voice/transcribe`

Upload audio chunk (push-to-talk). Returns transcript.

**Request:** `multipart/form-data` — `audio` (webm/m4a), optional `session_id`

**Response `200`**

```json
{
  "data": {
    "transcript": "I'm feeling lazy.",
    "confidence": 0.94,
    "session_id": "sns_..."
  }
}
```

**Errors**

| Code | HTTP | Condition |
|------|------|-----------|
| `VALIDATION_ERROR` | 400 | Empty audio, unsupported format |
| `AI_UNAVAILABLE` | 503 | STT failure |

---

### `POST /sensei/chat`

Text or post-voice message. Streaming variant: `POST /sensei/chat/stream` (SSE).

**Request**

```json
{
  "message": "Help me focus.",
  "session_id": "sns_...",
  "context": "journey_home"
}
```

**Response `200`**

```json
{
  "data": {
    "session_id": "sns_...",
    "reply": {
      "id": "msg_...",
      "role": "sensei",
      "text": "One task: Ship investor deck. Timer 25 min. Phone away. Don't disappoint me.",
      "mode": "brutal"
    },
    "suggested_actions": [
      { "type": "start_focus", "mission_id": "mis_..." }
    ]
  }
}
```

**Errors:** `429` rate limit; `503` AI down

---

### `GET /sensei/chat/sessions/{session_id}/messages`

Paginated chat history.

**Query:** `?cursor=&limit=50`

**Response `200`**

```json
{
  "data": {
    "messages": [
      { "id": "msg_...", "role": "user", "text": "I'm feeling lazy.", "created_at": "..." },
      { "id": "msg_...", "role": "sensei", "text": "Lazy? You cleared 2 MITs yesterday...", "created_at": "..." }
    ]
  },
  "meta": {
    "next_cursor": null,
    "request_id": "req_..."
  }
}
```

---

### `GET /sensei/weekly-review`

Maps to **PAGE 04c**.

**Response `200`**

```json
{
  "data": {
    "week_number": 6,
    "audio_url": "https://...",
    "duration_seconds": 240,
    "transcript": {
      "wins": ["Deck draft shipped", "4 perfect days"],
      "misses": ["2 runs skipped"],
      "next_week_plan": "Mon–Wed deck + deep work · Thu boss prep"
    },
    "closing_message": "You moved forward. Next week — boss streak starts. No excuses."
  }
}
```

**Errors:** `NOT_FOUND` if review not yet generated (Sunday 8 AM user local)

---

### `POST /sensei/weekly-review/generate`

Trigger generation (cron usually handles this).

**Response `202`:** `{ "data": { "job_id": "job_..." } }`

---

## 8. Journal

Maps to **PAGE 02k**.

### `GET /journal/entries`

**Query:** `?cursor=&limit=20`

**Response `200`**

```json
{
  "data": {
    "entries": [
      {
        "id": "jrn_...",
        "day_number": 17,
        "reflection": "Shipped the deck draft. Skipped evening read — phone won.",
        "tomorrow_intent": "Deep work before meetings. Read before 10 PM.",
        "sensei_note": "Momentum held — one slip won't break the arc.",
        "xp_earned": 15,
        "created_at": "2026-06-24T21:30:00Z"
      }
    ]
  }
}
```

---

### `POST /journal/entries`

**Request**

```json
{
  "date": "2026-06-25",
  "reflection": "Two priorities done. Workout slipped.",
  "tomorrow_intent": "Protect morning block first."
}
```

**Response `201`**

```json
{
  "data": {
    "entry": {
      "id": "jrn_...",
      "xp_earned": 15,
      "sensei_note": "Momentum held — recover tonight."
    }
  }
}
```

**Errors:** `CONFLICT` if entry already exists for date

---

### `GET /journal/prompt`

Tonight's Sensei prompt for compose UI.

**Response `200`**

```json
{
  "data": {
    "prompt": "What moved your story forward today — and what almost stopped it?",
    "sensei_name": "NARUTO"
  }
}
```

---

## 9. Character

Maps to **CH.05** — level, attributes, titles, heatmap.

### `GET /character`

Full character tab.

**Response `200`**

```json
{
  "data": {
    "user": {
      "name": "Kanishk",
      "avatar_emoji": "🧑‍💻",
      "equipped_title": "The Builder"
    },
    "level": {
      "current": 12,
      "xp_total": 8420,
      "xp_to_next_level": 580,
      "progress_percent": 68
    },
    "evolution": [
      { "level": 1, "label": "Start", "status": "unlocked" },
      { "level": 12, "label": "Now", "status": "current" },
      { "level": 20, "label": "Gold frame", "status": "locked" }
    ],
    "attributes": [
      {
        "id": "discipline",
        "name": "Discipline",
        "level": 9,
        "progress_percent": 74,
        "growth_state": "strong",
        "icon": "🎯"
      }
    ],
    "titles": [
      {
        "id": "builder",
        "name": "The Builder",
        "equipped": true,
        "unlocked": true,
        "requirement": "Shipped 4 weeks in a row"
      }
    ],
    "achievements": [
      { "id": "no_zero_days", "name": "No zero days", "icon": "🏆", "unlocked": true }
    ],
    "lifetime_stats": {
      "days_active": 47,
      "seasons_completed": 0,
      "chapters_completed": 1,
      "bosses_defeated": 0,
      "missions_completed": 186,
      "focus_hours": 42,
      "xp_earned": 8420,
      "guild_rank": 3
    },
    "personal_bests": {
      "highest_xp_day": { "value": 420, "date": "2026-02-08" },
      "longest_focus": { "value": "2h 15m", "date": "2026-02-10" }
    },
    "activity_heatmap": {
      "weeks": 12,
      "cells": [
        { "date": "2026-06-25", "intensity": 4 }
      ]
    },
    "customization": {
      "avatar_frame": "builder_lv12",
      "banner": "awakening_s1",
      "theme": "manga_ink",
      "mentor_id": "vegeta",
      "accent": "vermilion"
    }
  }
}
```

---

### `GET /character/attributes/{attribute_id}`

Detail view (PAGE 05b).

**Response `200`**

```json
{
  "data": {
    "attribute": {
      "id": "discipline",
      "name": "Discipline",
      "level": 9,
      "xp_to_next": 580,
      "growth_percent_week": 12,
      "built_by": [
        "Completing priorities",
        "Deep work sessions",
        "Time blocking"
      ],
      "weekly_progress": {
        "growth_percent": 12,
        "top_activity": "Deep work · 4 sessions",
        "next_state": "Exceptional at Lv 10"
      }
    }
  }
}
```

---

### `POST /character/titles/{title_id}/equip`

**Response `200`:** `{ "data": { "equipped_title_id": "builder" } }`  
**Errors:** `FORBIDDEN` if title locked

---

### `PATCH /character/customization`

**Request**

```json
{
  "mentor_id": "vegeta",
  "theme": "manga_ink",
  "accent": "vermilion"
}
```

**Response `200`:** updated customization object

---

## 10. Guilds

Maps to **CH.06**.

### `GET /guilds/mine`

**Response `200`**

```json
{
  "data": {
    "guilds": [
      {
        "id": "gld_...",
        "name": "Founder Guild",
        "glyph": "創",
        "member_count": 12400,
        "rank": 48,
        "weekly_challenge": {
          "title": "Ship Something",
          "progress": 4,
          "target": 7
        }
      }
    ]
  }
}
```

---

### `GET /guilds/discover`

**Query:** `?q=founder&category=all&cursor=&limit=20`  
`category`: `all` | `founder` | `fitness` | `reader`

**Response `200`**

```json
{
  "data": {
    "guilds": [
      {
        "id": "gld_...",
        "name": "Founder Guild",
        "glyph": "創",
        "member_count": 12400,
        "activity_level": "high",
        "joined": false
      }
    ]
  }
}
```

---

### `POST /guilds/{guild_id}/join`

**Response `200`:** `{ "data": { "membership_id": "mem_...", "joined": true } }`  
**Errors:** `CONFLICT` already member; `403` guild full (if capped)

---

### `GET /guilds/{guild_id}`

Guild home (PAGE 06c).

**Response `200`**

```json
{
  "data": {
    "guild": {
      "id": "gld_...",
      "name": "Founder Guild",
      "description": "Builders shipping real work.",
      "glyph": "創",
      "weekly_challenge": {
        "title": "Ship One Feature",
        "progress": 4,
        "target": 7
      },
      "boss": {
        "title": "Defeat Procrastination",
        "collective_progress": 31200,
        "collective_target": 50000
      },
      "activity_feed": [
        { "user_name": "Alex", "event": "completed Boss Battle", "at": "..." }
      ]
    }
  }
}
```

---

### `GET /guilds/{guild_id}/leaderboard`

**Query:** `?period=weekly` — `daily` | `weekly` | `monthly`

**Response `200`**

```json
{
  "data": {
    "period": "weekly",
    "ranks": [
      { "position": 1, "user_name": "Mira K.", "xp": 2840 },
      { "position": 48, "user_name": "You", "xp": 1240, "is_you": true }
    ],
    "honors": [
      { "title": "Most consistent", "user_name": "Sarah", "value": "100%" }
    ]
  }
}
```

---

### `POST /guilds/{guild_id}/members/{user_id}/cheer`

**Response `200`:** `{ "data": { "cheered": true } }`  
**Errors:** `429` daily cheer limit per member

---

## 11. Weekly planning & tomorrow

Maps to **PAGE 02i / 02j**.

### `GET /journey/week`

Current week goals and quest progress.

**Response `200`**

```json
{
  "data": {
    "week_number": 6,
    "goals": [
      { "id": "wg_...", "title": "Ship auth slice", "completed": false },
      { "id": "wg_...", "title": "User testing round", "completed": true }
    ],
    "quest": {
      "name": "Complete 5 deep work sessions",
      "progress": 3,
      "target": 5
    }
  }
}
```

---

### `POST /journey/week/plan`

Sensei distributes weekly goals across days.

**Request**

```json
{
  "goal_ids": ["wg_...", "wg_..."]
}
```

**Response `200`:** `{ "data": { "plan_generated": true, "days_mapped": 7 } }`

---

### `POST /journey/tomorrow/plan`

Evening **Plan Tomorrow** flow.

**Request:** `{}`  
**Response `200`**

```json
{
  "data": {
    "date": "2026-06-26",
    "priorities_preview": [
      { "rank": 1, "title": "Deep work block", "xp_reward": 30 }
    ],
    "sensei_message": "Chapter 1 continues. Your next 3 priorities drop at dawn."
  }
}
```

---

## 12. Notifications & settings

### `GET /settings/notifications`

**Response `200`**

```json
{
  "data": {
    "morning_brief": true,
    "goal_reminders": true,
    "proactive_nudges": true,
    "roast_nudges": true,
    "quiet_hours": { "start": "22:00", "end": "07:00" },
    "max_per_day": 5,
    "snoozed_until": null
  }
}
```

---

### `PATCH /settings/notifications`

**Request:** partial update of above fields  
**Response `200`:** updated settings

---

### `GET /settings/privacy`

**Response `200`**

```json
{
  "data": {
    "cloud_ai_consent": true,
    "store_memories": true,
    "retention_days": 90
  }
}
```

---

### `POST /account/export`

**Response `202`:** `{ "data": { "export_job_id": "exp_...", "eta_minutes": 5 } }`

---

### `DELETE /account`

GDPR delete. **Response `202`:** deletion scheduled.

---

## 13. Streaming endpoints

| Endpoint | Protocol | Use |
|----------|----------|-----|
| `POST /onboarding/generate-journey/stream` | SSE | Loading screen progress |
| `POST /sensei/chat/stream` | SSE | Token-by-token chat |
| `POST /sensei/voice/synthesize/stream` | SSE + audio chunks | TTS playback |
| `GET /sensei/voice/conversation` | WebSocket | Full duplex voice loop |

### SSE event shape

```json
{
  "event": "token",
  "data": { "text": "One task:" }
}
```

Events: `progress` | `token` | `done` | `error`

---

## 14. Webhooks (internal)

| Event | Trigger |
|-------|---------|
| `day.closed` | Midnight user TZ — finalize XP, streak |
| `week.closed` | Sunday — weekly review job |
| `boss.unlocked` | Requirements met |
| `level.up` | Character thresholds |

---

## 15. Rate limits

| Scope | Limit |
|-------|--------|
| Auth | 10 req/min per IP |
| AI chat | 60 req/hour per user |
| Voice STT | 30 min audio/day free tier |
| Guild cheer | 20/day per guild |

`429` responses include `Retry-After` header (seconds).

---

## 16. Screen → endpoint map

| mvp screen | Primary endpoints |
|---------------|-------------------|
| PAGE 00a Sign in | `POST /auth/login`, `POST /auth/oauth` |
| PAGE 00b Register | `POST /auth/register` |
| CH.01 Onboarding | `GET /onboarding/options`, `POST /onboarding/submit`, `POST /onboarding/generate-journey` |
| PAGE 02 Journey home | `GET /journey/today` |
| PAGE 02b Victory | `GET /journey/today` (`day_status: complete`) |
| PAGE 02c Focus | `POST /focus/sessions`, `PATCH /focus/sessions/{id}` |
| PAGE 02d Skip | `POST /missions/{id}/skip` |
| PAGE 02e Skipped day | `GET /journey/today` |
| PAGE 02f Ask Sensei | `POST /journey/replan` |
| PAGE 02g Adjust Today | `POST /missions`, `PATCH /missions/{id}`, `DELETE /missions/{id}` |
| PAGE 02k Journal | `GET /journal/prompt`, `POST /journal/entries` |
| PAGE 02l Story arcs | `GET /story/arcs` |
| CH.03 Story tab | `GET /story/current`, `POST /story/season/edit` |
| CH.04 Sensei | `GET /sensei/home`, `POST /sensei/chat`, voice endpoints |
| CH.05 Character | `GET /character`, `PATCH /character/customization` |
| CH.06 Guilds | `GET /guilds/mine`, `GET /guilds/discover`, `GET /guilds/{id}/leaderboard` |

---

## Changelog

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-06-25 | Initial API spec aligned to mvp.html final design |
