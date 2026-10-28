# Flicky — Type 110, J(atari)Cart 512 KB

Use `Flicky-Atari-XL-NTSC-JatariCart-512K.car` on an NTSC Atari, or `Flicky-Atari-XL-PAL-JatariCart-512K.car` on PAL. Both require a 64 KB XL/XE with BASIC disabled. Each file is 524,304 bytes: a 16-byte CART header followed by exactly 512 KB of ROM. The header identifies cartridge type 110 and includes the ROM checksum.

Copy the matching `.car` file to A8PicoCart storage, safely eject it and disconnect USB, then insert the cartridge into the powered-off Atari. Power on and select the file in the cartridge menu. Let the intro play or press fire to start. Left/right moves; fire jumps and throws a carried prop. In bonus rounds, fire jumps with the net; release and press again for another jump.

The A8PicoCart firmware must support **Type 110 / J(atari)Cart 512 KB**. The original Robin Edwards firmware's cartridge loader does not list type 110; older stock firmware is therefore insufficient. This release supplies the game cartridge, not a firmware update.

## Implementation

This is a source rebuild for the mapper's actual bus behavior. The common 8 KB kernel runs in RAM at `$8000–$9FFF`; banked code and assets share the ROM window at `$A000–$BFFF`. Bank 0 contains the bootstrap and kernel image. Even physical banks hold the former upper-window data; odd banks hold the corresponding hot code/data. The ROM bootstrap copies the common kernel into RAM and transfers execution there before switching banks.

Writes to addresses `$D500–$D53F` select the 64 banks. The data byte is irrelevant. The game maintains a RAM bank shadow because JCart does not provide SIC's readable control register. Foreground and audio/NMI selectors have separate self-modifying operands, with interrupted bank restoration tested at every cycle. Scenery and bonus collision pointers, saved bank restores, intro stream selectors, vectors and packed ROM ordering are relocated for the new window.

The SIC builds remain supported with the original default build commands. All 48 layouts, full 315-frame intro, sound, round-clear portrait, background policy and arcade-verified bonus jumping are retained. The NTSC/PAL prop policies are unchanged.

The current Type 110 release also includes the [MAME-verified scoring corrections](ARCADE_FIDELITY.md): arcade time-bonus values, default extra-life thresholds and the correct extra-life sound.

## Build and verification

```sh
python3 tools/build_variants.py --mapper jcart
```

This writes both named Type 110 cartridges to `release/`, and their SHA-256 hashes to `release/variants-jcart.json`. Individual development builds accept `--mapper jcart --variant ntsc` or `--mapper jcart --variant pal`. The source includes the converted assets and requires no original executable ROMs.

The release is checked with the cycle-counted 6502 test runner and Atari800 7.2.1 with real ANTIC/POKEY emulation, 64 KB XL, BASIC disabled and normal speed. Emulator checks play the full intro, enter and move through level 1, then select the bonus round before exercising ordinary fire-button jumps. No state edits occur during the observed gameplay segments. These checks are not a complete 48-round human playthrough or physical Atari/A8PicoCart verification.

See `evidence/tests-jcart-*.xml`, `evidence/jcart-ntsc.json`, `evidence/jcart-pal.json` and `evidence/jcart-release.json` for the release's results. Earlier performance and movie reports in this source package describe the previous SIC release.

Sources: [Atari800 cartridge specification, Type 110](https://github.com/atari800/atari800/blob/master/DOC/cart.txt), [J(atari)Cart author repository](https://github.com/jhusak/jataricart), [original A8PicoCart loader](https://github.com/robinhedwards/A8PicoCart/blob/main/source/Pico%20VSCode%20Project/a8_pico_cart/atari_cart.c).
