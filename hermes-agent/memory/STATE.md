# STATE.md — Hermes Persistent Memory & Broadcast State

## Persistent State Schema
The Hermes agent maintains a continuous state machine to ensure the 24/7 broadcast remains coherent across sessions.

| Component | Description | Persistence Level |
| :--- | :--- | :--- |
| **Current Segment** | The active show block (e.g., Headlines, Deep Dive). | Real-time |
| **Anchor Context** | Who spoke last, what was the tone, what is the current energy. | Short-term (Session) |
| **Visual State** | Current scene, active graphics, time since last cut. | Real-time |
| **Editorial Queue** | Upcoming stories scored by the Producer. | Mid-term (Daily) |
| **Audience Memory** | Top engagers, recurring themes in chat, total tips. | Long-term (Weekly) |

## State Transitions
Transitions are managed by the **Director** and **Producer** personas.

1. **BOOT → LIVE:** Triggered on startup. Load credentials, verify Buzz API, and execute the "Cold Open."
2. **LIVE → BREAKING:** Triggered by the News Researcher. Overrides the current queue.
3. **LIVE → NIGHT_MODE:** Triggered by system clock (e.g., 00:00 - 06:00). Shifts to lower-intensity production.
4. **LIVE → RECOVERY:** Triggered by API or logic failures. Anchors improvise while technical layer resets.

## Memory Retrieval Logic
Hermes uses a vector-based memory system to recall past broadcast moments.

- **The "Yesterday" Rule:** Before starting a "Deep Dive," the Producer persona queries memory for what was said about this topic in the last 24 hours to ensure continuity.
- **Audience Recognition:** When a top engager enters the Buzz room, the Community Manager persona retrieves their "Engagement Profile" to personalize on-screen callouts.
- **Narrative Loops:** The News Researcher tracks developing stories over days, ensuring anchors can say "As we reported yesterday..." with accuracy.

## Data Ingestion & Scoring
Every piece of raw data (news, tweets, market prices) is scored before being added to the memory state.

| Factor | Weight | Description |
| :--- | :--- | :--- |
| **Relevance** | 35% | Does this match the Buzz TV brand and current audience? |
| **Novelty** | 25% | Is this new information or a fresh perspective? |
| **Coherence** | 20% | Does it fit the current show's narrative flow? |
| **Actionability** | 15% | Can the anchors do something with this (debate, analyze)? |
| **Engagement** | 5% | Is it likely to trigger Buzz platform interactions? |
