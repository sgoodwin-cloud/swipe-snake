# Swipe Snake — Design & Development Doc

The single source of truth for Swipe Snake: what the game is, how the code is laid out, how releases work, and what's planned. Update this file whenever a decision is made or a milestone moves, so any new chat session can pick up from here.

_Last updated: 21 Oct 2026 · Current release: v12 (update fixes from Pocket Factory and Flip 7) on `main` · Next: play-test v10's mutation pool, then Milestone 4_

---

## 1. Quick reference

| Thing | Where |
|---|---|
| Repo (source of truth) | https://github.com/sgoodwin-cloud/swipe-snake |
| Live installable game | https://sgoodwin-cloud.github.io/swipe-snake/ |
| Test-bench artifact (private) | https://claude.ai/artifact/TN3VAjBSFPEFVhvJoNVB23 |
| Main device | iPhone 13, installed via Safari → Share → Add to Home Screen |
| Hosting | GitHub Pages, `main` branch, `/ (root)` |

**Working rhythm:** iterate in the artifact during a milestone (fast to test), then push to the repo at the end of the milestone so the phone picks it up.

---

## 2. Guiding principles

These are settled. Don't break them without a deliberate decision logged in §9.

1. **Fully offline.** Everything must work with no connection after the first load. No runtime network calls, no server, no artifact-only capabilities (shared storage, in-page Claude, etc.) in the game itself.
2. **Planning over reflexes.** Moderate base speed; tension comes from decisions, not twitch reactions. Brake is a core ability from the very first run.
3. **The snake evolves within a run; the handheld evolves between runs.** In-run upgrades are *Mutations* (biological). Meta upgrades are *Workshop* hardware (the device). Keep the two vocabularies separate.
4. **Meta progression adds variety, not raw power.** Unlocks mostly widen the mutation pool or add cosmetics. Only small starting perks affect power.
5. **Core game feel is never locked.** Bite burst, death shake and basic sound effects are on from run one. Only expressive extras (palettes, music, trails, skins) are unlockable.
6. **Phone-first.** Swipe controls, portrait, works one-handed. Keyboard support for desktop testing only.
7. **Tiny and self-contained.** Graphics drawn in code, sound generated in code. No image/audio asset files beyond icons.

---

## 3. Current state (v10)

**Title screen** with Run, Classic and a Sound toggle; the HUD shows runs, best floor reached and classic best.

