# Browser milestone experiment

Upstream baseline: `11514ea37d1e80d75bf2c1ba332bbd68fac5db4e`.
Fork: https://github.com/craze3/iw4L-browser
Branch: `browser-milestones`.

## Acceptance requirements

| Milestone | Required evidence | Current state |
| --- | --- | --- |
| 1. Browser rendering | A real MW2 map, working materials, moving camera, browser screenshots, and measured frame times at a recorded resolution on a recorded machine | Blocked pending the user's MW2 game-data path; no rendered map or browser benchmark |
| 2. Browser gameplay | Collision, movement, one gun, animation, audio, damage, and respawn in that browser build | Not run; depends on milestone 1 |
| 3. Browser multiplayer | Two browser clients playing through a native server, with movement and combat synchronization verified | Not run; depends on milestone 2 and a browser network transport |
| 4. Existing cross-game content | One Black Ops 1 map and weapon, repeated map changes, and observed memory behavior | Not run; also requires the user's Black Ops 1 game-data path |

No milestone is passed. Build checks alone cannot establish any of these outcomes.

## Experiments performed

- Created the GitHub fork and a local experiment branch at the upstream baseline.
- `cargo metadata --locked --format-version 1 --no-deps` succeeded.
- The first WebAssembly check stopped because the installed Rust 1.91.1 was below Bevy 0.19's Rust 1.95 minimum.
- Installed Rust 1.95.0 and its `wasm32-unknown-unknown` standard library. Pinned this fork's `rust-toolchain.toml` to that version and target. The machine's global default toolchain was not changed.
- With Rust 1.95, `cargo check --locked -p bootstrap --target wasm32-unknown-unknown` reached dependency compilation and failed. Mio explicitly rejects this target with Tokio's native networking enabled. The first attempt also selected Apple's Clang, which lacks the required WebAssembly backend for `zstd-sys`; a follow-up uses the existing Homebrew LLVM compiler.
- The follow-up with Homebrew LLVM exited 101 with the Mio networking errors; the Clang target error did not recur. Sanitized raw output is retained in [the initial build log](browser-evidence/wasm-baseline-rust195.log) and [the LLVM follow-up log](browser-evidence/wasm-baseline-llvm.log).

The networking error confirms a real porting requirement. It is not evidence that a browser port is impossible. No transport has been stubbed out or replaced, and no rendering implementation has been changed.

## Asset prerequisite

The repository contains no retail game data. No `.ff` or `.iwd` files were found in the searched Documents, Downloads, Steam, CrossOver, or Whisky locations. `IW4L_GAMES` is unset. An additional Spotlight query returned no results before it was stopped; that does not establish the absence of files outside the searched locations.

The user was asked for the local installation paths. MW2 2009 needs its game-data tree, including `zone` fastfiles and `main` archives; Black Ops 1 needs the corresponding tree for milestone 4. A single map file is insufficient because shared materials, models, animations, weapons, and audio live in other files.

Work stops at this missing-input condition, as requested. Synthetic geometry or a different game's browser build would not satisfy milestone 1.

## Resume procedure

1. Inspect the provided game-data directories and set an ignored local `.env` with `IW4L_GAMES`.
2. Establish the native baseline with the selected MW2 map and weapon so import errors can be distinguished from browser-port errors.
3. Add the browser platform path, including compatible material bindings/shader generation, asset access, and browser execution. Preserve the shared gameplay code and native baseline.
4. Verify milestone 1 in an actual browser and record console errors, screenshots, resolution, hardware, and frame-time measurements.
5. Continue milestones 2–4 in order, recording their direct runtime evidence. Treat the existing native networking dependency as implementation work, not as a reason to omit multiplayer.

For the compiler experiment on this macOS host, Homebrew LLVM can be selected without changing the system compiler:

```sh
CC_wasm32_unknown_unknown="$(brew --prefix llvm)/bin/clang" \
AR_wasm32_unknown_unknown="$(brew --prefix llvm)/bin/llvm-ar" \
cargo check --locked -p bootstrap --target wasm32-unknown-unknown
```

This is currently an expected failing portability check, not a runnable browser build.

## Cleanup

Both build checks have finished. The temporary Spotlight query was stopped. No game server, web server, browser automation process, container, tunnel, or proxy was started. No gameplay process is left running for this experiment.
