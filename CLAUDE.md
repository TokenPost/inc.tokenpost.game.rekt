# REKT — working notes

Beat-the-Boss style crypto mini-game for TokenPost. One file, no build step,
no dependencies, no backend. `index.html` is the whole game.

Live: https://tokenpostinc.github.io/rekt-game/  (GitHub Pages, HTTPS enforced)
Repo: TokenPostInc/rekt-game — note the `Inc`; `tokenpost.github.io` 404s.

Design, economy, and roadmap live in `DESIGN.md`. Read it before touching
prices, coin payout, or stage unlocks.

## Hard rules

These came from the client and from bugs that cost real time. Do not relitigate.

- **No founder names anywhere on screen.** Bosses are archetypes (THE CHEF,
  THE HYPE MAN). Coin names are fine.
- **Background props deal zero damage.** One tap arms the prop, it makes a mess
  and a banner, and that is all. Keeping props out of the damage model is why
  adding scenery never reopens combat balance.
- **Art: silhouette → SVG, flat field → CSS.** Anything with a real outline is
  SVG with bezier contours and split stroke weight (outer 4.5 / detail 2), with
  deliberate asymmetry and tilt. Floors, walls and gradients stay CSS. The
  client rejects art that reads as circles-and-straight-lines ("그림체가 딱딱하다").
- **No physics engine.** Hand-written fixed-120Hz accumulator, semi-implicit
  Euler. This was evaluated against Matter.js/Rapier and deliberately rejected.
- **No spec §14 features**: PvP, guilds, rankings, daily missions, season pass,
  NFT, wallet, on-chain, token issuance, crafting, skill trees, live price feeds.
- Success criterion is one question: *do you want to switch bosses and hit them
  again?* Time-to-kill must land in 30–60s on a candle-only starter loadout.

## Architecture, in one pass

- `BOSSES` — 10 entries, each with `stage`, `hp`, one unique `mech`, one unique
  reaction, voice, and quoted lines. Stage assignment gates them behind unlocks.
- `WEAPONS` — 6, tuned to comparable *sustained* DPS. The swing lock makes slow
  weapons strictly worse if you tune raw damage instead.
- `THEMES` — drives the ground line via both `AR.ground` and the `--stage-bottom`
  CSS var. **These two must stay in lockstep** (`ratio` and `100-ratio%`).
- Ragdoll: DOM nesting *is* the bone hierarchy, so child bones inherit parent
  rotation and forward kinematics is free. Per-joint spring-damper + hard limits.
- `makeProp()` overrides `p.remove` to restore the aim cursor — props delete
  themselves under the pointer, so `pointerleave` never fires on its own.
- Save is `localStorage` only: `{coins, owned, stages, stage}`. No account.

## Gotchas that have bitten before

- **`animationend` fires for every animation on an element.** Always check
  `e.animationName`. A second `animation-name` on a class also *restarts* the
  first animation. This killed the rain cloud twice.
- **`clearProps()` wipes `#props`, which holds fixtures too.** `start()` is
  ordered `clearProps()` → … → `placeFixtures()` on purpose. Don't reorder.
- **`placeFixtures()` needs `AR.ground`**, so it must run after `measure()`.
- Spawn into `#stage` (z-index 3), not `#props` (z-index 2), for anything that
  must appear in front of the puppet.
- String-replace code injection must assert its anchor exists — a silently
  missed `s.replace` has produced "invisible" features more than once.

## How to verify (this matters — there are no unit tests)

`index.html#test` runs `selfTest()`, but it throws rather than reporting, so
for headless use replace the call with a try/catch that sets `document.title`.

Headless Chrome notes, learned the hard way:
- **Virtual time starves `requestAnimationFrame`.** Always inject
  `window.requestAnimationFrame=cb=>setTimeout(()=>cb(performance.now()),16)`
  or mechanics silently never tick and your measurements are fiction.
- Report results via `document.title` and read them with `--dump-dom`.
- Never name a probe variable that collides with a game global (`DEF`, `pick`,
  `lang`) — `pick` is a `const` and assigning to it kills the whole script.
- **Headless won't go below ~500 CSS px wide**, so phone-width layout cannot be
  verified here. Check real phones by hand.
- Time-to-kill harness must drive the real `attack()` path with a modelled miss
  rate, and must not idle — a bot that idles 50% of wall time doubles every
  number and tells you nothing.

## Current state (2026-11-11)

All 10 bosses kill in 30–60s on candle-only. All 9 background props across the
4 stages deal 0 damage and throw no phantom MISS. `selfTest()` passes, 0 JS
errors booting every stage and starting every boss.

Next planned: airport stage (+2 bosses). Deferred on purpose: login/cloud save.
There is **no analytics**, so player counts are currently unmeasurable.
