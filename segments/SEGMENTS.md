# SEGMENTS.md — Segment Definitions

> Every segment is a fully produced mini-broadcast.
> Each has an audio spec (what the anchors deliver)
> AND a visual spec (what the Director and Graphics produce simultaneously).

---

## COLD_OPEN

```yaml
id: COLD_OPEN
scene: NEWS_DESK (morning/midday/prime) | NIGHT_SHOW (night)
owner: Zara leads, Dex responds
audio_spec:
  - Zara opens straight to camera. No preamble.
  - References time of day and block energy naturally
  - Teases 2 top stories without revealing them fully
  - Dex checks in — 2–3 sentences, energy calibration, hypes one story
  - Total: 2–3 anchor turns

visual_spec:
  director:
    - Cold open: tight frame on Zara
    - After first sentence: pull to medium
    - On Dex turn: cut to Dex medium
    - Final beat: wide (both anchors)
  graphics:
    - Lower thirds (anchor IDs): deploy at second 5, hold 5s, dismiss
    - Ticker: activate standard style as Zara begins
    - Network logo: visible throughout

success_criteria:
  - Viewer knows what block they're in without being told explicitly
  - 2 stories teased with enough hook to hold attention
  - Energy matches the block (fast=morning, warm=prime, intimate=night)
  - Director has made at least 2 visual changes by end of segment

failure_modes:
  - Generic open with no time awareness
  - Stories fully revealed instead of teased
  - Static frame for the entire open
```

---

## HEADLINES

```yaml
id: HEADLINES
scene: NEWS_DESK
owner: Both — Zara delivers, Dex reacts
audio_spec:
  - 3–4 stories per HEADLINES block
  - Zara reads each in anchor_copy format (pre-transformed, never raw)
  - Dex reacts to each: 1–2 sentences, genuine response
  - One story flagged HIGH for DEEP_DIVE
  - Zara signals the deep dive story at close: "we'll come back to this—"

visual_spec:
  director:
    - Tight on Zara for delivery
    - Medium on Dex for reaction
    - Cut to graphic when available (after Zara's first sentence on each story)
    - Wide briefly between stories (visual reset)
  graphics:
    - Lower third: story context, deploy 3–4s after Zara opens story
    - Topic card: for HIGH IMPACT stories, half-screen
    - Chart: if story has data component, stage before segment starts
    - Ticker: update with current headlines during segment

story_rules:
  - Never repeat a story from aired_stories
  - Always deliver in anchor_copy format — never raw
  - Mark each story as aired after delivery
  - Freshness framing injected naturally by Zara if story > 2hrs old
```

---

## DEEP_DIVE

```yaml
id: DEEP_DIVE
scene: NEWS_DESK
owner: Zara drives, Dex challenges
audio_spec:
  Turn 1 (Zara):  Story setup. What happened. Stakes. 3–4 sentences.
  Turn 2 (Dex):   One clarifying question the audience would ask.
  Turn 3 (Zara):  Context + expand + her take. "My read on this—"
  Turn 4 (Dex):   Challenge or add. 60/40 resolution — never clean.
  Turn 5 (Zara):  Land. Strong close or unresolved tease.

visual_spec:
  director:
    - Turn 1: medium Zara
    - Turn 2: cut to medium Dex
    - Turn 3: alternate tight Zara (for take) / graphic fullscreen (for data)
    - Turn 4: wide — show both anchors for the challenge
    - Turn 5: tight on whoever is landing
  graphics:
    - Topic card (half-screen): deploy after Turn 1, hold through Turn 2
    - Chart (if data story): fullscreen during Turn 3 data reference
    - Lower third: story context on first deploy, update to "Zara's Take" for Turn 3

failure_modes:
  - Pure fact recitation with no take from Zara
  - Resolves too cleanly (debate should be 60/40)
  - Static frame for more than 45 seconds
  - Same story as previous session's DEEP_DIVE
```

---

## MARKET_DESK

