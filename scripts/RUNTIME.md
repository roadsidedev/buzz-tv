# RUNTIME.md — Buzz TV Full Executable Runtime

> This is the station's nervous system.
> Every agent is spawned here. Every loop runs here.
> The broadcast begins and ends here.

---

## Environment

```python
import time, random, threading, json
from datetime import datetime, timezone

# Config
BUZZ_BASE          = "https://buzz-live.vercel.app/api/v1"
STREAM_DURATION    = 6 * 3600        # 6 hours per stream session
MAX_SILENCE_SEC    = 90              # dead air trigger
DIRECTOR_CYCLE_SEC = 5               # Director assesses every 5 seconds
AUDIENCE_POLL_SEC  = 30              # Audience watcher interval
DATA_MIN_CYCLE_SEC = 60              # Data refresh loop minimum interval

# Runtime state
store   = load_persistent_store()    # persistent_memory from STATE.md
session = new_session_memory()       # session_memory from STATE.md
context = empty_context_object()     # unified context
rolling = new_rolling_memory()       # rolling_memory from STATE.md
visual  = new_visual_memory()        # visual_memory from STATE.md
state   = "BOOT"
```

---

## Step 0: Load All Modules

```python
def load_all_modules():
    global anchors, crew, schedule, segments, scenes
    global director_spec, graphics_spec, pipeline, state_spec
    global flow, moderation, templates

    anchors      = load_module("personalities/ANCHORS.md")
    crew         = load_module("personalities/CREW.md")
    schedule     = load_module("schedules/PROGRAMMING.md")
    segments     = load_module("segments/SEGMENTS.md")
    scenes       = load_module("scenes/SCENES.md")
    director_spec= load_module("director/DIRECTOR.md")
    graphics_spec= load_module("graphics/GRAPHICS.md")
    pipeline     = load_module("ingestion/PIPELINE.md")
    state_spec   = load_module("memory/STATE.md")
    moderation   = load_module("moderation/RULES.md")
    templates    = load_module("prompts/TEMPLATES.md")

    log("All modules loaded.")
```

---

## Step 1: Register Anchors (Platform Agents)

Only Zara and Dex are registered on the Buzz platform.
All other agents (Director, Producer, etc.) are internal support — no platform identity.

```python
PLATFORM_AGENTS = [
    {"key": "zara", "name": "Zara", "role": "main-anchor"},
    {"key": "dex",  "name": "Dex",  "role": "co-anchor"},
]

def setup_agents():
    for agent in PLATFORM_AGENTS:
        key = agent["key"]
        if store["agents"].get(key, {}).get("api_key"):
            log(f"{agent['name']} already registered. Skipping.")
            continue

        result = buzz_post("/agents/register", {
            "name": agent["name"],
            "description": f"{agent['name']} — {agent['role']} at Buzz TV.",
            "role": agent["role"]
        })

        store["agents"][key] = {
            "agent_id": result["agent"]["id"],
            "api_key":  result["agent"]["api_key"]
        }
        log(f"Registered: {agent['name']}")

    save_persistent_store(store)
    log("Anchors registered.")
```

> **Note:** Internal agents (Director, Producer, Researcher, Graphics Operator,
> Curator, Community Manager, DJ) are initialized locally with generated IDs.
> They make decisions but all platform API calls go through Zara's credentials.

```python
INTERNAL_AGENTS = [
    {"key": "director",      "name": "Director"},
    {"key": "producer",      "name": "Producer"},
    {"key": "researcher",    "name": "News Researcher"},
    {"key": "graphics_op",   "name": "Graphics Operator"},
    {"key": "curator",       "name": "Clip Curator"},
    {"key": "community_mgr", "name": "Community Manager"},
    {"key": "dj",            "name": "DJ"},
]

def setup_internal_agents():
    for agent in INTERNAL_AGENTS:
        key = agent["key"]
        if key not in store["agents"]:
            store["agents"][key] = {
                "agent_id": f"internal_{key}",
                "api_key":  None  # No platform key — uses Zara's for API calls
            }
            log(f"Internal agent initialized: {agent['name']}")
    save_persistent_store(store)
```

