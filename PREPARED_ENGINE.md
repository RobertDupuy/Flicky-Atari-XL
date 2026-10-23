# What is prepared before the Atari runs

The runtime has fixed actor limits and fixed glyph destinations. ROM supplies already aligned artwork and already resolved commands. The game retains live input, collision responses, state changes, scoring and bounded memory copies.

| System | Prepared on the build machine | Work in play |
|---|---|---|
| Flicky and cats | Color planes, facing, animation frames, horizontal fit | Select pose; copy only changed planes into the inactive hardware buffer; change positions at presentation |
| Moving chicks | Every pose, variant, facing, four horizontal and four vertical phases | Look up aligned ink/mask addresses and occupied cells; copy or merge at most four characters |
| Props | 2,880 kind/rotation/flip/facing/alignment variants sharing padded columns | Use the same bounded mask path; floor props are painted during loading |
| Scenery collision | Every 4×4 solid-tile neighbourhood for 48 maps; rectangle and eight-probe masks | Index two RAM byte planes and apply a mask; reuse unchanged player samples |
| Cats | Jump/fall/death displacement sequences and poses | Advance a sequence; decide at tile boundaries and timed pursuit points |
| Thrown props | Gravity successor, integer displacement and fractional remainder tables | Table lookups, position additions, bounce and hit checks |
| Screen writes | Row-specific store programs with wrapping addresses resolved | Write prepared character codes and remember overwritten background cells |
| Sound | 19 exact POKEY event streams, 11,986 bytes | Read a duration and at most eight changed register values |
| Arcade bonus | Original trajectory data, full-height masked bird stamps, nine apparatus backgrounds, source-resolution contact silhouettes and result patches | Advance all twenty birds on the arcade clock; resolve visible collision owners and latched contacts; render the net and changed background |
| Round Clear | Original portrait paths, seven character sets, wide screen map, four player and four missile masks, immediate DLI programs | Static DMA; row color/font changes; timer and audio continue without actor rendering |
| Follower trail | Fixed-size history and prefix-sum representation | Record the live player path; calculate distances when the flock consumes them |

The player path cannot be fully pre-recorded because the player controls it. The 60-ish Hz physics and input clock remain live. Expensive image shifting, scaling, pixel priority, dynamic glyph allocation, general prop integration are absent from the normal moving-actor path. The original general compositor remains for cold floor-object preparation and the low-load bonus net.

## Memory and presentation

Two ANTIC character/font buffers and two hardware-sprite buffers are used. Foreground code prepares the inactive buffers. The vertical interrupt flips completed buffers and updates hardware positions; it does not copy full sprites. Seven character bands provide scenery and actor glyph capacity. The display list uses hardware horizontal scrolling, with coarse pointers updated only when the coarse column changes.

The first gameplay font address is prepared during vertical blank. Each display-list interrupt writes the prepared CHBASE value immediately after WSYNC, then prepares the successor. This fixes a one-scanline delay at the HUD/playfield boundary that could draw a row of stray dashes. `tools/raster_probe.py --check` installs contrasting diagnostic font bands in emulator RAM and verifies their actual NTSC and PAL scanline boundaries with ANTIC and player/missile DMA enabled.

Small moving actors have fixed four-character slots. Dirty cell records restore the old name-table entries on the next use of that buffer. Still chicks use prepared animations containing their scenery. Moving actors preserve existing pixels using precomputed transparency masks. Empty cells are skipped; empty backgrounds use direct copies; cells containing only gold pixels use OR operations. Background glyph addresses and empty flags are prepared per font band during loading. A shared character can tint a scenery accent gold because ANTIC selects its fourth color per character. There is no runtime shifting or per-pixel geometry calculation.

The SIC! cartridge holds 512 KB of code and converted data. Common code runs in RAM. Sound, sprite columns and idle-animation data use the upper cartridge window through short resident trampolines. All runtime game code is native NMOS 6502; there is no Z80 interpreter in this cartridge.

## Verification boundaries

Prepared geometry is compared with the retained original native scans across every map, every tile anchor, 98,304 eight-probe samples and 24,576 rectangle cases. Every chick pose and X/Y alignment is checked over patterned scenery, another chick, a font-band boundary and the horizontal wrap, including restoration. Prop rendering is compared against converted source pixels for all 2,880 alignments and rotations. Player color planes, all cat poses, front-buffer isolation, bounded dirty lists, prop motion, cat walking/jumping/landing/death and rescue-door triggers have separate checks.

Connected tests exercise pickup, throwing, hits, chick scattering, recovery/rescue, scores, bonus catches, extra lives, death, restart, round progression and all 48 map loads. Native audio events are compared with the converted POKEY register sequence, including loops. The hardware model distinguishes GTIA's read-only PAL identity from its write-only color register at the same address.

The emulator stress test is deliberately artificial: it injects independently changing positions and poses before rendering. It is useful for workload and buffer tests, not as a demonstration of enemy intelligence. Its own injection work is included in the measured budget. A normal free-play capture is supplied separately. Neither test certifies a physical cartridge or a complete manual playthrough.

The NTSC moving-prop budget is one held or airborne object. Acquiring another retires the previous projectile; all six original floor placements remain. The separate PAL build removes that retirement call: all six objects have independent motion, collision and lifetime state. The one-minute PAL test with three projectiles holds host/2; the all-six synthetic fixture is reported separately because it can miss deadlines. This is a measured performance boundary, not a cap on PAL gameplay. The main loop also gives queued gameplay ticks priority when the backlog reaches three, preventing drawing from starving the input clock.

Static room lettering is copied from prepared ROM blocks into both background/font buffers at load time. It reserves glyphs below the moving-actor pools and leaves collision maps untouched. Bonus lettering uses native 5×7 patterns spanning hardware character cells; its heading, tally and award rows remain over the playfield. A short original introduction delay precedes motion; the catch tally updates in place. Audio uses an independent looping music slot and a temporary effect slot; source reset commands no longer terminate the music. Normal rounds initialize their palette once. The bonus additionally uses a 512-byte raster program at `$1000–$11FF`: four lower-screen color boundaries distinguish red beams, shaded orange cats and the green-and-gold floor while preserving white and chick gold. VBI restores the upper palette before the shared interrupt path. Prepared fonts and raster code load inside the round-loading pause, before DMA/NMI are enabled.

The bonus uses a shared glyph pool beginning at 64 and retains all vertical source-pixel phases. Seesaws and cats are prepared together into nine background states, then chicks and the net are composited over them. The display presents each completed bonus picture at the next vertical blank. All twenty trajectories advance independently at the arcade clock rate even when drawing misses a refresh. The crowded bonus is not guaranteed to meet the normal host/2 presentation cadence. Original sprite silhouettes also decide collision ownership independently of ANTIC's reduced display resolution. See `ARCADE_BONUS.md` and the bonus evidence files.
