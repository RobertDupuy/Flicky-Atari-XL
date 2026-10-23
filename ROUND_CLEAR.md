# Round Clear portrait

The normal-round result screen now draws an original large Flicky portrait at
native Atari resolution. It does not rescale or reuse the small gameplay
sprite. The eyes, crown feathers, raised wings, white bib, beak, toes and
feather strokes are drawn separately in `tools/round_clear_art.py`.

The display uses 40 columns across 224 scanlines. ANTIC 4 supplies 160 color
pixels across the portrait, and ANTIC 2 supplies 320-dot lettering above and
below it. Seven prepared character sets occupy 7,168 bytes. ANTIC 4's fifth
color is selected per character for gold details; small conflicting blue
areas are assigned the neighboring blue shade during preparation.

The four player masks provide a curved navy body shadow, cheek colors and
warm beak shading. Four missile masks provide feather, cheek and toe details.
GTIA priority keeps the white and light-blue playfield in front of these
overlays. Gold and blue pixels sit behind them. The background is black.
PAL and NTSC use different blue hue values, checked with Atari800's standard
palettes. Physical televisions and user palette settings can differ.

Twenty-seven short display-list interrupts change colors and fonts at row
boundaries. Their values are immediate operands in prepared RAM code; the
color and font writes occur together at the left border. The ordinary
gameplay interrupt retains its existing timing. Portrait code lives at
`$4000` while the game is paused, safely outside either SIC! cartridge window.

Entering the screen loads prepared data with DMA disabled. While it is
visible, there is no actor rendering loop. The normal input, rational timer
and POKEY audio tail continue at vertical blank. The earned time bonus is
written in the high-resolution footer. After the existing 240 logical ticks,
the next round reloads gameplay graphics, restores the ordinary NMI vector,
palette and player/missile state, and resumes play.

## Verification

- Tests check every possible time award, exact loaded font/PMG bytes, the NMI
  vector, next-round map and restored gameplay mode.
- An interrupt sweep checks the portrait VBI at every cycle of an upper-ROM
  glyph transfer, including cartridge/shadow-register writes.
- `tools/round_clear_probe.py` captures both television standards in Atari800,
  then checks that the cartridge advances to the next round with music active,
  no renderer fault and no input-clock overflow.
- Normal gameplay cadence and the ordinary seven font-band boundaries are
  checked separately, with ANTIC DMA enabled.

The portrait preview is a real emulator capture. The diagnostic selects the
result state before capture; it is not presented as a human-played level or
as a physical Atari hardware photograph. Original executable ROMs are not
required to build this artwork.
