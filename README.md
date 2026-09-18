# Block Blast Bot (Android, fully offline)

Reads the Block Blast screen, solves all 3 visible pieces together, drags them into place, verifies the result, repeats.
No internet permission, no server, no downloads.

**Status: source only.** It has NOT been compiled or run on a phone. The solver logic was tested in a C port
(`tools/sim.c`); the Android parts (capture, gestures, screen reading) are untested. Expect to fix a few compile
errors or tune thresholds on first run.

## Build
- Android Studio: open this folder, let Gradle sync, Run. Or
- No PC: push this folder to a GitHub repo, run the "Build APK" workflow (Actions tab), download `app-debug.apk`.

## First-time setup (about 2 minutes)
1. Install the APK. If Samsung/Android blocks the accessibility toggle: App info -> ⋮ -> **Allow restricted settings**.
2. Open the app -> **1. Open accessibility settings** -> enable *Block Blast Bot*.
3. **2. Start screen capture** -> allow. (Android asks every time; that is a system rule.)
4. Open Block Blast. A small panel appears at the top-right:
   - **1 Calibrate**: tap top-left / bottom-right of the 8x8 grid, then top-left / bottom-right of the area holding the 3 pieces. A grid preview shows if you were accurate.
   - Start a **new game**, then **2 Learn skin** (learns the empty-cell colours of the current skin).
   - **Test read**: shows what the bot sees (8x8 grid + piece sizes). Compare with the screen.
   - **Start**. The first few moves auto-tune the finger offset (the game draws the dragged piece offset from your finger).
5. Changed skin? Just press **2 Learn skin** again on a fresh board. Geometry stays.

## How it works
| Step | What happens |
|---|---|
| Read board | 64 cell centres are compared with their own learned "empty" colour (per-cell, so checkerboard/gradient skins work). |
| Read pieces | Tray background is estimated per row; anything different is a piece. Shape = cells found on a grid whose size is auto-estimated. |
| Solve | 8x8 board as one 64-bit Long. All 6 piece orders x beam search (width 16) over precomputed placement masks, lines cleared after every placement. Scores holes, fragmentation, dead shapes, combos. |
| Move | Curved, slightly randomised drag (default ~110 ms). Waits longer after line clears (animation). |
| Verify | Reads the board again; finger offset is auto-corrected from where the piece really landed. 3 mismatches in a row -> re-tunes. |
| Save | Calibration, skin colours, timings, tuned offsets, combo stats and a solved-position cache are stored on the phone. |

## Numbers (simulation, not the real game)
Against a harsh generator that deals uniformly random pieces from 37 shapes: median 343 pieces placed before game over,
average 443, 7 of 200 games survived a 1500-piece cap; ~1.4 ms per solve in C. The real game deals friendlier sets.
Run it yourself: `gcc -O2 -o sim tools/sim.c && ./sim 200 1500`.

## Known limits / things to tune
- Only 3 pieces are visible, so it plans 3 moves ahead, never further. The next set is unknown.
- Exact combo/scoring rules of the game are not known to me; the solver just values consecutive line clears.
- Dark/very low-contrast skins: raise/lower *Board colour threshold* and *Tray piece threshold* in the app.
- Ads, pop-ups, "revive" screens: the bot stops when no pieces are found or no move exists; it does not dismiss them.
- The game may forbid automation in its terms; accounts/leaderboards could be affected. Use at your own risk.
- If the game sets FLAG_SECURE the screen cannot be captured at all.
- Very fast drags (< ~60 ms) may not register as a drag in the game.
- iOS cannot do this (no equivalent of accessibility gestures + screen capture for third-party apps).
