# Flicky — prepared-engine Atari XL build

A playable native 6502 game for a stock 64 KB Atari 800XL, supplied as a 512 KB SIC! cartridge. This build replaces the slow general sprite compositor with prepared character stamps, hardware sprites, compiled sound events and terrain lookup tables.

## Play

Download [Flicky-Prepared-Game.car](Flicky-Prepared-Game.car). Watch the [45-second NTSC gameplay movie with POKEY sound](Flicky-Atari-XL-45s.mp4).

Load `Flicky-Prepared-Game.car` as a SIC! 512 KB cartridge on an Atari XL emulator configured for 64 KB, with BASIC disabled. Both PAL and NTSC are supported. Press fire to start. Move left/right; fire jumps and throws a carried prop. Collect chicks, bring the following flock to an exit, and recover birds scattered by cats. Props are picked up automatically. Bonus rounds use the catching net. Fire restarts after game over.

The connected game includes 48 round layouts, six scenery accents, up to eight following chicks, two pursuing cats, six floor props, thrown-object combat, rescue scoring, time bonuses, extra lives, bonus catches, life restart, game over, a HUD and POKEY music/effects. Best score lasts until reset.

The current cartridge fixes the stray horizontal dashes caused by a font switch landing one scanline late at the HUD/playfield boundary. The first font address is now prepared during vertical blank. Real Atari800 raster checks verify all seven font-band boundaries in both NTSC and PAL; see `evidence/raster-before.json` and `evidence/raster-after.json` in the source archive.

## Performance and compromises

Gameplay pictures are presented every two host refreshes: approximately 29.96 FPS NTSC and 24.93 FPS PAL. Player controls and physics use the original rational arcade clock on either television standard. Cats use prepared two-tick motion; follower poses and collision decisions use a lower update rate. Loading a round and each rescue have intentional pauses.

The measured stress fixture continually changes eight chicks, two cats and two airborne props, while the normal player, collision, audio and update scheduling run. The acceptance check requires bounded logic backlog as well as display cadence. See `evidence/` inside the source archive for measured results, not just CPU estimates. Measurements use normal-speed Atari800 with ANTIC DMA on a 64 KB XL configuration. Physical Atari hardware has not been tested.

Flicky retains blue, white and gold. Cats are compact monochrome hardware sprites. Moving chicks and props use four prepared character cells with two-scanline vertical positioning; overlapping moving characters use priority, and can obscure small pieces of scenery. Wide poses are reduced offline. Cat pursuit is deliberately simpler than the arcade algorithm. This build has two cats, no chameleon, and one-player play. It is not an arcade-perfect port or a claim of superiority on every measure. Automated checks cover all 48 maps and connected game systems; they are not a manual 48-round playthrough.

## Rebuild the playable cartridge

Requires Python 3.12+, NumPy, and cc65 (`ca65`, `ld65`). The converted artwork, maps, sound events and generated data needed for the build are included. Original executable game ROMs are not needed for this path and are not included.

Download and extract [Flicky-Prepared-Game-Source.zip](Flicky-Prepared-Game-Source.zip), which contains the complete source, converted assets, tests, and evidence. Then:

```sh
unzip Flicky-Prepared-Game-Source.zip
cd Flicky-Prepared-Game
python3 tools/build_native.py --prepared
```

Output: `build/native/flicky-native-dev.car`. `CA65` and `LD65` may specify full tool paths. The source package was unpacked and rebuilt independently, and its cartridge hash was compared with the delivered cartridge.

Portable gameplay, geometry, rendering and native sound checks require a C compiler, pytest and NumPy:

```sh
python3 -m pytest tests/test_prepared_engine.py tests/test_prepared_logic.py tests/test_compromise_game.py tests/test_audio.py -q -k 'not False'
```

The SG binary-identity check skips when the separately owned reference ROM is absent. Legacy differential tests and `--convert-original` additionally require the original reference inputs; they are retained as development tools and are not needed to rebuild this release. Old general pixel-compositor checks do not describe the new renderer.

To build the separate stress cartridge:

```sh
FLICKY_BENCHMARK=1 python3 tools/build_native.py --prepared
cp build/native/flicky-native-dev.car build/native/flicky-stress-test.car
python3 tools/build_native.py --prepared
python3 tools/prepared_probe.py --stress --seconds 60
```

The probe additionally needs Atari800, Xvfb, python-xlib and Pillow. `tools/atari800.cfg` maps the arrow keys and right Control for the automated emulator run. The stress injector is absent from the delivered game cartridge.

`python3 tools/raster_probe.py --check` verifies the actual raster boundaries. `python3 tools/capture_movie.py` records 45 seconds of unmodified NTSC gameplay and emulator-generated POKEY audio; it additionally requires FFmpeg.

See `PREPARED_ENGINE.md` for the runtime/data split and `SG1000_STUDY.md` for the local disassembly findings that informed the bounded actor design.
