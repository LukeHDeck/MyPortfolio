# Handoff: finish and tune the lukehdecker.com redesign locally

You are picking up a portfolio redesign that was built in a cloud session that could not reach Mapbox. Your job is to run it locally against the real Mapbox style, tune it, fix anything that looks wrong, and get it ready to merge.

## Repo and setup

- Repo: `LukeHDeck/luke-portfolio-2026` (React 19 + Vite 6 + mapbox-gl 3, deployed on Vercel at https://lukehdecker.com).
- Work on branch `claude/dazzling-edison-8w3jsh` (already pushed, 3 commits on top of `main`). Do not commit to `main`. Do not open a pull request unless Luke asks.
- Setup:
  ```
  git fetch origin
  git checkout claude/dazzling-edison-8w3jsh
  npm install
  ```
- The Mapbox token goes in `.env.local` in the project root (git-ignored): `VITE_MAPBOX_TOKEN=pk...`. Luke has the token; ask him if the file is missing. Never commit it.
- Run with `npm run dev`, which serves http://localhost:3000.

## What was built (already done)

The old site had a landing page asking visitors to pick "Explore the Globe" or "Read the Portfolio", which were two separate copies of the same content. It is now **one page of chapters scrolling over a fixed Mapbox globe**. When a chapter crosses the middle of the viewport, the camera flies to that chapter's shot.

Chapter order (in `src/data/chapters.js`):
1. Intro (`#top`)
2. Mapbox BUILD 2025 talk (`#build-2025`)
3. Six projects, each flying to its real location:
   - `#sunbridge`: Osceola County FL
   - `#lakewood-ranch`: Lakewood Ranch FL
   - `#spaceflight`: Cape Canaveral
   - `#rush-hour`: Lambertville NJ
   - `#multifamily`: a Philadelphia concept site
   - `#tectonic`: Mid-Atlantic Ridge
4. Toolkit (`#toolkit`)
5. Contact (`#contact`)

Key pieces:
- `src/data/content.js`: all copy and project data, including each project's `camera` `{center, zoom, pitch, bearing}`. This is the single source of truth.
- `src/components/Story.jsx`: the page shell. It tracks the active chapter, handles deep links, the keyboard and the lightbox.
- `src/components/map/StoryMap.jsx`: the lazy-loaded map.
  - Camera follows the chapter, with a 150 ms settle delay and a move token so a fast scroll flies once.
  - Spins on intro and contact; a slow bearing drift on projects.
  - `?debug` mode.
- `src/components/map/enhanceStyle.js`: adds fog, a graticule layer and 3D buildings, but only if the custom style (`mapbox://styles/deckdog/cmikq7w36000v01s03jrwbymo`) lacks them. It picks light or dark colors from the style's background.
- `src/components/map/markers.js` (numbered DOM markers), `src/components/map/Hud.jsx` (coordinates, zoom, bearing, scale bar, compass).
- `src/utils/mapHelpers.js`: `moveCamera`, `startSpinAnimation`, and `paddingFor`, which keeps the subject clear of the card. Its card widths mirror `src/styles/story.css`.
- **Static fallback:** no WebGL, a failed style load, or `?static` renders the same page over an SVG graticule globe, and the map chunk is never downloaded.
- **Images:** originals live in `assets/originals/`. `npm run images` regenerates the WebP files in `public/content/`, plus `src/data/image-manifest.json`, `public/og.jpg` and `public/apple-touch-icon.png`.

## Your tasks

1. **Run it with the real style and walk every chapter** at desktop width and at about 390 px phone width. Check that:
   - each fly-in lands on something worth looking at
   - the subject sits in the open space beside the card (desktop) or above it (phone)
   - nothing important is hidden behind the card, the right-hand chapter rail or the bottom-right HUD
2. **Tune the cameras.**
   - Open http://localhost:3000/?debug. You can drag, rotate and zoom; press C to copy the current camera JSON (it is also logged to the console).
   - Paste good cameras into the matching `camera` in `src/data/content.js`, or into `src/data/chapters.js` for the intro, BUILD, toolkit and contact chapters.
   - The **Sunbridge center is an estimate**; verify it against https://sunbridgefl.com/Interactive-Map/.
   - The **multifamily concept site in Philadelphia** is a placeholder. Ask Luke if he wants a different spot.
3. **Check `enhanceStyle.js` against the real style.**
   - `?debug` logs the style's layer list. If the style already has buildings or fog, the code should skip adding its own; confirm it does.
   - If the added 3D buildings or graticule look wrong on this basemap, adjust the colors or opacity, or remove them.
4. **Check the details:**
   - the Mapbox logo and attribution are visible (required by Mapbox's terms)
   - the BUILD talk video plays
   - the lightbox works: arrow keys, Esc, focus returns to the image
   - `?static` looks good
   - old links `/#/map` and `/#/scroll` land on the intro
   - no console errors
   - `npm run build` succeeds, with mapbox-gl in its own lazy chunk (`StoryMap-*.js`), not the entry chunk
5. Commit fixes to the branch with clear messages and push. Then tell Luke what you changed and where to see it. Vercel should build a preview of the branch.

## Constraints from Luke

- **Keep the look:**
  - black and white monochrome UI, Inter, his custom dark Mapbox style
  - the project imagery carries the color
  - simple, sleek, eye-catching
- **The contact section** is only the headline "Let's make maps people use." plus the "Connect on LinkedIn" button and the small links. Do **not** add anything about job seeking, "open to roles", or "if your team needs someone".
- **No em dashes** anywhere in copy or comments (an earlier commit removed them all).
- **Don't invent facts** about client projects. Sunbridge and Lakewood Ranch are Cecilian Partners client work with details "available on request".
- The Mapbox token is public (`pk.`) but stays out of git.

## Optional, if time allows

- A glowing tectonic plate-boundary layer for the tectonic chapter. It needs a plate-boundary GeoJSON such as PB2002 (Bird 2003) or fraxen/tectonicplates, kept under about 80 KB, with attribution. Load it only when that chapter is reached.
- Suggest Luke add URL restrictions to his Mapbox token (lukehdecker.com, `*.vercel.app` previews, localhost).