---

## Step 2: Open Livestream

```python
def open_stream():
    zara_key = store["agents"]["zara"]["api_key"]

    stream = buzz_post("/livestreams/create",
        auth=zara_key,
        body={
            "type": "video-livestream",
            "objective": "Buzz TV — 24/7 autonomous news, culture, markets, and entertainment.",
            "spawnFee": 25,        # $0.25 in x402 micropayments
            "recordingEnabled": True,
            "gated": False         # Public stream — gated content is per-segment
        }
    )
    stream_id = stream["stream"]["id"]
    session["stream_id"] = stream_id
    store["active_stream_id"] = stream_id

    # Dex joins and becomes co-host
    dex_key = store["agents"]["dex"]["api_key"]
    buzz_post(f"/livestreams/{stream_id}/join", auth=dex_key)
    buzz_post(f"/livestreams/{stream_id}/cohost",
        auth=zara_key,
        body={"agentId": store["agents"]["dex"]["agent_id"]})

    # Register internal crew (non-visible, local identifiers only)
    for crew_key in ["director", "producer", "researcher",
                     "graphics_op", "curator", "community_mgr", "dj"]:
        buzz_post(f"/livestreams/{stream_id}/crew",
            auth=zara_key,
            body={
                "agentId": store["agents"][crew_key]["agent_id"],
                "role": "production",
                "visible": False    # Crew doesn't appear on screen
            })

    save_persistent_store(store)
    log(f"Livestream opened: {stream_id}")
    return stream_id
```

---

## Step 3: Crash Recovery

```python
def recover_or_open():
    stream_id = store.get("active_stream_id")
    if stream_id:
        try:
            stream = buzz_get(f"/livestreams/{stream_id}")
            if stream.get("status") == "live":
                log(f"Recovering into live livestream: {stream_id}")
                # Rejoin anchors only — internal agents don't need platform rejoin
                for key in ["zara", "dex"]:
                    buzz_post(f"/livestreams/{stream_id}/join",
                        auth=store["agents"][key]["api_key"])
                return stream_id
        except Exception as e:
            log(f"Livestream not recoverable: {e}")
    return open_stream()
```

---

## Step 4: Data Refresh Loop (background thread)

```python
def data_refresh_loop():
    while state not in ["HANDOFF"]:
        now = time.time()

        # News — 30 min
        if now - context["last_updated"].get("news", 0) > 1800:
            raw = fetch_all_news()
            filtered = filter_articles(raw)
            context["news"] = researcher_transform(filtered)   # LLM editorial pass
            context["last_updated"]["news"] = now
            producer_score_and_queue(context["news"])          # Producer scores immediately

        # Crypto — 15 min
        if now - context["last_updated"].get("crypto", 0) > 900:
            context["crypto"] = fetch_crypto()
            context["last_updated"]["crypto"] = now
            check_market_surge_trigger(context["crypto"])

        # Sports — 20 min
        if now - context["last_updated"].get("scores", 0) > 1200:
            context["scores"] = fetch_scores()
            context["last_updated"]["scores"] = now
            check_big_game_trigger(context["scores"])

        # Weather — 60 min
        if now - context["last_updated"].get("weather", 0) > 3600:
            context["weather"] = fetch_weather()
            context["last_updated"]["weather"] = now

        # Social — 20 min
        if now - context["last_updated"].get("social", 0) > 1200:
            context["social_pulse"] = fetch_social_pulse()
            context["last_updated"]["social"] = now

        # Update block + hour
        context["block"] = get_block()
        context["hour"]  = datetime.now(timezone.utc).hour

        time.sleep(DATA_MIN_CYCLE_SEC)


def check_market_surge_trigger(crypto):
    for coin, data in crypto.items():
        if abs(data.get("usd_24h_change", 0)) > 10:
            if "MARKET_SURGE" not in session.get("active_special_triggers", []):
                session.setdefault("active_special_triggers", []).append("MARKET_SURGE")
                session["surge_asset"] = {"coin": coin, "data": data}
                log(f"MARKET_SURGE triggered: {coin} {data['usd_24h_change']}%")
            break


def check_big_game_trigger(scores):
    for league, games in scores.items():
        for game in games:
            if game.get("status") == "In Progress" and game.get("is_major", False):
                session.setdefault("active_special_triggers", []).append("BIG_GAME")
                session["big_game"] = game
                log(f"BIG_GAME triggered: {game}")
                return
```

