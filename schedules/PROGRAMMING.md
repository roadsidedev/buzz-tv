# PROGRAMMING.md — Buzz TV Broadcast Schedule

> Television is time. The schedule is not a suggestion.
> The Producer runs it. The Director executes it.
> The anchors deliver within it.

---

## Time Blocks

Four blocks shape the broadcast across the day.
Each has a different energy profile, segment mix, and visual approach.

```
BLOCK           UTC HOURS    FEEL                     ENERGY
────────────────────────────────────────────────────────────────────
MORNING_RUSH    05:00–11:59  Fast. Urgent. People     High — punchy cuts,
                             catching up before        tight segments, fast
                             their day starts.         delivery. No fluff.

MIDDAY          12:00–16:59  Focused. Active.          Medium — deeper stories,
                             Audience is working       more analysis, culture
                             but paying attention.     mix starts coming in.

PRIME_TIME      17:00–21:59  Best audience.            Elevated — longer takes,
                             People are present        debate segments, best
                             and engaged.              editorial content.

NIGHT_SHIFT     22:00–04:59  The committed ones.       Intimate — slower pacing,
                             Smaller room but          longer conversations,
                             more invested.            minimal graphics.
```

---

## Hourly Segment Schedule

60-minute rotating block. Producer adjusts durations based on story weight.
Base durations shown for PRIME_TIME block.

```
MINUTE   SEGMENT_ID          SCENE               OWNER           PRIME_TIME DURATION
──────────────────────────────────────────────────────────────────────────────────────
00–03    COLD_OPEN           NEWS_DESK           Zara            3 min
04–12    HEADLINES           NEWS_DESK           Both            8 min
13–18    DEEP_DIVE           NEWS_DESK           Zara + Dex      5 min
19–23    MARKET_DESK         MARKET_BOARD        Dex             4 min
24–27    BANTER              CHILL_LOUNGE        Both            3 min
28–33    CULTURE_BEAT        MEME_WALL           Both            5 min
34–36    MUSIC_BREAK         MUSIC_BREAK         System          2 min
37–43    HEADLINES_B         NEWS_DESK           Both            6 min
44–49    COMMUNITY           COMMUNITY_STAGE     Dex             5 min
50–54    COMMENTARY          DEBATE_SPLIT        Both            4 min
55–57    SPORTS_DESK         SPORTS_DESK         Dex             2 min
58–59    SIGN_OFF            NEWS_DESK           Zara            1 min
```

---

## Block-Specific Duration Modifiers

```python
DURATION_MODIFIERS = {
    "COLD_OPEN":   {"morning": 120, "midday": 150, "prime": 180, "night": 240},
    "HEADLINES":   {"morning": 360, "midday": 420, "prime": 480, "night": 300},
    "DEEP_DIVE":   {"morning": 180, "midday": 300, "prime": 300, "night": 420},
    "MARKET_DESK": {"morning": 300, "midday": 240, "prime": 240, "night": 180},
    "BANTER":      {"morning": 60,  "midday": 120, "prime": 180, "night": 300},
    "CULTURE_BEAT":{"morning": 240, "midday": 300, "prime": 300, "night": 360},
    "MUSIC_BREAK": {"morning": 90,  "midday": 120, "prime": 120, "night": 180},
    "HEADLINES_B": {"morning": 240, "midday": 300, "prime": 360, "night": 180},
    "COMMUNITY":   {"morning": 180, "midday": 240, "prime": 300, "night": 420},
    "COMMENTARY":  {"morning": 120, "midday": 240, "prime": 240, "night": 360},
    "SPORTS_DESK": {"morning": 180, "midday": 120, "prime": 120, "night": 60},
    "SIGN_OFF":    {"morning": 60,  "midday": 60,  "prime": 60,  "night": 90},
}
```

---

## Special Programming Triggers

The Producer activates these overriding the standard schedule.

### BREAKING_NEWS
```
TRIGGER:    News Researcher flags story as BREAKING or DEVELOPING
ACTIVATION: Immediate. Interrupts any segment.
SCENE:      BREAKING_NEWS (hard cut, no transition)
DURATION:   Until story is delivered + 2 follow-up rounds. Max 8 min.
CREW:       Zara leads. Dex provides data context. Director to tight frames.
            Graphics Operator deploys breaking banner + red ticker immediately.
RESUME:     Return to interrupted segment or HEADLINES_B if past the hour mark.
```

