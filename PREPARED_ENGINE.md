# What is prepared before the Atari runs

The runtime has fixed actor limits and fixed glyph destinations. ROM supplies already aligned artwork and already resolved commands. The game retains live input, collision responses, state changes, scoring and bounded memory copies.

| System | Prepared on the build machine | Work in play |
|---|---|---|
| Flicky and cats | Color planes, facing, animation frames, horizontal fit | Select pose; copy only changed planes into the inactive hardware buffer; change positions at presentation |
| Moving chicks | Every pose, variant, facing, four horizontal and four vertical phases | Select and copy a four-character stamp |
| Props | 2,880 kind/rotation/flip/facing/alignment variants, deduplicated to 2,744 stamps | Select and copy a stamp; floor props are painted during loading |
| Scenery collision | Every 4×4 solid-tile neighbourhood for 48 maps; rectangle and eight-probe masks | Index two RAM byte planes and apply a mask; reuse unchanged player samples |
| Cats | Jump/fall/death displacement sequences and poses | Advance a sequence; decide at tile boundaries and timed pursuit points |
| Thrown props | Gravity successor, integer displacement and fractional remainder tables | Table lookups, position additions, bounce and hit checks |
| Screen writes | Row-specific store programs with wrapping addresses resolved | Write prepared character codes and remember overwritten background cells |
| Sound | 2,784 event programs for 19 commands | Execute immediate register-store commands and wait counters |
| Follower trail | Fixed-size history and prefix-sum representation | Record the live player path; calculate distances when the flock consumes them |

The player path cannot be fully pre-recorded because the player controls it. The 60-ish Hz physics and input clock remain live. Expensive image shifting, scaling, pixel priority, dynamic glyph allocation, general prop integration and audio-stream decoding are absent from the normal moving-actor path. The original general compositor remains for cold floor-object preparation and the low-load bonus net.

## Memory and presentation

Two ANTIC character/font buffers and two hardware-sprite buffers are used. Foreground code prepares the inactive buffers. The vertical interrupt flips completed buffers and updates hardware positions; it does not copy full sprites. Seven character bands provide scenery and actor glyph capacity. The display list uses hardware horizontal scrolling, with coarse pointers updated only when the coarse column changes.

The first gameplay font address is prepared during vertical blank. Each display-list interrupt writes the prepared CHBASE value immediately after WSYNC, then prepares the successor. This fixes a one-scanline delay at the HUD/playfield boundary that could draw a row of stray dashes. `tools/raster_probe.py --check` installs contrasting diagnostic font bands in emulator RAM and verifies their actual NTSC and PAL scanline boundaries with ANTIC and player/missile DMA enabled.

Small moving actors have fixed four-character slots. Dirty cell records restore the old name-table entries on the next use of that buffer. Still chicks use prepared animations containing their scenery. This avoids per-pixel background composition during ordinary play, at the cost of character-cell priority for moving overlaps.

The SIC! cartridge holds 512 KB of code and converted data. Common code runs in RAM. Large sound and idle-animation data use the upper cartridge window through short resident trampolines. All runtime game code is native NMOS 6502; there is no Z80 interpreter in this cartridge.

## Verification boundaries

Prepared geometry is compared with the retained original native scans across every map, every tile anchor, 98,304 eight-probe samples and 24,576 rectangle cases. Prop rendering is compared against converted source pixels for all 2,880 alignments and rotations. Player color planes, all cat poses, front-buffer isolation, bounded dirty lists, prop motion, cat walking/jumping/landing/death and rescue-door triggers have separate checks.

Connected tests exercise pickup, throwing, hits, chick scattering, recovery/rescue, scores, bonus catches, extra lives, death, restart, round progression and all 48 map loads. Native audio events are compared with the converted POKEY register sequence, including loops. The hardware model distinguishes GTIA's read-only PAL identity from its write-only color register at the same address.

The emulator stress test is deliberately artificial: it injects independently changing positions and poses before rendering. It is useful for workload and buffer tests, not as a demonstration of enemy intelligence. Its own injection work is included in the measured budget. A normal free-play capture is supplied separately. Neither test certifies a physical cartridge or a complete manual playthrough.
