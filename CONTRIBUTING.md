# Contributing to Buzz TV

---

## What We're Looking For

**High value:**
- Runtime implementations — actual Python/JS from RUNTIME.md pseudocode
- ElevenLabs TTS integration for real anchor voices
- Stability AI integration for AI-generated visual assets
- New scene types (e.g. Crypto Trading Floor, Interview Setup, Reaction Cam)
- New segment formats (e.g. Tech Desk, Interview, Prediction Recap)
- Additional data sources (Reddit, YouTube trending, additional sports leagues)
- Field reporter / guest correspondent personas
- Music API integration for real DJ breaks

**Not a good fit:**
- Changes that make anchors sound like assistants
- Removing the editorial transform layer (raw data must never reach anchors)
- Bypassing the Director for visual decisions (anchors don't control their own camera)
- Bypassing the Producer for story selection (score everything)
- Reducing the 9-agent topology without replacing with equivalent capability

---

## PR Checklist

- [ ] All 13 skill modules present and unbroken
- [ ] CI passes (validate.yml)
- [ ] `.env.example` updated if new env vars added
- [ ] New agents added to AGENT_ROSTER in RUNTIME.md
- [ ] New scenes added to SCENE_MAP in SCENES.md
- [ ] New segments have both audio spec AND visual spec in SEGMENTS.md
- [ ] New graphic types follow the 3-simultaneous-maximum rule
- [ ] The Director still controls all visual decisions
- [ ] The Producer still scores all stories before they air
- [ ] Anchors still sound like Zara and Dex — not like assistants

---

## Module Ownership

| Module | Downstream impact |
|--------|------------------|
| `ANCHORS.md` | All anchor prompts |
| `CREW.md` | Agent registration, turn sequences |
| `PROGRAMMING.md` | Segment durations, schedule, triggers |
| `SEGMENTS.md` | Turn sequences, data requirements, visual specs |
| `SCENES.md` | Director scene selection, transition matrix |
| `DIRECTOR.md` | Director loop, all visual decisions |
| `GRAPHICS.md` | Graphics queue, overlay rules |
| `PIPELINE.md` | Context object shape, data freshness |
| `STATE.md` | Memory objects, state machine |
| `MODERATION.md` | Prompt constraints for all agents |
| `TEMPLATES.md` | Every LLM call |
| `RUNTIME.md` | The execution loop — touch carefully |

---

## Questions

Open an issue with the `question` label.
