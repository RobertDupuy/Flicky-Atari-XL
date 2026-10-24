# Flicky — prepared-engine Atari XL build

Current update: plain black decorative wall backgrounds except rounds **20–22**, plus restored **bonus-round jumping**, verified against the original MAME ROM. Actor colors, room geometry, the full intro animation and the smaller Round Clear portrait are retained. The [updated gallery covers all 48 rounds in PAL and NTSC](Flicky-All-48-Rounds.html). See [ARCADE_BONUS.md](ARCADE_BONUS.md) and [ATTRACT_AND_VISIBILITY.md](ATTRACT_AND_VISIBILITY.md) for implementation and validation details.

A playable native 6502 game for a stock 64 KB Atari 800XL, supplied as a 512 KB SIC! cartridge. This build replaces the slow general sprite compositor with prepared character stamps, hardware sprites, compact sound events and terrain lookup tables.

## Play

Download the **[NTSC cartridge](Flicky-Atari-XL-NTSC-512K.car)** or **[PAL cartridge](Flicky-Atari-XL-PAL-512K.car)**. Both are 512 KB SIC! cartridges. The [PAL bonus preview](Flicky-Atari-XL-PAL-Arcade-Bonus-Preview.mp4) shows the new bonus implementation with POKEY sound. Round 3 is selected before recording; subsequent play uses ordinary keys in real time.

Load the cartridge matching your television standard as a SIC! 512 KB cartridge on an Atari XL emulator configured for 64 KB, with BASIC disabled. Use the PAL build on PAL and the NTSC build on NTSC. The clock still detects the hardware standard; the separate builds select the prop policy and presentation palettes. Press fire to start. Move left/right; fire jumps and throws a carried prop. Collect chicks, bring the following flock to an exit, and recover birds scattered by cats. Props are picked up automatically. During bonus rounds, **fire jumps with the net**; release and press again for another jump. Fire restarts after game over.

The connected game includes 48 round layouts, six scenery accents, up to eight following chicks, two pursuing cats, six floor props, thrown-object combat, rescue scoring, time bonuses, extra lives, bonus catches, life restart, game over, a HUD and POKEY music/effects. Best score lasts until reset. Room messages use the source wording GATHER UP PIOPIO, DEFEAT NYANNYAN, and TAKE PIOPIO HOME. They are placed in non-solid cells clear of stationary chicks and floor props; collision maps are unchanged. Music keeps advancing under ordinary sound effects and resumes after temporary jingles; game over intentionally stops it.

Bonus rounds now reproduce the arcade ROM's twenty independent birds, alternating entrance groups, changing flight patterns, two animated seesaws, bouncing cats and catching net. They appear on rounds 3, 7, 11 and every fourth round thereafter. Each catch awards **250 points** on completion; all twenty add **10,000**, for **15,000** total. Misses do not cost lives. The original catch tally, lettering and result layout remain over the playfield. Catch detection uses full-resolution source silhouettes, sprite overlap ownership and latched collision flags; ascending chicks cannot be caught. See [ARCADE_BONUS.md](ARCADE_BONUS.md) for the original-ROM analysis and verification boundaries.

The 512 KB cartridge keeps all 48 layouts and all sound commands, and adds prepared static room messages and bonus/results screens. Padded ink/mask columns share vertical animation phases; fixed lookup tables select the prepared addresses.

Bonus presentation now uses hand-drawn **5×7 Atari lettering**, red seesaw beams, shaded orange cats and a green-and-gold brick floor. Dedicated DLIs change the lower-screen colors while Flicky and the chicks keep their own colors. The arcade flight, catching and scoring rules are unchanged. The [PAL perfect-result capture](Flicky-Bonus-Colors-PAL.png) and [NTSC capture](Flicky-Bonus-Colors-NTSC.png) are emulator display diagnostics with the result selected for inspection.

Round Clear has original artwork drawn for a dedicated **160-color-pixel-wide display**, with **320-dot lettering**, seven character sets, DLI shading and player/missile color details. The portrait is now **60% of its previous width and height**; its paths are rasterized at that size and its hardware overlays repositioned to match. It does not enlarge the gameplay sprite. PAL and NTSC have separate portrait blues. The static screen suspends actor rendering while input, the result timer and POKEY audio continue; the next round restores the gameplay display. See [ROUND_CLEAR.md](ROUND_CLEAR.md) and the [PAL emulator capture](Flicky-Round-Clear-PAL.png).

The current cartridge also fixes the stray horizontal dashes caused by a font switch landing one scanline late at the HUD/playfield boundary. The first font address is now prepared during vertical blank. Real Atari800 raster checks verify all seven font-band boundaries in both NTSC and PAL; see `evidence/raster-before.json` and `evidence/raster-after.json`.

## Performance and compromises