**Run mode** (§6.1–6.5):
- 9 floors in 3 acts. Each floor opens on an intro card ("Floor N · Act A · eat X, then exit", the mutation just gained, and that floor's threats, e.g. "2 beetles · portals · shrinks at 5 food") and waits for a swipe, so you can plan before moving. An "Abandon run" button sits on the intro card (counts as a run ended).
- Food targets 6/8/10 by act; act speeds 160/145/130ms per move. Snake resets to length 3 each floor.
- Hitting the target opens an **exit** (hollow blinking tile, at least 5 tiles from the head where possible) with a sweep and jingle. Food **keeps spawning** after the exit opens, so you can risk extra pickups for score before leaving.
- Scoring: food = 10 × act number; floor clear = 20 × floor number; winning = +100.
- HUD: Floor n/9 · Act · Food eaten/target (score is no longer shown during a run; it's on the results screen only). Brake meter (12 segments) under the board, with Phase/Armour/Burrow charges on a line beneath it.
- Death → shake and flash (~0.75s) → results screen (floor, cause of death — hit a wall / bit yourself / caught by a beetle / crushed by the shrink / abandoned — score, new best, and the build). Winning floor 9 → "YOU WIN" results.
- **Draft** after floors 1–8: "MUTATE" screen with 3 cards (name + level, family, effect). Tap a card (or press 1/2/3); the cleared board stays dimmed underneath and the row below shows the clear bonus. Cards ignore taps for 0.6s so a swipe that just finished the floor can't pick by accident.
- Popups (+progress, BRAKE, BRAKE+, PHASE, SNAP, BURROW) rise from the snake. Wraparound draws dashes on every stretch of edge you can cross. Golden food is a plus sign that blinks before fading.

**Mutations (v10 pool, 17):** every one now affects whether you finish the run; none is score-only. Most stack to II or III; maxed ones stop being offered. Numbers live in `CFG.mut`.

| Mutation | Family | Max | Effect |
|---|---|---|---|
| Phase | Movement | II | Pass through your own body once (II: twice) per floor |
| Wraparound | Movement | I | Cross any edge where neither side is walled, come back on the opposite side |
| Burrow | Movement | II | Once (II: twice) per floor, hitting a wall tunnels you to the first open tile beyond it (up to 3 thick). Fails at the board edge, and a failed attempt doesn't use a charge |
| Deep Tank | Movement | II | Brake meter +50% per level |
| Quick Recovery | Movement | II | Brake refills ×2 (II: ×3) as fast |
| Cold Blood | Movement | II | Food eaten while braking counts ×2 (II: ×3) towards the target |
| Tremor | Movement | I | Beetles freeze while you brake (they fade back to show it) |
| Armoured | Body | II | Once (II: twice) per floor, biting yourself snaps the tail off at that point instead of killing you; the lost tail vanishes |
| Battery | Body | II | Every 5 tail segments beyond 3 adds 0.5s (II: 1s) to the Brake meter. New capacity arrives full ("BRAKE+"); losing length loses it |
| Spiked | Body | II | A beetle that walks into your body dies and counts as 1 (II: 2) food towards the target |
| Lean | Body | I | Every other bite doesn't grow your tail |
| Banquet | Appetite | II | 2 (II: 3) pieces of food on the board at once |
| Shortcut | Appetite | II | Every floor's food target is 1 (II: 2) lower, never below 2 |
| Magnet | Appetite | III | All food within 3/4/5 tiles drifts one tile towards your head every 2 moves |
| Golden Hour | Appetite | III | 20/32/45% of food spawns golden (a plus sign): counts 3 towards the target, fades after 45 moves |
| Gourmet | Appetite | III | A bite within 14/20/26 moves of the previous one refills 35% of the Brake meter ("BRAKE") |
| Second Helping | Appetite | I | Each food counts double towards the target; tail grows by 2 |

Rules: if both are held, Phase is spent before Armoured. Food progress multiplies: golden (3) × Cold Blood × Second Helping, so a golden bite while braking with Cold Blood II and Second Helping counts 18. Popups show progress ("+3") and refills ("BRAKE") rather than points. Draft weighting: each option's weight is 1 + 0.6 × mutations you already hold in its family. Classic mode never sees mutations.

**Floors and threats (v9):**

| Floor | Layout pool | Beetles | Portals | Shrinks |
|---|---|---|---|---|
| 1 | Open field | – | – | – |
| 2–3 | Pillars, Crossbars, Gatehouse | – | – | – |
| 4 / 5 / 6 | Lanes, Brackets, Arena | 1 / 2 / 3 | – | – |
| 7 / 8 | Divide, Crossroads, Gallery | 1 / 2 | yes | 8 only |
| 9 | The Nest (fixed finale, provisional until Ouroboros in M5) | 2 | yes | yes |

- **Layouts** are 16×16 text grids in `LAYOUTS` (`#` wall, `.` floor, letter pairs = portals); `FLOORS` says which pool, how many beetles and whether it shrinks. A run never repeats a layout while its pool has others left. Rows 7–9, columns 4–9 are always carved clear for the start. Gatehouse and Arena wall the edge, so Wraparound only works through Gatehouse's gaps and not at all in Arena.
- **Walls** (drawn as bricks) kill on contact. Phase and Armoured don't help.
- **Beetles** (a body with shuffling legs) patrol in a straight line, moving once per 2 snake moves (so Brake slows them too). They bounce off walls, edges, other beetles, food, the exit and your body. Head contact either way kills you. Never spawn within 5 tiles of your head or close to each other. They go through portals.
- **Portals** (framed tile with a pulsing core; second pair has a hollow core). Enter one, come out of the cell just past its pair in the same direction. Portal cells themselves are never occupied. Beetles and Magnet-drifted food use them too.
- **Shrink**: at half the food target, the outermost ring that isn't already solid flashes (and nothing spawns there) for 20 moves, then turns to wall with a thud. Anything of yours in it = death; beetles in it die; food and exit move; portals in it close.

**Brake:** hold a finger still (within 10px) for 250ms; time runs at 50% until you lift. Swiping while held steers and keeps braking. Desktop: hold Space. Meter: 3s from full, refills in 6s; if emptied, it's locked until it refills to 25%. Board gets a faint dark tint and sounds drop in pitch while braking. Run mode only.

**Classic mode:** the v5 game unchanged in feel (140ms → 60ms ramp, 1 point per food, no Brake), with game-over buttons for Menu and swipe-to-play-again after a short lockout.

**Game feel (always on):** bite pixel burst + bleep, death shake + flash + harsh bleep, exit sweep + jingle, floor-clear jingle, brake tint + blip. Web Audio unlocks on the first touch or key. Note: the iPhone's silent switch mutes Web Audio.

**Save:** versioned object in `localStorage` key `swipe-snake-save` (§4.4 shape, `v: 1`). On first load the old `snake-best` value migrates into `stats.classicBest` (old key left in place). Stats tracked: runs, wins, bestFloor (highest floor reached), bestRunScore, classicBest. Sound setting saved.

Keyboard: arrows/WASD steer, Space brakes (and starts a floor), Enter/R starts a run, C classic, M mute, Esc back to menu from Classic.

### Carried over from v5
Classic Snake with:
- 16×16 grid on a retro LCD-style canvas (single dark theme, Silkscreen + VT323 fonts from Google Fonts with monospace fallbacks; fonts get cached by the service worker after first load).
- Swipe anywhere on screen to steer, with a 28px dead zone at screen edges to avoid iOS gesture conflicts. Turns fire mid-swipe (22px threshold) and can chain without lifting. Up to 2 turns queued.
- Speed: starts at 140ms per move, 10ms faster every 5 pickups, floor of 60ms.
- Best score saved in `localStorage` key `snake-best`.
- Arrow keys / WASD on desktop.
- Installable PWA with offline cache and an "Update ready · tap to reload" banner.

**Update button — root cause found (14 Oct 2026):** in v5 and earlier, the swipe handler called `preventDefault()` on every `touchstart` in the safe zone, which includes the update banner. iOS then never fired the button's `click`, so tapping did nothing (not even "Updating…") and only a force-close applied the update. v6 fixed this by ignoring touches that start on a `<button>`. Because the banner you tap belongs to the *old* page, the fix only shows up from the v6 → v7 update onwards. Reproduced in headless Chromium with touch: v5 → v6 tap does nothing; v6 → v7 and v7 → v8 taps reload onto the new version.

v7 also makes the reload wait for the new worker to report `activated` (plus `controllerchange`), with a 5s timer only as a last resort, instead of a blind 2.5s timer that could reload onto the old page on a slow iPhone.

**Confirmed on the iPhone (14 Oct 2026):** the v6 → v7 update applied from the banner tap alone, no force-close needed.

---

## 4. Code structure

### 4.1 Files in the repo

| File | Purpose |
|---|---|
| `index.html` | The whole game: markup, CSS, game script, and the PWA/update script at the bottom |
| `sw.js` | Service worker: offline cache, update handling |
| `manifest.webmanifest` | Home-screen app metadata (name "Snake", standalone, portrait) |
| `icon-192.png`, `icon-512.png`, `apple-touch-icon.png` | Icons (LCD-style snake, generated by script) |
| `README.md` | Short public description |
| `DESIGN.md` | This document |
| `CLAUDE.md` | Pointer for Claude sessions working in the repo |

### 4.2 Inside `index.html` today (v10, ~1,320 lines)

- **CSS tokens** on `:root`: `--case`, `--bezel`, `--lcd`, `--lcd-dim`, `--pixel`, `--ink`, `--muted`, plus font tokens. The canvas reads LCD colours from these at start-up.
- **Markup:** `#hud` (filled per screen by `setHud`), `#board` with canvas + `#overlay` (`#ovTitle`, `#ovLines`, `#ovMsg`, `#ovMenu` buttons, filled by `showOverlay`), `.under` row holding the brake `#meter` or the `#hint`.
- **Game script** (one IIFE), sections numbered to match §4.3:
  1. `CFG` — all tuning numbers
  2. RNG — `mulberry32`; each run has a seed, each floor gets its own rng drawn from the run's rng. Cosmetic effects use `Math.random` so they never disturb the game sequence.
  3. Save — `loadSave` / `migrate` / `writeSave`
  4. Screens — `setScreen(name, data)`; `SCREENS.title`, `SCREENS.play` (mode `run` or `classic`, phases `ready → playing → dying | cleared | over`), `SCREENS.results`. Each has `enter`, `update(dt, now)`, `input(action, arg)`, `hud()`, and a `board` flag for drawing. `SCREENS.draft` sits between floors (`play` → `draft` → `play`).
  5. Run & board model — `run` (seed, rng, floor, score, `mutations: {id: level}`, `build`, `usedLayouts`, `cause`) vs `game` (snake, `foods` list of {x, y, golden, ttl}, exit, eaten, target, stepMs, accumulator, `walls` Uint8Array, `portals` Map, `pairs`, `beetles`, `shrink`). `travel(x, y, dir, who)` resolves one move through walls (and Burrow), portals and Wraparound for the snake, beetles and food alike. `placeFood()` tops the board up to its food count; `addEaten(n)` is the single place progress is counted and the exit opened. `step()` is one snake move, then beetles (`moveBeetles`) and the shrink (`updateShrink`). `die(won, cause)`, `clearFloor`, `endRun(won, cause)`.
  6. Content data — `LAYOUTS`, `FLOORS`, `parseLayout()`; `MUTATIONS` (id, name, family, max, `text(level)`, `hooks`), `FAMILIES`, `draftOptions()` (weighted pick, seeded from the run rng), `buildText()`
  7. Mutation hooks — `Hooks.emit` (onFloorStart, onEat, onStep, onSpawnFood), `Hooks.reduce` (foodValue, foodBonus, brakeMax, brakeRefill), `Hooks.first` (onSelfCollide → 'phase'/'snap', onWall → wrapped head, onBeetleTouch → points). Hooks iterate in `MUTATIONS` order and do nothing outside a run. **Adding a mutation = adding one entry to `MUTATIONS`.**
  8. Rendering — pre-rendered LCD background, walls pre-rendered per floor (`renderWalls`, redone after a shrink), `draw(now)`, `FX` (burst particles, exit sweep, shake offset)
  9. Audio — `Audio` synth (`bite`, `death`, `exit`, `clear`, `win`, `brake`, `click`)
  - Brake — `Brake` object (held, meter, exhausted lock, meter UI)
  10. Input — touch (swipe + hold detection), keyboard, mouse; inputs become actions (`dir`, `go`, `run`, `classic`, `mute`, `menu`) sent to the current screen
  - Boot — `requestAnimationFrame` loop with a time accumulator (dt capped at 100ms); `window.__snake` is a small test hook.
- **PWA script**: `APP_VERSION`, service worker registration, update banner logic.

### 4.3 Planned structure (from Milestone 1)

Stay with a single `index.html` until it gets unwieldy (roughly 1,500 lines), then split into `src/*.js` files loaded as plain scripts. If split, every file must be added to the `CORE` list in `sw.js` or it won't work offline.

Inside the script, organise into clearly commented sections:

1. **Config & tuning tables** — all numbers in one place (speeds, floor targets, meter sizes, mutation values).
2. **RNG** — a small seeded generator (e.g. mulberry32). Every random choice in a run uses it, so runs are reproducible and a future daily challenge is trivial.
3. **Save system** — one `localStorage` key holding a versioned JSON object (see §4.4), with migration on load and try/catch on every read/write.
4. **Screen state machine** — `title → run(floor) → draft → run(next floor) … → results → title`, plus `workshop` and `classic` from the title screen. One `setScreen(name, data)` function; each screen has `enter`, `update`, `draw`, `input` handlers.
5. **Run model** — run state (act, floor, mutations held, Brake meter, run score) separate from floor state (board, snake, food, hazards).
6. **Content data** — floor layouts, mutation definitions, Workshop items, achievements as plain data objects, so adding content means adding data, not logic.
7. **Mutation hooks** — mutations register for named events (`onEat`, `onSelfCollide`, `onFloorStart`, `onTick`, `onBrake`, `foodValue`, `scoreMultiplier`…) instead of being special-cased in the main loop.
8. **Rendering** — canvas drawing plus a tiny particle system and screen-shake offset.
9. **Audio** — Web Audio synth for bleeps; started from the first tap (iOS needs a user gesture); mute toggle saved.
10. **Input** — swipe, hold-to-brake, keyboard (Space = brake on desktop).
11. **PWA/update script** — unchanged.

The game loop should move from `setInterval` to `requestAnimationFrame` with an accumulator, so Brake can scale time smoothly and animations can run between grid steps.

### 4.4 Save data (planned)

```json
{
  "v": 1,
  "bits": 0,
  "unlocked": { "palettes": ["lcd"], "music": [], "effects": [], "skins": ["classic"], "mutations": ["…starter ids"], "perks": [] },
  "equipped": { "palette": "lcd", "music": null, "effects": [], "skin": "classic", "perk": null },
  "overclock": { "max": 0, "selected": 0 },
  "achievements": {},
  "stats": { "runs": 0, "wins": 0, "bestFloor": 0, "bestRunScore": 0, "classicBest": 0 },
  "settings": { "sound": true }
}
```

Migrate the old `snake-best` key into `stats.classicBest` on first load. Planned save-export: base64 of this JSON shown as a copyable code, with an import box — protects against iOS wiping web-app storage.

---

## 5. Release process

1. Make the change in `index.html` (and test in the artifact if it's a bigger change).
2. Bump **both** version markers, which must match:
   - `APP_VERSION` near the bottom of `index.html`, and the `v<span id="ver">` text
   - `VERSION` at the top of `sw.js` (`swipe-snake-vN`)
3. If any file was added, add it to `CORE` in `sw.js`.
4. Commit and push to `main`. GitHub Pages republishes in 1–2 minutes.
5. On the phone: open Snake with signal → "Update ready" banner → tap → it shows "Updating…" and reloads onto the new version.

### Known PWA / iOS gotchas

- **Stale files on update (fixed in v5):** the install step must fetch with `cache: 'reload'`, otherwise iOS's HTTP cache can feed the old page into the new cache. Keep this.
- **Buttons and the swipe handler:** never `preventDefault()` a `touchstart` that begins on a `<button>`, or iOS won't fire its `click`. This is what broke the update banner up to v5.
- **Old service workers carry old bugs:** a fix to the update logic only takes effect *after* the phone has that version. If the update flow itself breaks, the phone may need fully closing and reopening (app switcher) a couple of times.
- **Storage separation:** the home-screen app's storage is separate from Safari's. Deleting the icon deletes saves.
- **No vibration on iPhone.** Don't design anything that depends on haptics.
- **Audio needs a user gesture** before it can play on iOS.
- **Edge swipes** belong to iOS; keep the edge dead zone.
- **Artifact vs installed:** the artifact runs inside claude.ai's sandbox. To preview repo code as an artifact, publish the game markup/CSS/script without the `<!doctype>`/`<head>` wrapper; service-worker registration fails harmlessly there.

### Claude session notes

- Claude can push to this repo once it's attached to the session (the Claude GitHub App has access to all of Stuart's repos).
- Claude **can't** create repos or change GitHub Pages settings from a session; those are done by Stuart in GitHub (use Safari for Settings → Pages; the iOS app lacks it).

---

## 6. Game design spec (roguelike mode)

All numbers are starting values to be tuned in Milestone 6.

### 6.1 Run structure

- A **run** is **9 floors** in **3 acts** of 3 floors.
- Each floor has a **food target** (Act 1: 6, Act 2: 8, Act 3: 10). Hitting it opens an **exit** tile; entering the exit clears the floor.
- The snake **resets to base length (3) each floor** ("shedding its skin"). Mutations persist for the whole run.
- Between floors: **draft** — choose 1 of 3 mutations.
- Clearing floor 9 **wins** the run. Dying ends it. Either way you earn **Bits** (§6.6).
- Run score exists for bragging rights; winning is the goal. Mutations should affect finishing the run, never just score (decided 18 Oct 2026).

### 6.2 Pace

- Speed is fixed per act rather than ramping within a floor. Proposed: Act 1 = 160ms/move, Act 2 = 145ms, Act 3 = 130ms. (Classic mode keeps its own 140→60ms ramp.)
- Overclock levels (§6.7) can raise speed.

### 6.3 Brake (core ability)

- Available from the first run; not a draft pick.
- **Hold a finger still for ~250ms** to brake; time runs at 50%. **Swiping while holding still steers**, so you can slow down, line up a turn and swipe without lifting. Desktop: hold Space.
- Uses a **meter**: ~3 seconds of braking when full, refills in ~6 seconds when not braking. Shown beside the score.
- Draft mutations upgrade Brake rather than grant it (§6.4).

### 6.4 Mutations (in-run upgrades)

Four families. The draft gently favours families you already hold so builds come together.

**Body** — tail segments with effects
| Name | Effect |
|---|---|
| Spiked | Hazards that touch your body die and count towards the floor target |
| Armoured | First self-collision each floor snaps the tail off at that point instead of killing you |
| Battery | Tail length adds Brake capacity |
| Lean (added in M2) | Every other bite doesn't grow your tail — the anti-Battery pick |
| Bomb (later) | Every 5th segment detonates on a sharp turn, clearing nearby hazards |

**Appetite** — food
| Name | Effect |
|---|---|
| Magnet | Food within 3 tiles drifts towards your head |
| Golden Hour | Chance for food to spawn golden (counts 3 towards the target, expires) |
| Gourmet | Quick successive bites refill Brake |
| Banquet (added 18 Oct) | More food on the board at once |
| Shortcut (added 18 Oct) | Lower food target per floor |
| Second Helping | Each food counts double towards the floor target, but your tail grows by 2 |

**Movement & Brake**
| Name | Effect |
|---|---|
| Phase | Pass through your own body once per floor |
| Wraparound | Cross the board edge wherever it isn't walled (layouts can wall all, part or none of the border) |
| Deep Tank | Brake meter +50% |
| Quick Recovery | Brake refills twice as fast |
| Cold Blood | Food eaten while braking counts double towards the target |
| Burrow (added 18 Oct) | Tunnel through a wall once per floor |
| Tremor (added 18 Oct) | Beetles freeze while braking |

**Rare** — strong, with downsides; unlocked via the Workshop
| Name | Effect |
|---|---|
| Glass Jaw | Food counts ×2 towards the target; you lose Armoured and Phase and can't draft them |
| Constrictor | Close a loop with your body to capture everything inside: food counts as eaten, hazards die. The signature build-around mutation |
| Ouroboros' Gift (idea) | Biting your own tail cuts it and banks the cut length as progress towards the target (needs rework from the original score idea) |

Design intent: Constrictor + Spiked + Battery + Magnet should be a recognisable, satisfying "long-snake territory" build. Glass Jaw + Cold Blood + Deep Tank should be a "slow, precise, fast-clearing" build.

### 6.5 Acts and hazards

| Act | Floors | Adds |
|---|---|---|
| 1 | 1–3 | Open boards, then simple wall layouts |
| 2 | 4–6 | **Beetles**: walk in straight lines, bounce off walls *and your body*, kill on contact with your head. Phase/Armoured don't protect against them |
| 3 | 7–8 | **Portals** (paired) and **shrinking boards** (outer ring becomes wall partway through) |
| Finale | 9 | **Ouroboros**: a rival AI snake competing for the same food. Win by trapping it (it dies if it has no legal move) or by reaching the target first |

Layouts are small text grids in the content data (`#` wall, `.` floor, `E` exit spot, `P` portal pairs…).

### 6.6 Meta layer: the Workshop

The handheld's back panel. Currency: **Bits**, earned every run based on floors cleared (+ bonus for a win, + Overclock multiplier).

**Hardware (cosmetic only)**
- Screen palettes: Classic LCD (default), Amber, DMG Green, Backlit Blue, Night (inverted)
- Speaker modules: chiptune music tracks (generated in code), one per module
- LCD effects: ghosting trail, scanlines, bite flash style
- Snake skins: head/segment styles

**Cartridges (variety)** — add mutations to the draft pool. Brake upgrades and basic Body/Appetite/Movement mutations are available from the start; Rare mutations and later commons must be unlocked. Most Bits should go here.

**DIP switches (difficulty and starting options)**
- Starting perks (one equipped at a time): e.g. start with a random Body mutation; first draft offers 4 choices
- **Overclock** levels, unlocked after the first win. Each adds a modifier (faster acts, extra beetles, fewer draft choices, shorter Brake…) for more Bits. Same role as Heat in Hades.

**Achievements** unlock some cosmetics directly, so unlocks tell the story of how they were earned. Examples:
| Achievement | Unlock |
|---|---|
| Clear Act 1 for the first time | Amber palette |
| Clear a floor in under 20 turns | Ghosting trail |
| Win a run | Overclock 1 + DMG Green |
| Capture 3 beetles in one Constrictor loop | Constrictor skin |
| Win without braking | Night palette |

### 6.7 Overclock (draft list)
1. Acts 10% faster
2. +1 beetle per Act 2+ floor
3. Draft offers 2 choices instead of 3
4. Brake meter −30%
5. Food targets +2

### 6.8 Game feel (always on from run one)
- Bite: small pixel burst and a short bleep
- Death: brief board shake, harsh bleep, snake flashes
- Floor clear: exit opens with a sweep; short jingle
- Brake: subtle tint on the LCD and lowered pitch
- Mute toggle on the title screen, saved

### 6.9 Classic mode
The current game stays available from the title screen as **Classic**, with its own best score.

---

## 7. Milestones

Each milestone is a playable release. Tick items as they're done and note the version that shipped them.

### ☑ M1 — Run structure (v6)
- [x] Code reorganised into the sections in §4.3; seeded RNG; `requestAnimationFrame` loop (v6)
- [x] Title screen with Run and Classic modes (v6)
- [x] 9-floor run: food targets, exit tile, length reset per floor, act speeds (v6)
- [x] Brake: hold-to-brake, meter UI, keyboard Space (v6)
- [x] Results screen (floor reached, run score) (v6)
- [x] Versioned save data with migration from `snake-best` (v6)
- [x] Core game feel: bite burst, death shake, Web Audio bleeps, mute toggle (v6)
- [x] Play-test v6 on the iPhone, then merge `m1` → `main` (v6, 14 Oct 2026)
- [x] Confirm the update banner works on the iPhone without a force-close (v7)

### ☑ M2 — The draft (v8)
- [x] Pick-1-of-3 screen between floors (v8)
- [x] Mutation hook system (v8)
- [x] 12 mutations from Body, Appetite, Movement & Brake, most stacking to II/III (v8)
- [x] Draft weighting towards held families (v8)
- [x] Run summary shows the build (v8)
- [x] Play-test on the iPhone (15 Oct 2026): working; balance deferred to M6

### ☑ M3 — Acts and threats (v9)
- [x] Spiked mutation (v9)
- [x] Wall layouts per act: 3 per act + open floor 1 (v9)
- [x] Beetles (Act 2, and fewer in Act 3 and the finale) (v9)
- [x] Portals and shrinking boards (Act 3) (v9)
- [x] Provisional floor 9 challenge: The Nest (v9)
- [x] Death cause on the results screen (v9)
- [x] Play-test on the iPhone (18 Oct 2026): difficulty fine for now; score-only mutations felt pointless → v10 rework

### ☐ Between M3 and M4 — mutation rework (v10)
- [x] Battery, Golden Hour, Gourmet, Cold Blood and Spiked reworked to affect progress or Brake (v10)
- [x] New: Banquet, Shortcut, Burrow, Tremor (v10)
- [x] Score out of the in-run HUD (v10)
- [ ] Play-test: do the reworked and new mutations feel meaningful?

### ☐ M4 — The Workshop
- [ ] Bits earned per run
- [ ] Workshop screen: Hardware, Cartridges, DIP switches
- [ ] Palettes and first speaker module
- [ ] Achievements
- [ ] Save export/import code

### ☐ M5 — Depth
- [ ] Rare mutations including Constrictor
- [ ] Overclock levels
- [ ] Ouroboros finale

### ☐ M6 — Balance and polish
- [ ] Tuning pass on all numbers (known: Phase + Armoured + Wraparound is near-invincible on open boards; re-check once threats exist)
- [ ] Run history and stats screen
- [ ] More music tracks and cosmetics

---

## 8. Ideas parking lot (not scheduled)

- **Daily challenge:** seed the run from the date, so the household plays the same run each day. Works offline.
- **Turn-based puzzle mode:** the world only moves when you swipe; Snakebird-style hand-made puzzles with undo. Good candidate for an unlockable mode.
- **Ghost of your best run:** record seed + turn list; replay as a faint snake. Local only (fits offline principle).
- **Tail as currency:** spend segments to place walls or bridges.
- Ideas needing a connection (shared family scores, in-game Claude rival or level designer) are **out of scope** for the installed game because of the offline principle. They could live in a separate online artifact edition if ever wanted.

---

## 9. Decision log

| Date | Decision |
|---|---|
| 13 Oct 2026 | Build as an installable PWA on GitHub Pages for offline play; artifact used as test bench |
| 13 Oct 2026 | Swipe anywhere on screen with a 28px edge dead zone |
| 13 Oct 2026 | Speed-up every 5 pickups in classic mode |
| 13 Oct 2026 | Install step fetches with `cache: 'reload'`; update banner gives "Updating…" feedback (v5) |
| 14 Oct 2026 | Game must remain fully offline; roguelike layer chosen over online features |
| 14 Oct 2026 | Moderate base speed; Brake is a core ability from the first run, upgraded via mutations |
| 14 Oct 2026 | Mutations (snake) vs Workshop (handheld) split; core game feel never locked |
| 14 Oct 2026 | M1: each floor starts on an intro card and waits for a swipe (planning time) |
| 14 Oct 2026 | M1: food keeps spawning after the exit opens — extra score for extra risk |
| 14 Oct 2026 | M1: run scoring 10 × act per food, 20 × floor per clear, +100 for a win; classic stays 1 per food so old bests still compare |
| 14 Oct 2026 | M1: Brake is Run-only; Classic stays pure. An emptied meter locks until 25% refilled |
| 14 Oct 2026 | M1: M1 work pushed to branch `m1`, merged to `main` only after a phone play-test |
| 14 Oct 2026 | v6 play-tested successfully and merged to `main` |
| 14 Oct 2026 | v7: update button reloads only once the new worker is active; root cause of dead banner taps traced to v5's touch handler |
| 14 Oct 2026 | v7 update confirmed working on the iPhone from the banner tap; Milestone 1 complete |
| 14 Oct 2026 | M2: mutations stack in levels (I–III) so the draft has upgrade depth, not just breadth |
| 14 Oct 2026 | M2: Spiked deferred to M3 (no hazards yet); new Body mutation **Lean** fills the slot and pulls against Battery |
| 14 Oct 2026 | M2: Armoured's snapped-off tail just vanishes (answers the open question) |
| 14 Oct 2026 | M2: Phase is used before Armoured when both are held, so you keep your length |
| 14 Oct 2026 | M2: released straight to `main` (no branch) after M1 went smoothly |
| 15 Oct 2026 | M3: walls are a standard part of layouts, including along the board edge. Wraparound works only through edge cells that aren't walled |
| 15 Oct 2026 | M3: beetles bounce off the snake's body as if it were a wall (herding tool; sets up Spiked and Constrictor) |
| 15 Oct 2026 | M3: board stays 16×16 for now; revisit after play-testing |
| 15 Oct 2026 | M3: portals send you out of the cell just past the pair, same direction; portal cells are never occupied. Beetles and Magnet food use them too |
| 15 Oct 2026 | M3: shrink starts at half the food target (not a timer), warns for 20 moves (so Brake buys time) and closes the outermost ring that isn't already solid |
| 15 Oct 2026 | M3: beetles move once per 2 snake moves and also bounce off food and the exit; beetle counts 1/2/3 in Act 2, 1/2 in Act 3, 2 in the finale |
| 15 Oct 2026 | M3: Spiked pays 2× / 4× the act's food value per squashed beetle, as instant points (no pickup) |
| 15 Oct 2026 | M3: floor 9 is a fixed hand-made finale (The Nest) until Ouroboros arrives in M5 |
| 18 Oct 2026 | Play-testing showed score doesn't matter to the player; finishing the run does. Mutations must affect run completion, not score. Score stays only on the results screen |
| 18 Oct 2026 | v10: reworked Battery (length → Brake capacity), Golden Hour (counts 3), Gourmet (quick bites refill Brake), Cold Blood (braking bites count double), Spiked (beetle = food) |
| 18 Oct 2026 | v10: added Banquet (more food at once), Shortcut (lower targets), Burrow (tunnel through walls, not the edge), Tremor (beetles freeze while braking). Thick Skin, Homing and a survive-a-death mutation held back as too strong for now; survive-a-death is a Rare candidate for M5 |
| 18 Oct 2026 | v10: golden food drawn as a plus sign so it can't be confused with portals |
| 15 Oct 2026 | M2 play-tested: works as expected. Mutations feel very strong (Phase + Armoured + Wraparound is near-invincible on open boards), but balance is deferred to M6 until the M3–M5 threats exist to balance against |
| 21 Oct 2026 | v11: update fixes from Pocket Factory. The update banner is now a strip above the game (it was a floating button near the home bar, which iPhone could make untappable), and updates fetch each file at `?v=<VERSION>` and check `APP_VERSION` before installing, so GitHub Pages' CDN can't hand an update stale files |
| 21 Oct 2026 | v12: installs retry the version check up to four times with fresh addresses before failing, since a failed install can leave an iPhone stuck on the old version (it happened to Flip 7 v4) |

---

## 10. Open questions

- Exact Brake trigger timing (250ms hold, 10px drift) — check on the phone that it doesn't misfire when starting a swipe slowly.
- There's no way to quit mid-floor on the phone (only "Abandon run" on the intro card). Add a pause button if it's missed.
- Should food after the exit opens be worth more (or less) than normal?
- Is 16×16 big enough once walls and beetles arrive? Staying at 16×16 for M3 (Stuart suspects it'll prove cramped); revisit after M3 play-testing.
- Bits economy: how many runs should it take to unlock Constrictor? Target roughly 4–6.
