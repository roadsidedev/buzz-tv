# STATE.md — Broadcast State Machine & Memory System

> Television has three memory requirements that radio doesn't:
> conversational continuity, visual continuity, and channel identity.
> All three must be maintained simultaneously.

---

## Broadcast State Machine

```
┌──────────────────────────────────────────────────────────────────┐
│                    BUZZ TV STATE MACHINE                         │
│                                                                  │
│  BOOT ─────────────────────────────────────────► LIVE           │
│                                                    │             │
│  LIVE ──── segment ends ──────────────────────► SCENE_CHANGE    │
│  LIVE ──── music cue ─────────────────────────► BREAK           │
│  LIVE ──── BREAKING flag ─────────────────────► BREAKING_NEWS   │
│  LIVE ──── 90s no output ─────────────────────► RECOVERY        │
│  LIVE ──── 6hr mark ──────────────────────────► HANDOFF         │
│  LIVE ──── viewer count > 15 ─────────────────► AUDIENCE_HOT    │
│  LIVE ──── block == night ────────────────────► NIGHT_MODE      │
│                                                    │             │
│  SCENE_CHANGE ──── transition complete ───────► LIVE            │
│  BREAK ─────────── break ends ────────────────► SCENE_CHANGE    │
│  BREAKING_NEWS ─── story delivered ───────────► LIVE            │
│  RECOVERY ─────── hosts back ─────────────────► LIVE            │
│  HANDOFF ──────── new stream open ────────────► BOOT            │
│  AUDIENCE_HOT ─── count < 5 ──────────────────► LIVE            │
│  NIGHT_MODE ────── dawn block ────────────────► LIVE            │
└──────────────────────────────────────────────────────────────────┘
```

---

## State Definitions

```python
STATES = {
    "BOOT": {
        "description": "Loading modules, registering agents, prefetching data, staging assets",
        "max_duration": 180,
        "on_timeout": "RECOVERY",
        "visual": "Pre-stream holding slate"
    },
    "LIVE": {
        "description": "Active segment. All agents running. Broadcast at full production.",
        "max_silence": 90,
        "on_silence": "RECOVERY",
        "visual": "Current scene active"
    },
    "SCENE_CHANGE": {
        "description": "Director executing transition between scenes.",
        "max_duration": 5,       # transition itself
        "on_complete": "LIVE",
        "visual": "Transition animation"
    },
    "BREAK": {
        "description": "Music break. Full visual experience. No anchors on camera.",
        "max_duration": 180,
        "on_complete": "SCENE_CHANGE",
        "visual": "MUSIC_BREAK scene"
    },
    "BREAKING_NEWS": {
        "description": "Breaking story interrupt. Red ticker. Zara leads.",
        "max_duration": 480,
        "on_complete": "LIVE",
        "visual": "BREAKING_NEWS scene"
    },
    "RECOVERY": {
        "description": "Dead air or failure. Emergency anchor fill.",
        "max_duration": 120,
        "on_complete": "LIVE",
        "visual": "Hold current scene"
    },
    "HANDOFF": {
        "description": "Stream rotation. Continuity message before close.",
        "max_duration": 60,
        "on_complete": "BOOT",
        "visual": "NEWS_DESK with handoff lower third"
    },
    "AUDIENCE_HOT": {
        "description": "High viewer activity. Community-driven mode.",
        "triggers": ["viewer_count > 15", "tip_surge"],
        "exits": ["viewer_count < 5"],
        "modifies": "COMMUNITY segment extends, Director adds audience graphics"
    },
    "NIGHT_MODE": {
        "description": "Intimate format. Reduced crew. Slower everything.",
        "triggers": ["block == night"],
        "exits": ["block == morning"],
        "modifies": "All transitions dissolve, ticker off, graphics minimal"
    }
}
```

---

## Three-Layer Memory Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│  LAYER 3: PERSISTENT MEMORY (cross-session, survives restarts)  │
│  Agent credentials, viewer profiles, channel identity,          │
│  proven segments, successful bits, long-term reputation         │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │  LAYER 2: SESSION MEMORY (6-hour stream)                  │  │
│  │  Aired stories, callbacks, running jokes, debate history, │  │
│  │  tip log, visual continuity, segment history              │  │
│  │                                                            │  │
│  │  ┌──────────────────────────────────────────────────────┐  │  │
│  │  │  LAYER 1: ROLLING MEMORY (current segment)          │  │  │
│  │  │  Last 10 anchor turns, active graphics,             │  │  │
│  │  │  current scene, audience events queued              │  │  │
│  │  └──────────────────────────────────────────────────────┘  │  │
│  └────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Rolling Memory (Segment Scope)

