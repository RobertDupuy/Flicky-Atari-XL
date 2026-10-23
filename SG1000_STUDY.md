# What the SG-1000 binary changed in this port

The examined file is the actual 32,768-byte SG-1000 v0 ROM, CRC32 `BD24D27B`, MD5 `b52e308e1c0005bc1dba2ba900ad44d0`. The conclusions below come from disassembling that binary, not from descriptions of the game or the Coleco conversion. `python tools/study_sg.py` reproduces the annotated instruction excerpts and their manifest in `build/sg-study/`. The full linear decode is also retained in `build/sg1000-disassembly.txt`; unlike the selected excerpts, that raw decode includes data interpreted as instructions.

| Binary evidence | What it does | Decision in the Atari engine |
|---|---|---|
| `$0219–$0299`, `$029B–$02A7`, `$0425–$045B` | Interrupt work and the main play loop communicate through a frame byte; major systems are separate bounded calls. | Keep input/music timing separate from a display submission. No Z80 execution in the cartridge. |
| `$05C0–$062C`, `$1D12–$1D31` | Scroll uses an eight-position fine phase and a 32-column coarse coordinate. The fine phase indexes a small table of shifted tile codes. | Keep ANTIC hardware scroll, existing tile maps, and prepared character animation. Do not simulate the arcade display mixer. |
| `$1C1B–$1C4B`, `$10EE–$1119` | A point offset from a sprite is divided into character coordinates. Chick pickup checks the resulting character number against a small range. | Use explicit world-space pickup/hit boxes and solid-tile probes. Visual occlusion does not decide who was hit. |
| `$10CA–$10ED` | Eight chicks; four-byte sprite entries and three-byte state entries. | Bound the flock to eight; retain the already implemented arcade-style delayed trail and scattered-bird recovery. |
| `$1539–$1570`, `$1591–$161B` | Six object slots, a short state dispatcher, a carried-object latch, and a timed throw. | Implement six floor/carried/projectile objects. Reuse all fifteen earlier Atari object types, rotation art, cat hit animation, and scoring. |
| `$1C9D–$1D11`, `$210F–$215E` | The map is traversed as twenty rows of sixteen bytes, with two cell classes per byte. | Collision decisions stay in a small tile map, independent of sprite pixels. Keep the richer converted Atari scenery where it fits. |
| `$0357–$0361` | The layout counter resets when it reaches 41. | Retain the existing Atari sequence of 48 rounds, including its bonus-round positions. |

The SG code makes the simplifying rules explicit. Its point probes are not a justification for reproducing pixel-perfect arcade collision ownership more efficiently. They show that the ownership problem can be removed from the game engine.

The prepared Atari renderer restores changed cells and copies aligned four-character stamps into fixed glyph slots. Hardware sprites draw Flicky and the cats. Stationary chicks have staggered poses. Floor objects are pinned once per buffer and restore their saved background when picked up. The general mask/ink compositor is used during floor-prop loading and for the bonus net. These are Atari-specific choices, not claims about how the SG implementation draws every actor.

The Atari build adds a title and instructions, five-color scenery and actors, a live score/best-score/lives display, native POKEY sound and throwable-object combat. It preserves the earlier port's movement, map conversion, chick trails, cat spawning, death/restart, and stage progression. Prepared bounded pursuit replaces the old normal-round cat decision loop. The bonus has since been replaced with an implementation based directly on the original arcade ROM, including source-resolution sprite collision ownership; see `ARCADE_BONUS.md`. The additional chameleon enemy and two-player mode are outside this one-player compromise build.

This is a static study of the listed routines and data structures. It is not a claim that the entire SG binary has been dynamically traced or translated.
