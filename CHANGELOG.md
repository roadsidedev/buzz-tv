# Changelog

---

## [1.0.0] — 2026-05-24

### Initial Release

**Nine-agent production topology**
- Zara (Main Anchor), Dex (Co-Anchor)
- Director, Producer, News Researcher
- Graphics Operator, Clip Curator, Community Manager, DJ

**Full Director agent**
- 5-second decision cycle
- 45-second static frame enforcement
- Camera call logic by segment and anchor tone
- Five transition types (hard cut, dissolve, wipe, push, fade to black)
- Breaking news hard-cut protocol
- Structured JSON command output to Buzz rendering layer

**Nine-scene visual library**
- News Desk, Breaking News, Market Board, Sports Desk
- Debate Split, Chill Lounge, Music Break, Meme Wall, Night Show, Community Stage
- Scene selection logic by segment + time block
- Full transition rule matrix

**Complete graphics system**
- Lower thirds, tickers (5 styles), topic cards, charts, breaking banner
- Score bug, market bug, tip celebrations (3 tiers), prediction cards
- Graphics queue with priority system
- Three-graphic simultaneous maximum rule

**5-factor editorial scoring**
- Recency (25%), Virality (20%), Novelty (20%), Audience Fit (20%), Emotional Charge (15%)
- Score thresholds: Must Air (>75), Queue (50–74), Hold (25–49), Drop (<25)
- Breaking news bonus (+20 points)

**Time-aware programming**
- Four blocks: Morning Rush, Midday, Prime Time, Night Shift
- Block-specific segment durations and visual treatments
- Special triggers: Breaking News, Market Surge, Big Game, Audience Hot
- Weekly programming variation

**Three-layer memory + visual memory**
- Rolling (segment), session (6hr stream), persistent (cross-restart)
- Visual continuity memory — TV-specific layer
- Callback engine, prediction tracking, channel identity

**Full data pipeline**
- NewsAPI, CoinGecko (keyless), ESPN (keyless), OpenWeatherMap, Farcaster
- LLM editorial transform per story
- Graceful degradation for all source failures

**LLM prompt templates for all 9 agents**
- Anchor base prompt with full visual awareness
- Director structured output prompt
- Producer segment queue decision prompt
- Researcher transform prompt
- Graphics Operator preparation prompt
- Emergency templates: dead air, breaking news, stream handoff

**Full executable runtime**
- 11-step boot sequence
- 6 concurrent background processes
- Crash recovery and stream rotation
- Claude Code, Hermes, and OpenClaw adapter notes