```python
rolling_memory = {
    "current_segment_id": str,
    "current_scene": str,
    "anchor_transcript": [
        {"speaker": str, "text": str, "timestamp": int}
    ],  # last 10 turns
    "active_graphics": [str],        # currently deployed graphic IDs
    "last_cut_timestamp": int,       # Director uses this for 45s rule
    "topics_this_segment": [str],
    "audience_events_queued": []
}
```

---

## Session Memory (Stream Scope)

```python
session_memory = {
    "session_id": str,
    "stream_id": str,
    "started_at": int,

    # Editorial
    "aired_stories": set(),          # story IDs that have aired
    "segment_history": [str],        # segments run this session
    "current_callbacks": [
        {
            "type": "joke|debate|reference|prediction",
            "content": str,
            "speaker": str,
            "segment_origin": str,
            "call_back_in": str      # target segment
        }
    ],
    "unresolved_debates": [
        {"topic": str, "zara_position": str, "dex_position": str}
    ],
    "predictions_made": [
        {"content": str, "speaker": str, "segment": str}
    ],

    # Visual continuity
    "scenes_used": [str],            # which scenes have aired this session
    "graphics_deployed": [str],      # graphic IDs used this session
    "last_scene": str,
    "last_energy_reset_time": int,

    # Audience
    "viewers_seen": {
        "viewer_id": {
            "name": str,
            "first_seen": int,
            "times_seen": int,
            "total_tips": float,
            "questions_asked": int,
            "greeted": bool
        }
    },
    "tips_this_session": [{"name": str, "amount": float, "segment": str}],
    "questions_queue": [{"viewer": str, "text": str, "score": int}],

    # Broadcast metrics
    "peak_viewer_count": int,
    "total_segments_aired": int,
    "breaking_news_count": int
}
```

---

## Persistent Memory (Cross-Session)

```python
persistent_memory = {
    # Agent credentials
    "agents": {
        "zara": {"agent_id": str, "api_key": str},
        "dex":  {"agent_id": str, "api_key": str},
        "director":     {"agent_id": str, "api_key": str},
        "producer":     {"agent_id": str, "api_key": str},
        "researcher":   {"agent_id": str, "api_key": str},
        "graphics_op":  {"agent_id": str, "api_key": str},
        "curator":      {"agent_id": str, "api_key": str},
        "community_mgr":{"agent_id": str, "api_key": str},
        "dj":           {"agent_id": str, "api_key": str},
    },
    "active_stream_id": str,

    # Channel identity
    "channel": {
        "total_sessions": int,
        "total_hours_broadcast": float,
        "total_tips_received": float,
        "peak_concurrent_viewers": int,
        "high_engagement_topics": [str],
        "successful_segments": [str],
        "audience_favorite_moments": [str]
    },

    # Viewer profiles
    "viewer_profiles": {
        "viewer_id": {
            "name": str,
            "visit_count": int,
            "total_tips": float,
            "preferred_topics": [str],
            "questions_asked": int,
            "last_seen": int,
            "is_vip": bool,
            "memorable_interactions": [str]  # callbacks worth making
        }
    },

    # Anchor persona continuity
    "anchor_lore": {
        "ongoing_bits": [str],
        "retired_bits": [str],
        "predictions": [{"content": str, "resolved": bool, "outcome": str}],
        "rivalries": [str],         # ongoing narrative threads
        "running_debates": [str]    # multi-session debate threads
    }
}
```

---

## Visual Continuity Memory

TV requires an additional memory layer radio doesn't need.

```python
visual_memory = {
    "scenes_aired_this_hour": [str],    # avoid visual monotony
    "last_graphic_type": str,           # avoid same graphic back to back
    "scene_durations": {                # how long each scene has aired
        "NEWS_DESK": int,               # seconds this session
        "MARKET_BOARD": int,
        # etc.
    },
    "pending_asset_queue": [],          # staged but not yet deployed
    "active_overlay_count": int,        # must stay ≤ 3
}
```

---

## Callback Engine

```python
def queue_callback(event_type, content, speaker, origin_segment, target_segment):
    session_memory["current_callbacks"].append({
        "type": event_type,
        "content": content,
        "speaker": speaker,
        "segment_origin": origin_segment,
        "call_back_in": target_segment
    })

def get_callbacks_for_segment(segment_id):
    return [c for c in session_memory["current_callbacks"]
            if c["call_back_in"] == segment_id]

# Prediction tracking
def log_prediction(content, speaker, segment):
    session_memory["predictions_made"].append({
        "content": content,
        "speaker": speaker,
        "segment": segment
    })
    persistent_memory["anchor_lore"]["predictions"].append({
        "content": content,
        "resolved": False,
        "outcome": None
    })
    # Queue callback for SIGN_OFF or next session COLD_OPEN
    queue_callback("prediction", content, speaker, segment, "SIGN_OFF")
```
