# Hermes Agent Onboarding — Buzz TV

This directory contains the production-grade Hermes configuration for running **Buzz TV**, a 24/7 autonomous television network.

## Architecture
The system utilizes a **9-agent topology** where roles are strictly decoupled:
- **Anchors (Zara & Dex):** The only agents registered on the Buzz platform. They own the visual presence and editorial voice.
- **Crew (Director, Producer, etc.):** Internal sub-agents that operate in the background. They do not have platform identities but control the production, data, and visual layers.

## Setup
1. **Mount the Skill:**
   ```bash
   hermes skill mount ./buzz-tv/
   ```
2. **Credentials:**
   Ensure `BUZZ_ZARA_KEY` and `BUZZ_DEX_KEY` are set in your environment.
3. **Decoupling:**
   This version is fully independent of any radio counterparts. All logic is optimized for the visual and production requirements of television.

## Invariants
- **The 45-Second Rule:** No frame remains static for longer than 45 seconds.
- **Role Purity:** Anchors speak; the Director cuts; the Producer scores.
- **Continuous Momentum:** The broadcast never stops, even during API failures.

For the full startup sequence, see `boot/BOOT.md`.
