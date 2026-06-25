# Pokédex Calendar

**A generative art engine that renders a different illustration every moment of the year.**

151 Pokémon across a calendar year, drawn in the browser from the current hour, the season, and the phase of the moon. Open it at dawn and you get one thing. Open it at noon and the leaves are denser, the palette warmer, the animation faster. Come back in December and the marks are sparse. That is what December looks like.

[**▶ Live viewer**](https://studiodavidpreli.github.io/pokedex_viewer.html/pokedex_viewer.html) · [SVG repository](https://github.com/StudioDavidPreli/pokemonSVG) · [Studio David Preli](https://davidpreli.com)

---

## What it is

The engine doesn't display the 151 illustrations directly. It **reads** them — maps each silhouette, finds the edges, measures how deep the body is at every point along the outline, and uses all of that to decide where things grow. The SVG is the skeleton. The engine fills it in according to the time of day you showed up.

Nothing in the output is stored. Every time you load the page the engine starts from scratch, reads the clock, and builds the image for that exact moment. The whole thing runs as a single self-contained HTML file — no server, no database, no build step.

## Four signals, one image

| Signal | Source | Controls |
|--------|--------|----------|
| **Time of day** | `getHours()` | Color temperature, bug/firefly population, animation energy |
| **Season** | `getAnnualData()` | Leaf density (peaks in summer), annual hue shift, winter marks |
| **Moon phase** | `getMoon()` | Leaf and inner-growth density, scaled from a known lunar epoch |
| **Pokémon type** | Dex lookup | Leaf shape and palette — every one of the 18 types has its own growth form |

All four route through a single `_now()` wrapper instead of calling `new Date()` directly, so the date/time picker can override the real clock and every downstream calculation responds instantly.

## How it works

1. **Date → Pokémon lookup.** 151 Pokémon distributed across 366 days by linear interpolation. Day 0 is Bulbasaur, day 365 is Mew. Runs in the browser on every load.
2. **SVG fetch and mask build.** The silhouette is fetched from raw GitHub, base64-encoded, and drawn to an offscreen 500×500 canvas. Every edge pixel gets a normal vector via a Sobel operator and a thickness value via a ray cast inward. Thin areas (ears, tails) get sparse fine marks; wide areas (torso) get dense layered growth.
3. **Time signals.** The four functions above read the current moment and return values the renderer uses directly.
4. **Generative elements.** Two leaf populations grow from the edge data — one per type, growing outward and inward along the surface normals. Each leaf is shaped by the type table, shifted by season and hour, and modulated by moon factor and local body thickness. A multi-pass Perlin topo background renders first; drifting flow marks (fireflies and flies) move along its contour lines.
5. **Animation loop.** Runs at 22fps. Each frame restores the static background, then redraws every leaf offset by a sine function seeded with its own noise value. Everything moves but nothing goes anywhere — it reads as breathing rather than animation.

Each Pokémon name is hashed to seed a `mulberry32` PRNG, so the same name always produces the same noise field. The only variation between two renders of the same Pokémon is wall-clock time.

## Controls

- **Click the canvas** — reposition the silhouette and randomize all parameters
- **Scroll** — zoom the silhouette in and out
- **Date / time picker** — jump to any moment in the year and see what it looks like

## Stack

`p5.js` · `Canvas 2D` · `SVG` · `GitHub Pages`

No framework, no bundler, no server. The viewer weighs about 58KB before the p5 library loads. The SVGs live in a [separate repository](https://github.com/StudioDavidPreli/pokemonSVG) and are fetched live from `raw.githubusercontent.com`; adding a new SVG to that repo makes it immediately available to the viewer without any other change.

## Running locally

```bash
git clone https://github.com/StudioDavidPreli/pokedex_viewer.html.git
cd pokedex_viewer.html

# serve over http so the SVG fetch works (file:// will be blocked by CORS)
python3 -m http.server 8000
# then open http://localhost:8000/pokedex_viewer.html
```

The viewer fetches its SVGs from the public `pokemonSVG` repo, so no local assets are required.

## Embedding

The viewer is embedded in the portfolio via iframe. Cross-origin height reporting uses `postMessage` — the viewer measures its own content height after the SVG loads and sends that to the parent, which resizes the iframe to match. A 50ms debounce prevents a resize loop.

---

## Philosophy

> The image you see right now does not exist anywhere. It was built when you loaded the page and it will not be built again in exactly this way.

Most images are made once. This one is made continuously. The engine is always reading its context — it doesn't know you're looking at it, but it knows what time it is.

The Pokémon framing isn't decorative. The original 151 were designed as a closed system with its own internal logic about types and habitats. Using that system as a calendar gives the year a shape it wouldn't otherwise have. February is Bulbasaur. October is around Gengar. New Year's Eve is Dragonite or Dragonair, depending on the year.

---

*Studio David Preli — [davidpreli.com](https://davidpreli.com)*