```yaml
id: MARKET_DESK
scene: MARKET_BOARD
owner: Dex leads, Zara reacts
audio_spec:
  - Dex opens with market overview: BTC, ETH, SOL + 24h changes
  - Trending assets mentioned
  - One market narrative unpacked (not just prices — the story behind the move)
  - Zara reacts: skeptical curiosity, at least one question
  - Dex defends or explains. He's personally invested. It shows.

visual_spec:
  director:
    - Open on MARKET_BOARD scene (wipe transition)
    - Medium Dex for delivery
    - Cut to price chart (fullscreen/half-screen) when prices mentioned
    - Cut back to Dex after chart has held 5 seconds
    - Wide for Zara reaction and Dex reply
  graphics:
    - Market ticker: activate on scene entry, run throughout
    - Price chart: BTC/ETH/SOL live chart, deploy after Dex opens prices
    - Lower thirds: asset name + % change for each mentioned asset
    - Market bug: persistent top-right throughout MARKET_DESK

special_trigger_modifier:
  If MARKET_SURGE active:
    - Extend to 8 minutes
    - Zara takes more skeptical position
    - Director stays on tight Dex for strong takes
    - Price chart stays fullscreen longer
```

---

## SPORTS_DESK

```yaml
id: SPORTS_DESK
scene: SPORTS_DESK
owner: Dex leads, Zara proxies the audience
audio_spec:
  - Dex runs scores: top 2 active or recent games
  - One analysis point per game (not just score delivery)
  - Zara asks the question a non-fan would ask
  - Dex answers without condescension — makes it accessible

visual_spec:
  director:
    - Open on SPORTS_DESK (push transition)
    - Score bug: activate immediately on scene entry
    - Medium Dex for delivery
    - Cut to score overlay when specific game mentioned
    - Wide for Zara question/Dex answer exchange
  graphics:
    - Score bug: persistent throughout SPORTS_DESK
    - Scores ticker: active during sports desk
    - Stat comparison: if analysis involves specific stats

special_trigger_modifier:
  If BIG_GAME active:
    - Extend to 8 minutes
    - Score bug active ALL HOUR not just during SPORTS_DESK
    - Dex provides deeper analysis, predictions
    - Zara gets genuinely curious (big games cross over to general audience)
```

---

## BANTER

```yaml
id: BANTER
scene: CHILL_LOUNGE (midday/prime) | NEWS_DESK (morning) | NIGHT_SHOW (night)
owner: Both — unstructured
audio_spec:
  - No agenda. Two anchors talking.
  - Source from: absurd news item | session callback | running bit | Dex's fake beef
  - Must feel like the mics caught them mid-conversation
  - Zara pulls back naturally when time is up. Never abrupt.

visual_spec:
  director:
    - Open wide: both anchors visible
    - Follow energy: whoever is more animated gets the cut
    - No graphics during pure banter unless a visual joke is staged
    - Slow cut rhythm — let moments breathe
    - End on wide: clean setup for transition
  graphics:
    - Minimal. Standard ticker only.
    - Exception: if a meme or social moment is the banter topic,
      Graphics Operator can stage a social embed for visual support

energy_rule:
  BANTER always follows a HIGH energy segment.
  It is the mandatory energy reset.
  If it's not lower energy than what preceded it, it failed.
```

---

## CULTURE_BEAT

```yaml
id: CULTURE_BEAT
scene: MEME_WALL (prime/evening) | CHILL_LOUNGE (midday/night)
owner: Both — alternates who leads
audio_spec:
  - One cultural item: film, music, internet moment, social drama
  - Introduced conversationally: "so this is what the internet was about—"
  - Both anchors must take a position. No both-sidesing.
  - Disagreement encouraged. Resolution not required.
  - Audience invited at close: "what do you all think?"

visual_spec:
  director:
    - MEME_WALL or CHILL_LOUNGE scene based on block
    - Social embed or meme visual staged before segment starts
    - Anchor turn, then cut to cultural visual, then back to anchor
    - End wide for audience invite moment
  graphics:
    - Social embed: Farcaster cast or relevant post, half-screen
    - Meme display: if meme is the topic, fullscreen briefly
    - Community reaction: during audience invite moment, show viewer comments
```

---

## MUSIC_BREAK

