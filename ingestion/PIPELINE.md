# PIPELINE.md — Data Ingestion & Editorial Pipeline

> The pipeline is the News Researcher's domain.
> Raw data in. Broadcast-ready editorial packages out.
> The Producer scores. The anchors never see raw data.

---

## Four-Stage Pipeline

```
STAGE 1: INGEST    → Fetch from all sources on schedule
STAGE 2: FILTER    → Remove stale, duplicate, low-signal items
STAGE 3: TRANSFORM → Convert to broadcast-ready anchor copy + visual suggestions
STAGE 4: SCORE     → 5-factor editorial scoring → deliver to Producer
```

---

## Source Registry

### News
```python
NEWS_SOURCES = {
    "top": {
        "url": "https://newsapi.org/v2/top-headlines?country=us&pageSize=10&apiKey={NEWS_API_KEY}",
        "refresh": 1800
    },
    "tech": {
        "url": "https://newsapi.org/v2/top-headlines?category=technology&pageSize=5&apiKey={NEWS_API_KEY}",
        "refresh": 1800
    },
    "business": {
        "url": "https://newsapi.org/v2/top-headlines?category=business&pageSize=5&apiKey={NEWS_API_KEY}",
        "refresh": 1800
    },
    "entertainment": {
        "url": "https://newsapi.org/v2/top-headlines?category=entertainment&pageSize=5&apiKey={NEWS_API_KEY}",
        "refresh": 2400
    },
    "sports": {
        "url": "https://newsapi.org/v2/top-headlines?category=sports&pageSize=5&apiKey={NEWS_API_KEY}",
        "refresh": 1200
    }
}
```

### Crypto (keyless)
```python
CRYPTO_SOURCES = {
    "prices": {
        "url": "https://api.coingecko.com/api/v3/simple/price?ids=bitcoin,ethereum,solana,sui&vs_currencies=usd&include_24hr_change=true&include_market_cap=true",
        "refresh": 900
    },
    "trending": {
        "url": "https://api.coingecko.com/api/v3/search/trending",
        "refresh": 3600
    },
    "global": {
        "url": "https://api.coingecko.com/api/v3/global",
        "refresh": 1800
    }
}
```

### Sports (keyless)
```python
SPORTS_SOURCES = {
    "nba": "https://site.api.espn.com/apis/site/v2/sports/basketball/nba/scoreboard",
    "nfl": "https://site.api.espn.com/apis/site/v2/sports/football/nfl/scoreboard",
    "soccer": "https://site.api.espn.com/apis/site/v2/sports/soccer/usa.1/scoreboard",
    "refresh": 1200
}
```

### Social Pulse (keyless)
```python
SOCIAL_SOURCES = {
    "farcaster": {
        "url": "https://api.warpcast.com/v2/trending-casts?limit=5",
        "refresh": 1200
    }
}
```

### Weather
```python
WEATHER = {
    "url": "https://api.openweathermap.org/data/2.5/weather?q={SHOW_CITY}&appid={WEATHER_API_KEY}&units=imperial",
    "refresh": 3600,
    "fallback_city": "New York"
}
```

---

## Editorial Transform Prompt

```
SYSTEM:
You are the News Researcher for Buzz TV, a 24/7 autonomous television network.
Your job is to transform raw data into broadcast-ready editorial packages.

RULES:
- Never use wire service citation language ("Reuters reports...", "AP says...")
- Convert passive journalistic prose to active broadcast voice
- Every story needs a visual suggestion — what should be on screen when this airs?
- Flag BREAKING if story is tagged developing/breaking AND less than 30 minutes old
- Anchor copy should sound like a television anchor, not a news reader
- Keep anchor_copy to 3–4 sentences. TV is tight.

OUTPUT JSON (array of items):
[
  {
    "id": "unique_story_id",
    "headline": "Short punchy broadcast headline",
    "anchor_copy": "How Zara would deliver this on air. 3-4 sentences.",
    "visual_suggestion": "lower_third | chart | topic_card | social_embed | none",
    "visual_detail": "Specific description of what the graphic should show",
    "talking_points": ["angle 1", "angle 2"],
    "energy": "shocking | important | fun | heavy | wild | light",
    "is_breaking": false,
    "deep_dive_worthy": false,
    "freshness_hours": 0.5,
    "freshness_framing": ""
  }
]

RAW DATA:
{raw_data}
```

