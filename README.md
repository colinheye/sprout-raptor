# Sprout & Raptor

A one-thumb endless runner starring a blonde toddler and his velociraptor. Runs in any
modern phone or desktop browser. No build step, no dependencies, no server — the whole
game is one `index.html` with procedurally drawn art and synthesized sound.

![Gameplay](docs/preview-gameplay.png)

## How to play

| | |
|---|---|
| **Tap / click / Space** | Jump |
| **Hold** | Jump higher |
| **Tap again mid-air** | Double jump |
| **M** | Mute |

Keep the pair out of rocks, stumps, bone piles, boulders, tall rock spires and
low-swooping pterodactyls. Each egg is worth **5 m**. Your best run is saved on the device.

The game is designed for landscape. In portrait it shows a "turn your phone" prompt
(tap *play anyway* if you want to play in a narrow window).

## Try it right now

Double-click `index.html`. That's it — it works straight off the filesystem.

Or serve the folder if you want the home-screen/offline features (a service worker
needs `http(s)`):

```sh
python3 -m http.server 8000     # then open http://localhost:8000
```

## Share it with friends (free)

Any static host works, because there is nothing to compile. Pick one:

**Cloudflare Pages** — fast, free, no CLI.
1. Go to <https://pages.cloudflare.com> → *Upload assets*.
2. Drag this whole folder in.
3. You get a URL like `https://sprout-raptor.pages.dev` — send that to friends.

**Netlify Drop** — the least ceremony.
1. Go to <https://app.netlify.com/drop>.
2. Drag the folder onto the page. Done.

**GitHub Pages** — if you want it in git.
```sh
git init && git add . && git commit -m "Sprout & Raptor"
gh repo create sprout-raptor --public --source=. --push
```
Then in the repo: *Settings → Pages → Source: Deploy from branch → main / (root)*.
The game lands at `https://<user>.github.io/sprout-raptor/`.

A custom domain is the only thing that costs money (~$10–15/year). Everything above
is free at this scale, and stays free for a few hundred players.

### Send it to a phone
- A plain link is enough — no app store, no install.
- iOS Safari: *Share → Add to Home Screen* → it opens fullscreen with the raptor icon.
- Android Chrome: *⋮ → Add to Home screen / Install app*.
- Once visited, the service worker keeps it playable offline.

## Tweaking the game

Most knobs are near the top of the `<script>` in `index.html`:

| What | Where | Default |
|---|---|---|
| Jump strength / gravity | `PHYS` | `jump: 880`, `gravity: 2500` |
| Run speed and ramp | `update()` → `game.speed = clamp(420 + game.t * 15.5, 420, 1120)` | 420 → 1120 (caps at ~45 s) |
| Obstacle spacing | `spawnObstacle()` → `game.speed * rand(0.78, 1.28)` | |
| Egg frequency | `spawnEggArc()` → `game.speed * rand(2.3, 3.6)` | |
| Colours | the `C` palette object | dusk jungle |
| Character art | `drawRig()` / `drawBoy()` / `drawObstacle()` | vector, no image files |

Difficulty is governed by that one speed line plus the spawn gaps: if you raise the
top speed, raise the gaps too (they are multipliers of speed, so time-between-hazards
stays roughly constant).

## What's in here

```
index.html            the whole game (HTML + CSS + JS, no assets to load)
manifest.webmanifest  installable-app metadata (name, icon, landscape)
sw.js                 network-first offline cache
icon.svg / .png       app icons
docs/                 preview images
tests/                automated playtests (see below)
```

## Tests

`tests/` holds headless harnesses used while building; open them in a browser with the
folder served, or run them headless:

```sh
python3 -m http.server 8777
# fairness: 6 minutes of a scripted player that jumps at hazards
chrome --headless=new --virtual-time-budget=25000 --dump-dom \
  "http://localhost:8777/tests/sim.html?mode=bot&frames=21600"
# hazards really kill: same run with a player that never jumps
chrome --headless=new --virtual-time-budget=25000 --dump-dom \
  "http://localhost:8777/tests/sim.html?mode=idle&frames=21600"
# tap path through the overlays: start, jump, double jump, restart
chrome --headless=new --virtual-time-budget=15000 --dump-dom \
  "http://localhost:8777/tests/input.html"
# jump heights
chrome --headless=new --virtual-time-budget=8000 --dump-dom \
  "http://localhost:8777/tests/sim.html?mode=apex"
```

Current results: scripted player survives 6 min / 22 km / 0 deaths at max speed, an
idle player dies at the first hazard (~85 m), tap-jump / double-jump / restart all pass.

`index.html` also takes two optional query flags: `?autoplay=1` skips the menu
(add `&t=15` to fast-forward) and `?dev=1` exposes `window.__SR` for debugging and
frame-stepping.

## Not included (and what it would cost to add)

- **Global leaderboards / multiplayer** — needs a backend. Cloudflare Workers + KV is
  free at this scale; an hour or two of work.
- **Music and better SFX** — currently synthesised beeps; real audio means asset
  licensing.
- **More content** — new enemy types are ~20 lines each in `drawObstacle()` /
  `spawnObstacle()`; a second character is a new `draw*()` function.
