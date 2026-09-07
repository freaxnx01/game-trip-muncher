# Trip Muncher

A retro home-computer browser game: the family station wagon is crossing the US
on a cross-country vacation, and you are the thing that eats it. Five stages,
city to city, rendered on a 280×192 pixel canvas inside a simulated CRT monitor
and beige "OKTA-80" chassis.

Play it: <https://github.freaxnx01.ch/game-trip-muncher/>

## Controls

- **Arrows / WASD** — steer (8-way)
- **Space / Enter** — start, advance
- **M** — toggle sound
- **Touch** — drag on the screen to steer; the spider fires automatically

## Stack

Buildless static page. `index.html` is a Claude Design (`dc`) document hydrated
by the generic `support.js` runtime; all game logic is plain JavaScript drawing
immediate-mode on one 2D canvas. No build step, no dependencies to install —
GitHub Pages serves the repo root as-is.

The full design reference (colours, sprite specs, route data, timings) is in
[`docs/design-handoff.md`](docs/design-handoff.md).

## Versioning

The git tag `vX.Y.Z` is the source of truth; `version.js` mirrors it for the
in-page badge. See [`CHANGELOG.md`](CHANGELOG.md).