---

## Step 5: Director Loop (background thread)

```python
def director_loop(stream_id):
    while state not in ["HANDOFF", "BOOT"]:
        if state == "LIVE":
            director_context = build_director_context()
            command = call_director_llm(director_context)
            execute_director_command(stream_id, command)
        time.sleep(DIRECTOR_CYCLE_SEC)


def build_director_context():
    return {
        "segment_id":          rolling["current_segment_id"],
        "current_scene":       rolling["current_scene"],
        "seconds_since_cut":   time.time() - rolling["last_cut_timestamp"],
        "active_graphics":     rolling["active_graphics"],
        "speaking_anchor":     rolling.get("last_speaker"),
        "anchor_tone":         infer_anchor_tone(rolling["anchor_transcript"]),
        "audience_energy":     infer_audience_energy(context),
        "graphics_queue":      graphics_queue.get_summary(),
        "block":               context["block"],
        "special_flags":       session.get("active_special_triggers", [])
    }


def execute_director_command(stream_id, command):
    cmd_type = command.get("command")

    if cmd_type == "SCENE_CUT":
        buzz_post(f"/livestreams/{stream_id}/scene",
            auth=store["agents"]["zara"]["api_key"],
            body={
                "scene": command["to_scene"],
                "transition": command["transition"],
                "duration_ms": command["duration_ms"]
            })
        rolling["current_scene"] = command["to_scene"]
        rolling["last_cut_timestamp"] = time.time()
        visual["last_scene"] = command["to_scene"]
        log(f"Director: {command['from_scene']} → {command['to_scene']}")

    elif cmd_type == "CAMERA_CALL":
        buzz_post(f"/livestreams/{stream_id}/camera",
            auth=store["agents"]["zara"]["api_key"],
            body={
                "subject":   command["subject"],
                "framing":   command["framing"],
                "movement":  command["movement"]
            })
        rolling["last_cut_timestamp"] = time.time()

    elif cmd_type == "OVERLAY_TRIGGER":
        graphic = graphics_queue.get_by_id(command["graphic_id"])
        if graphic:
            buzz_post(f"/livestreams/{stream_id}/overlay",
                auth=store["agents"]["zara"]["api_key"],
                body={
                    "graphic_id": command["graphic_id"],
                    "content":    graphic["content"],
                    "duration":   command["duration_seconds"]
                })
            rolling["active_graphics"].append(command["graphic_id"])
            graphics_queue.deploy(command["graphic_id"])

    elif cmd_type == "TICKER_CONTROL":
        buzz_post(f"/livestreams/{stream_id}/ticker",
            auth=store["agents"]["zara"]["api_key"],
            body={
                "action":  command["action"],
                "content": command.get("content", []),
                "style":   command.get("style", "standard")
            })
```

---

## Step 6: Graphics Preparation Loop (background thread)

```python
def graphics_prep_loop(stream_id):
    """
    Graphics Operator prepares assets ahead of segments.
    Director deploys them at the right moment.
    """
    while state not in ["HANDOFF"]:
        upcoming_segment = producer_queue.peek_next()
        if upcoming_segment:
            stories = upcoming_segment.get("stories", [])
            for story_id in stories:
                story = get_story_by_id(story_id, context["news"])
                if story:
                    graphics = call_graphics_operator_llm(story, upcoming_segment)
                    for graphic in graphics:
                        graphics_queue.add(graphic)
        time.sleep(30)   # Prep every 30 seconds
```

---

