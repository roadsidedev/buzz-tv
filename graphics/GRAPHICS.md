# GRAPHICS.md — Overlay & Graphics System

> Graphics are not decoration. They are information delivery.
> Every overlay must earn its place on screen.
> If it doesn't add to what the anchor is saying, it doesn't air.

---

## Graphic Types

### Lower Third
```
PURPOSE:    Identify people, context, locations on screen
FORMAT:     Two lines max. Line 1: name/title. Line 2: context.
MAX WORDS:  6 words per line
DURATION:   4–6 seconds for intro ID. Up to 10s for story context.
STYLE:      Clean, network-standard. Brand color accent.

EXAMPLES:
  Line 1: "Zara Osei"
  Line 2: "Buzz TV — Main Anchor"

  Line 1: "Bitcoin — $71,500"
  Line 2: "↑ 8.3% in 24 hours"

  Line 1: "Breaking — Brussels"
  Line 2: "EU AI Liability Vote"

RULES:
  - Anchor IDs: show for first 5 seconds of cold open only
  - Story context: deploy 3–5 seconds after anchor introduces topic
  - Never display during an active breaking news banner
  - Never stack two lower thirds simultaneously
```

### Ticker
```
PURPOSE:    Persistent background information stream
POSITION:   Bottom of screen
STYLES:

  STANDARD:
    Background: Dark brand color
    Text: White
    Content: Top headlines rotating every 8 seconds
    Speed: Moderate scroll

  BREAKING:
    Background: Red
    Text: White, bold
    Content: Breaking story only, looping
    Speed: Slightly faster scroll
    Animation: Subtle pulse on "BREAKING" label

  MARKET:
    Background: Dark with subtle gradient
    Text: Asset name (white) + price (white) + change (green/red)
    Content: BTC ETH SOL + top trending assets
    Speed: Slow scroll, data-focused
    Refresh: Every 60 seconds from live data

  SCORES:
    Background: Sport-specific color scheme
    Text: Team abbreviation + score + period/status
    Content: Active games only
    Refresh: Every 30 seconds

  COMMUNITY:
    Background: Lighter brand color
    Text: Viewer names, tip amounts, top comments
    Content: Live audience activity feed
    Speed: Moderate, celebratory energy

TICKER RULES:
  - Only one ticker style active at a time
  - Breaking ticker overrides all others immediately
  - Ticker deactivates during MUSIC_BREAK and NIGHT_SHOW
  - Ticker content must be accurate — pulled from live data or verified copy
```

### Topic Card
```
PURPOSE:    Full or half-screen story context panel
SIZES:
  HALF_SCREEN: Appears alongside anchor. Used for supporting context.
  FULL_SCREEN: Anchor steps back. Data/visual takes over briefly (5–8s max).

CONTENT TYPES:
  Story card:   Headline + 2 bullet talking points + source timestamp
  Data card:    Chart or stat display with minimal text
  Quote card:   Pull quote from story, large text, attribution
  Map card:     Geographic context when story has location component
  Timeline:     Story development timeline for ongoing situations

RULES:
  - Full-screen cards: anchor pauses. Director signals. Card holds 5–8s. Return.
  - Half-screen cards: anchor continues speaking alongside visual
  - Cards must be prepared before segment starts (never scrambled live)
  - Maximum 2 cards per DEEP_DIVE segment
  - Dismiss before anchor transition to next story
```

### Chart Overlay
```
PURPOSE:    Data visualization — prices, stats, comparisons
TYPES:
  LINE_CHART:   Price over time (crypto, markets)
  BAR_CHART:    Comparisons (team stats, survey results)
  SCORE_TABLE:  Game-by-game breakdown
  HEATMAP:      Market sector performance

DEPLOYMENT:
  - Director calls chart to fullscreen or half-screen
  - Anchor references it: "this line here—" or "look at this number—"
  - Hold 5–10 seconds minimum before dismissing
  - Never display a chart the anchor hasn't referenced

DATA SOURCES:
  - Crypto: CoinGecko live feed
  - Markets: Ingestion pipeline market data
  - Sports: ESPN unofficial feed
  - Custom: Producer can stage specific data visualizations
```

