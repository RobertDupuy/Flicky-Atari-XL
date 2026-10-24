# Bonus rounds reconstructed from the original arcade ROM

This implementation replaces the earlier simplified bonus. The reference is the user's verified Sega System 1 `flicky` ROM set, with program ROMs `epr-5978a` and `epr-5979a`. The game was run in MAME 0.264 and its decrypted instructions were also executed in an independent Z80 harness. The Atari cartridge contains native 6502 code and converted data, not the original executable ROMs or a Z80 interpreter.

## What the original does

**Flicky can jump with the net.** This was confirmed by pressing Button 1 in
the original `flicky` set in MAME, then tracing the player routine at `$3537`
and controls at `$3816`. The earlier Atari bonus code omitted the vertical
player update; it now calls the existing ROM-verified jump and pose routines.
Fire starts a jump on a fresh press, holding fire does not repeat it, and
releasing the stick in the air retains horizontal momentum. The net follows
Flicky's vertical position and the original descending-bird catch rule remains.
Landing uses the bottom opaque pixels of the source animation poses.

Twenty chicks enter in five groups of four, alternating sides. The cats bounce on two seesaws below the catching platform. Each bird has its own fractional position, velocity, acceleration, launch state, flight clock and animation. The first four bonus rounds have different entrance spacing; later rounds change the horizontal flight program. Falling motion advances on alternate logic ticks. Birds that fall offscreen are missed without losing a life.

The net catches descending birds. Original sprite silhouettes decide contact at the arcade's horizontal resolution, independently of the Atari picture. Later sprite slots hide earlier birds where their pixels overlap. Hardware collision bits remain latched until the original scan clears them; the first pending collision ends that tick's scan even if the selected bird is ascending or already gone. The new native implementation reproduces these rules, including right-edge clipping.

Catches are tallied across the playfield. At the end, each catch earns 250 points. Catching all twenty adds 10,000 points, making a perfect round worth 15,000. The two seesaws, their mirrored animation, the bouncing cats and the net retain the arcade artwork and animation sequence.

## Atari lettering and color

The heading, score calculation and perfect-bonus award use hand-drawn **5×7 native color-pixel letters**, with a real one-pixel gap between letters. A letter can cross hardware character boundaries; it is not an eight-pixel arcade glyph squeezed into four color pixels. The twenty tally icons are drawn separately. All font and text patches are prepared in ROM and copied at transitions, with no per-frame text rasterization.

Bonus-only display-list interrupts distinguish red seesaw beams, orange cats and a green-and-gold brick floor. The original floor joints are recovered from the graphics data. Cat eye pixels stay white and pale fur stays orange. Four color boundaries at playfield lines 176, 192, 208 and 216 provide the lower-screen palette and shading; white and chick gold remain constant. Each VBI restores the upper playfield palette before the shared buffer-flip, input and audio code runs.

The 512-byte bonus raster program runs in `$1000–$11FF`, outside both SIC! cartridge windows. Font switches retain their WSYNC deadline. The ordinary NMI vector and display-list flags are restored on the next normal round. The original flight, net movement, collision ownership, catch rules and score calculation are unchanged.

| Original address | Purpose | Native implementation |
|---|---|---|
| `$145E–$159D` | Four twenty-bird starting tables | Converted `ab_initial` data |
| `$159E–$1606` | Flight-program pointers and steps | Converted pattern tables |
| `$4060–$414F` | Bird entry, launch, fractional flight and poses | `bonus_bird_step` |
| `$4152–$41F1` | Net position, latched collisions and catches | `bonus_catch` and ownership masks |
| `$41F4–$4247` | Seesaw phase and facing | Nine prepared apparatus backgrounds |
| `$52CC` | Bouncing bonus cats | Prepared cat pose and height cycle |
| `$0573–$068D` | Catch score and result lettering | Native BCD score and prepared result patches |

## Verification

`tools/check_bonus_player.py` executes the source player and movement routines
with floor contacts from the independent source sprite renderer. Its 943-state
fixture covers standing jumps, held fire, release/repress and left/right moving
jumps in the central playfield, comparing sixteen player/movement fields with
zero mismatches. This establishes the tested trajectories, not every possible
screen-edge interaction. The fixture is included for ROM-free regression tests.
`tools/bonus_jump_probe.py` checks ordinary fire presses in both real-time
Atari800 variants, including a long hold through landing and a second jump.

`tools/check_arcade_bonus.py` executes the original bird routine and compares every native state field and animation selection. Sixteen round selections, covering every pattern and later reuse, produce **288,400 actor states with zero mismatches**. The resulting golden data is included so regression tests do not require the private ROMs.

`tools/check_bonus_collision.py` independently renders the original sprite descriptors through the System 1 reference renderer and executes the original net routine. It checks **608 individual contacts**, **1,824 ascending/stationary rejections**, and **1,680 crowded or latched-collision cases**, including screen edges. These checks report zero mismatches. A second golden fixture preserves the crowded cases for ROM-free tests.

`tests/test_arcade_bonus.py` also checks the converted bird pixels over patterned backgrounds, hardware-player uploads, net attachment, every score from zero through twenty catches, native lettering and tally pixels in both buffers, and rendering of every bonus pattern without glyph overflow or front-buffer damage. Bonus VBI interruption is injected at 189 positions in an upper-ROM glyph copy, followed by a check that the next round restores the ordinary raster. The complete PAL and NTSC test reports accompany the source package.

`tools/bonus_presentation_probe.py` selects all nine apparatus states and the perfect result in Atari800, then checks real GTIA palette boundaries with ANTIC/PMG DMA active. These selected-state screenshots are display diagnostics. They are separate from the continuous ordinary-input bonus preview.

The real-time emulator checks select round 3 before recording, then use ordinary keyboard input with read-only observation. No state is edited and the emulator is not paused during the recording. These are diagnostic bonus previews, not evidence of reaching the bonus through ordinary level progression. Their timing reports include render faults and input-queue overflow.

## Remaining fidelity limits

The tests establish the listed behavior; they do not certify every scanline of a full MAME session. Atari ANTIC uses reduced horizontal resolution and a smaller palette. Source silhouettes retain full collision resolution, but their displayed pixels are converted for Atari graphics. The existing POKEY music conversion is also different from the arcade sound hardware.

All twenty birds continue on the original rational arcade logic clock. Drawing is slower during crowded flights, so the bonus presents each completed picture at the next vertical blank and does not guarantee a fixed 25/30 FPS. Normal-round pacing and the separate NTSC/PAL prop policies remain unchanged. This is a substantial arcade-reference implementation, not a claim that the entire Atari port is arcade-perfect.

## Reproduce

The normal build uses included converted assets and requires no original ROMs:

```sh
python3 tools/build_variants.py
FLICKY_VARIANT=pal python3 -m pytest tests/test_arcade_bonus.py -q
```

The research checks require the separately supplied, verified ROM data under `private-roms/`. `tools/arcade_bonus_mame.lua` provides a reproducible MAME research fixture; it selects the bonus before sampling and is not a gameplay replay. `tools/arcade_bonus_assets.py` additionally uses the palette snapshot captured by `tools/run_reference.py` at `build/arcade-bonus/original-600.bin`. None of these private ROM or full-memory inputs are distributed in the source package.

Hardware reference: [MAME Sega System 1 driver](https://github.com/mamedev/mame/blob/master/src/mame/sega/system1.cpp) and [MAME Lua device API](https://docs.mamedev.org/luascript/ref-devices.html).