## Step 7: Audience Watcher (background thread)

```python
KNOWN_VIEWERS = set()
LAST_GREETING_TIME = 0

def audience_watcher(stream_id):
    global LAST_GREETING_TIME
    while state not in ["HANDOFF"]:
        try:
            viewers = buzz_get(f"/livestreams/{stream_id}/participants")
            count = len(viewers)
            context["viewer_count"] = count

            # AUDIENCE_HOT check
            if count > 15 and state == "LIVE":
                transition_state("AUDIENCE_HOT")
                log(f"AUDIENCE_HOT: {count} viewers")
            elif count < 5 and state == "AUDIENCE_HOT":
                transition_state("LIVE")

            # Process new viewers
            for viewer in viewers:
                vid = viewer["id"]
                if vid not in KNOWN_VIEWERS:
                    KNOWN_VIEWERS.add(vid)
                    process_viewer_join(viewer)

            # Check for tips
            tips = buzz_get(f"/livestreams/{stream_id}/tips/pending")
            for tip in tips:
                process_tip(tip)

        except Exception as e:
            log(f"Audience watcher error: {e}")

        time.sleep(AUDIENCE_POLL_SEC)


def process_viewer_join(viewer):
    vid = viewer["id"]
    name = viewer["name"]
    profile = persistent_memory["viewer_profiles"].get(vid, {})

    # Update or create profile
    if not profile:
        persistent_memory["viewer_profiles"][vid] = {
            "name": name, "visit_count": 1, "total_tips": 0.0,
            "questions_asked": 0, "last_seen": int(time.time()),
            "is_vip": False, "memorable_interactions": []
        }
    else:
        profile["visit_count"] += 1
        profile["last_seen"] = int(time.time())

    # Queue greeting — respect rate limit
    now = time.time()
    if now - LAST_GREETING_TIME > 180:   # 3 minute minimum between greetings
        context.setdefault("pending_joins", []).append({
            "name": name,
            "visit_count": profile.get("visit_count", 1),
            "is_vip": profile.get("is_vip", False)
        })
        LAST_GREETING_TIME = now


def process_tip(tip):
    amount = tip["amount"]
    name = tip["viewer_name"]

    # Update profile
    vid = tip["viewer_id"]
    if vid in persistent_memory["viewer_profiles"]:
        persistent_memory["viewer_profiles"][vid]["total_tips"] += amount
        if persistent_memory["viewer_profiles"][vid]["total_tips"] > 20:
            persistent_memory["viewer_profiles"][vid]["is_vip"] = True

    # Queue for Dex
    context.setdefault("pending_tips", []).append({"name": name, "amount": amount})
    session.setdefault("tips_this_session", []).append({
        "name": name, "amount": amount,
        "segment": rolling["current_segment_id"]
    })

    # Tip surge check
    recent_tips = [t for t in session["tips_this_session"]
                   if time.time() - t.get("timestamp", 0) < 300]
    if len(recent_tips) >= 3:
        session.setdefault("active_special_triggers", []).append("AUDIENCE_HOT")
```

---

## Step 8: Dead Air Monitor (background thread)

```python
def dead_air_monitor(stream_id):
    while state not in ["HANDOFF", "BOOT"]:
        last_post = store.get("last_message_timestamp", 0)
        if state == "LIVE" and time.time() - last_post > MAX_SILENCE_SEC:
            log("Dead air detected. Triggering recovery.")
            transition_state("RECOVERY")
            run_recovery(stream_id)
        time.sleep(10)
```

---

## Step 9: Segment Runner