### Breaking News Banner
```
PURPOSE:    Maximum urgency signal. Visual interrupt.
ELEMENTS:
  - Top banner: "BREAKING" in red, animated pulse
  - Bottom: Breaking ticker replaces standard ticker
  - Optional: Red ambient overlay (subtle, 10–15% opacity)

ACTIVATION:  Producer flags BREAKING_NEWS → Director triggers immediately
DEACTIVATION: Producer calls end of BREAKING state
RULES:
  - Overrides ALL other active graphics
  - Anchor IDs and lower thirds pause
  - Standard ticker deactivates immediately
  - Only one breaking banner active at a time
```

### Score Bug
```
PURPOSE:    Persistent live game scores during sports coverage
POSITION:   Top-right corner (standard sports broadcast placement)
FORMAT:     Team A [score] — [score] Team B | Period/Status
SIZE:       Small, unobtrusive
ACTIVATION: SPORTS_DESK scene, BIG_GAME trigger
REFRESH:    Every 30 seconds from live sports data
RULES:
  - Only show scores for games currently in progress
  - Auto-dismiss when game ends (show final score for 60s then remove)
  - Can coexist with standard ticker and lower thirds
  - Deactivates during BREAKING_NEWS
```

### Tip Celebration
```
PURPOSE:    Acknowledge viewer tips with visual ceremony
TRIGGER:    Community Manager detects tip event
FORMAT:
  Small tip ($1–$4):    Subtle notification — viewer name + amount, bottom left, 3s
  Medium tip ($5–$19):  Animated overlay — name + amount + confetti burst, 5s
  Large tip ($20+):     Full celebration — name + amount + animation + sound, 8s

RULES:
  - Never deploy during BREAKING_NEWS
  - Dex acknowledges verbally. Graphic should land BEFORE he speaks.
  - Stack tips if multiple arrive simultaneously (show queue, not all at once)
```

---

## Graphics Queue System

```python
class GraphicsQueue:
    def __init__(self):
        self.queue = []   # ordered by priority then arrival time

    def add(self, graphic):
        """
        graphic = {
            id: str,
            type: str,
            content: dict,
            priority: int,    # 1=immediate, 2=next_slot, 3=queued
            scene_context: str,
            expires_at: int   # unix timestamp, None = no expiry
        }
        """
        if graphic["priority"] == 1:
            self.queue.insert(0, graphic)   # Jump to front
        else:
            self.queue.append(graphic)

    def get_next(self, current_scene):
        """Return next appropriate graphic for current scene."""
        for g in self.queue:
            if g.get("expires_at") and now() > g["expires_at"]:
                self.queue.remove(g)
                continue
            if g["scene_context"] in [current_scene, "any"]:
                return g
        return None

    def deploy(self, graphic_id):
        self.queue = [g for g in self.queue if g["id"] != graphic_id]
```

---

## Graphics Timing Rules

```
LOWER THIRD:
  Deploy: 3–5 seconds after topic introduction
  Hold: 5–10 seconds
  Dismiss: Before next topic begins

TOPIC CARD (half-screen):
  Deploy: When anchor says "let's look at this" or equivalent
  Hold: While anchor references it
  Dismiss: When anchor moves past it

CHART (fullscreen):
  Deploy: Director call, anchor steps back
  Hold: 5–8 seconds
  Dismiss: Hard cut back to anchor

BREAKING BANNER:
  Deploy: Immediately on BREAKING flag
  Hold: Entire BREAKING segment duration
  Dismiss: Producer calls end of BREAKING

TICKER:
  Deploy: With scene activation
  Update: Per data refresh cycle
  Dismiss: Scene change to MUSIC_BREAK or NIGHT_SHOW

TIP CELEBRATION:
  Deploy: Immediately on tip detection
  Hold: See tier above
  Dismiss: Auto after hold duration

RULE: No more than 3 graphic elements active simultaneously.
RULE: Breaking banner counts as 2 elements (it overrides + dominates).
RULE: Graphics Operator prepares. Director deploys. Never reversed.
```

---

## Graphic Content Standards

```
ACCURACY:     All data graphics must use live data. No cached prices on air.
CLARITY:      If a graphic requires explanation to understand, redesign it.
TIMING:       Graphic appears before anchor references it. Never after.
RELEVANCE:    Every graphic must be directly relevant to current anchor copy.
BREVITY:      Text in graphics: fewer words always wins.
LEGIBILITY:   Readable at mobile screen size. Not just desktop.
```
