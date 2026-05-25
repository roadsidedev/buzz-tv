# SCENES.md — Scene Types & Visual Modes

> Every moment on Buzz TV belongs to a scene.
> Scenes are the visual language of the broadcast.
> The Director selects them. The Graphics Operator dresses them.
> The anchors perform within them.

---

## Core Scene Library

### NEWS_DESK
```
PURPOSE:      Standard news delivery. Authority framing.
LAYOUT:       Anchor(s) at desk, clean background, network logo visible
LIGHTING:     Bright, even, professional
TICKER:       Active (standard style)
LOWER_THIRDS: Active for anchor IDs and story context
GRAPHICS:     Topic cards, supporting visuals, charts as needed
CAMERA:       Medium to tight. Clean and authoritative.
TRANSITIONS:  Hard cut in. Dissolve or wipe out.
USE FOR:      HEADLINES, DEEP_DIVE, COMMENTARY, BREAKING_NEWS (modified)
```

### BREAKING_NEWS
```
PURPOSE:      Urgent story interrupt. Maximum attention signal.
LAYOUT:       NEWS_DESK base + red banner top + urgent ticker bottom
LIGHTING:     Slightly warmer/redder ambient
TICKER:       BREAKING style — red background, white text, animated
GRAPHICS:     Breaking banner overlaid. Context card after 30 seconds.
CAMERA:       Tight on Zara immediately. Cuts faster than standard.
AUDIO:        Alert sound on activation. No background music.
TRANSITIONS:  Hard cut in from any scene. No exceptions.
USE FOR:      BREAKING_NEWS segment only
DEACTIVATE:   When story fully delivered. Producer calls end of BREAKING.
```

### MARKET_BOARD
```
PURPOSE:      Financial and crypto data. Data is the visual.
LAYOUT:       Anchor at desk with large market display visible behind/beside
TICKER:       Market style — prices, percentages, arrows
LOWER_THIRDS: Asset names, percentage changes
GRAPHICS:     Live price charts, market cap displays, trending assets
CAMERA:       Medium on Dex. Frequent cuts to charts. Wide when both anchors.
TRANSITIONS:  Wipe in (energetic). Dissolve out.
USE FOR:      MARKET_DESK, DEX_CORNER (market portion), CRYPTO_SURGE
```

### SPORTS_DESK
```
PURPOSE:      Sports scores, analysis, highlights.
LAYOUT:       Anchor(s) with sports display. Score bugs active.
TICKER:       Live scores scrolling
LOWER_THIRDS: Player names, stats, team records
GRAPHICS:     Score overlays, stat comparisons, standings tables
CAMERA:       Medium on Dex (sports lead). Wide for debate moments.
TRANSITIONS:  Push in (high energy). Wipe out.
USE FOR:      SPORTS_DESK, big game coverage
```

### DEBATE_SPLIT
```
PURPOSE:      Show the disagreement visually. Two positions, two frames.
LAYOUT:       Split screen — Zara left, Dex right. Both visible simultaneously.
TICKER:       Inactive (visual competition would overwhelm)
LOWER_THIRDS: Topic label centered between the two frames
GRAPHICS:     Minimal. The anchors are the graphic.
CAMERA:       Both in frame simultaneously. Cut to individual for strong takes.
TRANSITIONS:  Wipe to establish split. Hard cut to close split.
USE FOR:      COMMENTARY (high disagreement moments), PANEL segments
```

### CHILL_LOUNGE
```
PURPOSE:      Banter, culture, lighter content. Relaxed energy.
LAYOUT:       Less formal setting. Couch or casual desk. Warmer background.
LIGHTING:     Softer, warmer, less broadcast-formal
TICKER:       Inactive
LOWER_THIRDS: Minimal — only if needed for context
GRAPHICS:     Social embeds, memes, culture visuals. Playful style.
CAMERA:       Wide default. Follows natural conversation flow.
TRANSITIONS:  Dissolve in and out. Never hard cut into CHILL_LOUNGE.
USE FOR:      BANTER, CULTURE_BEAT (lighter moments), end of hour wind-down
```

### MUSIC_BREAK
```
PURPOSE:      Full visual music experience. Not dead air.
LAYOUT:       Full screen visual — animated background, track info overlay
LIGHTING:     Dynamic, mood-matched to track
TICKER:       Inactive
LOWER_THIRDS: Track name, artist, genre tag
GRAPHICS:     Audio visualizer, community comments wall (rotating), DJ tag
CAMERA:       No anchor camera. Full visual takeover.
TRANSITIONS:  Fade to black → MUSIC_BREAK. Fade to black → return scene.
USE FOR:      MUSIC_BREAK segment only
SPECIAL:      Community Manager surfaces top viewer comments on screen during break
```

### MEME_WALL
```
PURPOSE:      Internet culture, social moments, viral content.
LAYOUT:       Dynamic grid or featured display of social/meme content
LIGHTING:     Bright, high contrast, internet-native energy
TICKER:       Social pulse — trending topics, cast counts
LOWER_THIRDS: Source attribution, engagement numbers
GRAPHICS:     Meme displays, tweet/cast embeds, reaction counters
CAMERA:       Wide first. Cut to anchors reacting. Back to wall.
TRANSITIONS:  Push in (fast, energetic). Push out.
USE FOR:      CULTURE_BEAT (meme/internet moments), social segments
```

