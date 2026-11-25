# No Lag Kart

**Mario Kart World 1.8.0 on Apple silicon Macs, in docked mode, without the stutter.** This is a fork of [nxnative](https://github.com/seedds/nxnative), which packs your own copy of a Switch or Switch 2 game into a Mac app: the game's own code runs on the Apple chip and its graphics are rebuilt on Metal. It is not an emulator.

Nothing from the game is in this repository: you need your own dump of Mario Kart World with update 1.8.0. This project is not affiliated with Nintendo. nxnative's own README, with its other games and its iPhone and iPad builds, is [README-nxnative.md](README-nxnative.md).

[How much better](#how-much-better-than-nxnative) · [Requirements](#requirements) · [How to play](#how-to-play) · [Settings](#settings) · [If you still get stutters](#if-you-still-get-stutters) · [What this fork changes](#what-this-fork-changes) · [Known limits](#known-limits) · [Credits](#credits)

## How much better than nxnative

Against nxnative as it was on 14 November 2026 (`main` at `b06f4e0e`), on the same Mac with the same game files and the same race:

- **4 players in split screen:** from 21 fps to the game's full 30 fps.
- **2 players in split screen:** from 46 fps to 60 fps.
- **1 player:** from 56 fps to 60 fps.
- **Late frames** (hitches you can see): from hundreds or thousands a race to practically none.

| Players | nxnative (14 Nov 2026) | No Lag Kart |
|---|---|---|
| 1 | 56 fps, 760 late frames a race | **60 fps, 3 late frames a race** |
| 2 (split screen) | 46 fps, 2,687 late frames | **60 fps, 1 late frame** |
| 4 (split screen) | 21 fps, 3,731 late frames \* | **30 fps, 1 late frame** |

Each value is the average of two runs of one whole 150cc race (Mushroom Cup, race 1, from the countdown to the finish), in console (docked) mode, fullscreen on a 14-inch MacBook Pro (M2 Pro, 16-core GPU, 16 GB, macOS 26.7.1) with Game Mode on and sound off. Both builds ran with MetalFX upscaling and no frame hold; nxnative has no GPU clock keeper, which this fork turns on. Players 2 to 4 were scripted inputs. 30 fps is the game's own rate for 3 and 4 players.

A late frame stayed on screen more than 2 ms longer than its step (16.7 ms at 60 fps, 33.3 ms at 30). With 2 and 4 players, the one late frame left is the frame at the cut into the race.

\* Below 30 fps the game itself slows down, so nxnative's 4-player race had not finished when the run ended: its count covers 220 s of racing, where this fork finished the race in 194 s.

- **The GPU sets the pace.** A frame's GPU time (median) was 14, 14 and 29 ms with 1, 2 and 4 players here, against 15, 18 and 42 ms on nxnative. A frame has 16.7 ms at 60 fps and 33.3 ms at 30.
- **No shader compiles while racing.** After the same warm-up races, nxnative still compiled 7 to 45 pipelines in each race; this fork compiled none.
- **The same picture or more.** Nothing in this fork lowers what the game draws. Its dynamic resolution even picks a larger render size here (15 to 26 % more render-target megabytes a frame than on nxnative), because its frames fit their time.
- **Memory** is alike in both: the footprint peaked at 10 to 11 GiB in these races, and this fork's at 12.5 GiB in a whole three-player Grand Prix.

## Requirements

- **A Mac with Apple silicon** and macOS 26.6 or later. This fork was tested on one Mac only: a 14-inch MacBook Pro, M2 Pro (10-core CPU, 16-core GPU), 16 GB, macOS 26.7.1. In split screen the GPU needs most of a frame's time there (14 of 16.7 ms with two players, 29 of 33.3 with four), so a Mac with a smaller GPU will likely not hold these frame rates: see [If you still get stutters](#if-you-still-get-stutters). Macs with 8 GB were not tested.
- **[Xcode](https://developer.apple.com/download/all/?q=Xcode%2027.2) 27.2** and its Metal toolchain: the shaders are built only with the Metal compiler they were tested with.
- **[Homebrew](https://brew.sh)** and **Python 3.12 or newer** (an arm64 one).
- **Your own dump of Mario Kart World with update 1.8.0** (v917504), as a folder: see step 3. No `prod.keys` are needed.
- **About 55 GB of free disk space** besides the dump (nxnative's rule: twice the game's size plus 10 GB). The finished app is 25 GB.

## How to play

**1. Install the tools** (once), in Terminal. Select Xcode 27.2 first: `sudo xcode-select -s /Applications/Xcode.app` (with several Xcode versions installed, the path of your 27.2, such as `/Applications/Xcode-27.2.app`).

```
xcodebuild -downloadComponent MetalToolchain
brew install xcodegen zstd glslang
python3 -m venv ~/.venvs/nxnative
~/.venvs/nxnative/bin/python3 -m pip install lz4 "capstone>=5,<6" fonttools nsz zbic "backports.zstd; python_version < '3.14'"
```

The virtual environment keeps the packages away from Homebrew's Python, which refuses a plain `pip install`.

**2. Get this repository:**

```
git clone https://github.com/kingbumi31/no-lag-kart.git && cd no-lag-kart
```

**3. Put the game in a folder.** Update 1.8.0's RomFS is the whole patched game, so the update's folder alone is enough. `exefs/` must hold `main.npdm`:

```
~/Games/MarioKartWorld/
  updates/917504/exefs/     main, main.npdm, rtld, sdk, subsdk0, subsdk1
  updates/917504/romfs/
```

**4. Build** the Mac app:

```
~/.venvs/nxnative/bin/python3 scripts/build_ipa_from_nsp.py ~/Games/MarioKartWorld --platform mac-app
```

The app, `Mario Kart World.app`, appears in the game's folder. The tested settings are built in: no extra file or option is needed. On the M2 Pro a rebuild in which every shader changed took 18 minutes (shader split 3:41, Mac shaders 7:35, pipeline archive 2:30, the app and its signature 2:13); a first build also packs the 23.5 GB game image first. Keep the game folder where it is: moved or renamed, it makes the next build pack that image again. All the script's options: [The packing script in detail](docs/development.md#the-packing-script-in-detail).

**5. Start it** from Finder. The first start is slow once: macOS checks the 25 GB bundle (about three minutes here before anything shows), then the app builds its graphics pipelines behind "Preparing shaders" (about a minute). Later starts take seconds. The settings window that opens before the game already holds the tested settings: click **Play**.

Players 2 to 4 each need a controller. Controllers, quitting, saves and the settings window: [Playing](docs/playing.md) and [Settings](docs/settings.md).

## Settings

The app starts with the settings every measurement above was made with. **They are tested, and I do not recommend changing them.** The settings window shows them, and **Reset to tested defaults** goes back to them:

| Setting in the window | Tested value | Why |
|---|---|---|
| Frame rate | 60 fps | the cap; with 3 and 4 players the game switches to its own 30 fps by itself |
| Console mode | Docked | the game draws for 2560x1440, with larger reflection and shadow maps than in handheld mode |
| Upscaling | MetalFX | MetalFX scales the picture up to the screen |
| Game's upscaler | Game's own | MetalFX in its place changed nothing in frame rate |
| Display mode | Fullscreen | every measurement was made in fullscreen, and macOS's Game Mode needs it |
| Hold frames when late | Off | a late frame does not lengthen the frames after it |
| Encode thread | On | Metal's encoding on a thread of its own: with four players the replay thread took 23.9 ms a frame instead of 26.8 |
| Frame buffering, Metal 4 pipelines, Pipeline prebuild | Off, On, On | nxnative's defaults |

Three more are built in and not in the window: the GPU clock is kept at its top (with it off, fullscreen with Game Mode averaged 1,199 of 1,398 MHz; about 4.5 W more in a race), the game's submissions reach the GPU in segments of 8 draws while the GPU waits (so the GPU starts sooner), and uploads come from a 128 MB resident ring (which halves the work at a scene cut). They are the game profile's defaults, in [`dm/games/0400C3F00006E000/game.tsv`](dm/games/0400C3F00006E000/game.tsv) with their reasons.

A line such as `DM_FPS_HOLD=on` in `~/Library/Application Support/nxnative/0400C3F00006E000/env.txt` (make the file if it is not there) overrides a setting at the next start; delete the line to go back.

## If you still get stutters

On a Mac that does not hold the frame rates above, two settings trade picture for smoothness in split screen. Both exist only in this fork, and both go into `env.txt` (above):

- **`DM_NVN_GPU_TS_PAD=1.25`** (or a milder `1.10`): the game's dynamic resolution reads each frame's GPU time 25 % (10 %) longer than it was, so it lowers its render size before frames run late. The picture is a little softer when the GPU is busy.
- **`DM_OPMODE_MULTI=0`**: handheld mode once the game has found two or more players; until then the game stays in console mode. Split screen then draws for 1920x1080 with handheld's smaller reflection and shadow maps, so its picture is clearly softer.

What they did on the M2 Pro with an earlier build of this fork (13 November), which still stuttered in split screen: the same race as above, one run per line.

| `env.txt` line | Two players | Four players | Drawn per frame (two / four players) |
|---|---|---|---|
| none (average of two runs) | 56 fps, 787 late frames | 28 fps, 924 late frames | 393 / 715 MB |
| `DM_NVN_GPU_TS_PAD=1.25` | 58 fps, 410 | 29 fps, 314 | 6 % / 15 % less |
| `DM_OPMODE_MULTI=0` | 60 fps, 29 | 30 fps, 9 | 25 % / 29 % less |
| `DM_OPMODE_MULTI=0` and `DM_NVN_GPU_TS_PAD=1.25` | 60 fps, 3 | 30 fps, 1 | 28 % / 30 % less |

"Drawn per frame" is the render-target megabytes the game draws into each frame, which follows the render size it picks. The pad alone halves the late frames or better; handheld split screen removes nearly all of them. On the current build the M2 Pro has only the one late frame at the cut into the race left, so there the two settings only cost picture: the pad at 1.25 draws 12 % and 19 % less (two and four players), handheld split screen 32 % and 34 % less. The current build with neither line is as smooth as the earlier one with both, and draws about half again as much (52 % and 54 % more).

### If the whole game runs slow on an older Mac

On a Mac with a smaller or older GPU, the GPU clock keeper itself can be the cause: the whole game, one player included, runs at about 20 fps and slower than real time (the three-second countdown takes visibly longer). Turn the keeper off with this line in `env.txt`:

```
DM_GPU_KEEP=0
```

Measured on a MacBook Pro, M1 Pro (6+2-core CPU, 14-core GPU), 16 GB, macOS 27.2.1, fullscreen, one player in console mode, with `scripts/bench.py` and the route `gp-start-150` (the 150cc race's start), three runs per line (the times are the median run):

| `env.txt` line | fps per run | Frame time p50 | Frame time p99 | GPU time p50 |
|---|---|---|---|---|
| none (the keeper at full duty, the profile's default) | 20.9, 21.0, 49.5 | 47.7 ms | 61 ms | 44.4 ms |
| `DM_GPU_KEEP=auto` | 46.9, 29.5, 32.0 | 16.9 to 39.7 ms | 52 to 56 ms | 13.6 to 35.6 ms |
| `DM_GPU_KEEP=0` | 59.5, 59.5, 59.5 | 16.6 ms | 21 to 22 ms | 13.1 to 13.4 ms |

With the keeper at full duty, most runs stayed at about 21 fps with 45 ms of GPU time a frame (the GPU reported 99 % busy) and some ran at about 50 fps, at random; with it off, every run held 60 fps with 13 ms of GPU time. `DM_NVN_FAST_WAKE=0`, `DM_NVN_SUBMIT_STREAM=0`, `DM_SPIN_POLL=0`, `DM_SPIN=0`, `DM_YIELD_BACKOFF=0`, `DM_NVN_ENC_THREAD=0` and `DM_NVN_BARRIER=game` made no difference there, and neither did handheld mode alone. In normal play afterwards (about 200 windows of 300 frames over several races), no window's median frame time went above 17 ms and the game ran at real speed.

When benchmarking on such a Mac, pass `--pso-archive` a copy of the app's own pipeline archive (`~/Library/Application Support/nxnative/0400C3F00006E000/pso_archive`): with an empty one, preparing the pipelines took 83 s before the first frame there, and the route's presses came before the title took them.

## What this fork changes

Each change went in only after a run on the Mac showed it helps this game, or because it corrects behaviour on a path the game takes. Every one with its measurement, and what was tried and left out: [`mkw/TRIAGE.md`](mkw/TRIAGE.md).

**Frame pacing**
1. The frame step never holds a frame for 3 vsyncs, as the longest step the game takes is 2: no stretch at two thirds of the game's speed.
2. Update 1.8.0 gets the frame step too (its frame-interval function is found by a second fingerprint): no slow motion after a late frame.
3. The streamed submission: the GPU starts on a frame 0.9 ms after the game submits it, where it waited for the host's copy of the whole frame.

**Less GPU work**
4. Render pipelines are compiled whole: GPU time 14.1 → 12.6 ms a frame, late frames 48 → 1 to 6 a minute.
5. The sky's cube map: task and mesh draws join the merged passes, 13 → 6 passes a view.
6. A sparse texture shares storage only with the pools it has pages in: the sky's cube map 6 → 1 pass a view.
7. The reflection maps' filters are drawn once for each cube face: 1,440 → 950 µs a view.
8. The reflection filters' face-layered pass, one layered pass up to 64x64: with it and the cube array filter split up to 64x64, two players hold 60 fps in docked mode.

**Less CPU work, fewer freezes**
9. A memory pool's finalize costs what the pool holds: the 50 to 70 ms freeze once per stretch of track went from 14 of 17 runs to 1 of 12.
10. A submission's commit looks only at the pages its shaders wrote: the replay thread 1.4 ms a frame faster at a 4-player race's start.
11. The per-draw feedback checks are cheaper: the replay thread 0.5 to 0.8 ms a frame faster with four players.
12. Upload sources are copied into a 128 MB resident ring: a scene cut's residency work halves, its frame 20 to 28 ms shorter.
13. The host's speed profile (PGO) recorded again on the final code. Building the host with such a profile took a whole 4-player Grand Prix from 71 late frames to 24.

**Shaders and pipelines**
14. A warm-up line that names a shader at another stage than the game's is left out: the merged warm-up list (15,062 pipelines from ten lists) no longer stops the prepare step, so nothing compiles while racing.
15. A table row after a control section is no shader entry: 1.8.0's shader packages split cleanly.
16. A pipeline archive write and the pipeline compiles no longer overlap (a port of an upstream fix): no Metal crash when the archive fills after a shader rebuild.

**Settings and switches**
17. The tested settings, without the two stopgaps of 13 November: the GPU's submissions in segments of 8 draws while it waits, the upload ring, the cube array filter split up to 64x64. Since 15 November they are the game's defaults in the settings window, with Fullscreen, and the app reports version 1.8.0.
18. The GPU-time pad counts from the frame's first submission, so `DM_NVN_GPU_TS_PAD` works on this game (off by default).
19. Handheld mode for a split screen (`DM_OPMODE_MULTI`; off by default).
20. Handheld mode from a chosen player count (`DM_OPMODE_MULTI_FROM`; off by default).
21. MetalFX up to a chosen player count (`DM_PRESENT_UPSCALE_PLAYERS`; off by default: MetalFX stays on for every player count).

**Testing**
22. A test that read a count too early waits for the earlier command buffers.
23. The benchmark stops the game by its process ID, so a hung system call no longer leaves a game running beside the next start.

Besides these, the branch carries 15 upstream pull requests and 5 later upstream commits (listed in [`mkw/TRIAGE.md`](mkw/TRIAGE.md)), among them the GPU clock keeper of pull request #1847. The state of the build: [`mkw/README.md`](mkw/README.md). How the port works and what is known about the game: [`mkw/KNOWLEDGE.md`](mkw/KNOWLEDGE.md).

## Known limits

- **One Mac:** a 16 GB M2 Pro MacBook Pro, its built-in screen, fullscreen at a fixed 60 Hz. Other Macs, TVs and external screens, real controllers for players 2 to 4 (scripted here), free roam and the other modes were not tested with this build. iPhone and iPad builds of this fork were not tested either.
- **Heavier courses:** over a whole Grand Prix, a few minutes of heavier courses still bring late frames, up to about two a second (two players on the Star Cup, four on the Banana Cup).
- **Scene cuts:** one frame of about 70 to 115 ms at each scene cut, where several hundred textures are shown for the first time.
- **Sound** was off in the timed runs. An earlier check with the sound path running but muted found no late frame with one player and up to 5 a minute with four (0 to 1 without it).

## Credits

- **[nxnative](https://github.com/seedds/nxnative)** by seedds and its contributors, which this fork is built on, and the projects nxnative builds on ([its acknowledgements](README-nxnative.md#acknowledgements)).
- The authors of the upstream pull requests in the branch, listed in [`mkw/TRIAGE.md`](mkw/TRIAGE.md).
- The warm-up lists merged into this build's: seedds/nxnative pull request #2097, Thijs2004/nxnative-mkw, azmasm/switch2mac, cheemu21/plumber-racer-2 and pahorema/kartnative ([`mkw/pso/README.md`](mkw/pso/README.md)).

## License

MIT, as nxnative ([LICENSE](LICENSE)). Third-party code keeps its own license ([dm/third_party](dm/third_party/README.md)).
