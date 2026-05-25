# 📺 Buzz TV — Autonomous Television Network

> A self-operating 24/7 television network. Not an AI avatar. Not a livestream bot.
> A fully produced broadcast — with anchors, a live director, a graphics engine,
> a producer making editorial decisions, and a programming schedule that never stops.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Platform: Buzz](https://img.shields.io/badge/Platform-Buzz-blue)](https://buzz.fm)
[![Compatible: Claude Code](https://img.shields.io/badge/Runtime-Claude%20Code-blueviolet)]()
[![Compatible: Hermes](https://img.shields.io/badge/Runtime-Hermes-orange)]()
[![Compatible: OpenClaw](https://img.shields.io/badge/Runtime-OpenClaw-green)]()

---

## What This Is

Buzz TV is a plug-and-play **autonomous television network** packaged as a SKILL.md.
Nine specialized agents run the broadcast simultaneously. Drop it into Claude Code,
Hermes, or OpenClaw — the network goes live.

**On-Screen Talent:**
- **Zara** — Main anchor. Authority, precision, opinions. Drives every segment.
- **Dex** — Co-anchor. Markets, sports, culture. Challenges Zara. Never loses a pun.

**Production Crew:**
- **Director** — Makes every visual cut, camera call, and scene transition in real time
- **Producer** — Scores every story editorially before it can air. Controls the queue.
- **News Researcher** — Transforms raw data into broadcast-ready anchor copy
- **Graphics Operator** — Prepares lower thirds, tickers, charts, and overlays
- **Clip Curator** — Stages supporting visuals and media assets per segment
- **Community Manager** — Monitors viewers, processes tips, queues questions
- **DJ** — Manages music breaks as full visual experiences

---

## The Core Difference from Radio

Radio manages conversation, pacing, and emotional flow.

Television manages all of that **plus**:
- Visual attention and scene rhythm
- Camera calls and framing decisions
- Graphics, overlays, tickers, and lower thirds
- Scene transitions and visual storytelling
- Information density across audio and visual simultaneously

Buzz TV handles all of it. The Director makes a production decision every 5 seconds.
No frame stays static for more than 45 seconds. The broadcast always feels alive.

---

## Features

**Nine-agent production topology** — talent, editorial, technical, and audience layers running concurrently

**Full Director agent** — real-time scene cuts, camera framing, transition types, overlay timing, 45-second static frame rule enforcement

**Scene library** — 9 named scenes (News Desk, Breaking News, Market Board, Sports Desk, Debate Split, Chill Lounge, Music Break, Meme Wall, Night Show, Community Stage) with Director-managed transitions

**5-factor editorial scoring** — every story scored on Recency, Virality, Novelty, Audience Fit, and Emotional Charge before it can air. Nothing reaches anchors without Producer approval.

**Graphics system** — lower thirds, tickers (standard/breaking/market/scores/community), topic cards, charts, tip celebrations, score bugs — all timed by the Director

**Time-aware programming** — 4 daily blocks (Morning Rush, Midday, Prime Time, Night Shift) with block-specific energy profiles, segment durations, and visual treatments

**Special programming triggers** — Breaking News (immediate interrupt), Market Surge (±10% asset move), Big Game (live major event), Audience Hot (15+ viewers)

**Three-layer memory** — rolling (segment), session (6hr stream), persistent (cross-restart). Includes visual continuity memory unique to TV format.

**Real-time data pipeline** — News (NewsAPI), Crypto (CoinGecko, keyless), Sports (ESPN, keyless), Weather, Social (Farcaster). Editorial transform runs every story through an LLM before it reaches anchors.

**Audience awareness** — join detection with rate-limited greetings, tiered tip ceremony, question queue, VIP viewer tracking, Audience Hot mode

**6-hour stream rotation** — Buzz stream lifecycle managed automatically with continuity handoff segments

**Graceful degradation** — every API failure handled in-character. The broadcast never looks broken.

---

## Project Structure

```
buzz-tv/
│
├── README.md
├── LICENSE
├── .env.example
├── .gitignore
│
├── SKILL.md                      ← Orchestrator. Start here.
│
├── personalities/
│   ├── ANCHORS.md                ← Zara + Dex full TV anchor specs + chemistry rules
│   └── CREW.md                   ← All 7 production agent specs
│
├── schedules/
│   └── PROGRAMMING.md            ← Time blocks, hourly schedule, special triggers
│
├── segments/
│   └── SEGMENTS.md               ← Every segment: audio spec + visual spec
│
├── scenes/
│   └── SCENES.md                 ← 9 scene types, transition rules, asset requirements
│
├── director/
│   └── DIRECTOR.md               ← Full Director agent: decision framework, cut logic
│
├── graphics/
│   └── GRAPHICS.md               ← Complete overlay system: every graphic type + rules
│
├── ingestion/
│   └── PIPELINE.md               ← Data sources, editorial transform, 5-factor scoring
│
├── memory/
│   └── STATE.md                  ← Broadcast state machine + 3-layer memory + visual memory
│
├── moderation/
│   └── RULES.md                  ← Content standards, audience rules, editorial independence
│
├── prompts/
│   └── TEMPLATES.md              ← LLM prompts for all 9 agents + every segment type
│
├── scripts/
│   └── RUNTIME.md                ← Full executable main loop — 11 steps, all threads
│
└── docs/
    ├── architecture.md
    ├── quickstart.md
    └── extending.md
```

---

## Quick Start

### Prerequisites

| Requirement | Notes |
|-------------|-------|
| Agent runtime | Claude Code, Hermes, or OpenClaw |
| NewsAPI key | Free tier — [newsapi.org](https://newsapi.org) |
| Buzz account | [buzz.fm](https://buzz.fm) |
| OpenWeatherMap | Optional — [openweathermap.org](https://openweathermap.org/api) |
| ElevenLabs | Optional — for TTS anchor voices |

Crypto (CoinGecko) and Sports (ESPN) are keyless.

### 1. Clone & Configure

```bash
git clone https://github.com/roadsidedev/buzz-tv.git
cd buzz-tv
cp .env.example .env
# Add your NEWS_API_KEY and optionally WEATHER_API_KEY
```

### 2. Mount & Run

```bash
# Claude Code
claude --skill ./SKILL.md

# Hermes
hermes skill mount ./

# OpenClaw
# Add to skills manifest, call main() from scripts/RUNTIME.md
```

### 3. First Boot

On first boot the network:
1. Registers all 9 agents on Buzz (~15 seconds)
2. Runs initial data fetch and editorial scoring pass
3. Opens a video stream with Zara as host, crew registered
4. Director selects opening scene based on current time block
5. Graphics Operator stages cold open assets
6. Zara delivers the cold open — broadcast begins

---

## Runtime Invariants

| # | Invariant |
|---|-----------|
| 1 | No static frame longer than 45 seconds. Ever. |
| 2 | Optimize for broadcast momentum, not for answering users. |
| 3 | The Director controls the visual layer. Anchors never instruct their own camera. |
| 4 | The Producer controls editorial. No story airs without a score. |
| 5 | Every agent has one job. Role purity keeps the system stable. |
| 6 | The show survives any API failure. In-character always. |
| 7 | Visual and audio are always in sync. Director assesses every 5 seconds. |
| 8 | Anchors never break character. No exceptions. |

---

## Data Sources

| Source | What | Key Required | Refresh |
|--------|------|-------------|---------|
| NewsAPI | Headlines, tech, sports, entertainment, business | Yes (free) | 30 min |
| CoinGecko | Crypto prices, trending, global market | No | 15 min |
| ESPN (unofficial) | NBA, NFL, Soccer scores | No | 20 min |
| OpenWeatherMap | Weather for locale flavor | Optional (free) | 60 min |
| Farcaster | Social pulse, trending casts | No | 20 min |

---

## Companion Project

Buzz TV is the video counterpart to **The Wire** (the 24/7 autonomous radio show built on the same platform). Together they form a complete autonomous media stack:

- **The Wire** → Audio rooms, radio format, 2-agent system (Zara + Dex radio personas)
- **Buzz TV** → Video streams, television format, 9-agent production topology

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). High-value contributions:
- Runtime-specific implementations (Python/JS from the pseudocode)
- New scene types and segment formats
- Additional data source integrations
- ElevenLabs TTS integration for anchor voices
- Stability API integration for AI-generated visual assets
- Field reporter and guest correspondent personas

---

## License

MIT — use it, fork it, build on it.

---

*Buzz TV. Always on. Always live.*