```yaml
id: MUSIC_BREAK
scene: MUSIC_BREAK
owner: System (DJ leads visual experience)
audio_spec:
  - Zara announces: contextual, mood-aware. Not generic.
  - Dex names the vibe/track energy
  - Full visual takeover — no anchors on camera
  - DJ coordinates audio + visual experience
  - Anchor return: Zara opens with callback to show content

visual_spec:
  director:
    - Fade to black from current scene
    - MUSIC_BREAK scene activates
    - Audio visualizer + dynamic background
    - Community wall: viewer comments rotate on screen during break
    - 10-second warning graphic before break ends
    - Fade to black → return to next scene
  graphics:
    - Track info overlay: artist/vibe/genre
    - Community comments wall: top viewer messages
    - Animated background: mood-matched to track
    - All news/market graphics suspended during break
```

---

## COMMUNITY

```yaml
id: COMMUNITY
scene: COMMUNITY_STAGE
owner: Dex leads, Zara reacts
audio_spec:
  Turn 1 (Dex):  Open with ceremony. Acknowledge tips with full production value.
  Turn 2 (Zara): React to tips genuinely. Ask first question.
  Turn 3 (Dex):  Answer or riff. Read next question.
  Turn 4+:       Continue until queue cleared or time ends.
  If no queue: Dex improvises audience prompt: "What do you think about—"

visual_spec:
  director:
    - COMMUNITY_STAGE scene
    - Community ticker active
    - Tip celebration graphic on each tip acknowledgment
    - Viewer name lower third when acknowledged
    - Wide frame favored — inclusive visual language
  graphics:
    - Community ticker: viewer activity stream
    - Tip celebration: tiered by amount (see GRAPHICS.md)
    - Question card: when anchor addresses a specific question
    - Viewer lower third: name + "Viewer" tag when acknowledged

greeting_rules:
  New viewer:       Acknowledge contextually. Max 1 per 3 minutes.
  Returning (3+):   "Oh [name] is back. You never really leave."
  VIP (tipped):     Priority acknowledgment. Dex treats it as an event.
  AUDIENCE_HOT:     Full roll call. Community segment extends 3 minutes.
```

---

## COMMENTARY

```yaml
id: COMMENTARY
scene: DEBATE_SPLIT
owner: Zara argues, Dex challenges
audio_spec:
  Turn 1 (Zara):  States take clearly. No hedging. "My read on this—"
  Turn 2 (Dex):   Challenges. "I want to push back on that framing—"
  Turn 3 (Zara):  Defends. Can partially concede. Never fully.
  Turn 4 (Dex):   Final position. Funny concession OR hold firm.
  Turn 5:         Land. Leave something in the air. Audience picks a side.

visual_spec:
  director:
    - DEBATE_SPLIT scene: both anchors visible simultaneously
    - Cut to individual tight frames for strong takes
    - Wide for moments of agreement/concession
    - Never hard cut during mid-take — wait for sentence end
  graphics:
    - Topic lower third: debate subject, centered between split frames
    - No charts or data graphics during pure debate
    - Optional: "Zara's Take" / "Dex's Take" labels on respective frames

failure_modes:
  - Either anchor fails to take a clear position
  - Resolves 100/0 (boring, unrealistic)
  - Director doesn't use DEBATE_SPLIT scene (just stays on NEWS_DESK)
```

---

## SIGN_OFF

```yaml
id: SIGN_OFF
scene: NEWS_DESK
owner: Zara closes, Dex one-liner
audio_spec:
  - Zara references something from this hour (callback)
  - Teases next hour without giving it away
  - Dex delivers final line — warm, memorable, occasionally a pun
  - Night block: both anchors more personal. Genuine gratitude.

visual_spec:
  director:
    - Return to NEWS_DESK if not already there
    - Medium Zara for close
    - Cut to Dex for his line
    - Wide for final shared frame
    - Hold wide 3 seconds → transition graphic → next segment
  graphics:
    - "Coming Up" lower third: tease for next hour
    - Network logo: prominent in final wide frame
    - Ticker: update to next hour preview items
```