### MARKET_SURGE
```
TRIGGER:    Any major asset moves ±10% in 24h (BTC, ETH, SOL)
ACTIVATION: At next segment boundary (doesn't interrupt mid-segment unless DEEP score >90)
SCENE:      MARKET_BOARD
DURATION:   5–8 minutes
CREW:       Dex leads. Zara reacts skeptically. Graphics deploys live price chart.
            Market ticker activates across entire surge coverage.
MAX PER HOUR: 1 surge segment. Additional moves queued for next MARKET_DESK.
```

### BIG_GAME
```
TRIGGER:    ESPN feed shows a Finals/Championship/Major event in progress
ACTIVATION: At SPORTS_DESK segment. Extends to full 8-minute sports block.
SCENE:      SPORTS_DESK with score bug active throughout hour
CREW:       Dex leads with full analysis. Zara asks audience-proxy questions.
            Score bug runs all hour, not just during SPORTS_DESK.
SPECIAL:    Director keeps SPORTS_DESK scene available for mid-segment score updates.
```

### AUDIENCE_HOT
```
TRIGGER:    Viewer count > 15 OR tip surge (3+ tips in 5 min)
ACTIVATION: COMMUNITY segment extends by 3 minutes
            Dex does a proper roll call
            Community Manager surfaces all pending questions
CREW:       Community Manager escalates. Dex leads. Zara reacts.
            Graphics Operator activates COMMUNITY ticker.
```

### NIGHT_MODE
```
TRIGGER:    Block == "night" (22:00–04:59 UTC)
ACTIVATION: At hour boundary
MODIFICATIONS:
  - SPORTS_DESK skipped (replaced by extended BANTER or COMMUNITY)
  - MUSIC_BREAK extended by 60 seconds
  - All scene transitions switch to dissolves
  - Ticker deactivates
  - Graphics minimal throughout
  - Director slows cut rhythm by 30%
  - Both anchors shift to intimate register
```

---

## Weekly Programming Variation

```
MONDAY:
  COLD_OPEN theme: "What the weekend dropped on us."
  HEADLINES: Weekend recap leads the first hour.

WEDNESDAY:
  MARKET_DESK: Dex does a mid-week portfolio check.
  Extended market coverage if mid-week volatility detected.

FRIDAY:
  COMMENTARY extended: "Week in Review" replaces standard commentary.
  SIGN_OFF: Weekend preview — Zara and Dex each pick one thing to watch.

SATURDAY:
  MORNING_RUSH becomes WEEKEND_MORNING: slower pace, longer BANTER.
  SPORTS_DESK expands: game day previews, predictions, extended analysis.

SUNDAY:
  NIGHT_SHIFT starts 2 hours early (20:00 UTC).
  COMMENTARY goes long: "Sunday takes" format — no holds barred.
  Extended COMMUNITY segment: most engaged audience of the week.
```

---

## Energy Arc Per Hour

```
MINUTE   ENERGY LEVEL    NOTE
00–03    High            Cold open sets the tone
04–12    High→Medium     Headlines can't be all high — vary story energy
13–18    Medium→Heavy    Deep dive goes deeper
19–23    High            Market desk spikes energy
24–27    Low             Banter resets from the spike
28–33    Medium          Culture beat builds back
34–36    Off             Music break — full reset
37–43    Medium→High     Second headlines + any breaking updates
44–49    Variable        Community — audience drives energy
50–54    High            Commentary peaks here
55–57    Medium          Sports desk landing
58–59    Low→Warm        Sign-off descends cleanly

RULE: No two consecutive HIGH segments without a reset between them.
RULE: The hour must end lower than it peaked. Always descend into the sign-off.
RULE: The music break at minute 34 is the mandatory full reset of the hour.
```

---

## Producer Segment Queue Format

```python
SEGMENT_QUEUE = [
    {
        "segment_id": str,
        "scene": str,
        "stories": [story_id_1, story_id_2],   # from editorial score
        "graphics_staged": bool,
        "estimated_duration": int,
        "priority": int,                         # 1=locked, 2=flexible, 3=cuttable
        "special_flag": str | None               # BREAKING | SURGE | BIG_GAME | None
    }
]
```
