# Sources and third-party notices

`vendor/z80.h` is the unmodified floooh/chips Z80 emulator by Andre Weissflog.
It retains its zlib/libpng license and copyright notice inside the file. It is
used only as the independent host test oracle.

Source: https://github.com/floooh/chips/blob/master/chips/z80.h

The Sega 315-5051 decode table and algorithm in `tools/import_roms.py` are adapted
from MAME's `src/devices/machine/segacrpt_device.cpp`.

Copyright Nicola Salmoria, David Haywood. BSD-3-Clause:

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice, this
   list of conditions and the following disclaimer.
2. Redistributions in binary form must reproduce the above copyright notice,
   this list of conditions and the following disclaimer in the documentation
   and/or other materials provided with the distribution.
3. Neither the name of the copyright holder nor the names of its contributors
   may be used to endorse or promote products derived from this software without
   specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

The converted Flicky artwork, maps and sound are derived from the user's supplied game data. Sega's original game content is not relicensed by this project. This prepared source package excludes the original executable game ROMs and the old translated/reference cartridges.

Additional unmodified host-only reference headers retain their embedded zlib licenses:

- `vendor/z80.h`: floooh/chips commit 055d581d6b7042dee217908566a772a4502d8b12.
- `vendor/z80dasm.h`: floooh/chips-test commit 604d5d739ec94cae5dbab1efdc58e76bc55b68d4.
- `vendor/m6502.h`: floooh/chips commit 20d2d35d961b979771b3b11a735f0b29e5512fb3.

Atari800 and the cc65 toolchain are external build/test dependencies and are not redistributed in this source package.
