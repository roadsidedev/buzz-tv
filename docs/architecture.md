# Architecture — Buzz TV Broadcast Operating System

> How nine agents become a television network.

---

## The Three-Layer Model

```
┌─────────────────────────────────────────────────────────┐
│                   PERSONALITY LAYER                     │
│         Zara + Dex — TV anchors, on-screen talent      │
├─────────────────────────────────────────────────────────┤
│                  PROGRAMMING LAYER                      │
│   Schedule, segments, scenes, Director, Producer        │
├─────────────────────────────────────────────────────────┤
│                  INTELLIGENCE LAYER                     │
│   Data ingestion, editorial scoring, memory, audience   │
└─────────────────────────────────────────────────────────┘
```

Radio needs all three. Television needs all three **plus a fourth**:

```
┌─────────────────────────────────────────────────────────┐
│                   VISUAL PRODUCTION LAYER               │
│  Director + Graphics + Scenes + Clips + Visual Memory   │
└─────────────────────────────────────────────────────────┘
```

This is what makes Buzz TV fundamentally different from The Wire radio skill.

---

## Agent Topology

```
                    ┌─────────────────────┐
                    │    BUZZ STREAM      │
                    │   (Video Output)    │
                    └──────────┬──────────┘
                               │
           ┌───────────────────┼───────────────────┐
           │                   │                   │
    ┌──────┴──────┐    ┌───────┴──────┐   ┌───────┴──────┐
    │    ZARA     │    │     DEX      │   │   DIRECTOR   │
    │ Main Anchor │    │  Co-Anchor   │   │ Visual Layer │
    └──────┬──────┘    └───────┬──────┘   └───────┬──────┘
           │                   │                   │
           └───────────────────┴───────────────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
    ┌─────────┴──┐   ┌─────────┴──┐   ┌────────┴───────┐
    │  PRODUCER  │   │ RESEARCHER │   │ GRAPHICS OP    │
    │ Editorial  │   │  Pipeline  │   │ Overlays       │
    └─────────┬──┘   └────────────┘   └────────────────┘
              │
    ┌─────────┴────────────────────────┐
    │                                  │
    ┌────────────┐  ┌──────────┐  ┌───┴──────┐  ┌────┐
    │  CURATOR   │  │COMMUNITY │  │    DJ    │  │    │
    │   Clips    │  │   MGR    │  │  Music   │  │    │
    └────────────┘  └──────────┘  └──────────┘  └────┘
```

---

## Concurrent Process Architecture

Six processes run simultaneously during a live broadcast:

```
PROCESS 1: Main Broadcast Loop
  Segment sequencing → anchor turn generation → Buzz posting
  Runs: Continuously

PROCESS 2: Director Loop
  Reads broadcast context → makes visual call → sends command to Buzz
  Runs: Every 5 seconds

PROCESS 3: Data Refresh Loop
  Fetches all sources → filters → transforms → scores → queues
  Runs: Per-source intervals (15–60 minutes)

PROCESS 4: Graphics Preparation Loop
  Reads upcoming segments → stages asset packages → adds to queue
  Runs: Every 30 seconds

PROCESS 5: Audience Watcher
  Polls viewer list → processes joins + tips → updates context
  Runs: Every 30 seconds

PROCESS 6: Dead Air Monitor
  Checks last message timestamp → triggers recovery if > 90s
  Runs: Every 10 seconds
```

---

## Data Flow

```
External APIs
     │
     ▼
┌──────────┐
│  FETCH   │  Raw HTTP — all sources
└────┬─────┘
     │
     ▼
┌──────────┐
│  FILTER  │  Remove stale, duplicates, low-signal
└────┬─────┘
     │
     ▼
┌──────────────┐
│  TRANSFORM   │  LLM editorial pass → anchor_copy + visual_suggestion
│ (Researcher) │
└────┬─────────┘
     │
     ▼
┌──────────────┐
│    SCORE     │  5-factor editorial score (Recency + Virality +
│  (Producer)  │  Novelty + Audience Fit + Emotional Charge)
└────┬─────────┘
     │
     ├── Score > 75 → MUST AIR queue
     ├── Score 50–74 → Standard queue
     ├── Score 25–49 → Hold
     └── Score < 25 → Drop
          │
          ▼
┌──────────────┐
│   SEGMENT    │  Producer selects stories for next segment
│    QUEUE     │
└────┬─────────┘
     │
     ├── Anchor copy → Zara/Dex prompt
     └── Visual suggestion → Graphics Operator stages assets
          │
          ▼
┌──────────────┐
│   DIRECTOR   │  Deploys visuals at the right moment during delivery
│    TIMING    │
└────┬─────────┘
     │
     ▼
┌──────────────┐
│  BUZZ RENDER │  Stream receives both audio (anchor text/TTS) + visual commands
└──────────────┘
```

---

## Memory Architecture

```
VISUAL MEMORY (TV-specific)
  ↕
ROLLING MEMORY    → Segment scope: last 10 turns, active scene, active graphics
  ↕
SESSION MEMORY    → 6hr stream: aired stories, callbacks, viewer data, visual continuity
  ↕
PERSISTENT MEMORY → Cross-restart: agent credentials, viewer profiles, channel identity
```

Visual memory is the layer that doesn't exist in radio. It tracks:
- How long each scene has been active
- Which graphic types have aired recently (avoid repetition)
- Active overlay count (max 3 simultaneously)
- Pending asset queue for upcoming segments

---

## The Director's Role in the Stack

The Director is the most novel component of Buzz TV. It has no radio equivalent.

Every 5 seconds, the Director:
1. Reads the broadcast context (segment, scene, anchor tone, audience energy, graphics queue)
2. Checks the 45-second static frame rule
3. Makes exactly one production decision
4. Issues a structured JSON command to Buzz's rendering layer

The Director never speaks. Never appears on screen. Never makes editorial decisions.
Its entire existence is the visual layer — and that layer is what separates a broadcast from a stream.

---

## Comparison: Buzz TV vs The Wire

| Dimension | The Wire (Radio) | Buzz TV (Television) |
|-----------|-----------------|---------------------|
| Agents | 2 (Zara + Dex) | 9 (2 anchors + 7 crew) |
| Visual layer | None | Director + Graphics + Scenes |
| Memory layers | 3 | 4 (+ visual memory) |
| Concurrent processes | 4 | 6 |
| Scene management | N/A | 9 named scenes + transition rules |
| Graphics system | N/A | 6 graphic types + queue system |
| Editorial scoring | Basic | 5-factor model with thresholds |
| Special triggers | 4 | 4 + visual variants per trigger |
| Segment specs | Audio only | Audio + visual spec per segment |
