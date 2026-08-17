# AlpineFall Support

This repository is the support hub for **AlpineFall**.

Use it to:
- raise issues about bugs, crashes, gameplay balance, performance, and platform/device problems,
- review and discuss reported problems,
- triage issues with labels, severity, and reproducibility details,
- track fixes from report to resolution.

## How to raise an issue

When opening an issue, please include:
- platform and hardware details,
- game version/build,
- clear reproduction steps,
- expected vs actual behavior,
- logs/screenshots/video (if available).

The more detail you provide, the faster a report can be reproduced, triaged, and fixed.

---

## AlpineFall — Game Brief

### What It Is

**AlpineFall** is a physics-driven alpine ski-and-freestyle game built in Unity that simulates skiing the way it *actually* works — energy, edges, gravity, and consequence — rather than bolting arcade controls onto a pretty mountain. You carve real turns, charge tree-lined pistes, and throw spins and flips off kickers where landing quality is the score, not a button combo.

A curated run catalog carries you from a forgiving nursery bowl up to a double-black couloir, each run authored with gradient, relief, vegetation, surface conditions, weather, time of day, objectives, and medal thresholds — all rendered under HDRP with volumetric snow, wind, spindrift, and weather that can white you out mid-descent.

### The Pitch

> **The mountain doesn't care. Ski it anyway.**

AlpineFall is a love letter to the sport, not a score-attack. Every crash is earned, every stomped landing is *yours*. Feel the difference between groomed corduroy and wind-packed powder under your skis. Link clean carves to hold your line, tuck for the hairpins, and risk a misty 720 off a booter — but land it, because a flat-backed slam from the *same* jump is a different run entirely.

- **Nine hand-built runs** with more to come across Green → Freestyle → Blue → Black → Double Black, from Sunnegg's nursery bowl to the Nordwand Couloir.
- **Consequence-based injury** — damage comes from *absorbed energy*, not speed. Ski 120 km/h safely on smooth piste; catch an edge at 30 and feel it.
- **Freestyle that respects physics** — rotation, hang time, and landing absorption score the trick. Over-rotate into a wipeout and you score nothing.
- **Living alpine weather** — bluebird, golden hour, overcast, night skiing, and full whiteout heavy snow, cross-fading as you descend.

### Technical Details

**Engine & rendering** — Unity **6000.5.8f1** (Unity 6), **HDRP** with three pipeline presets (Performant / Balanced / High Fidelity), procedural winter-environment authoring.

**Deterministic simulation core** — Game logic lives in a pure, allocation-light `Alpinefall.Simulation` namespace with no Unity dependencies, driven by a **`double`-precision** vector type (`Double3`) and a custom hashed `DeterministicRandom` (PCG-style `xorshift`/`murmur3`-ish multiply-xor mixing). This makes terrain, physics, and scoring reproducible — a foundation for deterministic tests and future replays/netcode.

**Physics model** (`SkiPhysics.cs`) — Closed-form force balance: gravity (`9.81 * sin(slope)`, with a slope-drive boost tapering off past ~50°), aerodynamic drag, snow resistance, lateral skid resistance, braking, and carving quality. Terminal speed is solved by 100-iteration bisection.

**Trick & landing system** (`TrickSystem.cs` / `SkiJumpPhysics`) — Rotation, flip, off-axis (Cork/Misty), grabs, and shifties are recognized from net angular displacement (±40° landing window); landing quality folds in rotation error, alignment, impact speed, and grab release.

**Health & injury** (`Health.cs`) — Damage derives from joules absorbed with a 2400 J free allowance and 190 J per point; crash-cause multipliers (tree ×1.9, rock ×1.45) and fatigue/stun degrade control authority dynamically.

**Terrain pipeline** — Every run is a pure `RunDefinition` (seed, gradient points, piste wander via 3 harmonics, relief meso/macro/micro, forest bands, glades, cliffs, surface bands) compiled into geometry by `RunTerrain`/`AlpinefallWorldBuilder`, with an **edit-mode test suite** (`Tests/EditMode/…`) covering parity between simulation and rendering paths.

**Weather** — Seven authorable `WeatherState` presets, fully cross-fadeable (`Lerp` on every field), with independent time-of-day sun rotation and late-weather blending over a run's bottom third.
