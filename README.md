libmobile for BGB
=================

This is one of the software implementations of [libmobile](https://github.com/REONTeam/libmobile), interfacing with the [BGB](http://bgb.bircd.org/) emulator.

It provides BGB with the ability to emulate the [Mobile Adapter GB](https://bulbapedia.bulbagarden.net/wiki/Mobile_Game_Boy_Adapter) and connect games with homebrew servers.

Parts of our modifications in this fork were developed with assistance from AI tools, including code generation. This statement applies to our modifications and does not describe the upstream project's development process.

This fork is maintained at [zenaror/libmobile-bgb](https://github.com/zenaror/libmobile-bgb), on `feature/full_server`. Its integration tests simulate the BGB link and local network services; passing them does not establish compatibility with a particular game, production mail server or physical adapter. Those checks require separately recorded emulator or hardware results.

Project map
-----------

| Responsibility | Canonical location |
| --- | --- |
| Agent entry point and local restrictions | [AGENTS.md](AGENTS.md) |
| Usage, integration notes, validation and artifact retention | This README |
| Frontend code (`source/` is the equivalent of `src/`) | [source/](source/) |
| Core protocol implementation and callback contracts | [subprojects/libmobile/](subprojects/libmobile/) and its [mobile.h](subprojects/libmobile/mobile.h); maintained separately |
| Synthetic integration suite | [test.py](test.py), in the repository root |
| Build configuration and helper flags | [configure.ac](configure.ac), [Makefile.am](Makefile.am), [CMakeLists.txt](CMakeLists.txt), [meson.build](meson.build), [tools/](tools/) |
| Release tooling | [releng/](releng/); see the release toolchain notes below |
| Planning, decisions and pending validation | Existing OMM scope `libmobile-bgb`, with a version-checked handoff; technical contracts remain in this repository and the core headers |

The core is a pinned submodule with its own history. Review its actual diff before an update; develop core changes in its own repository. Another checkout of this frontend is not a second project or a reason to overwrite its branch or local changes.


Usage
-----

Open up the BGB emulator, select "Link-\>Listen" in its right click menu, and then fire up the program by running `./mobile` (or `./mobile.exe` on windows).

The mobile adapter has two means of communication, it can either directly call a different phone number and communicate over the telephone line, in P2P (peer-to-peer, or player-to-player) fashion, or it can instead call an ISP (Internet Service Provider) to connect to servers on the internet. As a general rule of thumb, any feature that communicates directly with another player will use the P2P mode, and will not require any prior configuration to function. The primary use for the P2P mode are direct battles and trades in Pocket Monsters: Crystal Version. Practically every other game and functionality connects to internet servers, and requires the user configure their account through the use of the Mobile Trainer cartridge.

By default, when dialing a number through a game's P2P functionality, the emulator will interpret any 12-digit phone number as an IP address. This works by splitting the 12 digits into chunks of three, which form the four components of an IP address. For example, '127000000001' is the number corresponding to the IP address '127.0.0.1'. The emulator will attempt to establish a TCP connection to that IP address, on port 1027. The receiving party must be reachable on the specified port, and have selected the listening option in-game, prior to receiving the call.

To avoid complicated matters with regards to opening TCP ports, an alernative communication method is provided through the use of a [relay server](https://github.com/REONTeam/mobile-relay/). By configuring one such server's IP address through the `--relay` option, each user will be assigned a phone number upon attempting to use any P2P functionality, which they may use to dial eachother. A secret key will be stored in mobile_config.bin, which will be used to preserve the assigned phone number across restarts of the emulator.

For the emulator to reach servers on the internet, a DNS server must be configured. This is done through the `--dns1` and/or `--dns2` options. The system's DNS resolver is avoided because all of the original game servers are unreachable on the open internet. The exact IP addresses to configure here will depend on the third-party game server that may be utilized.


Compilation
-----------

To compile the git sources, the following packages must be installed: `gcc`, `make`, `autoconf`, `automake`, `libtool`. Initialize the pinned core before configuring. For a new checkout of this fork:

```
git clone --branch feature/full_server https://github.com/zenaror/libmobile-bgb
cd libmobile-bgb
git submodule update --init
autoreconf -vfi
mkdir build
cd build
../configure --without-system-libmobile
make
```

`--without-system-libmobile` selects the pinned core instead of an installed system library. From an existing checkout, start at the submodule initialization step and use the task's own build directory.

On windows, you will need a Unix environment, such as [msys2](https://www.msys2.org/). The currently recommended package to install to provide `gcc` is `mingw-w64-x86_64-gcc`. One should use the MINGW64 environment to use it.

For the Linux cross-build with the x86_64 MinGW toolchain, use a separate directory:

```
mkdir build-windows
cd build-windows
../configure --host=x86_64-w64-mingw32 --without-system-libmobile
make
```

Alternatively, a `meson` build is also provided. See its [quickstart guide](https://mesonbuild.com/Quick-guide.html) for more information.

The windows build is reproducible: compiling the same source twice, even at different times, produces a byte-identical `mobile.exe`. This means the sha256 checksum published alongside a release can be used to verify a download -- a mismatch means a different file, never just a different build time.

Released binaries are built with whatever toolchain is on hand on the machine doing the release (plain `gcc`/mingw, dynamically linked against that machine's glibc/mingw runtime, not stripped), not with `releng/release-mingw.sh` or `releng/release-musl.sh`. Those scripts exist for a fully static, hermetic build (musl on Linux, `-m32` mingw on Windows) that isn't tied to the releasing machine's library versions; skipping them is a deliberate choice, not an oversight. Known consequence: a `mobile-linux` built this way may fail to run on a system with an older glibc than the one it was built against, and both binaries carry the releasing machine's build path in their debug info (visible via `strings`). Rebuild locally if either of these matters for your use.

Integration and validation
--------------------------

`mobile` runs as a separate process connected to BGB's TCP link endpoint. [source/bgblink.c](source/bgblink.c) handles link packets; [source/main.c](source/main.c) connects serial transfers, configuration, timers, sockets and device-auth callbacks to libmobile. Core timers use BGB timestamps at 2^21 units per second. A large timestamp discontinuity restarts the adapter with `mobile_stop()`/`mobile_start()`. The nonblocking HTTP client in [source/device_auth.c](source/device_auth.c) uses host time for its own deadlines. The core resolves the device-auth hostname through configured DNS; the frontend receives the address and connects on HTTP port 80. There is no device-auth server override.

For the existing synthetic suite, install Python 3 and run from the repository root after the Linux build:

```
ln -sf build/mobile mobile && python3 test.py
```

The two device-auth tests check effective UID 0 to bind their fake local services on ports 80 and 110; otherwise they are skipped. The frontend itself does not need root. A user/network namespace (`unshare -rn`) can supply a root identity inside an isolated test environment where supported; its loopback networking must be usable. Record skipped tests explicitly. Do not treat synthetic results as hardware or production evidence, or run this suite only to perform documentation maintenance or cleanup.

Known integration limits include device identity depending on the host installation, synchronous stderr requiring a consumer that drains it, and final shutdown having no further loop cycles to send queued device-auth events. On Unix only SIGINT has the explicit cleanup handler. Consult the core's [callback contracts](subprojects/libmobile/mobile.h) before changing socket behavior, resets or authentication.

Cleanup and retention
---------------------

At task completion, checkpoints and pauses, classify artifacts by their origin and purpose. Ignored or untracked files are not automatically disposable. Check task ownership, active processes, real paths and symlinks before removal; preserve another session's work and unresolved failures. `/tmp` is not permanent reference storage.

| Category | Reproduction and retention |
| --- | --- |
| Objects, libraries and intermediate executables in `build/` or `build-windows/` | Generated by the task's configure/make steps. Before cleaning, retain frontend/core revisions, flags, environment, commands, hashes and results sufficient to reproduce the work. Keep binaries involved in unexplained failures or pending acceptance. |
| `__pycache__/`, the `mobile` link and synthetic test configs | Remove only items known to have been created by the completed task, after its processes exit and useful evidence is saved. Removing a link never authorizes deleting its target. `test.py` registers exit cleanup for `config_test.bin`, `config_test_p2.bin` and `config_test_notoken.bin`; do not execute tests just to clean them. |
| Logs and intermediate inspection/test results | Retain a durable summary, commands, revisions, environment, hashes and relevant failure cases before discarding reproducible duplicates. Do not copy credentials or real configuration contents into checkpoints. |
| Real configurations, originals, saves, fixtures and console/hardware evidence | Preserve these potentially unique inputs, including `mobile_config.bin` and the legacy `config.bin`, even when ignored. Synthetic test cleanup does not authorize removing user data or files of uncertain origin. |
| Accepted releases and Git rewrite recovery | Keep original/published bundles, hash maps, manifests, scripts and verification/publication records while needed for reproduction, recovery, old clones or pending consumers/submodules. Keep historical backups private and outside published refs. A disposable verification clone may be removed after its durable source bundle is checked. No automatic expiry applies. |

The native clean removes build objects, libraries and executables, including the core's build outputs. Inspect `make -C build -n clean` or `make -C build-windows -n clean` first, only in a configured directory owned by the task. Check the emitted recipes and paths before executing without `-n`. Do not clean the source directory or use `distclean`, `maintainer-clean`, `uninstall`, `git clean -xfd`, broad deletion or age-based deletion as routine task cleanup.

Record removed paths, readily available counts/sizes, retained evidence locations, retention conditions and uncertain artifacts in the existing handoff. Cleanup grants no additional Git publication permission and does not trigger a global OMM backup sync.