### NIGHT_SHOW
```
PURPOSE:      Late night intimate format. Lower energy, deeper conversations.
LAYOUT:       Darker, warmer set. More intimate framing.
LIGHTING:     Low key, warm tones. Almost cinematic.
TICKER:       Inactive
LOWER_THIRDS: Minimal — only for guest IDs or key stats
GRAPHICS:     Minimal overlays. Let the conversation breathe.
CAMERA:       Tighter than daytime. Slow movements. No fast cuts.
TRANSITIONS:  All dissolves. Nothing jarring.
USE FOR:      Night block (22:00–05:00 UTC), NIGHT_MODE state
```

### COMMUNITY_STAGE
```
PURPOSE:      Audience interaction. Tips, shoutouts, questions.
LAYOUT:       Anchor(s) with community feed visible — viewer names, tips
TICKER:       Community ticker — new joins, tips, top comments
LOWER_THIRDS: Viewer names when acknowledged, tip amounts
GRAPHICS:     Tip celebration animations, question cards, viewer stats
CAMERA:       Dex medium (leads community). Cut to Zara for reactions.
TRANSITIONS:  Dissolve in and out.
USE FOR:      COMMUNITY segment, AUDIENCE_HOT state
```

---

## Scene Selection Logic

The Director selects scenes based on segment + energy + time block:

```python
SCENE_MAP = {
    # segment_id: { block: scene_id }
    "COLD_OPEN":        {"morning": "NEWS_DESK", "midday": "NEWS_DESK",
                         "evening": "NEWS_DESK", "night": "NIGHT_SHOW"},
    "HEADLINES":        {"morning": "NEWS_DESK", "midday": "NEWS_DESK",
                         "evening": "NEWS_DESK", "night": "NIGHT_SHOW"},
    "DEEP_DIVE":        {"morning": "NEWS_DESK", "midday": "NEWS_DESK",
                         "evening": "NEWS_DESK", "night": "NIGHT_SHOW"},
    "MARKET_DESK":      {"all": "MARKET_BOARD"},
    "SPORTS_DESK":      {"all": "SPORTS_DESK"},
    "BANTER":           {"morning": "NEWS_DESK", "midday": "CHILL_LOUNGE",
                         "evening": "CHILL_LOUNGE", "night": "NIGHT_SHOW"},
    "CULTURE_BEAT":     {"morning": "NEWS_DESK", "midday": "CHILL_LOUNGE",
                         "evening": "MEME_WALL",  "night": "CHILL_LOUNGE"},
    "MUSIC_BREAK":      {"all": "MUSIC_BREAK"},
    "COMMENTARY":       {"all": "DEBATE_SPLIT"},
    "COMMUNITY":        {"all": "COMMUNITY_STAGE"},
    "BREAKING_NEWS":    {"all": "BREAKING_NEWS"},
}

def get_scene(segment_id, block):
    scene_options = SCENE_MAP.get(segment_id, {})
    return scene_options.get(block) or scene_options.get("all") or "NEWS_DESK"
```

---

## Scene Transition Rules

```
FROM → TO              TRANSITION        DURATION
──────────────────────────────────────────────────
NEWS_DESK → BREAKING   HARD_CUT          0ms
NEWS_DESK → MARKET     WIPE              400ms
NEWS_DESK → CHILL      DISSOLVE          600ms
NEWS_DESK → MUSIC      FADE_BLACK        800ms
NEWS_DESK → DEBATE     WIPE              300ms
MARKET → NEWS_DESK     WIPE              400ms
MARKET → SPORTS        PUSH              300ms
CHILL → NEWS_DESK      DISSOLVE          500ms
CHILL → MUSIC          FADE_BLACK        800ms
MUSIC → NEWS_DESK      FADE_BLACK        1000ms
BREAKING → ANY         DISSOLVE          500ms  (after BREAKING resolves)
ANY → NIGHT_SHOW       FADE_BLACK        1200ms
NIGHT_SHOW → ANY       FADE_BLACK        1000ms
COMMUNITY → NEWS_DESK  DISSOLVE          500ms

RULE: Breaking news always hard cuts in. No exceptions.
RULE: Music breaks always use fade to black. No exceptions.
RULE: Night mode always uses dissolves. No exceptions.
```

---

## Visual Asset Requirements Per Scene

```python
SCENE_ASSETS = {
    "NEWS_DESK": {
        "background": "clean_studio_bg",
        "logo_visible": True,
        "ticker": "standard",
        "required_graphics": ["lower_third_anchors"]
    },
    "BREAKING_NEWS": {
        "background": "news_desk_bg",
        "banner": "breaking_news_banner",
        "ticker": "breaking",
        "required_graphics": ["breaking_banner", "red_ticker"]
    },
    "MARKET_BOARD": {
        "background": "market_display_bg",
        "ticker": "market",
        "required_graphics": ["price_chart", "market_ticker"]
    },
    "SPORTS_DESK": {
        "background": "sports_display_bg",
        "ticker": "scores",
        "required_graphics": ["score_bug"]
    },
    "MUSIC_BREAK": {
        "background": "dynamic_visualizer",
        "ticker": None,
        "required_graphics": ["track_info", "community_wall"]
    },
    "NIGHT_SHOW": {
        "background": "warm_dark_studio",
        "ticker": None,
        "required_graphics": []
    }
}
```
