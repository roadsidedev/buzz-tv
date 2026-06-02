# BOOT.md — Hermes Startup Playbook

## Phase 1: Identity & Configuration
1. Load `soul.md` to establish the 9-agent production topology.
2. Initialize `hermes.config.yaml` and verify environment variables.
3. Decouple logic from radio counterparts; ensure no cross-talk with "The Wire" or other radio-specific modules.

## Phase 2: Platform Registration
1. **Register Zara:** `POST /agents/register` as the primary host.
2. **Register Dex:** `POST /agents/register` as the co-host.
3. **Internal Setup:** Initialize the 7 Crew agents as local sub-processes. Do NOT register Crew agents on the Buzz platform.
4. **Link Identity:** Authenticate Zara and Dex with their respective `BUZZ_API_KEY`s.

## Phase 3: Stream Initialization
1. **Create Livestream:** Zara issues `POST /livestreams/create` with type `video-livestream`.
2. **Join Dex:** Dex joins the room and Zara sets him as `co-host`.
3. **Background Sync:** Crew agents connect to the stream's metadata and audience feeds via Zara's session.

## Phase 4: First Broadcast Flow
1. **Data Ingestion:** News Researcher fetches initial data; Producer scores the top stories.
2. **Visual Staging:** Director selects the "News Desk" scene; Graphics Operator preps the cold open card.
3. **Cold Open:** Zara delivers the first headline. The 24/7 broadcast loop begins.
