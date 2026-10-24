# Attract animation, scenery visibility and delivery awards

The 512 KB PAL and NTSC cartridges replace the static title with the original
arcade motorcycle, balloons, airplane, bombs and FLICKY letter unscrambling
sequence. Research used the original uncoined MAME attract sequence; the Atari
runs prepared native data and handwritten 6502 code, with no arcade emulator.

The adaptation uses 160 by 224 color pixels, seven ANTIC character sets,
separate display buffers, scenery palette changes and four hardware players
multiplexed every eight scanlines. It contains 315 sampled poses. Decoding
holds each completed picture until the next one is ready; playback is slower
than the arcade (about 30 seconds PAL and 35 seconds NTSC). The supplied video
is an unretimed recording of the actual cartridge. Fire can start the game
while the animation plays or after the settled logo.

Scenery wall fills have been replaced with sparse dark patterns. Original tile
collision masks and room geometry are unchanged. Stationary chicks are baked
against the corrected backgrounds, and hardware players have priority over
all playfield colors so scenery cannot hide Flicky or the cats. All 48 numbered
rounds were captured in both video standards through the ordinary round loader.
The screenshot gallery uses diagnostic round selection, not played completion.
Bonus rounds occur every fourth round, beginning with round 3.

Delivery score actors now render native outlined digits. Original train awards
are 100, 200, 300, 400, 500, 1000, 2000 and 5000 points. The original positions,
four-actor history and lifetimes are retained; the last award is allowed to
finish before Round Clear. Values depend on the length of the delivered train,
so separate partial deliveries restart at 100.

Prepared builds need no original executable ROM. They include the converted
animation in build/native/attract-pixels.npz and corrected wall glyphs in
build/native/scenery-wall-glyphs.bin. The research-only tools can regenerate
those inputs when separately supplied original ROMs and MAME are available.

Validation covers every animation frame, player bitmap and raster program;
interrupts during stream decoding; buffer isolation; all eight delivery labels
at all four horizontal phases; music/effect continuity; the original bonus
physics and scoring; and all 48 rooms. Emulator probes also check startup,
fire-to-start, bonus DLI boundaries and return from Round Clear.