Normal-round pictures are presented every two host refreshes: approximately 29.96 FPS NTSC and 24.93 FPS PAL. Bonus pictures appear at the next vertical blank after rendering; their rate varies with the twenty-bird workload and drops below normal-round cadence during crowded flights. Player controls and bonus physics use the original rational arcade clock on either television standard. Normal-round cats use prepared two-tick motion; follower poses and collision decisions use a lower update rate. Loading a round and each rescue have intentional pauses.

The measured stress fixture continually changes eight chicks, two cats and a selectable number of moving props, while the normal player, collision, audio and update scheduling run. NTSC uses one moving prop. PAL retains all six independent props: picking up another does not remove earlier projectiles. PAL holds its two-refresh cadence with three active props; the deliberately extreme all-six stress case drops to about 20 FPS. There is no three-prop cap. The acceptance check requires bounded logic backlog as well as display cadence. See `evidence/` for measured results, not just CPU estimates. Measurements use normal-speed Atari800 with ANTIC DMA on a 64 KB XL configuration. Physical Atari hardware has not been tested.

Flicky retains blue, white and gold. Cats are bright orange hardware sprites. Chicks use separate NTSC/PAL color values so both television standards show yellow/gold birds. Moving chicks and props use masked character cells with two-scanline vertical positioning: transparent pixels preserve the scenery and other chicks. ANTIC shares its fourth color per character, so a background accent can take the chick’s gold color within a shared cell. Six floor props remain in both builds. NTSC permits one held or airborne prop at a time; picking up the next prop retires the previous projectile. PAL leaves previous projectiles active, with independent motion, hits and expiry for all six props. The player carries one prop at a time in either build. Wide poses are reduced offline. Cat pursuit is deliberately simpler than the arcade algorithm. This build has two cats, no chameleon, and one-player play. It is not an arcade-perfect port or a claim of superiority on every measure. Automated checks cover all 48 maps and connected game systems; they are not a manual 48-round playthrough.

## Rebuild the playable cartridge

Requires Python 3.12+, NumPy, Pillow, and cc65 (`ca65`, `ld65`). The converted artwork, maps, sound events and generated data needed for the build are included. Original executable game ROMs are not needed for this path and are not included.

Download and extract [Flicky-Prepared-Game-Source.zip](Flicky-Prepared-Game-Source.zip), then run these commands inside its `Flicky-Prepared-Game` directory.

```sh
python3 tools/build_native.py --prepared --variant ntsc
python3 tools/build_native.py --prepared --variant pal
```

Output: `build/native/flicky-native-dev.car` (each build replaces this development output). `python3 tools/build_variants.py` writes both named cartridges into `release/`. `CA65` and `LD65` may specify full tool paths. The source package was unpacked and rebuilt independently, and its cartridge hash was compared with the delivered cartridge.

Portable gameplay, geometry, rendering and native sound checks require a C compiler, pytest and NumPy:

```sh
FLICKY_VARIANT=pal python3 -m pytest tests/test_prepared_engine.py tests/test_prepared_logic.py tests/test_compromise_game.py tests/test_stamp_background.py tests/test_native_sic.py tests/test_audio.py tests/test_presentation_audio.py tests/test_arcade_bonus.py tests/test_attract_delivery.py tests/test_bonus_jump_visibility.py -q
```

Run the same command with `FLICKY_VARIANT=ntsc` to check the NTSC prop policy. The PAL-specific test verifies that picking up and throwing the sixth prop preserves the other five and that expiry removes only that prop.

The SG binary-identity check skips when the separately owned reference ROM is absent. Legacy differential tests and `--convert-original` additionally require the original reference inputs; they are retained as development tools and are not needed to rebuild this release. Old general pixel-compositor checks do not describe the new renderer.

To build a separate PAL stress cartridge:

```sh
FLICKY_BENCHMARK=1 FLICKY_STRESS_PROPS=3 python3 tools/build_native.py --prepared --variant pal
cp build/native/flicky-native-dev.car build/native/flicky-stress-test.car
python3 tools/build_native.py --prepared --variant pal
python3 tools/prepared_probe.py --stress --mode pal --seconds 60
```

`FLICKY_STRESS_PROPS=6` exercises the unrestricted maximum; that deliberately excessive load is a capacity test and does not meet every two-refresh deadline. The stress injector is absent from both delivered game cartridges. Probe checks additionally need Atari800, Xvfb, python-xlib and Pillow. `tools/atari800.cfg` maps arrow keys and right Control for automated input.

`python3 tools/raster_probe.py --check` verifies the actual NTSC and PAL raster boundaries. `tools/capture_arcade_bonus.py` selects round 3 before recording a diagnostic preview. During recording it uses read-only observation and ordinary keyboard input, with no state edits, pauses or speed changes. Catch counts, score and results come from the game. FFmpeg encodes the emulator's video and POKEY audio into the MP4. Older bonus movies show the superseded implementation.

See `PREPARED_ENGINE.md` for the runtime/data split and `SG1000_STUDY.md` for the local disassembly findings that informed the bounded actor design.
