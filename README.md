# MeshSense (mricho fork)

A personal fork of [Affirmatech/MeshSense](https://github.com/Affirmatech/MeshSense) with extra features for antenna placement and link debugging.

**For everything about MeshSense itself — installation, headless usage, dev setup, FAQ — see the [upstream README](https://github.com/Affirmatech/MeshSense/blob/master/README.md).** This file only documents what's different in the fork.

## What this fork adds

- **1-minute minimum traceroute interval.** Upstream floors "Traceroute Rate Limit (Minutes per Node)" at 15 min; this fork floors it at 1 min (the firmware's actual rate limit).
- **Route-back + per-hop SNR in the log.** RouteDiscovery rows now show both directions and the dB value for each hop, e.g. `NBDY -(8.50dB)-> ?64ae0d19 -(5.25dB)-> !ac1cd197 | back: !ac1cd197 -(6.00dB)-> NBDY`.
- **Watch mode.** A 👁 toggle next to the traceroute button (`↯`) on each node row. Continuously traceroutes that node at the configured rate-limit interval until toggled off. One node at a time, server-side, not persisted across restarts. Great for placing/aiming antennas while watching SNR drift in real time. API: `POST /watch { "destination": <nodeNum> | null }`.

---

## Instructions for AI assistants updating this README

When asked to update this README:

1. **Do not document upstream MeshSense.** No install steps, headless usage, dev setup, dependencies, screenshots, or general MeshSense features. Those belong in the upstream repo and the link to it stays at the top of this file.
2. **Only describe what this fork adds or changes** vs. `Affirmatech/MeshSense`. Each entry goes in the bullet list under "What this fork adds" — short, one paragraph max, with a concrete example or API surface where helpful.
3. **Keep the file short.** Header, one-paragraph intro, upstream link, bullet list, this instructions section. That's it. If the bullet list grows past ~10 items, group them under sub-headings rather than expanding each bullet.
4. **Don't restore upstream content** that has been intentionally removed, even if a section feels "missing." If you think something genuinely belongs here that isn't a fork-specific change, ask before adding it.
5. **Preserve this instructions section** verbatim unless explicitly told to change it.