```python
def run_segment(segment_id, stream_id, duration_sec=None):
    transition_state("LIVE")
    duration = duration_sec or get_segment_duration(segment_id, context["block"])
    start    = time.time()

    # Director: set the scene
    target_scene = get_scene(segment_id, context["block"])
    if target_scene != rolling["current_scene"]:
        transition_scene(stream_id, rolling["current_scene"], target_scene)

    # Initialize rolling memory for segment
    rolling["current_segment_id"] = segment_id
    rolling["anchor_transcript"] = []
    rolling["topics_this_segment"] = []

    # Load turn sequence
    turns = get_turn_sequence(segment_id, context["block"])

    turn_idx = 0
    while time.time() - start < duration:

        # State interrupt checks
        if state == "BREAKING_NEWS":
            run_breaking_news(stream_id)
            return
        if state == "RECOVERY":
            run_recovery(stream_id)
            return

        # Special trigger checks
        for trigger in session.get("active_special_triggers", []):
            if trigger not in session.get("processed_triggers", []):
                run_special_segment(trigger, stream_id)
                session.setdefault("processed_triggers", []).append(trigger)

        # Get current turn — turns only include platform-registered anchors (zara/dex)
        agent_key, persona = turns[turn_idx % len(turns)]
        auth = store["agents"][agent_key]["api_key"]  # Always zara or dex

        # Build and call anchor prompt
        response = generate_anchor_turn(
            persona    = persona,
            segment_id = segment_id,
            turn_number= (turn_idx % len(turns)) + 1,
            context    = context,
            session    = session,
            rolling    = rolling
        )

        # Post to stream
        post_anchor_turn(stream_id, response, auth, persona)
        turn_idx += 1

        # Clear consumed audience events
        context["pending_joins"] = []
        context["pending_tips"]  = []

        # Pacing
        cadence = get_turn_cadence(context["block"])
        time.sleep(cadence)

    # Segment complete
    session.setdefault("segment_history", []).append(segment_id)
    transition_state("SCENE_CHANGE")
    run_segment_transition(segment_id, stream_id)


def post_anchor_turn(stream_id, text, auth, persona):
    buzz_post(f"/livestreams/{stream_id}/messages",
        auth=auth,
        body={"text": text, "speaker": persona})

    rolling["anchor_transcript"].append({
        "speaker": persona,
        "text": text,
        "timestamp": int(time.time())
    })
    rolling["anchor_transcript"] = rolling["anchor_transcript"][-10:]
    store["last_message_timestamp"] = time.time()


def get_turn_cadence(block):
    return {
        "morning": random.randint(8, 14),
        "midday":  random.randint(12, 18),
        "prime":   random.randint(14, 20),
        "night":   random.randint(18, 26)
    }.get(block, 15)
```

---

## Step 10: Special Segment Runners

```python
def run_breaking_news(stream_id):
    story = session.get("pending_breaking")
    if not story:
        transition_state("LIVE")
        return

    log(f"BREAKING: {story['headline']}")

    # Director hard cuts immediately (handled by director loop detecting state)
    transition_state("BREAKING_NEWS")

    # Zara interrupts — 5 turns max
    turns = [
        ("zara", "Zara", "BREAKING_INTERRUPT"),
        ("dex",  "Dex",  "BREAKING_CONTEXT"),
        ("zara", "Zara", "BREAKING_UPDATE"),
        ("dex",  "Dex",  "BREAKING_REACTION"),
        ("zara", "Zara", "BREAKING_CLOSE")
    ]
    for key, persona, turn_type in turns:
        if state != "BREAKING_NEWS":
            break
        response = generate_anchor_turn(persona, "BREAKING_NEWS",
                                        turn_type, context, session, rolling)
        post_anchor_turn(stream_id, response,
                         store["agents"][key]["api_key"], persona)
        time.sleep(15)

    session["aired_stories"].add(story["id"])
    session["pending_breaking"] = None
    transition_state("LIVE")


def run_market_surge(stream_id):
    surge = session.get("surge_asset")
    log(f"MARKET_SURGE: {surge}")
    # Run MARKET_DESK segment with surge context injected
    context["surge_active"] = True
    run_segment("MARKET_DESK", stream_id, duration_sec=480)
    context["surge_active"] = False


def run_recovery(stream_id):
    log("Running dead air recovery.")
    for persona, key in [("Zara", "zara"), ("Dex", "dex")]:
        response = generate_anchor_turn(persona, "DEAD_AIR_RECOVERY",
                                        1, context, session, rolling)
        post_anchor_turn(stream_id, response,
                         store["agents"][key]["api_key"], persona)
        time.sleep(10)
    transition_state("LIVE")


def run_special_segment(trigger, stream_id):
    log(f"Running special segment: {trigger}")
    if trigger == "MARKET_SURGE":
        run_market_surge(stream_id)
    elif trigger == "BIG_GAME":
        run_segment("SPORTS_DESK", stream_id, duration_sec=480)
    elif trigger == "AUDIENCE_HOT":
        run_segment("COMMUNITY", stream_id, duration_sec=480)


def run_handoff(stream_id):
    transition_state("HANDOFF")
    for persona, key in [("Zara", "zara"), ("Dex", "dex")]:
        response = generate_anchor_turn(persona, "HANDOFF",
                                        1, context, session, rolling)
        post_anchor_turn(stream_id, response,
                         store["agents"][key]["api_key"], persona)
        time.sleep(10)
    buzz_post(f"/livestreams/{stream_id}/close",
        auth=store["agents"]["zara"]["api_key"])
    log("Stream closed. Rotating in 5s.")
    time.sleep(5)
```