---

## Editorial Scoring System (Producer Layer)

Every transformed story gets a 5-factor score before it can air.

```python
def editorial_score(story, context):
    """
    Returns 0–100. Producer uses this to decide what airs and when.
    """

    # Factor 1: Recency (0–25 points)
    age_hours = story["freshness_hours"]
    if age_hours < 0.5:   recency = 25
    elif age_hours < 1:   recency = 22
    elif age_hours < 2:   recency = 18
    elif age_hours < 4:   recency = 12
    elif age_hours < 8:   recency = 6
    else:                 recency = 0

    # Factor 2: Virality — is it trending online? (0–20 points)
    in_social_pulse = any(
        story["headline"].lower() in pulse.lower()
        for pulse in context["social_pulse"]
    )
    virality = 20 if in_social_pulse else random.randint(3, 10)

    # Factor 3: Novelty — has it aired? (0–20 points)
    novelty = 0 if story["id"] in session["aired_stories"] else 20

    # Factor 4: Audience fit — does our audience care? (0–20 points)
    HIGH_FIT_CATEGORIES = ["technology", "crypto", "finance", "sports", "culture"]
    audience_fit = 20 if story.get("category") in HIGH_FIT_CATEGORIES else 10

    # Factor 5: Emotional charge — will it create a reaction? (0–15 points)
    ENERGY_SCORES = {
        "shocking": 15, "wild": 14, "heavy": 12,
        "important": 10, "fun": 8, "light": 5
    }
    emotional = ENERGY_SCORES.get(story["energy"], 7)

    total = recency + virality + novelty + audience_fit + emotional

    # Breaking news bonus
    if story["is_breaking"]:
        total = min(100, total + 20)

    return total

# Producer thresholds
MUST_AIR_THRESHOLD    = 75
QUEUE_THRESHOLD       = 50
HOLD_THRESHOLD        = 25
# Below 25: drop from queue
```

---

## Freshness Handling

```python
FRESHNESS_FRAMING = {
    (0, 30):   "",                              # Minutes — no qualifier
    (30, 60):  "",                              # Still fresh — no qualifier
    (60, 120): "earlier this morning—",
    (120, 240):"this came in a couple hours ago—",
    (240, 480):"we've been tracking this—",
    (480, 999):"going back to a story from earlier—"
}
# If story is > 8 hours old and score < 50: drop it entirely

def get_freshness_framing(age_minutes):
    for (min_age, max_age), framing in FRESHNESS_FRAMING.items():
        if min_age <= age_minutes < max_age:
            return framing
    return "going back to a story from earlier—"
```

---

## Visual Asset Pipeline (Clip Curator)

Every story that scores > 50 gets a visual asset staged by the Curator.

```python
def stage_visual_assets(story):
    suggestion = story["visual_suggestion"]
    detail     = story["visual_detail"]

    if suggestion == "chart":
        return build_chart(detail)          # Pull from live data
    elif suggestion == "topic_card":
        return build_topic_card(story)      # Headline + bullets
    elif suggestion == "social_embed":
        return fetch_social_embed(detail)   # Farcaster/social link
    elif suggestion == "lower_third":
        return build_lower_third(detail)    # Text overlay spec
    else:
        return None                         # No graphic needed
```

---

## Graceful Degradation

```
NEWS API DOWN:
  → Use cached news (max 4 hours)
  → Anchor framing: "this came in earlier—"
  → Producer activates COMMENTARY or COMMUNITY to fill
  → Retry every 5 minutes silently

CRYPTO API DOWN:
  → Dex works from last known prices, flags uncertainty
  → "I don't have the live number but last I saw—"
  → MARKET_DESK shortened to analysis, not prices

SPORTS API DOWN:
  → Dex works from memory, announces it
  → SPORTS_DESK becomes sports analysis, not score delivery
  → Score bug deactivated

ALL SOURCES STALE (>4 hours):
  → Producer activates COMMENTARY extended + COMMUNITY extended
  → Zara and Dex hold the broadcast on editorial content
  → Director maintains broadcast presentation — no visual degradation
  → The stream never looks broken even when data is thin
```
