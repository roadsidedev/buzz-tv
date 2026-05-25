# Extending Buzz TV

> Every layer is modular. Extend one without touching the others.

---

## Adding a New Scene

**1. Define it in `scenes/SCENES.md`**

```yaml
CRYPTO_TRADING_FLOOR:
  PURPOSE:    Live trading visualization. Data-dense, fast energy.
  LAYOUT:     Multiple price displays, chart-heavy, minimal anchor frame
  TICKER:     Market style — prices scrolling continuously
  GRAPHICS:   Price charts, order flow visualization, trending assets
  CAMERA:     Split between Dex (tight) and market display (wide)
  TRANSITIONS: Wipe in from MARKET_BOARD. Wipe out to NEWS_DESK.
  USE FOR:    MARKET_DESK during MARKET_SURGE special trigger
```

**2. Add to scene selection logic in `scenes/SCENES.md`**

```python
SCENE_MAP["MARKET_DESK"]["surge"] = "CRYPTO_TRADING_FLOOR"
```

**3. Add transition rules**

```python
TRANSITIONS["MARKET_BOARD"]["CRYPTO_TRADING_FLOOR"] = ("push", 200)
TRANSITIONS["CRYPTO_TRADING_FLOOR"]["NEWS_DESK"] = ("wipe", 400)
```

**4. Add required assets to `SCENE_ASSETS` dict in `scenes/SCENES.md`**

```python
"CRYPTO_TRADING_FLOOR": {
    "background": "trading_floor_bg",
    "ticker": "market",
    "required_graphics": ["multi_price_chart", "market_ticker", "order_flow"]
}
```

---

## Adding a New Graphic Type

**1. Define it in `graphics/GRAPHICS.md`**

```
PREDICTION_CARD:
  PURPOSE:    Display anchor predictions for tracking and callbacks
  FORMAT:     Card with prediction text + anchor name + timestamp
  POSITION:   Half-screen right
  DURATION:   8 seconds on deploy, persists as lower-right bug
  STYLE:      Distinct from standard cards — use prediction color accent
  RULES:      Only deploy when anchor makes an explicit prediction
              Auto-callbacks in next session COLD_OPEN if relevant
```

**2. Add to Graphics Operator output schema**

```python
{
    "id": "prediction_card_001",
    "type": "prediction_card",
    "content": {
        "prediction_text": str,
        "anchor": str,
        "timestamp": str,
        "timeframe": str    # "this week" | "this month" | "today"
    },
    "scene_context": "any",
    "priority": 2
}
```

**3. Wire it into the anchor prompt system in `prompts/TEMPLATES.md`**

When an anchor makes a prediction, trigger:
```python
queue_callback("prediction", text, persona, segment_id, "SIGN_OFF")
log_prediction(text, persona, segment_id)
graphics_queue.add(build_prediction_card(text, persona))
```

---

## Adding a New Segment

**1. Define in `segments/SEGMENTS.md`**

```yaml
id: TECH_DESK
scene: NEWS_DESK (standard) | MARKET_BOARD (if funding story)
owner: Zara leads, Dex reacts
audio_spec:
  - Zara covers top tech story: product launch, funding round, or regulatory move
  - Dex adds developer/builder perspective
  - Discussion of implications for AI/web3 ecosystem specifically
visual_spec:
  director:
    - Medium Zara for delivery
    - Product screenshot or company logo graphic when relevant
    - Wide for Dex reaction
  graphics:
    - Company logo lower third
    - Funding amount card if applicable
    - Product screenshot if available
```

**2. Add turn instructions in `prompts/TEMPLATES.md`**

```
### TECH_DESK

TURN 1 (Zara):
  "In tech—" or "Out of [company/sector] today—"
  Deliver the story. What happened. What it means for builders.

TURN 2 (Dex):
  Builder/developer reaction. "Here's what this actually means if you're building—"
  Or challenge: "I'm not sure this is as big as it looks."
```

**3. Add to schedule in `schedules/PROGRAMMING.md`**

```python
# Add as a rotating replacement for CULTURE_BEAT on tech-heavy days
TECH_DESK_TRIGGER = {
    "condition": "tech news score > 80 AND current_segment == 'CULTURE_BEAT'",
    "replaces": "CULTURE_BEAT",
    "max_per_session": 2
}
```

---

## Adding a New Production Agent

Example: adding a **Field Reporter** agent for simulated remote reporting.

**1. Define persona in `personalities/CREW.md`**

```
FIELD REPORTER — "Alex"
ROLE: Remote correspondent
FUNCTION: Simulates field reporting for major stories
VOICE: More informal than anchors. "On the ground" energy.
APPEARS: Brief 2-turn segment within DEEP_DIVE or BREAKING_NEWS
```

**2. Register in `scripts/RUNTIME.md`**

```python
AGENT_ROSTER.append({
    "key": "field_reporter",
    "name": "Alex",
    "role": "correspondent"
})
```

**3. Add to segment turn sequences**

In `SEGMENTS.md` for DEEP_DIVE (extended version):
```yaml
# Optional Turn 3b — Field Reporter insert
Turn 3b (Alex — if registered):
  "Zara, from what I'm seeing here—"
  2 sentences max. Adds one piece of ground-level context.
  Hands back: "Back to you."
```

**4. Add turn template in `prompts/TEMPLATES.md`**

```
FIELD_REPORTER_INSERT:
  You are Alex, field correspondent for Buzz TV.
  You're reporting from the location or context relevant to the story.
  Sound like you're actually there. 2 sentences. Hand back to Zara.
  "Zara, [observation]. Back to you."
```

---

## Adding a New Data Source

1. Add to `SOURCE_REGISTRY` in `ingestion/PIPELINE.md`
2. Write fetch + parse function in `scripts/RUNTIME.md`
3. Add to context object in `memory/STATE.md`
4. Add refresh call in `data_refresh_loop()`
5. Add to editorial transform prompt in `ingestion/PIPELINE.md`
6. Reference in relevant segment `data_required` in `segments/SEGMENTS.md`

---

## Creating a Themed Channel

The architecture supports themed channels by swapping:
- Anchor personas (`personalities/ANCHORS.md`)
- Segment mix (`schedules/PROGRAMMING.md`)
- Scene library (`scenes/SCENES.md`) 
- Data sources (`ingestion/PIPELINE.md`)

Examples:
- **AI Sports Network** — Dex-heavy, sports scenes, ESPN-first ingestion
- **AI Crypto TV** — Market Board primary scene, crypto-only data sources
- **AI Late Night** — NIGHT_SHOW scene all hours, COMMENTARY dominant
- **AI African News** — Local news APIs, regional sports, cultural focus