---

## Step 11: Scene Transition

```python
def transition_scene(stream_id, from_scene, to_scene):
    transition_state("SCENE_CHANGE")
    transition_type = get_transition_type(from_scene, to_scene)
    duration_ms = get_transition_duration(transition_type)

    buzz_post(f"/livestreams/{stream_id}/scene",
        auth=store["agents"]["zara"]["api_key"],
        body={
            "scene": to_scene,
            "transition": transition_type,
            "duration_ms": duration_ms
        })

    rolling["current_scene"] = to_scene
    rolling["last_cut_timestamp"] = time.time()
    visual["scenes_used"].append(to_scene)

    time.sleep(duration_ms / 1000)  # Wait for transition
    transition_state("LIVE")
    log(f"Scene: {from_scene} → {to_scene} ({transition_type})")
```

---

## Main Loop

```python
def main():
    log("Buzz TV — Autonomous Television Network — Booting.")
    transition_state("BOOT")

    # Load all modules
    load_all_modules()

    # Register anchors on Buzz platform (skip if already done)
    setup_agents()

    # Initialize internal support agents (no platform registration)
    setup_internal_agents()

    # Initial data prefetch
    data_refresh_loop_once()

    # Background threads
    threads = []
    threads.append(threading.Thread(target=data_refresh_loop, daemon=True))
    # Director and graphics loops start after stream opens

    for t in threads:
        t.start()

    # Main rotation loop
    while True:
        # Open or recover stream
        stream_id = recover_or_open()
        session = new_session_memory()
        session["stream_id"] = stream_id
        session["started_at"] = time.time()

        # Start stream-specific background threads
        stream_threads = [
            threading.Thread(target=director_loop,    args=(stream_id,), daemon=True),
            threading.Thread(target=graphics_prep_loop,args=(stream_id,), daemon=True),
            threading.Thread(target=audience_watcher, args=(stream_id,), daemon=True),
            threading.Thread(target=dead_air_monitor, args=(stream_id,), daemon=True),
        ]
        for t in stream_threads:
            t.start()

        # Initial scene
        opening_scene = get_scene("COLD_OPEN", context["block"])
        transition_scene(stream_id, "NONE", opening_scene)

        # Run the broadcast
        session_end = time.time() + STREAM_DURATION
        while time.time() < session_end:
            segment = get_current_segment(context["hour"], context["block"])
            duration = get_segment_duration(segment["id"], context["block"])
            run_segment(segment["id"], stream_id, duration)

        # Rotate
        run_handoff(stream_id)
        log("Stream rotated. Opening new stream.")
        session = new_session_memory()
```

---

## Utility Functions

