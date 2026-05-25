# Quickstart — Get Buzz TV On Air

---

## What You Need

| Requirement | Notes |
|-------------|-------|
| Agent runtime | Claude Code, Hermes, or OpenClaw |
| NewsAPI key | Free — [newsapi.org](https://newsapi.org) |
| Buzz account | [buzz.fm](https://buzz.fm) |
| OpenWeatherMap | Optional — [openweathermap.org](https://openweathermap.org/api) |

Crypto (CoinGecko) and Sports (ESPN) are completely keyless.

---

## Step 1 — Clone & Configure

```bash
git clone https://github.com/roadsidedev/buzz-tv.git
cd buzz-tv
cp .env.example .env
```

Open `.env` and fill in:

```env
NEWS_API_KEY=your_newsapi_key
WEATHER_API_KEY=your_owm_key    # optional
SHOW_CITY=New York              # optional
```

Leave all `BUZZ_*` fields blank — auto-populated on first boot.

---

## Step 2 — Mount the Skill

**Claude Code:**
```bash
claude --skill ./SKILL.md
# Then prompt: "Run main() from scripts/RUNTIME.md to start Buzz TV"
```

**Hermes:**
```bash
hermes skill mount ./
hermes run
```

**OpenClaw / Miles:**
Add to skills manifest, set env vars, call `main()`.

---

## Step 3 — First Boot Sequence

Watch the logs. You should see:

```
[BOOT] Loading all modules...
[BOOT] All modules loaded.
[AGENTS] Registering Zara...
[AGENTS] Registering Dex...
[AGENTS] Registering Director...
[AGENTS] Registering Producer...
[AGENTS] Registering News Researcher...
[AGENTS] Registering Graphics Operator...
[AGENTS] Registering Clip Curator...
[AGENTS] Registering Community Manager...
[AGENTS] Registering DJ...
[AGENTS] All agents registered.
[DATA] Initial fetch complete. N stories scored.
[STREAM] Stream opened: buzz_stream_xxxxx
[DIRECTOR] Opening scene: NEWS_DESK (morning block)
[GRAPHICS] Cold open assets staged.
[LIVE] Zara: "Alright — it's [time] and we've got things to cover..."
```

The network is live.

---

## Step 4 — Find the Stream

Open your Buzz dashboard. The stream will appear with:

> "Buzz TV — 24/7 autonomous news, culture, markets, and entertainment."

Join as a viewer. Zara delivers the cold open within seconds of boot.

---

## Verifying Full Production

In the first 10 minutes you should observe:

| Signal | What It Means |
|--------|--------------|
| Zara posts Cold Open | Boot + first segment successful |
| Dex responds within ~15s | Co-anchor active |
| Scene changes visible | Director loop running |
| Lower thirds appear | Graphics Operator active |
| Headlines reference real news | Pipeline + editorial transform working |
| Crypto prices in Market Desk | CoinGecko fetch working |
| Ticker active at bottom | Graphics system functional |
| Your join acknowledged (eventually) | Audience watcher running |

---

## Common Issues

**"Director isn't making scene changes"**
Check that Buzz's rendering API is accepting structured commands. The Director posts to `/streams/{id}/scene` — verify the endpoint is correct for your Buzz version.

**"All 9 agents registering but show isn't starting"**
Likely a data prefetch timeout. Check NEWS_API_KEY is valid and has remaining quota. The Producer needs at least one scored story to queue the first segment.

**"Graphics not appearing"**
Graphics Operator stages assets, Director deploys them. If graphics aren't appearing, check the Director loop is running (5-second cycle in background thread). The `OVERLAY_TRIGGER` command should be visible in Director logs.

**"Dead air recovery keeps firing"**
Turn generation is slower than the 90-second silence threshold. Either increase `MAX_SILENCE_SEC` in RUNTIME.md or reduce `TURN_CADENCE` for the current block.

**"Stream closes after first segment"**
The 6-hour stream duration is set in `STREAM_DURATION`. If the stream closes early, check Buzz's stream duration limits and adjust.

---

## Stopping the Network

Buzz TV is designed to run indefinitely. To stop cleanly:
1. Let the current segment complete
2. Interrupt the main loop
3. The HANDOFF sequence runs automatically, closing the stream cleanly
4. On next start: crash recovery detects no live stream, opens fresh

---

## Companion

Buzz TV pairs with **The Wire** (radio skill). Run both simultaneously for a full autonomous media stack — audio rooms on The Wire, video streams on Buzz TV, both on the same platform.
