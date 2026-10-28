# Arcade fidelity audit — 28 October 2026

The current Type 110 builds fix three scoring differences verified by executing
the original Flicky parent ROM in MAME 0.264. This is a targeted fidelity update,
not a claim that the entire Atari adaptation is arcade-exact.

## Changes

| Behavior | Previous Atari build | Current build / original arcade |
| --- | --- | --- |
| Clear under 20 seconds | 2,000 points | 20,000 points |
| Clear in 20–29 seconds | 1,000 points | 10,000 points |
| Clear in 30–39 seconds | 500 points | 5,000 points |
| Clear in 40–49 seconds | 300 points | 3,000 points |
| Clear in 50–59 seconds | 100 points | 1,000 points |
| Clear in 60 seconds or more | 0 points | 0 points |
| Extra lives | Every 10,000 points, capped at nine lives | 30,000, 80,000 and 160,000; no further awards |
| Extra-life sound | Command `$8F` | Command `$90` |

The Round Clear screen now prints the corrected award. The extra-life milestone
index survives death and round changes and resets for a new game. The original
has no nine-life cap; the three default milestones normally take three starting
lives to at most six.

The cause of the time-bonus error was a unit mismatch. The arcade keeps BCD
scores in **tens of points**; this Atari implementation keeps **actual points**.
The old time-bonus port copied the source values without that conversion.

## Reference and verification

The supplied ROM passed size, CRC and SHA-1 validation, and MAME's `-verifyroms`.
The parent program chips are `epr-5978a.116` (CRC `296f1492`) and
`epr-5979a.109` (CRC `64b03ef9`). MAME's default Bonus Life DIP is
`30000 80000 160000`; that is the setting implemented here.

* **255 staged MAME cases** cover every ordinary score event, BCD carries,
  scores around the extra-life thresholds, time-bonus boundaries and long
  rounds. MAME also records the actual extra-life sound write. All outputs were
  independently reproduced by executing the same ROM routines on a separate
  Z80 core, then matched against the current 6502 implementation.
* **31 selected regression checks** exercise retained original-ROM algorithms
  and current life transitions. Coverage includes all 256 fractional velocities,
  acceleration and gravity, 3,000 world-motion ticks, 4,096 player-control
  cases, 4,096 moving-chick cases, trail history, spawn schedules and deaths.
  The cat algorithm comparisons cover retained reference routines; the live
  game still uses the simpler prepared pursuit described below.
* **Fresh bonus comparisons** execute the original ROM again: 288,400 bird
  states across 16 round selections; 608 sprite-contact cases; 1,824 ascending
  or stationary rejection cases; and 1,680 crowded/latched collision cases.
  All match. A fresh MAME bonus snapshot supplies the original platform for
  943 player states across 16 fields, including jumping, release/repress,
  landing and direction changes; all match.
* The PAL and NTSC release suites check the current renderer, all 48 layouts,
  music/effects, prop policies, score/result presentation, bonus fixtures and
  mapper/NMI behavior. Exact counts and release hashes are in
  `evidence/arcade-fidelity.json` and the accompanying JUnit XML files.
* Normal-speed Atari800 checks exercise the complete 315-frame intro, ordinary
  level-one movement, two bonus jumps, the actual 20,000-point award and result
  display, and the automatic return to round two. The result/bonus states are
  selected diagnostically before observation; they are not presented as a
  continuous human playthrough. No physical Atari/A8PicoCart was available.

The initial broad legacy test run had 29 passes and 13 failures. It mixed tests
for the retired pixel renderer with current routines, and compared raw score
bytes in different units. The score comparison now normalizes units, moving
chick tests explicitly exclude the deliberately simplified idle animation, and
the live death test uses the current contact path. Eleven historical rendering,
collision and old-background checks remain outside the selected suite; their
exclusion is not evidence that those differences have been eliminated.

## Remaining fidelity limits

* Normal-round cats use simpler pursuit and prepared two-tick movement. The
  original cat routines still exist but are not the live pursuit implementation.
* Normal-round pickups and several enemy/terrain contacts use bounding boxes
  instead of the arcade's full sprite-pixel ownership and collision hardware.
  Followers update at half rate, and stationary birds have simplified poses.
  The bonus round has its own more exact collision implementation.
* This build has two cats, no chameleon, and one-player play. NTSC permits one
  moving prop; PAL keeps six independent props. Those established policies
  have been preserved.
* Atari resolution, colors, sprite fitting and display cadence differ from the
  arcade. Normal pictures target two host refreshes; crowded bonus scenes can
  be slower. The custom intro, smaller Round Clear portrait and requested black
  decorative walls (except rounds 20–22) remain.
* The score/HUD remains a six-digit implementation. This audit's staged score
  coverage stops below its upper limit; it does not certify arcade behavior at
  million-point scores.

The next gameplay priorities are normal-round collision silhouettes and live
cat decisions. Both need cycle-budget measurements on the 64 KB target before
replacing the prepared fast paths.

## Reproducing the scoring comparison

Build the cartridge with `python3 tools/build_variants.py --mapper jcart`.
The shipped fixture in `tests/fixtures/arcade-scoring-mame.json` contains only
diagnostic inputs and outputs. It can be tested without original ROMs:

```sh
FLICKY_MAPPER=jcart FLICKY_VARIANT=pal python3 -m pytest tests/test_arcade_scoring.py -q
```

To regenerate it, privately supply the verified parent ROM, import it with
`tools/import_roms.py`, and place a MAME-compatible parent archive in your
chosen ROM directory. Run MAME with the Lua diagnostic:

```sh
mkdir -p build/mame-audit/scoring
FLICKY_ORACLE_DIR="$PWD/build/mame-audit/scoring" \
  mame flicky -rompath /path/to/roms -video none -sound none -nothrottle \
  -skip_gameinfo -autoboot_delay 0 -autoboot_script tools/arcade_scoring_mame.lua \
  -seconds_to_run 20 > build/mame-audit/scoring/mame.log 2>&1
python3 tools/check_arcade_scoring.py
```

The diagnostic writes RAM/register inputs and executes unmodified original
scoring instructions at `$178D` and time-bonus selection at `$06F1`. It clears
HALT between isolated calls and taps the real sound-command port. It is a
component comparison, not an unassisted arcade playthrough. The source archive
contains no original executable ROMs or full original-machine RAM dumps.