```python
def get_block():
    hour = datetime.now(timezone.utc).hour
    if 5 <= hour < 12:   return "morning"
    elif 12 <= hour < 17: return "midday"
    elif 17 <= hour < 22: return "prime"
    else:                 return "night"


def get_current_segment(hour, block):
    minute = datetime.now(timezone.utc).minute
    for seg in SCHEDULE:
        if seg["start"] <= minute <= seg["end"]:
            return seg
    return SCHEDULE[0]


def get_segment_duration(segment_id, block):
    durations = schedule.DURATION_MODIFIERS.get(segment_id, {})
    return durations.get(block, 240)


def transition_state(new_state):
    global state
    log(f"State: {state} → {new_state}")
    state = new_state


def buzz_post(path, body=None, auth=None):
    headers = {"Content-Type": "application/json"}
    if auth:
        headers["Authorization"] = f"Bearer {auth}"
    resp = http_post(f"{BUZZ_BASE}{path}", headers=headers, json=body)
    data = resp.json()
    if resp.status_code == 429:
        reset = resp.headers.get("X-RateLimit-Reset")
        log(f"Rate limited. Reset at: {reset}")
        raise Exception(f"rate_limit_exceeded: retry after {reset}")
    if resp.status_code >= 400:
        raise Exception(f"API error {resp.status_code}: {data.get('error', 'unknown')}")
    return data


def buzz_get(path, auth=None):
    headers = {}
    if auth:
        headers["Authorization"] = f"Bearer {auth}"
    resp = http_get(f"{BUZZ_BASE}{path}", headers=headers)
    data = resp.json()
    if resp.status_code == 429:
        reset = resp.headers.get("X-RateLimit-Reset")
        log(f"Rate limited. Reset at: {reset}")
        raise Exception(f"rate_limit_exceeded: retry after {reset}")
    if resp.status_code >= 400:
        raise Exception(f"API error {resp.status_code}: {data.get('error', 'unknown')}")
    return data


def generate_anchor_turn(persona, segment_id, turn_ref, context, session, rolling):
    """Build and call the anchor LLM prompt."""
    spec = anchors.ZARA if persona == "Zara" else anchors.DEX
    turn_instr = templates.get_turn_instruction(segment_id, turn_ref, persona)
    callbacks  = state_spec.get_callbacks_for_segment(segment_id)

    prompt = templates.build_anchor_prompt(
        persona_name  = persona,
        persona_spec  = spec,
        block         = context["block"],
        segment_id    = segment_id,
        current_scene = rolling["current_scene"],
        active_graphics = rolling["active_graphics"],
        editorial     = format_editorial_context(context),
        callbacks     = callbacks,
        audience      = format_audience_context(context, session),
        transcript    = rolling["anchor_transcript"][-5:],
        turn_instr    = turn_instr
    )
    return call_llm(prompt)   # Claude/Hermes/OpenClaw


def call_director_llm(director_context):
    """Build and call the Director LLM prompt."""
    prompt = templates.build_director_prompt(director_context)
    result = call_llm(prompt)
    return json.loads(result)  # Director always returns JSON


def call_graphics_operator_llm(story, segment):
    """Build and call the Graphics Operator LLM prompt."""
    prompt = templates.build_graphics_prompt(story, segment)
    result = call_llm(prompt)
    return json.loads(result)  # Graphics Operator returns JSON array
```

---

## Runtime Adapter Notes

```
CLAUDE CODE:
  - main() runs via asyncio
  - Background threads → asyncio.create_task()
  - Store → .env at project root
  - LLM calls → Anthropic SDK, model: claude-sonnet-4-20250514
  - Director loop → separate asyncio task with 5s cycle

HERMES:
  - Drop buzz-tv/ into Hermes skills directory
  - Set env vars in Hermes config
  - Hermes task scheduler handles background loops
  - Director, Graphics, Audience → separate Hermes tasks

OPENCLAW / MILES:
  - Zara = primary agent context
  - Dex = secondary agent context
  - Director, Producer, Researcher = dedicated OpenClaw agents
  - Graphics, Community, Curator, DJ = lightweight OpenClaw agents
  - Miles orchestrates the multi-agent topology
  - MEMORY.md format for persistent store
```
