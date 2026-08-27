# OPUS-GODS-EYE-INGEST-2026-08-26-HANDOFF

**What this is** — A read-only study of `gods-eye-view` (Bilawal Sidhu's open-sourced spatial-intelligence client, formerly WorldView), read from the full git checkout of your fork, not from a capped upload.

**What it asks of Joshua** — One drop: save this file to `C:\Users\abstr\Dropbox\Joshua\OPUS-GODS-EYE-INGEST-2026-08-26-HANDOFF.md`. Nothing else.

**What it changes** — It converts "I forked a repo" into a decision-ready map: eight named patterns with file pointers and effort classes, a commercial-shipping blocklist, and the exact Windows path to first pixel.

**What happens if it's ignored** — The fork stays a bookmark. The three product shapes keep re-deriving problems this repo already solved and paid for in field tests.

**Current status** — Complete. 171 of 428 tracked files read and cited; 1,500 file:line pointers audited, zero broken; 33 named absences.

---

## A note on method, because it changes what "NOT FOUND" means

The fire card anticipated reading the fork through claude.ai project knowledge, where a file cap could silently truncate the repo. That is not what happened. The repo was present as a complete git checkout at `/home/user/gods-eye-view` — **428 tracked files, 428 files on disk, zero untracked strays; the working tree is exactly the git index** (verified: `git ls-files | wc -l`, `find` sweep).

So NOT FOUND in this report never means "didn't sync." It means **a document in this repo names a thing that does not exist in this repo**. That is a stronger claim and a more useful one.

Every claim below is tagged:

- **(verified: path:line)** — the line was read.
- **(inferred: from path)** — reasoned from named evidence.
- **(unverified)** — could not be confirmed from the tree.

Nothing was executed. `node_modules/` is absent and this box runs Node v22.22.2, outside the declared engines range (verified: `node -v`; package.json:18). So `npm test`, `npm run build`, and every performance number in this report are **read from source or read from docs, never observed**. Treat the entire build/test/perf surface as unexecuted.

---

## 2. WHAT IT IS

A browser client that draws live public signals onto a photorealistic 3D Earth, with a voice agent that can drive the whole console.

- The planet is Google Photorealistic 3D Tiles, loaded straight into CesiumJS. There is no framework — vanilla JavaScript, Cesium, Vite (verified: package.json:45-56; src/main.js:93).
- Sixteen data layers register at boot; thirteen of them appear in the README's marketing table, and ten of those need no key at all (verified: src/main.js:213-229; src/data/layerState.js:277-293; README.md:171-188).
- Every layer carries a feed-state chip that will say `DEGRADED` / `STALE` / `FALLBACK` / `UNAVAILABLE` rather than showing an empty layer as healthy (verified: src/data/manager.js:13-20, :72-108).
- Voice is the OpenAI Realtime API over WebRTC with 28 tools. The API key never reaches the browser — the client gets a short-lived ephemeral session token (verified: vite.config.js:5534-6156, :5186-5207; src/voice/gevRealtime.js:13).
- There is **no server**. Every proxy is Vite dev-server middleware living inside a single 7,383-line `vite.config.js` (verified: vite.config.js:7342-7363; `find . -name server -type d` returns nothing).

The honest framing the project gives itself: *"An evolving open-source client for exploration and learning — a fast, hackable foundation, not a hardened production service"* (verified: README.md:340).

What it is **not**: a production service, a deployable app, or a codebase with any CI. There is no `.github/` directory of any kind (verified: `find .github` returns nothing).

---

## 3. ARCHITECTURE

### 3a. The stack

- **Framework: none.** No React/Vue/Svelte, no JSX/TSX anywhere. Runtime dependencies are exactly `@mapbox/vector-tile`, `cesium`, `egm96-universal`, `mgrs`, `pbf`, `satellite.js` (verified: package.json:45-51).
- **Renderer: CesiumJS `^1.124.0`**, driven directly via `new Cesium.Viewer('cesiumContainer', {...})` with every stock widget disabled (verified: package.json:46; src/main.js:93-125).
- **Build: Vite `^6.0.0`** plus `vite-plugin-cesium ^1.2.23` (verified: package.json:55-56).
- **Node engines: `>=24.14.0 <25 || >=26 <27`** — Node 25 deliberately excluded (verified: package.json:17-18).
- **What the other deps buy:** `satellite.js` → SGP4 propagation; `@mapbox/vector-tile` + `pbf` → TomTom flow vector-tile decode; `egm96-universal` → geoid undulation for the vertical datum; `mgrs` → military grid formatting (verified: package.json:45-51; call sites for the first three verified at src/data/satellites.js, src/data/tomtomTiles.js, src/data/geoid.js:32-39; `mgrs` call site — unverified).
- **Dev deps:** `puppeteer ^24.37.5` (39 QA harnesses), `sharp ^0.34.5`, `ws ^8.21.0` (the AIS websocket bridge) (verified: package.json:53-57).

**The plugin array is the whole backend.** Twenty entries: `cesium()` plus **19 hand-written proxy plugins** (verified: vite.config.js:7342-7363).

> Reconciliation note, because two numbers circulate: the array holds **20** entries, of which **19** are hand-written proxies mounting **24** `/api/*` routes (several plugins mount more than one route). All three numbers are correct and measure different things (verified: vite.config.js:7342-7363; route census in §4).

**Dev and preview are not the same app.** Nineteen plugins implement `configureServer`; only **nine** also implement `configurePreviewServer` (verified: `grep -c` over vite.config.js). OpenSky, CCTV, TomTom, FIRMS, Overpass, GBFS, terrain, adsbdb and celestrak routes therefore 404 under `npm run build && npm run preview`. The practical consequence, which no document states: **this repo has no working production-shaped mode. It is a dev server or nothing.**

### 3b. The `src/` map

| Path | Files | What it is for | Representative file |
|---|---|---|---|
| `src/*.js` (top level) | 34 `.js` + 39 `.test.mjs` | App shell and cross-cutting singletons: bootstrap, the monolithic UI, HUD, camera/cinematics, render governor, share links, and a large family of pure "policy" modules that exist to be unit-testable | `src/ui.js` — 10,293 lines / 455 KB |
| `src/annotations` | 7 `.js` + 5 `.test.mjs` | The voice whiteboard: resolve names to world anchors, hold mark state with TTL fade, route geometry to a world-space or screen-space renderer | `src/annotations/hybridAnnotationRenderer.js` |
| `src/data` | 87 `.js` + 101 `.test.mjs` | One module per layer, plus the layer manager, the share/persistence codec, the detection mesh, the context store, per-source adapters | `src/data/manager.js` — 2,288 lines |
| `src/data/local_data` | 14 files, 5 dataset dirs | Bundled datasets, each with its own provenance README | `dams/`, `datacenters/`, `natural_earth/`, `neighborhoods/`, `telegeography_submarine_cables/` |
| `src/overlays` | 3 `.js` + 3 `.test.mjs` + 1 harness | One shared DPR-aware 2D canvas every layer paints labels and cards onto | `src/overlays/worldOverlay.js` — 2,364 lines |
| `src/scenes` | 3 `.js` + 2 `.test.mjs` | Deterministic cinematic playback: shot-list director, canned recipes, pure reconcile decisions | `src/scenes/director.js` — 1,391 lines |
| `src/styles` | 6 `.js`, **0 tests** | GLSL post-process presets, each `{name, uniforms, fragmentShader}` | `src/styles/thermal.js` |
| `src/voice` | 3 `.js` + 4 `.test.mjs` | The Realtime WebRTC session, the action runner, a pure cost/tier model | `src/voice/gevRealtime.js` — 2,610 lines |

(verified throughout: `find`/`wc -l` sweeps; representative files read.)

Three things worth knowing about this map:

- **The 28 voice tool schemas are not in `src/voice/`.** They live server-side at `vite.config.js:5534`. `src/voice/gevActions.js` holds only the client-side handlers (verified: vite.config.js:5534, :5180). README.md:254 places them in `src/voice/`; CONTRIBUTING.md:28 gets it right. The two docs contradict each other.
- **`src/ui.js` is the adoption liability.** 10,293 lines holding `StyleManager`, `OrbitController`, `IntelHUD` construction, the post-process stage registry, the `LayerStateCoordinator` wiring, and the entire panel system (verified: `wc -l`; src/ui.js:2113, 2393-2397, 2946-2982, 4497-4511). If you fork rather than lift, you own this file.
- **The README's own `src/` tree omits `annotations/`, `overlays/`, and `styles/` entirely**, and lists only 5 of the 34 top-level files (verified: README.md:248-258; `grep` returns no match for those three paths in README.md).

### 3c. Entry point

`index.html` has exactly one script tag and no inline module (verified: index.html:897):

```html
<script type="module" src="/src/main.js"></script>
```

`src/main.js` is 336 lines. `initLogoGaze()` fires at module scope; everything else is one `async function init()`. The order that matters (all verified in src/main.js):

1. **Loading screen grabbed** — `#loading-screen` + `.loader-status`; every stage writes a status string into it (`70-71`).
2. **Cesium ion token** applied only if present (`77-80`).
3. **Google Maps key is fatal if missing** — `throw new Error('GOOGLE_MAPS_API_KEY not found…')` (`83-86`). This is the single hardest gate in the app.
4. **Cesium viewer constructed** with all stock widgets off, `baseLayer: false`, `msaaSamples: 4`, and a hand-built `#cesium-credits` div appended to `<body>` because Google's ToS requires visible attribution (`93-125`).
5. **`registerDataCredits(viewer)`** — static per-layer license credits (`141`).
6. **Google Photorealistic 3D Tiles**, and this is a *soft* dependency — on failure it warns and falls back to the Cesium globe rather than aborting (`160-173`).
7. **`MapStackController`** with `initialStack: tileset ? 'photoreal' : 'osm'` (`177-191`).
8. **`StyleManager`** — the big one; its constructor builds `OrbitController`, `IntelHUD`, the cockpit controller, then runs an init cascade (`194`).
9. **`DataLayerManager`**, then registration (`210-229`).
10. **`finalizeRegistrations(LAYER_STATE_REGISTRY)`** — seals the registry (`231`).
11. **Toggle panel built, then `styleManager.attachDataManager(dataManager)`** — the second call constructs the `LayerStateCoordinator` and starts share/local restoration (`242-243`).
12. **`installRenderGovernor(viewer)` deliberately last**, so every module above can register pre-install continuous-render holds (`277`).
13. **One try/catch around all of it** — a throw ends with red error text in `.loader-status`, never a blank screen (`329-333`).

**The layer-count reconciliation, settled.** There are 13 textual `dataManager.register(...)` call sites in `main.js`: twelve named layers (`213-225`) plus one inside a loop over the 4-element `localDataLayers` array (`228`) (verified: src/main.js:213-229; src/data/localLayers.js:47-52). That is **16 registered layers**, exactly matching `LAYER_STATE_REGISTRY`'s 16 frozen entries (verified: src/data/layerState.js:277-293). Of those, **15 appear in the toggle panel** — `militaryAwareness` sets `showInTogglePanel: false` because it is the Contacts engine, not a user toggle (verified: src/data/militaryAwareness.js:1724, honored at src/data/manager.js:2026). The README's "Thirteen live layers" is a marketing table whose first row is Map Stack, which is not a data layer at all (verified: README.md:171-188).

### 3d. The module-wiring contract — the most reusable thing in the repo

A layer is a plain object with a stable `id` and duck-typed lifecycle methods. Nothing is subclassed. `src/data/earthquakes.js` is the minimal reference case (verified: src/data/earthquakes.js:131-136):

```js
const layer = {
  id: 'earthquakes', name: 'Earthquakes (24h)', source: 'USGS', updateInterval: 60000,
  init(viewer) {…}, enable(viewer) {…}, disable(viewer) {…},
  async update(viewer) {…}, destroy(viewer) {…}, getStats() {…},
};
```

The parts worth stealing wholesale:

- **Registration is sealed.** `register()` throws after `finalizeRegistrations()` (verified: manager.js:132-137).
- **The seal is a bidirectional proof.** `finalizeRegistrations(LAYER_STATE_REGISTRY)` computes `missing` (registered layers with no URL disposition) and `extra` (dispositions with no layer) and **throws on either** (verified: manager.js:219-240). Net effect: you cannot add a layer without also giving it a share-link token, and you cannot leave a stale token behind. Adding a layer is a two-file change by construction.
- **Returning `false` from any lifecycle method is a rejection, not a value.** Each enable phase does `if (x === false) throw lifecycleRejectedError(layerId, phase)` (verified: manager.js:33-37, 863, 894, 908).
- **A per-entry serialized `toggleChain`** exists to stop double-armed intervals. The comment names the exact bug: *"the interval is armed after the user already turned the layer off, and a subsequent enable arms a SECOND interval → 2× poll → OpenSky 429"* (verified: manager.js:195-200).
- **Failure leaves honest state.** A failed enable compensates with `disable()` and sets `entry.lifecycleUncertain = !cleanupConfirmed` — the manager admits when the real state is unknowable (verified: manager.js:838-844).
- **`origin` is a first-class concept.** `'user' | 'voice' | 'tool'` are "explicit intent" and are the only origins that cancel a pending restore and get persisted (verified: manager.js:43-45).

### 3e. TRACE ONE LAYER END TO END — aircraft, poll to pixel

Ten hops. The civil-flights layer is `src/data/flights.js` (5,344 lines) plus fourteen collaborator modules and one Vite middleware.

**a. Where the poll is scheduled**

- The layer declares its own cadence: `updateInterval: 30000` (verified: src/data/flights.js:3896).
- The manager arms it: `entry.intervalId = setInterval(…, refreshInterval)` (verified: src/data/manager.js:522-531). No `refreshInterval` is declared, so **30 s is the live period**.
- Enable runs one immediate fetch first, then arms the interval (verified: manager.js:905-909, :934).
- Every tick explicitly asks for a frame and re-arms detection, because in idle-render mode a poll would otherwise never paint: `governorRequestRender('layer-tick:…')` and `markDetectionSourcesChanged(…)` (verified: manager.js:315, :321).

**b. Server side — `/api/opensky`, the credit governor, the fallback**

- Middleware at `server.middlewares.use('/api/opensky', …)` (verified: vite.config.js:2914); upstream `https://opensky-network.org/api/states/all?extended=1` (verified: vite.config.js:3009).
- **The credit governor exists because of a real burn.** The block comment: *"The global /states/all this proxy fetches costs 4 CREDITS per call against OpenSky's ~4000/day authenticated budget — a day with the app open burned the whole quota in ~8h"* (verified: vite.config.js:89-93). Three levers:
  - Adaptive TTL stepped by the `X-Rate-Limit-Remaining` header: `>2400 → 9 s`, `>1200 → 30 s`, `>400 → 90 s`, else `300 s` (verified: vite.config.js:113-120).
  - 429 cooldown honoring `x-rate-limit-retry-after-seconds`, clamped 30 s–30 min (verified: vite.config.js:3051-3056).
  - Serve-stale during cooldown with `X-OpenSky-Cache: STALE` rather than propagating the 429 (verified: vite.config.js:3058-3068).
- Only `upstream.ok` responses are cached (verified: vite.config.js:3125-3132).
- **The adsb.lol fallback has four distinct triggers** — snapshot older than 120 s, cold start inside a cooldown, a fresh 200 whose snapshot is already stale, and upstream failure with no cache (verified: vite.config.js:2925-2931, :2958, :3028-3040, :3074-3079).
- The fallback is **viewport-scoped, not worldwide**: `api.adsb.lol/v2/lat/{lat}/lon/{lon}/dist/250`, anchor snapped to 0.25° (verified: vite.config.js:2811-2825).
- It is **re-shaped into OpenSky's own state-vector array** server-side, with knots→m/s and feet→m conversions, so the renderer never learns it is on a different feed except through headers `X-Flight-Source` / `X-Flight-Coverage` (verified: src/data/adsbLolFallback.js:37-71; vite.config.js:2873-2874).

**c. Client fetch and parse**

- The URL carries the camera subpoint, which is what gives the server its fallback anchor: `new URLSearchParams({ lat, lon })` (verified: src/data/flights.js:325-337).
- Provenance is read straight off the headers into layer state (verified: src/data/flights.js:4164-4165).
- `data.states` is an array of positional arrays, destructured with the index map documented inline (verified: src/data/flights.js:4185-4199).
- **Sticky merge.** Missing fields hold last-known-good: *"OpenSky intermittently drops callsign/velocity/track for aircraft it still positions — hold last-known-good instead of regressing to the ICAO hex / a 0° (north) heading"* (verified: src/data/flights.js:4220-4223; src/data/aircraftMeta.js:10-21).
- **Freshness is dated from the source snapshot, not receipt**: *"Freshness belongs to the source snapshot, not the moment this browser received a cached 200 response"* (verified: src/data/flights.js:4613-4615).

**d. The deliberate one-interval render lag — the constant, quoted**

```js
// Render-behind smoothing (PRD WS-C C2 — approved product decision):
// the fleet renders at now - RENDER_DELAY_SEC so positions interpolate
// BETWEEN two known fixes instead of extrapolating ahead and snapping back
// when the next poll lands. All consumers (labels, HUD, detection,
// frame_overhead) share this delayed clock; removing the delay reintroduces
// the back/forward oscillation and is a regression.

/** @constant {number} Display latency in seconds (= one poll interval). */
const RENDER_DELAY_SEC = 30;
```

(verified: src/data/flights.js:589-599.) The 30 is `updateInterval: 30000` restated in seconds, and the JSDoc says so. The delayed clock also governs the trail warm-up gate, the trail seed, the trail head/body split, and the ground-corridor lookahead (verified: src/data/flights.js:1236-1242, :2967-2972, :2946-2954, :1817).

**e. Interpolation and dead reckoning**

- Position history is stamped with the **feed's** fix epoch: *"OpenSky positions arrive 5-15s stale and receipt-time stamping is what caused the back/forward oscillation"* (verified: src/data/flights.js:4408-4416).
- `_deadReckon` has three branches (verified: src/data/flights.js:1099-1196):
  - **Bracketing pair → straight interpolation.** Scan history newest-first for `a.time ≤ renderTime ≤ b.time`, then `Cartesian3.lerp`. This is the common case and the whole point of the lag.
  - **Warm-up** → extrapolate the *oldest* fix backward, capped at 60 s. The comment records both rejected alternatives: *"Holding the oldest fix froze the icon; extrapolating the NEWEST fix to wall-clock now made the icon jump back ~one poll interval."*
  - **Coasting** → forward dead reckoning bounded by `staleCoastLimitSeconds`, keyed on `last_contact` rather than `time_position` so an aircraft still sending velocity does not hard-freeze.
- The projection is a **constant-rate-turn arc**, not a straight line: *"a plane in a standard-rate turn is ~90° of arc wrong per 30 s if extrapolated straight"* (verified: src/data/flights.js:1216-1217; `arcOffsetEnu` at src/data/motionModel.js:354).
- A late kinematics change never rewrites history — it appends a synthetic forward fix, because *"mutating the historical fix reprojects the entire stale interval and snaps the rendered aircraft"* (verified: src/data/flights.js:4451-4467).

**f. Altitude — why planes sit on aprons instead of floating**

The priority chain is a separate pure module so both flight layers share it verbatim (verified: src/data/renderAltitude.js:29-40):

```js
export function pickRenderAltitudeM({ geoAltM, baroAltM, onGround, surfaceM, geoidN }) {
  if (onGround && Number.isFinite(surfaceM)) return surfaceM;
  if (Number.isFinite(geoAltM)) return geoAltM;
  if (Number.isFinite(baroAltM)) return baroAltM + (Number.isFinite(geoidN) ? geoidN : 0);
  return null;
}
```

- `geo_altitude` is already WGS84 ellipsoidal and used verbatim; `baro_altitude` is MSL and only a visual fallback, corrected by the geoid undulation N. The module explicitly disclaims exactness (verified: src/data/renderAltitude.js:4-12).
- N comes from bundled EGM96 via a lazy dynamic import so the ~2.7 MB grid stays out of the eager bundle (verified: src/data/geoid.js:32-39).
- The single choke point is `cachedGroundFloor` — **rendered-mesh cell first, then real DEM cell**, deliberately rejecting geoid-fallback entries because of *"the 'sea-level poison' … at Austin sits ~165 m below the airport"* (verified: src/data/groundFloor.js:473-479; src/data/terrainHeights.js:71-84).
- Mesh probes exclude aircraft billboards and models so *"a vertical probe can't land on an aircraft instead of the pavement"* (verified: src/data/flights.js:4589-4591).
- Field evidence is recorded in situ: *"SWA696 showed 450 ft at Austin's 542 ft field elevation … the whole fleet sat buried at AUS in 2D"* (verified: src/data/flights.js:4325-4330).
- **Known limitation, documented but not in the README:** grounded aircraft float low for 1–2 poll cycles at a cold airport, and cold-start floor latency runs 30–60 s at fresh fields (verified: docs/KNOWN-ISSUES.md:55-67).

**g. Heading — world-stable icons**

Billboards are camera-facing quads, so real-world course has to be solved in screen space. The fix builds a 2000 m forward **vector** in local ENU, transforms it to world, and projects onto the camera basis (verified: src/data/iconOrientation.js:53-79):

```js
const dx = Cesium.Cartesian3.dot(_scratchWorldForward, camera.rightWC);
const dy = -Cesium.Cartesian3.dot(_scratchWorldForward, camera.upWC);
return Math.atan2(-dx, -dy);
```

Projecting the *vector* rather than a probe *point* is what survives a behind-camera forward point during a >180° tracked orbit. The module header names the bug it replaced: *"broke in tracked-entity orbit mode, where camera.heading is expressed in the entity's reference frame — the tracked icon stayed glued to the viewport"* (verified: src/data/iconOrientation.js:7-15). The fleet rotation pass runs only on camera-pose change or once a second; the tracked icon is per-frame via a `CallbackProperty` (verified: src/data/flights.js:2637, :3608-3622).

**h. Glyph → 3D model**

Two independent regimes that deliberately disagree.

- **Fleet:** camera altitude below `MODEL_ALT_CEIL_M = 800000` m, then distance plus caps `MODEL_MAX = 150` / `MODEL_MAX_ALL = 350`, with add/keep hysteresis and four ordered passes so *"a retained off-screen model can never starve an on-screen plane"* (verified: src/data/flights.js:1511-1514, :164-177, :2703-2710).
- **Tracked contact:** a separate hysteretic latch at `TRACKED_MODEL_ENTER_ALT_M = 150_000`, exit at ×1.15. A first pass at 1,000,000 m was rejected in field test — *"the model 'pops to 3D far too early' — a 26 m airframe held at its minimum pixel size from ~1 Mm out reads as a floating toy"* (verified: src/data/trackedModelRegime.js:15-21, :54-61). The module flags the resulting inversion itself: between 150 km and 800 km the surrounding traffic is 3D while the selected plane is still a billboard.
- **The swap is gap-proof.** The billboard stays shown until the model actually renders, and the matrix is written *before* the readiness test because *"a first rendered frame on a stale load-start matrix is the one-frame jump this ordering prevents"* (verified: src/data/flights.js:1710-1734).

**i. What Cesium actually draws**

- **Fleet:** one `BillboardCollection` — *"all aircraft are drawn as billboards in a single BillboardCollection for GPU-efficient batching (handles 5000+ aircraft)"* (verified: src/data/flights.js:6-8, :3908-3910).
- **Models:** `Cesium.Model` instances in a `PrimitiveCollection`; every model is its own draw call — *"no instancing yet"* — which is what the caps exist for (verified: src/data/flights.js:162-163, :3911).
- **Tracked contact:** a pure billboard `Entity` with **no model graphic**, because `viewer.trackedEntity` derives follow-camera framing from the bounding sphere and a glTF graphic reports PENDING until loaded (verified: src/data/flights.js:3570-3579). It hides by going transparent, not `show = false`, so the sphere stays ready.
- **Trail:** entity polylines with a `depthFailMaterial`, plus a cheap two-point head entity bridging the last body point to the live icon so the 400-point body only rebuilds at poll cadence (verified: src/data/trailRenderer.js:66-85; src/data/flights.js:2985-3000).
- **The tick:** `scene.preRender` self-throttled to ~12 Hz (`FLEET_DR_INTERVAL_MS = 80`). Position writes are gated at 1 m² because *"assigning Billboard.position dirties the whole collection's vertex buffer"* (verified: src/data/flights.js:2628, :2730-2733).

**j. The freshness label — where the vocabulary actually lives**

**The enum is defined in `src/data/manager.js`, not in the flights layer** (verified: src/data/manager.js:13-20, :70):

```js
const FEED_STATE_LABELS = Object.freeze({
  nominal: 'ON', loading: 'LOADING', degraded: 'DEGRADED',
  stale: 'STALE', fallback: 'FALLBACK', unavailable: 'UNAVAILABLE',
});
```

- `layerFeedState(stats)` normalizes heterogeneous per-layer stats into those six words, in strict precedence order, with an explicit carve-out so guidance states (`zoom-in`, `empty`, `idle`) read as normal operation rather than faults — *but a genuinely stale cache still reads STALE* (verified: src/data/manager.js:72-108, comment at :86-89).
- **The adsb.lol tell is a string sniff.** `(!hasExplicitFallback && /\badsb\.lol\b/i.test(source))` — that string arrives from the proxy header `X-Flight-Source`. This is the entire mechanism by which the UI knows the layer degraded to the regional feed (verified: src/data/manager.js:100; vite.config.js:2873; src/data/flights.js:4164).
- Rendering: `_syncToggleButton` sets the label, a `data-feedState` attribute, and one `feed-<state>` class; `_buildMetaText` composes `STATE · source · error · retry Ns` (verified: src/data/manager.js:2254-2276, :2210-2251).
- Voice consumes the same function and phrases refusals from it — *"is enabled, but ${feed.source} is unavailable"* (verified: src/voice/gevActions.js:503, :526-528).

**The README's freshness vocabulary is wrong.** README.md:39 says *"including partial, delayed, simulated, and unavailable states."* Of those four words only `unavailable` is a feed state (verified by exhaustive grep):

| Literal | Where it actually lives |
|---|---|
| `'RECONSTRUCTED ESTIMATE'` | **`src/data/rocketLaunches.js:191` only** — the space-missions ascent-provenance label. Nothing to do with aircraft. |
| `'simulated'` / `'sim'` | The **traffic** layer: `_uncoveredMode = 'sim'` (src/data/traffic.js:219-220), mapped to `'fallback'` by `layerFeedState` (src/data/manager.js:98). |
| `'unavailable'` | A genuine enum member (src/data/manager.js:19, :70). |
| `'partial'` | Not a feed state — a satellite refresh outcome (src/data/satellites.js:1751). |
| `'delayed'` | **No occurrence anywhere in `src/` as a state string.** All 57 case-insensitive hits are prose in comments. The concept exists as the render-behind clock; the word is never a label. |

There are in fact **three freshness tiers**, and only the first is the enum:

1. The six chip words above.
2. Per-layer label strings layered on top — `KEY REQUIRED`, `LIVE · updated Xm ago`, `STALE · cached Xh` (src/data/firmsHeatmap.js:304-315); `SIMULATED — add TomTom key for live` (src/data/traffic.js:1278-1279); `RECONSTRUCTED ESTIMATE` / `ASCENT REPLAY` (src/data/rocketLaunches.js:191, :2685).
3. `src/loadingFeedback.js:25` adds `keyRequired` as a first-class field, which is **outside** the six-word enum.

**The one-paragraph version.** A 30 s interval calls `update()`, which fetches `/api/opensky?lat=…&lon=…`. A Vite middleware serves it from a 9 s–300 s cache tuned to OpenSky's remaining daily credits, honors 429 cooldowns, serves last-good on failure, and when the snapshot is >120 s old silently substitutes a 250 nm adsb.lol snapshot re-shaped into OpenSky's own array format, flagged only by a header. The client keeps ~20 fields per aircraft with sticky merging, stamps each fix with the feed's own epoch, and resolves render height through geo-altitude → baro+geoid → warm mesh/DEM cell. A `preRender` tick at 12 Hz renders every plane at **now − 30 s** so it interpolates between two known fixes rather than extrapolating and snapping back, falling to constant-rate-turn dead reckoning when a fix is missing. Icon rotation is a camera-basis projection of the world course vector. Below 800 km the nearest 150 billboards hand off to glTF models, but only once the model actually renders, while the selected plane waits until 150 km on its own latch. The chip above it reads ON / LOADING / DEGRADED / STALE / FALLBACK / UNAVAILABLE.

---

## 4. THE DATA FEEDS TABLE

`DATA_SOURCES.md` is the authority. Every row below was cross-checked against the module that actually calls the endpoint.

Key column: **none** = works with no account · **free** = no-cost registration · **metered** = billing-enabled account.

### Map stack

| Layer | Feed / endpoint | Provider | Protocol | Key | License / terms flag | Code pointer |
|---|---|---|---|---|---|---|
| Google Photorealistic 3D Tiles | `createGooglePhotorealistic3DTileset` (Map Tiles API) | Google | OGC 3D Tiles, **client-direct** | **metered — required**; app throws without it | Google Maps Platform ToS. Content may not be cached/stored/rehosted. On-globe credit mandatory | src/main.js:160, :83-90; vite.config.js:7374 |
| Bing Aerial | `createWorldImageryAsync` via Cesium ion | Bing via ion | ion imagery, client-direct | **free** (optional); stack disabled without it | ion plan terms; token client-exposed by design | src/mapStackController.js:12-26, :258 |
| OSM raster | `tile.openstreetmap.org` | OpenStreetMap | XYZ raster, **client-direct** | **none** | ⚠️ ODbL 1.0 + OSMF tile policy | src/mapStackController.js:260-261 |
| Terrain (keyless) | `terrain.reearth.land/cesium-mesh/ellipsoid` | Re:Earth / Mapterhorn | quantized-mesh, client-direct | **none** | ⚠️ CC BY 4.0 (mesh); EGM2008 public domain | src/mapStackController.js:45, :324-327 |
| Terrain heights | `/api/terrain/heights` → Re:Earth | Re:Earth | JSON via proxy, 30-day disk cache | **none** | as above | src/data/terrainHeights.js:102; vite.config.js:2282 |

### Live feeds

| Layer | Feed / endpoint | Provider | Protocol | Key | License / terms flag | Code pointer |
|---|---|---|---|---|---|---|
| Live Flights (primary) | `/api/opensky` → `opensky-network.org/api/states/all` | OpenSky | REST via proxy; OAuth/Basic/anon | **none** anon; **free** OAuth | 🔴 **NonCommercial**; operational REST use may require a prior written agreement | src/data/flights.js:273; vite.config.js:2914, :3009 |
| Live Flights (fallback) | `api.adsb.lol/v2/lat/…/dist/250` | adsb.lol | REST via proxy, 250 nm cap | **none** | ⚠️ ODbL 1.0 | vite.config.js:2825, :2863-2876 |
| Flight track history | `/api/opensky-track` | OpenSky | REST via proxy | **none** | as OpenSky | vite.config.js:4896 |
| Flight enrichment | `/api/adsbdb/{route,type}` → `api.adsbdb.com` | adsbdb.com | REST via proxy, 24 h TTL, persisted to `.gev-cache/` | **none** | 🚩 **Undocumented** — absent from DATA_SOURCES.md, README, and DATA_CREDITS | vite.config.js:2416, :2388-2389 |
| Military Flights | `/api/adsblol/mil` → `api.adsb.lol/v2/mil` | adsb.lol | REST via proxy, 12 s cache | **none** | ⚠️ ODbL 1.0 | vite.config.js:4718, :4726 |
| Military traces | `/api/adsblol/trace` | adsb.lol | tar1090 trace JSON, 60 s cache | **none** | ⚠️ ODbL 1.0 | vite.config.js:4920, :4933 |
| Live Vessels | `/api/ais-live` ← `wss://stream.aisstream.io` | AISStream.io | server holds the **WebSocket**; browser polls JSON | **free** (server-side only) | Free beta, **no formal ToS** | src/data/aisLiveVessels.js:55; vite.config.js:4760 |
| Satellites | `/api/celestrak/:group` | CelesTrak | TLE text via proxy, 6 h TTL | **none** | US-gov origin, no license; citation requested | src/data/satellites.js:1096; vite.config.js:1543 |
| Earthquakes | `earthquake.usgs.gov/…/all_day.geojson` — **client-direct, no proxy** | USGS | GeoJSON from the browser | **none** | U.S. public domain | src/data/earthquakes.js:28 |
| Traffic — roads | `/api/overpass` → 4 Overpass mirrors | OpenStreetMap | Overpass QL POST, 24 h/7 d/30 d TTL | **none** | ⚠️ ODbL 1.0 | src/data/traffic.js:45; vite.config.js:2589 |
| Traffic — live flow | `/api/tomtom/flow/{z}/{x}/{y}.pbf` | TomTom | MVT protobuf via proxy, 120 s TTL, daily budget governor | **BYOK**; keyless → simulation | TomTom developer terms (proprietary) | src/data/traffic.js:1293; vite.config.js:1847 |
| CCTV — Austin | `/api/cctv/*` → `data.austintexas.gov` + `cctv.austinmobility.io` | City of Austin | JSON catalog + JPEG frames, 15 min cache | **none** | City of Austin Open Data ToU | src/data/cctv.js:106-109; vite.config.js:4494 |
| CCTV — Caltrans | `cwwp2.dot.ca.gov` districts 4,7,11,3 | Caltrans | JSON + images, origin-pinned | **none** | Public Caltrans data (courtesy) | vite.config.js:3426-3430, :3985 |
| CCTV — TfL JamCams | `api.tfl.gov.uk` + TfL S3 | Transport for London | JSON + images via proxy | **none**; optional **free** `TFL_APP_KEY` | ⚠️ **Attribution REQUIRED** — "Powered by TfL Open Data. Contains OS data © Crown copyright and database rights" | vite.config.js:3439-3440, :4027-4034 |
| CCTV — frame fallback | `maps.googleapis.com/maps/api/streetview` | Google Street View Static | image via proxy, `no-store`, never persisted | **metered** — reuses the Maps key | Google ToS. 🚩 **Undocumented** in DATA_SOURCES.md | vite.config.js:4459-4488 |
| Radio | `/api/radio/stations`, `POST /api/radio/click/:uuid` | Radio Browser | REST via proxy; audio plays **direct from the broadcaster** | **none** | Directory **PDDL 1.0**; broadcaster stream terms apply; direct playback exposes listener IP | src/data/radio.js:37; vite.config.js:1254 |
| Bikeshare | `/api/gbfs/{encoded}` → 8 allowlisted hosts | GBFS operators | GBFS JSON via proxy | **none** | Per-feed, attribution-only | src/data/bikeshare.js:528; vite.config.js:3249 |
| Active Fires | `/api/firms` → 3 VIIRS NRT sources | NASA FIRMS | CSV via proxy, merged, 24 h clamp, 30 min TTL | **free** (server-side); layer empty without it | CC0 / U.S. public domain; EOSDIS acknowledgement requested | src/data/firmsHeatmap.js:39; vite.config.js:2101 |
| Space Missions | `/api/launches` → `ll.thespacedevs.com/2.3.0` | Launch Library 2 | REST via proxy, 15 min cache, serve-last-good | **none** (15/hr); optional **free** token | TSD terms: use/share freely, don't forward without added value | src/data/rocketLaunches.js:16; vite.config.js:1680 |
| Mapped Installations | `/api/military-installations` → Overpass | OpenStreetMap | Overpass QL, ≤10° bbox, 700-element cap | **none** | ⚠️ ODbL 1.0; must be labeled incomplete mapped context | src/data/militaryInstallations.js:422; vite.config.js:6806 |
| Installation candidates | `/api/google/{text-search,nearby-places}` | Google Places | REST via proxy | **metered** | Google Maps Platform ToS | vite.config.js:5276, :5390 |
| Cockpit place label | `/api/regional-brief` → Nominatim | OSM Nominatim | REST via proxy, ≤1 req/s | **none** | ⚠️ ODbL 1.0 + Nominatim usage policy | vite.config.js:7134, :7032 |
| Cockpit headlines (primary) | `news.google.com/rss/search` | Google News | RSS via proxy | **none** | 🔴 **Personal, noncommercial use only** | vite.config.js:7054 |
| Cockpit headlines (fallback) | `api.gdeltproject.org` | GDELT | REST via proxy, only when RSS fails | **none** | Unrestricted incl. commercial, **with citation + link** | vite.config.js:7070 |
| Cockpit weather + WX | `/api/weather-effects` → Open-Meteo | Open-Meteo | REST via proxy, 0.1° cells, 5 min TTL | **none** | ⚠️ **CC BY 4.0 with adjacent-link attribution** | vite.config.js:7208, :6906 |
| Walking / driving routes | `/api/route` → `routing.openstreetmap.de` | FOSSGIS OSRM | REST via proxy, 10 min cache | **none** | ODbL 1.0 + FOSSGIS policy. 🚩 **Undocumented** | vite.config.js:2706, :2760 |

### Bundled static datasets — counts verified byte-for-byte against the files

| Layer | File | Provider | Key | License flag | Code pointer |
|---|---|---|---|---|---|
| Submarine Cables (**712** cables + **1,917** landing points) | `telegeography_submarine_cables/*.json` | TeleGeography | **none** | 🔴 **CC BY-NC-SA 3.0 — NonCommercial + ShareAlike.** Not covered by MIT | src/data/telegeographySubmarineCables.js:12-19 |
| Datacenters (**4,351**) | `datacenters/datacenters.geojsonl` | OSM extract | **none** | ⚠️ **ODbL 1.0** | src/data/localLayers.js:6, :13-23 |
| Dams (**704**) | `dams/dams.geojsonl` | OpenInfraMap / OSM | **none** | ⚠️ **ODbL 1.0** + Open Infrastructure Map credit | src/data/localLayers.js:7, :25-30 |
| Natural Earth (**1,046** land + **292** marine) | `natural_earth/{regions,marine}.json` | Natural Earth 10m | **none** | Public domain | src/data/naturalEarthRegions.js:122 |
| SF Neighborhoods (**41**) | `neighborhoods/san-francisco.json` | DataSF `j2bu-swwd` | **none** | **PDDL 1.0** — public domain | src/data/neighborhoodPolygons.js:17 |
| 3D models (**9** `.glb`) | `public/models/*.glb` | Sketchfab creators | **none** | ⚠️ **CC BY 4.0** — attribution + license link + modification notice; commercial OK | public/models/README.md:9-20 |

(All counts verified by direct count of the files: `wc -l` on the JSONL, `"type":"Feature"` counts on the JSON. Every count DATA_SOURCES.md asserts is correct.)

### The 24 `/api/*` routes

`/api/radio` :1254 · `/api/celestrak` :1543 · `/api/launches` :1680 · `/api/tomtom` :1847 · `/api/firms` :2101 · `/api/terrain/heights` :2282 · `/api/adsbdb` :2416 · `/api/overpass` :2589 · `/api/route` :2706 · `/api/opensky` :2914 · `/api/gbfs` :3249 · `/api/cctv` :4494 · `/api/adsblol/mil` :4718 · `/api/ais-live` :4760 · `/api/opensky-track` :4896 · `/api/adsblol/trace` :4920 · `/api/openai/hud-summary` :4962 · `/api/realtime/debug-log` :5020 · `/api/realtime/token` :5045 · `/api/google/nearby-places` :5276 · `/api/google/text-search` :5390 · `/api/military-installations` :6806 · `/api/regional-brief` :7134 · `/api/weather-effects` :7208 (all verified as `middlewares.use` registration lines in vite.config.js).

Two layers bypass the proxy entirely and fetch cross-origin from the browser: **Earthquakes** and the **OSM / Re:Earth map-stack tiles**. Nothing in the tree states whether that is intentional (unverified).

### Restricted sources, called out in their own right

- 🔴 **TeleGeography submarine cables — NonCommercial.** The only source in the tree that forbids commercial use outright. The folder README is unambiguous: *"the data may not be used commercially. If you use this project commercially, **remove these files** … or obtain a commercial license from TeleGeography"* (verified: telegeography_submarine_cables/README.md:14-16). **But its machine-readable sidecar is weaker than its prose README** — `source.json:7`'s `license_note` never names CC BY-NC-SA at all (verified). Any tooling that reads `source.json` will under-report the restriction.
- 🔴 **OpenSky — NonCommercial**, and *"operational use of the REST API in a live product can require a prior written agreement with OpenSky — even for non-profit/government use"* (verified: DATA_SOURCES.md notes). The in-app credit already carries `(non-commercial)` (verified: src/data/dataCredits.js:36).
- 🔴 **Google News RSS — personal, noncommercial only.** *"commercial deployments must disable/replace it or obtain separate permission"* (verified: DATA_SOURCES.md cockpit note).
- ⚠️ **ODbL 1.0 covers seven surfaces, not two**: bundled datacenters and dams, plus five live feeds — adsb.lol (three routes), Overpass roads, Overpass installations, Nominatim, OSM raster tiles, and FOSSGIS OSRM.
- ⚠️ **Google Maps Platform — no caching.** Tiles, geocodes and Places results *"may not be cached, stored, rehosted, or committed."* The code honors this: Street View fallback frames go out with `Cache-Control: no-store` and are never persisted (verified: vite.config.js:4664-4665).

### Where DATA_SOURCES.md and the code disagree — ranked

1. **`api.adsbdb.com` is an undocumented live third-party feed.** It supplies airline, route, aircraft type and registration, and **persists results to `.gev-cache/adsbdb.json` for 24 h**. It appears in no documentation surface — not DATA_SOURCES.md, not README, not LICENSE, not DATA_CREDITS (verified: vite.config.js:2388-2389, :2325-2332). This directly violates the file's own closing rule.
2. **FOSSGIS OSRM routing is undocumented** — it powers the walking-route flythrough the README advertises (verified: vite.config.js:2760).
3. **Google Street View Static API is undocumented** — a live CCTV frame fallback billed against the same key (verified: vite.config.js:4459-4471). Handling is compliant; the gap is purely documentary.
4. **`LICENSE:39-40` still carves out a bundled FIRMS snapshot** that was deleted 2026-07-16 (verified: DATA_SOURCES.md:96; no such folder on disk).
5. **The dams layer's in-app source label says `'USACE'`** while its provenance is an OpenInfraMap/OSM ODbL extract (verified: src/data/localLayers.js:31 vs dams/README.md:11). For a provenance-first product, this is the exact failure mode to design against.
6. GBFS coverage is understated (8 allowlisted hosts, DATA_SOURCES.md names 2); the DataSF layer has no DATA_CREDITS entry; `LL2_API_TOKEN` and `TFL_APP_KEY` are documented but missing from `.env.example`; `config/cctv_sources.shinjuku.json` ships demo MP4 references with a non-license "license" string and is referenced by nothing in the tree.

---

## 5. THE VOICE AGENT

### 5a. Session wiring

**Transport is WebRTC only. There is no WebSocket path** — `grep -c WebSocket` returns 0 for both voice modules (verified).

- `RTCPeerConnection` is built in the browser; the OpenAI event stream rides a data channel named `oai-events` (verified: src/voice/gevRealtime.js:423, :448).
- The SDP offer is POSTed **straight from the browser to OpenAI** at `api.openai.com/v1/realtime/calls`, not through the Vite proxy, using the ephemeral bearer (verified: src/voice/gevRealtime.js:478-496).
- The ephemeral token comes from `/api/realtime/token`; the server POSTs to `api.openai.com/v1/realtime/client_secrets` with the real key and pipes the upstream body through untouched (verified: src/voice/gevRealtime.js:13; vite.config.js:5045, :5186-5207).
- Model strings: `standard` → `gpt-realtime-2`, `mini` → `gpt-realtime-2.1-mini` (verified: src/voice/voiceCost.js:52, :68). Session config pins `voice: 'marin'`, `reasoning.effort: 'low'`, semantic VAD, and server-side truncation at 3,000 post-instruction tokens with a 0.5 retention ratio (verified: vite.config.js:5084-5113).
- The server echoes `X-GEV-Voice-Tier` / `X-GEV-Voice-Model` so the client **prices against the model actually served**, not the tier it asked for (verified: vite.config.js:5202-5206).

**The system prompt is a server-side array of ~64 strings** at `vite.config.js:5116-5179`. It is never sent from the browser. The honesty guardrails, quoted:

- *"Only control the app by calling the provided tools. Never invent tool names or arguments."* (verified: vite.config.js:5120)
- *"Confirmations echo the RESULTING state, never the request… Never claim an action without ok=true in the tool result."* (verified: vite.config.js:5166)
- *"State counts VERBATIM — never estimate, round, or hedge ('a few', 'less than a dozen'): if a tool returns 46, say 46."* (verified: vite.config.js:5131)
- *"For a missing field say exactly 'Operator details are unavailable'… Never silently omit missing enrichment or infer it from the callsign."* (verified: vite.config.js:5125)
- *"If an annotate_map result has partial:true or any failedLabels, do not pretend those places appeared… an honest miss beats a misleading guess."* (verified: vite.config.js:5176)

There is a **second enforcement layer beyond the prompt**: a per-tool `responseInstructionForToolResult()` is attached to the follow-up `response.create`, re-stating the constraint for the specific result shape (verified: src/voice/gevRealtime.js:2028-2103). Place names from failed lookups are never interpolated into the instruction channel — prompt-injection hygiene (verified: src/voice/gevRealtime.js:2078-2081).

### 5b. How scene context reaches the agent

**It is PULLED by a tool, not pushed per turn and not injected into session state.** No `session.update` is ever sent (verified: grep over src/voice/gevRealtime.js).

`getSceneContext(...)` at `src/voice/gevActions.js:2601` returns:

- `camera: {latitude, longitude, heightM}` to 6 decimal places (verified: :2613-2617).
- `style` — the active visual style (verified: :2619).
- `enabledLayers[]` — `{id, name, count, source}` for **enabled layers only** (verified: :2604-2611).
- `basemap.viewScale` — one of `global | continental | regional | metro | city | local`, from camera-height thresholds 12e6 / 3e6 / 750e3 / 100e3 / 10e3 m (verified: :2693-2700).
- `viewportSamples[]` — 7 screen points picked against the ellipsoid (verified: :2857-2889).
- `viewportPlaces` — dominant country/region/locality, ≤24 visible labels, ≤20 street labels (verified: :3029-3039).
- `target` — the center-screen pick, cached 2.5 s by camera signature (verified: :2780-2797).

**Street names** come from Google Geocoding `address_components` filtered to `types.includes('route')`, called directly from the browser (verified: src/voice/gevActions.js:2936-2983). **Altitude gates decide what is even fetched**: reverse-geocode center only ≤750 km, viewport-sample geocoding only ≤3000 km and >10 km, Nearby Places only ≤25 km (verified: :2702-2712). Everything is raced against a 1,500 ms budget so a slow Google call cannot stall the turn (verified: :243, :2669-2673).

**Visual grounding is tightly gated.** The viewport screenshot fires only after `get_entity_context`, only at `viewScale === 'local'`, and only when structured context does not already answer the question (verified: src/voice/gevRealtime.js:1376-1380, :2015-2026). It is capped twice — a 1.08 MP pixel budget and a hard 200 KB encoded ceiling, over which the image is dropped rather than risking a send that would strand the turn (verified: :31-32, :2209-2215). **At most one screenshot lives in context**: the previous item is deleted before a new one is added (verified: :1384-1404).

### 5c. The 28 tools — the count is exactly right

Single source of truth: `const GEV_REALTIME_TOOLS = [` at `vite.config.js:5534`, closing at `:6156` (verified). Independently confirmed three ways: 28 `name:` definitions, 28 `type: 'function'` entries, and 28 unique dispatch branches in `src/voice/gevActions.js` — **the two sorted name sets diff empty**. There is no client-only or server-only tool (verified).

**Grouped into the README's "four jobs"** — the grouping is derived from each tool's own description text; the README gives spoken examples per job but never an explicit tool-to-job map (inferred: from vite.config.js:5534-6156):

- **🎥 Direct it — camera verbs (6):** `fly_to_location`, `adjust_camera_zoom`, `zoom_to_globe`, `move_camera`, `fly_route`, `frame_overhead`
- **🖊️ Annotate it (2):** `annotate_map`, `clear_annotations`
- **🔎 Interrogate it (5):** `analyst_query`, `get_entity_context`, `get_current_view_state`, `next_iss_pass`, `select_nearest_aircraft`
- **🎛️ Operate it — console control (15):** `set_layer_visibility`, `show_data_layers_menu`, `set_panel_open`, `set_context_mode`, `control_cockpit`, `set_visual_style`, `set_hud`, `set_detection`, `set_map_stack`, `set_post_processing`, `control_scene`, `control_cctv`, `control_radio`, `track_entity`, `stop_tracking`

6 + 2 + 5 + 15 = 28. Two boundary cases: `select_nearest_aircraft` is a query verb that *mutates* (it enables the layer, waits for arrival, refreshes, filters, selects), and the README files "Track that plane" under Operate it even though `track_entity` is a camera verb.

### 5d. The cost guard

`src/voice/voiceCost.js` is a **pure module** — no DOM, no network, no imports — shared by the browser UI, the server token endpoint, and the tests (verified: src/voice/voiceCost.js:1-18).

The constants, quoted (verified: src/voice/voiceCost.js:170-172):

```js
export const VOICE_COST_LIMITS = Object.freeze({
  warnUsd: 2,
  capUsd: 5,
});
```

The doc comment immediately above: *"`warnUsd`: soft — one visual cue + one console line, session continues. `capUsd`: hard — the session is closed through the normal stop path"* (verified: :161-169).

The accounting design is the interesting part:

- Rates are USD per 1M tokens across eight buckets, stamped with the date they were read: `VOICE_MODEL_RATES_VERIFIED_ON = '2026-08-18'` (verified: :45, :55-80).
- **Every ambiguity resolves toward over-estimating, deliberately.** Missing `input_token_details` → the whole input is billed as *audio*; any aggregate-minus-details residual is added to audio too. The stated reason is that under-metering is what lets a cap be overrun (verified: :234-244, :278-299).
- **An unrecognised model id bills at the most expensive known rates**, derived by ranking on `audioOutput` rather than hardcoded (verified: :122-127, :143-155).
- A disabled threshold round-trips as the string `'off'` because `JSON.stringify(Infinity)` is `null` and would silently re-arm the cap (verified: :183, :210-217).
- Corrupt or unparseable persisted values fall back to the defaults, so a bad entry can never *disarm* the cap (verified: :192-207).

**What happens at the cap:** `handleCostCap()` sets a latch, then calls `stop({preserveStatus:true})`, which aborts in-flight tool controllers, closes the data channel, closes the peer connection, and calls `stream.getTracks().forEach(t => t.stop())` — **the mic hardware is released, not merely muted** (verified: src/voice/gevRealtime.js:1897-1917, :770-828). Two independent gates then block new tool dispatch (verified: :1073, :1108-1111).

**Documented limitation:** in-flight tools run to completion and are not rolled back — a camera flight already executing may finish. The latch only guarantees no *new* tool is dispatched (verified: src/voice/gevRealtime.js:1742-1752). And if a response was still active at teardown its usage never arrives, so the tracker is flagged `markIncomplete()` rather than fabricating tokens — notably it is *not* called a lower bound, because the estimate can also run high (verified: :813; voiceCost.js:398-414).

### 5e. The analyst query path

`src/data/analystEngine.js` is 247 lines, pure, node-testable, and renders nothing. Providers are injected so it never touches Cesium or the network (verified: :1-25).

*"How many flights over Texas"* resolves like this:

1. `analyst_query` restricts `layers` by enum to five: flights, military, ais-live-vessels, local-firms, earthquakes (verified: vite.config.js:6099-6119).
2. `providers.getRecords(layerKey)` returns `[]` unless the layer is **enabled**, then calls the layer's own `getAnalystRecords()` (verified: src/voice/gevActions.js:3311-3320).
3. `"over Texas"` → resolve the region ring, first from the bundled Natural Earth pack, then falling back to geocode + admin-area lookup. Containment is point-in-ring (verified: src/data/analystEngine.js:148-159).
4. Filters support `gt|gte|lt|lte|eq|neq|contains`; **null fields fail closed** (verified: :30-36).
5. The result always carries `coverage.note: 'client-side data only — answers cover what the enabled layers currently hold'` (verified: :221-239).

Three honesty annotations are added on top of the raw result, and each records the incident that forced it:

- **Warm-up:** a layer enabled by voice within 45 s gets *"enabled moments ago — data is still loading; counts will rise for ~30-45s. Say so."* (verified: src/voice/gevActions.js:3382-3393)
- **Viewport-loaded caveat:** for view/radius scopes over flights, *"counts cover loaded data; the flights layer loads by viewport"* (verified: :3394-3403).
- **Contacts reconciliation:** when Contacts is active, the payload carries a prose `countsReconciliation` string stating explicitly which number answers "how many aircraft are nearby" and which measured something else. The comment records the field incident: the model answered 15 and explained away the 111 sitting in the same payload (verified: :3412-3453).

---

## 6. KEYS + SECURITY

### 6a. The brokering pattern

There is no separate server. Every proxy is a Vite plugin registered in one array; each calls `server.middlewares.use('/api/<name>', handler)` (verified: vite.config.js:7342-7363).

Env resolution runs first, and the precedence matters: `loadEnv(mode, __dirname, '')` with an **empty prefix** loads every var, then copies into `process.env` **only where undefined** — so shell exports beat `.env` (verified: vite.config.js:7336-7339).

The canonical request shape, using `/api/realtime/token`:

1. Browser issues a same-origin GET. No key of any kind leaves the browser.
2. Method gate → 405 for anything but GET/POST (verified: :5045-5051).
3. Opt-in throttle (verified: :5053-5054).
4. Secret read from the process, never from the request: `process.env.OPENAI_API_KEY`; missing → sanitized 503 (verified: :5056-5061).
5. **Client input is normalized to a closed set before it can influence upstream** — an unknown `?tier=` degrades to `standard` rather than reaching OpenAI as a model id (verified: :5063-5081).
6. The secret is attached server-side as an outbound header (verified: :5186-5193).
7. What returns is an **ephemeral client secret**, not the API key (verified: :5194-5206).
8. Failure is sanitized: `502 {error: …}` — never credentials, never the upstream URL (verified: :5207-5211).

### 6b. Credential table

| Credential | Env var | Exposure | How the browser uses it | Pointer |
|---|---|---|---|---|
| Google Maps API key | `GOOGLE_MAPS_API_KEY` | **Browser-exposed** (also used server-side) | Baked in by `define`; set as `Cesium.GoogleMaps.defaultApiKey`, re-published on `window.__GOOGLE_MAPS_API_KEY__` | vite.config.js:7374; src/main.js:83-90 |
| Cesium ion token | `CESIUM_ION_TOKEN` | **Browser-exposed** | `Cesium.Ion.defaultAccessToken` | vite.config.js:7375; src/main.js:77-80 |
| OpenAI API key | `OPENAI_API_KEY` | Server only | Never sees it — gets an ephemeral secret | vite.config.js:5056, :5189 |
| OpenSky OAuth id/secret | `OPENSKY_CLIENT_ID` / `_SECRET` | Server only | Polls `/api/opensky` | vite.config.js:1372-1373 |
| OpenSky Basic user/pass | `OPENSKY_USERNAME` / `_PASSWORD` | Server only — **absent from `.env.example`** | Same | vite.config.js:2970-2971 |
| AISStream key | `AISSTREAM_API_KEY` | Server only | Polls `/api/ais-live` | vite.config.js:6344 |
| TomTom key | `TOMTOM_API_KEY` | Server only | Fetches `/api/tomtom/…` | vite.config.js:1835 |
| NASA FIRMS key | `FIRMS_MAP_KEY` | Server only | Polls `/api/firms` | vite.config.js:1992 |
| Launch Library 2 token | `LL2_API_TOKEN` | Server only — **absent from `.env.example`** | Polls `/api/launches` | vite.config.js:1602-1609 |
| TfL app key | `TFL_APP_KEY` | Server only — **absent from `.env.example`** | Never sees it | vite.config.js:4004 |

### 6c. What actually reaches the browser

The `define` block is four lines and emits exactly two values (verified: vite.config.js:7372-7376):

```js
define: {
  'import.meta.env.GOOGLE_MAPS_API_KEY': JSON.stringify(env.GOOGLE_MAPS_API_KEY),
  'import.meta.env.CESIUM_ION_TOKEN': JSON.stringify(env.CESIUM_ION_TOKEN),
},
```

Three findings here, all of which cut in the operator's favour but contradict the docs:

- **SECURITY.md:31 overstates the block.** It claims *"only these two keys plus two non-secret CCTV feature flags."* There are no CCTV flags in it. `CCTV_AUTO_CALIBRATE` and `CCTV_DRAPE_MESH` — both annotated "(client flag)" at `.env.example:120-121` — appear **zero times anywhere else in the repository**: not in `vite.config.js`, not in `src/`, not in `index.html` (verified: repo-wide grep). They are dead variables. **The security posture is stricter than documented; the doc is wrong.**
- **`define` is not the only client-exposure path.** `envPrefix` is never set, so Vite's default `VITE_` prefix applies and three non-secret values reach the browser from `.env`: `VITE_AIS_LIVE_API_URL`, `VITE_AIS_LIVE_MAX_ROWS`, `VITE_AIS_LIVE_LABEL_MAX_ROWS` (verified: src/data/aisLiveVessels.js:946, :953, :961; no `envPrefix` match in vite.config.js). None are secrets — but SECURITY.md's enumeration is incomplete on both counts.
- **`define` applies to `vite build` too.** `dist/` contains both keys in plaintext. Publishing a build publishes the keys (inferred: from vite.config.js:7373-7376 living in the shared config object).

### 6d. The LAN-exposure threat model, in operator terms

**Default posture:** `host: env.HOST || 'localhost'`, and `allowedHosts` is restricted to `['localhost','127.0.0.1','.local']` unless HOST is `0.0.0.0`/`::` (verified: vite.config.js:7365, :7368-7370).

**What changes at `HOST=0.0.0.0`:** the listener binds every interface **and — separately — Vite's Host-header check is switched off entirely** (`allowedHosts: true`). The DNS-rebinding protection SECURITY.md credits to `allowedHosts` exists *only* in the local-only branch. Opting into LAN mode removes it (verified: vite.config.js:7368-7369).

**What an attacker on the LAN can spend.** There is no authentication, no session, no CSRF token, and **no `Origin`/`Referer` check on any `/api/*` handler** — the only gates are method, shape validation, and the opt-in limiters (inferred: from reading all 24 handlers). Concretely, per request:

- `GET /api/realtime/token` mints a **full OpenAI Realtime session secret with the entire GEV tool surface attached**. One request equals one billable voice session the attacker drives directly against OpenAI, outside your server (verified: vite.config.js:5045, :5091-5182).
- `GET /api/cctv/frame/:id` falls back to the **billed Google Street View Static API**. This path has **no rate limiter at all** (verified: vite.config.js:4460-4487; no limiter call in :4494-4690).
- `GET /api/tomtom/flow/...` — billed tiles, capped only by the soft daily budget (verified: :1912-1918).
- `POST /api/realtime/debug-log` — **unauthenticated, unthrottled, accepts up to 8 MB per request and `appendFileSync`s it with no rotation and no file size ceiling** (verified: :5020-5043, :1327-1329). A disk-fill and log-poisoning primitive, not a spend primitive. It is installed in **both** dev and preview servers and is gated on neither `DEV` nor the presence of an API key.

**What the throttles do not do — five things:**

- They are **off by default**. Unset / `0` / non-numeric returns `null`, and enforcement on `null` is a no-op returning true (verified: vite.config.js:439-441, :464).
- They are **not billing caps** (verified: SECURITY.md:52).
- They cover **four endpoints only** — realtime token, HUD summary, and the two Google Places routes. The billed Street View path and the billed TomTom tile path are covered by neither `GEV_RATELIMIT_*` var (verified: vite.config.js:4970, :5053, :5284, :5398).
- They are per-IP, in-memory, process-local, keyed on the raw socket peer. `X-Forwarded-For` is deliberately not trusted — but the flip side is that a LAN attacker with IPv4 plus a rotating IPv6 privacy address gets a fresh bucket per address (verified: vite.config.js:483-488).
- They reset on restart.

**Where the SSRF posture is genuinely strong.** This is the best-engineered part of the security surface:

- **CCTV frames fetch only server-registered URLs** — clients cannot pass an upstream URL. Scheme check, 8 s abort, `content-type: image/*` required (verified: vite.config.js:4626-4631, :4374-4399).
- **Radio is the strictest path in the repo.** Destination must be `<label>.api.radio-browser.info` with no userinfo/port/hash and one of exactly three path shapes; **DNS is resolved first and every returned address must be globally routable or the request is refused before any socket opens**; the connection is then **TLS-pinned to the validated address** via a custom `lookup`; redirects are `manual` and any 3xx is an error; 12 s timeout, 4 MB cap (verified: vite.config.js:874-887, :939-948, :951-976, :1019-1027). The test suite asserts 22 forbidden address forms including `169.254.169.254`, `::ffff:127.0.0.1`, `fc00::/fd00::/fe80::`, `64:ff9b::`, `2001:db8::` — and that a hostname resolving to any forbidden address is rejected with `fetchCount === 0`, meaning **nothing is dialed** (verified: src/data/radioProxy.test.mjs:395-471).
- `/api/gbfs` forces HTTPS, an allowlisted host, and only two path shapes (verified: :3238-3244).
- `/api/route` clamps to a 3-value profile enum, 2–12 coordinates, per-leg ≤600 km, total ≤2500 km (verified: :2717-2775).

### 6e. Rate-limit and hardening env vars

| Var | Default | Effect |
|---|---|---|
| `GEV_RATELIMIT_OPENAI_PER_MIN` | **unset ⇒ unlimited** | N/min/IP across the two OpenAI routes; global backstop N×20; 429 + `Retry-After: 5` |
| `GEV_RATELIMIT_GOOGLE_PER_MIN` | **unset ⇒ unlimited** | Same for the two Places routes; 429 body preserves the `places: []` contract |
| `HOST` | `localhost` | `0.0.0.0` binds all interfaces **and** disables the Host-header check |
| `PORT` | 4173 (launcher) / 5173 (config) | Listen port |
| `TOMTOM_DAILY_TILE_BUDGET` | 40,000/UTC day | Over cap: serve cached/stale, or 429 |
| `AISSTREAM_SILENCE_TIMEOUT_MS` | 120,000; `0` disables | Feed reported stale after N ms; socket recycled at 2.5× |
| `OPENSKY_AUTH_MODE` | `oauth` | `{basic, oauth, auto, anon}`; invalid warns and falls back |

(verified: .env.example:13-22, :41-49, :64-72, :107-109, :94-99, :51-54; vite.config.js:439-459, :7365-7370, :1738, :124-127.)

Always-on hardcoded limiters, documented as *"not a hard security boundary (dev-only), just a backstop"*: Overpass 90/min, military-installations 90/min, route 60/min, regional-brief 30/min, weather-effects 45/min (verified: vite.config.js:392-396, :423-425).

### 6f. The redacted debug log

**Redaction happens in the browser, before the POST. The server does none.** This is the load-bearing detail.

- `sanitizeDebugValue` walks to depth 10 and replaces any key matching `/(?:api[_-]?key|authorization|bearer|client[_-]?secret|token|secret|password)/i` with `'[Redacted]'` (verified: src/voice/gevRealtime.js:2143-2178).
- `sanitizeDebugString` strips, in order: any `data:image/` string, `sk-`/`sk-proj-` keys, `Bearer …`, `"client_secret":"…"`, `"value":"ek_…"`, then truncates at 50,000 chars (verified: :2161-2174).
- **The server adds no redaction of its own** — it stamps `loggedAt` and spreads the received record (verified: vite.config.js:5032-5035). SECURITY.md:44's claim that the log *"strips API keys… before writing"* is accurate for GEV's own client, but the stripping is the client's doing. **Anything else that can POST to that endpoint writes unfiltered JSON into `.gev-logs/realtime-conversations.jsonl`.**

---

## 7. RUN-IT-LOCALLY REALITY (WINDOWS)

The one-sentence version: the repo's *convenience* path is macOS-only bash; the repo's *actual* path — `npm run dev` reading a `.env` — is fully platform-neutral, and on Windows you lose the launcher, not the app. There is **no platform branching anywhere in the app or in the `.mjs` npm scripts** (verified: repo-wide grep for `process.platform` / `darwin` / `win32` returns zero hits outside the `.sh` files).

The word "Windows" appears **twice** in the entire documentation set — `.env.example:3` and `docs/PERFORMANCE.md:134`. There is no Windows Quick Start, and no `.ps1`, `.cmd`, or `.bat` anywhere (verified).

### 7a. Minimum to first pixel

**One mandatory key: `GOOGLE_MAPS_API_KEY`.** It is the only hard requirement in the boot path (verified: src/main.js:83-86):

```js
const googleApiKey = import.meta.env.GOOGLE_MAPS_API_KEY;
if (!googleApiKey) {
  throw new Error('GOOGLE_MAPS_API_KEY not found. Set it as an environment variable.');
}
```

It goes in `.env` at the repo root, beside `package.json`. Vite loads dotenv from `vite.config.js`'s own directory with an **empty prefix**, so every var in the file is loaded (verified: vite.config.js:66, :7336).

The exact PowerShell sequence:

```powershell
cd C:\Users\abstr\...\gods-eye-view
node -v                          # must be 24.14.x–24.x or 26.x
Copy-Item .env.example .env
# edit .env, set GOOGLE_MAPS_API_KEY=AIza...
npm install
npm run dev -- --host localhost --port 4173
```

Then open **`http://localhost:4173`**.

Four things to know about that sequence:

- **Do not use `npm install --omit=dev`.** `vite` itself is a devDependency (verified: package.json:55) — omitting dev deps leaves you with no dev server at all.
- **The flags are droppable only if your `.env` carries them.** A copied `.env.example` gives you `PORT=4173`. A `.env` without a `PORT` line gives you **5173**, not 4173 (verified: vite.config.js:7366; .env.example:64).
- **Use `localhost`, not the LAN IP, even locally.** `http://<ip>:4173` is not a secure browsing context, so `navigator.mediaDevices.getUserMedia` is undefined and the mic short-circuits at its capability gate (verified: src/voice/gevRealtime.js:345; the browser secure-context rule itself — inferred).
- **`npm install` triggers a Chrome download.** `puppeteer` carries `hasInstallScript: true` in the lockfile — a large network-dependent step on the way to a dev server that never uses it. Windows prebuilt binaries for `sharp`, `esbuild`, and `rollup` **are** in the lockfile (40 `win32` entries), so there is no compile-from-source hazard on x64 (verified: package-lock.json:3012, :508-553).

**Expected cold start**, as loading-screen text: `Configuring viewer...` → `Loading Google 3D Tiles...` → `Initializing systems...` → `Flying to Austin, TX...`, then the screen hides after the restore promise and a hard 1,000 ms floor both settle (verified: src/main.js:74, :156, :175, :203-205, :251-256).

The README's "median 1.86 s" is a macOS number. `docs/PERFORMANCE.md:134` states explicitly: *"This report does not establish Windows performance."* (verified). There is no Windows counterpart in this repo.

**Failure mode with no key:** `init()` throws before the Cesium viewer is ever constructed; the catch writes the error into `.loader-status` in red and the loading screen never hides (verified: src/main.js:83-86, :327-331).

### 7b. What works with zero keys

**Does the app boot with no Google key? No.** This is the most important honest answer in this section. With the key absent the app does not degrade — it dies on the loading screen. **Zero of the layers light up, because `DataLayerManager` is never constructed** (verified: src/main.js:85; registration at :210-238 is downstream of the throw).

The distinction that matters:

- **No key at all** → hard abort, red error, no globe.
- **Key present but invalid or tiles unreachable** → the app *does* boot. The tileset call is wrapped in its own try/catch, logs *"Google 3D Tiles unavailable, falling back to Cesium globe"*, and continues with `initialStack: 'osm'` — you land on the keyless OSM globe with every data layer alive (verified: src/main.js:157-190; src/mapStackController.js:65).

So the practical floor is: **a syntactically-present Google key gets you into the app; the OSM fallback only rescues you after that gate.**

With the Google key and nothing else:

**Fully live, keyless (10 of the README's 13 rows):** Live Flights (OpenSky falls through to anonymous when credentials are empty — verified: vite.config.js:2974-2991), Military Flights, Satellites, Earthquakes, CCTV Mesh (all three providers), Radio, Bikeshare, Space Missions, Mapped Installations, and the OSM map stack. Plus the bundled datasets, which touch no network at all.

**Silently degrades — the one to watch:** **Traffic.** It does not error; it runs a built-in simulation on hardcoded per-road-class speeds. The label reads `SIMULATED — add TomTom key for live` (verified: src/data/traffic.js:1274-1280, :1293-1296). **If you do not read the chip label, keyless traffic looks like working traffic.**

**Honestly gated with a visible "no key" state (3):**

- **Live Vessels** — 503 with `status: 'missing-key'` (verified: vite.config.js:4791, :6290-6291).
- **Active Fires** — `503 {error:'no_key'}`, never touches upstream; the client renders `KEY REQUIRED` as a first-class UI state (verified: vite.config.js:1970-1972; src/data/firmsHeatmap.js:308-309).
- **Bing map stacks** — without an ion token the assignment is simply skipped; you get Google 3D + OSM only (verified: src/main.js:77-80).

**Voice** is gated the same way: `/api/realtime/token` returns `{error: 'OPENAI_API_KEY is not set'}` (verified: vite.config.js:5057-5061).

### 7c. What `dev-fresh.sh` does that Windows must do by hand

`scripts/dev-fresh.sh` is 362 lines and is **not wired into `package.json` at all** — there is no `npm run dev:fresh`, despite README.md:78, SECURITY.md:50-51, CONTRIBUTING.md:15, TESTING.md:68 and docs/opensky-auth.md all instructing you to run it (verified: package.json:34-43).

| macOS step | What it does | Windows equivalent |
|---|---|---|
| `:7`, `:11` | `PORT=4173`, `HOST=localhost` | `.env` line `PORT=4173`; HOST already defaults to localhost |
| `:17-24` | CCTV caps: Austin 250, Caltrans `4,7,11,3` / 300, TfL on / 250, overall 900 | **Nothing to do** — byte-identical to the server-side defaults (verified: vite.config.js:3420-3441). Windows `npm run dev` gets the same ~800-camera mesh |
| `:35-55` | Pre-read the Google key from shell env, else parse `.env` | Not needed — Vite loads `.env` itself |
| `:58-75` | `security find-generic-password` Keychain sweep; **Keychain wins over both shell env and `.env`** | **No equivalent.** `.env` lines |
| `:76-80` | Hard `exit 1` if no Google key, with a hint | No preflight — you find out in the browser as red text |
| `:88-124` | Parse an OpenSky credentials JSON; validate `OPENSKY_AUTH_MODE` | Put `OPENSKY_CLIENT_ID`/`_SECRET` in `.env`. The mode validation is duplicated server-side, so nothing to do (verified: vite.config.js:1432-1447) |
| `:132-209` | Keychain sweep for six optional keys | `.env` lines for all six |
| `:221-232` | `pkill` vite; `lsof -tiTCP:$PORT \| xargs kill -9` | `Get-NetTCPConnection -LocalPort 4173 -State Listen \| ForEach-Object { Stop-Process -Id $_.OwningProcess -Force }` |
| `:234-235`, `:362` | `rm -rf node_modules/.vite` plus `--force` | `Remove-Item -Recurse -Force .\node_modules\.vite`; add `--force` to the npm command |
| `:242-271` | LAN branch: discover IP, print a 9-line key-brokering warning | **You lose the warning, not the risk.** Set both `GEV_RATELIMIT_*` vars yourself before ever binding `0.0.0.0` |
| `:273-318` | Status echo: one line per key stating the exact degradation | You get none of it. §7b above is that table, derived from code |
| `:324-360` | `put_env_if_set` **removes** a var from the child rather than exporting it empty | **This bug class bites you on Windows** if you use `$env:FOO=""` — `vite.config.js:7337` treats `''` as *defined*, so it shadows the `.env` value. Use `Remove-Item Env:\FOO` |

(All line references verified against scripts/dev-fresh.sh.)

**The other three scripts:** `dev-secure.sh` has **no `.env` fallback at all** and caps CCTV at 48; it is the only `.sh` wired into npm (`dev:secure`) and **that npm script is broken on Windows**. `dev-cctv.sh` defaults `HOST=0.0.0.0` — LAN-exposed by default, **with no warning print**. `opensky-import-client.sh` is a pure Keychain writer and a no-op on Windows by design; it is also wired into npm and also broken there (verified: dev-secure.sh:81-84, :150-151; dev-cctv.sh:8-12; opensky-import-client.sh:7-10; package.json:36-37).

### 7d. Windows gotchas, evidenced

- **Two npm scripts are broken on Windows** — `dev:secure` and `opensky:import` run `./scripts/*.sh` as the script body. `npm run dev`, `build`, `preview`, `test`, `test:track`, `qa:map-source-tray` are pure Node/vite and unaffected (verified: package.json:35-42).
- **Path separators are a non-issue.** No hardcoded POSIX paths and no shell-outs in the server — grep for `spawn(`, `execSync`, `'/tmp`, `os.homedir`, `/bin/sh` across `vite.config.js` returns zero hits. Cache paths use `path.join(process.cwd(), …)`, and the test discoverer explicitly normalizes `path.sep` (verified: vite.config.js:1979-1980; scripts/run-unit-tests.mjs:33). Someone already thought about this.
- **The `engines` field is not actually enforced.** README.md:62 and CONTRIBUTING.md:7 both claim it is *"enforced by `package.json`"*. There is **no `.npmrc`**, so `engine-strict` is unset; there is no `preinstall` guard; and no runtime version check exists outside the test harness — which *skips* rather than fails (verified: absence of `.npmrc`; package.json:34-43; scripts/run-unit-tests.mjs:11-22).
- **Cosmetic macOS-isms only.** `dev-fresh.sh:274` says `Cmd+Shift+R`; use `Ctrl+Shift+R`. In-app shortcuts are safe — both keyboard handlers treat `metaKey` and `ctrlKey` symmetrically, and `index.html` contains no `Cmd`/`⌘` strings at all (verified: src/ui.js:1029; grep over index.html).
- **`.env` is parsed, never executed.** A unit test proves `$(touch …)` and backticks survive as literal text with no side effects, so an API key containing `$`, backticks or quotes is safe to paste verbatim (verified: src/devFreshDotenv.test.mjs:8-27).

### 7e. Ready-to-paste minimal `.env`

```dotenv
# ── REQUIRED ─────────────────────────────────────────────────────────
# Map Tiles API must be enabled. CLIENT-EXPOSED — restrict by HTTP referrer
# (allow http://localhost:4173/*) and set a Google Cloud budget alert.
GOOGLE_MAPS_API_KEY=PASTE_YOUR_GOOGLE_MAPS_KEY_HERE

# ── DEV SERVER ───────────────────────────────────────────────────────
PORT=4173
# Leave HOST unset. localhost is the default and keeps every brokered key
# on this machine. HOST=0.0.0.0 exposes all of them to your LAN.

# ── OPTIONAL: no-cost developer keys ─────────────────────────────────
AISSTREAM_API_KEY=
FIRMS_MAP_KEY=
TOMTOM_API_KEY=
CESIUM_ION_TOKEN=

# ── OPTIONAL: metered ────────────────────────────────────────────────
OPENAI_API_KEY=

# ── OPTIONAL: OpenSky ────────────────────────────────────────────────
OPENSKY_AUTH_MODE=anon
OPENSKY_CLIENT_ID=
OPENSKY_CLIENT_SECRET=

# ── OPTIONAL: read by the server, undocumented in .env.example ───────
LL2_API_TOKEN=
TFL_APP_KEY=

# ── LAN ONLY: set these before ever using --host 0.0.0.0 ─────────────
# Per-IP, in-memory app guards. NOT billing caps.
# GEV_RATELIMIT_OPENAI_PER_MIN=30
# GEV_RATELIMIT_GOOGLE_PER_MIN=60
```

Blank lines are safe — the proxies test with `Boolean(process.env.X)`, so `KEY=` reads as unset (verified: vite.config.js:1874, :1992).

---

## 8. PATTERNS WORTH STEALING

Eight patterns, each verified by reading the implementation. Shapes: **(a)** land-title workbench · **(b)** evidence-first intelligence board · **(c)** 3D geological/strata visualization.

Baseline license fact for all eight: the code is MIT, and the MIT grant explicitly does not cover bundled datasets or runtime feeds (verified: LICENSE:1-10, :25-31).

---

### 1. Versioned share-link codec with frozen "absent token" semantics

- **What it is** — Camera pose, visual style plus its per-style parameters, which layers are on with their options, and which panels are open, all compressed into a `#`-fragment under schema version `v=2`, written back with a 500 ms-debounced `history.replaceState` so the address bar always *is* the current view. The distinguishing move: **an omitted token's meaning is frozen at the value it had when links of that vintage were authored, separately from today's default.**
- **Where it lives** — Grammar documented inline at `src/sharelink.js:14`; writer at `:474-517`; reader with per-field parse fallbacks at `:154-233`; token tables rather than ad-hoc strings at `:35-53` and `:55-86`; the frozen-omission rule `absentTokenValue()` at `src/data/layerState.js:111`, used by both encoder (`:434`) and decoder (`:478-486`); fail-closed decode with payload caps at `src/data/layerState.js:25-26`, enforced at `:450-451`, where **any unknown token rejects the whole payload** at `:456` (all verified).
- **Maps to** — **(a)** directly. This *is* the shareable handoff link, and the frozen-omission rule is what keeps a handoff sent last quarter from silently re-rendering under this quarter's defaults. **(b)** secondarily, for pinning a board's exact source mix.
- **Effort class: weekend.** The mechanism is small (617 lines) but the value is the discipline — registry-driven tokens, per-field clamps, explicit fail-closed decode. Porting means writing your own registries.
- **License** — Pure MIT. `layerState.js` imports nothing external; `sharelink.js` imports only Cesium and siblings (verified: :1-8 of each). Cesium's own license is never stated in this repo (unverified).

---

### 2. Live-target handoff — a shared subject is *pending*, not bookmarked

- **What it is** — A share link can carry one tracked subject. On the recipient's side the ID is **not** a bookmark that resolves or fails on first refresh. It arms the owning layer's deferred-restore latch, a 1 s background poll watches for the subject to arrive, and a terminal verdict issues only after a per-source expiry window elapses. The failure wording distinguishes *"the link is old"* from *"the source can't see it."*
- **Where it lives** — Per-source expiry policy at `src/data/layerState.js:252-270` (flights 90 s, military 45 s, satellites 300 s, each with its own human label); the ID grammar, bounded and **never truncated** because *"half an address is a DIFFERENT aircraft, not a shorter name for the same one"* (`:9-19`); exactly-one-subject invariant, fail-closed on ambiguity (`:388-400`); the pending watch settling `found` / `expired` / `unavailable` / `cancelled` (`:919-1015`); **"expired" measured from the share's age, not from how long this client watched** (`:833-843`); recipient-intent claiming a restore lane at `src/sharelink.js:351-358` (all verified).
- **Maps to** — **(a)** strongly. "Hand off a live thing, not a coordinate" is exactly a tract handoff where status may have moved between send and open. The lane-authority idea — the recipient's own click beats a delayed restore — is the part most workbenches get wrong.
- **Effort class: weekend.** The state machine has real edge cases already paid for: abort/generation guards, owner-layer-disabled abandonment, revoke-both-halves-together.
- **License** — Pure MIT, dataset-agnostic. The subjects in this repo come from OpenSky and adsb.lol, but none of that is touched by the handoff machinery.

---

### 3. One honest feed-state vocabulary + a two-tier attribution registry

- **What it is** — Heterogeneous per-layer `getStats()` shapes normalized by one pure function into six words, which then drive a CSS class, a `data-feedState` attribute, a chip label, and a tooltip reading `SOURCE · age · retry in Ns`. Attribution is a parallel registry: always-on credits registered once at init, plus **conditional credits registered only at the moment their source actually activates**.
- **Where it lives** — The vocabulary at `src/data/manager.js:13-20`; the normalizer at `:72-108`, including the carve-out that guidance states (`zoom-in`/`empty`/`idle`) are normal operation *but a genuinely stale cache still reads STALE* (`:86-89`); chip binding at `:2254-2273`; tooltip composition at `:2212-2251`; the credit table at `src/data/dataCredits.js:28-178` with the rule stated at `:11-19`; dynamic credits fired on live activation at `src/data/traffic.js:1300`; and the resilience half — memoize success, rate-limit failure, **never memoize a transient rejection into a session-long outage** — at `src/data/retryableLoad.js:21-84` (all verified).
- **Maps to** — **(b)** directly and almost completely. This *is* "label every source's freshness and confidence," down to the refusal to let a fabricated confidence score stand in for a real state. **(a)** for the provenance column on a tract.
- **Effort class: afternoon** for `layerFeedState` + the chip binding + `retryableLoad` (~250 lines of pure units); **weekend** if you also want the credit-registry discipline and the doc-and-code-must-agree convention.
- **License** — Pure MIT, no data. **One caution worth internalizing:** the app's own display strings can drift from its license doc — `src/data/localLayers.js:31` labels the dams layer `'USACE'` while the provenance is an OpenInfraMap/OSM ODbL extract (verified). For a provenance-first product, that drift is the failure mode to design against.
- **Scope note:** the six-word enum is real, but there are three tiers (see §3e). Per-layer label strings sit on top of it, and `src/loadingFeedback.js:25` adds `keyRequired` outside it. Design your vocabulary as one tier, not three.

---

### 4. Bundled-polygon → first-class layer pipeline

- **What it is** — Three stages. A deterministic Node build script pulls a public polygon dataset, simplifies and rounds it reproducibly, and writes a versioned file. A sidecar Markdown file records the exact URL, byte count, retrieval date, license evidence, and transform. A generic layer factory turns the resulting GeoJSON/GeoJSONL into a Cesium layer with entities, picking, per-feature context registration, labels, and a real error state.
- **Where it lives** — Build script at `scripts/build-sf-neighborhoods.mjs` (header records source URL, dataset ID, license, and the five-step transform; Douglas-Peucker at `:52-70`); provenance sidecar at `src/data/local_data/neighborhoods/SOURCE.md` — retrieval date, HTTP status, byte count, `licenseId: "PDDL"` evidence, vertex counts before and after, and an explicit *"re-run it to reproduce byte-for-byte"* claim; **layer declaration is data, not code** at `src/data/localLayers.js:13-35`; the factory at `src/data/localGeojson.js:280-291`; the honest empty-vs-dead contract at `:391-398` — a failed load surfaces `error` so the chip reads UNAVAILABLE instead of reporting zero as nominal; and rollback discipline the naive version misses, awaiting `dataSources.add()` so a throw cannot orphan a source Cesium inserts on a later microtask (`:449-457`, `:568-576`) (all verified).
- **Maps to** — **(a)** core. This is the tract layer end to end, and the sidecar-README convention is the provenance overlay's data model rather than an afterthought. **(c)** for bringing a strata or boundary GeoJSON in as a first-class pickable layer.
- **Effort class: weekend.** Build script is ~145 lines and self-contained; the layer factory is 892 lines, of which the last ~400 are ground-sampling and label-cohort work you may not need on day one.
- **License** — Code MIT; the data it carries is not. SF neighborhoods is PDDL 1.0 and safe. The two datasets `localLayers.js` actually registers are **ODbL 1.0**. The submarine-cables layer registered alongside them is **CC BY-NC-SA 3.0, NonCommercial**. **Take the pipeline; do not take the cable folder.**

---

### 5. Cross-layer pick arbitration + degenerate-pick rejection + one selection slot

- **What it is** — Three small pieces that make click-to-entity behave across heterogeneous layers. A registry where each layer declares a predicate for "this pick id is mine," so siblings stop fighting over one click. A numeric-magnitude guard that rejects `pickPosition()` results which are finite but not a real place on the globe. A single window-scoped store holding the one selected entity's metadata that every downstream surface reads.
- **Where it lives** — `resolvePickId()` coercing string / numeric / record / Cesium-Entity ids to one canonical string at `src/data/pickRegistry.js:16-45`, with `isOwnedByOtherLayer` at `:47-84` and a broken predicate deliberately unable to break click handling (`:78-81`); the bug it fixes documented at `:1-12` — two layers issuing competing camera commands from one click; the degenerate-pick guard at `src/data/scenePick.js:16-48`, whose floor of 6,000,000 m exists because `(500,0,0)` converts *without complaint* to a point 6,378 km underground that reverse-geocodes as 0°,0°; a worked click path at `src/data/localGeojson.js:578-621`; the selection store at `src/data/contextStore.js:31-51` (all verified).
- **Maps to** — **(a)** core. Per-tract selection with status, provenance and annotation overlays stacked on the same map *is* the multi-owner click problem this solves. `scenePick` is what keeps a click on empty sky from selecting a tract at 0°,0°.
- **Effort class: afternoon.** `pickRegistry.js` is 84 lines and `scenePick.js` is 48. Both are dependency-free and copy-paste-able as-is.
- **License** — Pure MIT, zero imports in both files, no data, no feed.

---

### 6. Hybrid world/screen annotation overlay with a lossless GeoJSON codec

- **What it is** — Every mark is world-anchored (lon/lat) in the data model, but rendering is routed per mark type: polygon footprints and routed paths are draped into the 3D scene via `ClassificationType.CESIUM_3D_TILE` so they conform to real building and ground geometry, while captions, pins, arrows and reticles draw as screen-space SVG. A separate Cesium-free module converts the runtime model to and from a GeoJSON FeatureCollection with `gev:`-namespaced properties.
- **Where it lives** — Routing table and rationale at `src/annotations/hybridAnnotationRenderer.js:4-59`; the prototype-chain `liveProxy` letting both sub-renderers read per-frame `alpha`/`bornAt`/`expiring` while keeping their own state isolated (`:136-141`); partial-failure discipline — route the entry *before* filling it so a WebGL loss mid-add is still addressable (`:29-34`); draping mechanics including why there is no terrain to clamp to under photoreal tiles at `src/annotations/worldAnnotationRenderer.js:10-17`; the GeoJSON codec at `src/annotations/annotationGeoJson.js:1-80`; and engine safety rails — a hard cap of 120 live marks, a 22 s default TTL with `persist=true` exempt, and a tri-state deferred retry that **never retries a definitive "no polygon" answer** (`src/annotations/annotationEngine.js:27-79`) (all verified).
- **Maps to** — **(a)** directly. Tract outlines that drape onto real relief instead of floating as flat rings, with captions that stay legible; the GeoJSON codec is your save/export format. **(c)** for annotating strata or borehole locations over a 3D surface.
- **Effort class: heavy.** 4,580 lines across seven modules, of which the resolver alone is 1,948. The **renderer split and the codec are the transplantable core** (150 + 435 + 213 lines); the resolver is name→geometry lookup you would replace wholesale with your parcel index.
- **License — split, and this matters.** The engine, the three renderers, and the codec are pure MIT + Cesium. The **resolver drags in two non-MIT upstreams**: Google Geocoding directly with an API key at `:613` (proprietary, BYOK, no caching allowed) and OSM Overpass at `:820` (ODbL). Take the renderers and codec cleanly; treat the resolver as a slot to fill.
- **One honest caveat:** the GeoJSON codec is **well-built but unwired**. The only references to it anywhere in the tree are its own test file. You get a clean, round-trip-tested codec — not a persistence feature (verified: repo-wide grep).

---

### 7. Real vertical datum + sampling against the *rendered* mesh

- **What it is** — A four-layer height stack that keeps every consumer agreeing on one surface. EGM96 geoid undulation converts orthometric (MSL) inputs to ellipsoidal heights (`h = H + N`). A batched, coordinate-rounded proxy resolves DEM ground height per point. A coarse ~111 m grid caches those so moving objects collapse onto reusable cells. And a rationed one-shot sampler probes the *actually rendered* photogrammetric mesh — whose surface sits **above** bare earth — reporting validated cells back so reads answer mesh-first.
- **Where it lives** — The datum statement at `src/data/geoid.js:1-21` (`h = H + N`, N roughly −106…+85 m worldwide), lazy 2.7 MB grid import at `:33-38`; the batched resolver with two hard-won constraints — chunk at 200 points to stay under a Node header-size ceiling, round to 5 dp so the disk cache key is stable — at `src/data/terrainHeights.js:15-58`; the coarse cell grid with its explicit *"visual floor, not a survey"* framing at `src/data/groundFloor.js:12-64`; **the single choke point everything reads, mesh-first then DEM**, at `:473-480`; and the mesh sampler's rationing — near-viewer only (15 km), ≤40 samples/call, one-shot per cell, skipped above 25 km camera height, with an **asymmetric acceptance window** against the DEM prior because the mesh legitimately sits above bare earth — at `src/data/meshFloorSampler.js:1-38` (all verified).
- **Maps to** — **(c), and it is the load-bearing pattern.** Here is why: strata visualization is *entirely* a question of what "zero" means. A seam surface digitized against MSL, a borehole logged against a local datum, and a Cesium scene that only speaks ellipsoidal height will disagree by tens of metres — which for strata is not a cosmetic error, it is the whole signal. This stack gives you (i) the MSL↔ellipsoid conversion, (ii) a real ground surface to hang sections from, (iii) the distinction between *bare-earth DEM* and *rendered mesh*, which is precisely the distinction between "where the rock starts" and "where the pixels are", and (iv) a caching and rationing discipline that makes per-point height queries affordable at strata-mesh vertex counts. **(a)** secondarily: parcel corners with survey elevations land on the same conversion problem.
- **Effort class: heavy.** 1,084 lines across four modules, and the mesh sampler additionally depends on the server-side `/api/terrain/heights` proxy and on knowing which map regime is active. The geoid wrapper alone is an afternoon; the full stack is not.
- **License — mixed but mostly clean.** Code MIT. The geoid math comes from `egm96-universal`, described in a source comment as MIT with an embedded NGA grid (the package's actual license text is not in this checkout — **unverified**). The DEM path is Re:Earth Terrain / Mapterhorn — mesh CC BY 4.0, keyless. **The mesh sampler's surface is Google Photorealistic 3D Tiles, which may not be cached, stored, or rehosted.** You may sample it live as this code does, but the sampled heights are the product of a Google surface — decide deliberately whether to persist them.

---

### 8. Label arbiter under a ref-counted render governor

- **What it is** — Label placement treated as a bounded allocation problem, not a per-label decision. Competing layers declare demand; a work-conserving allocator hands out collective capacity by one of two deterministic strategies; a pooled uniform spatial hash resolves collisions without per-candidate allocation; and time-based hysteresis stops the cohort strobing as the camera moves. Separately, the whole scene sits in Cesium's `requestRenderMode` unless some animator holds a named, identity-keyed hold.
- **Where it lives** — Timing constants that produce the stability at `src/data/labelArbiter.js:7-12` (150 ms fade-in, 300 ms fade-out, **2,500 ms minimum lifetime, 1,200 ms exit cooldown**); two allocation strategies, both work-conserving with unused entitlement always borrowed, at `:155-220` — `ELASTIC` (equal split + remainder walk) and `WEIGHTED` (`sqrt(count) × semanticWeight`, largest-remainder apportionment, guaranteed floor of 1 per layer); the spatial hash at `:240-300`; a golden-row differential test locking selection, placement corners, quotas, fade rows and even spatial cell counts at `src/data/labelArbiterDifferential.test.mjs:10-30`; and the render governor at `src/renderGovernor.js:1-113`, whose problem statement is *~60% GPU with zero layers and a parked camera* (all verified).
- **Maps to** — **(a)** a tract board is exactly a fixed screen budget with competing label sources (tract ID, status, provenance badge), and the hysteresis is the difference between a workbench and a flickering demo. **(c)** the same problem for strata unit labels at varying camera distance. **(b)** quota allocation across sources is a natural fit for a board that must not let one chatty feed crowd out the rest.
- **Effort class: heavy** for the arbiter (1,125 lines, and the golden test implies the behavior is load-bearing); **afternoon** for the render governor in isolation (137 lines, zero dependencies) — **the single highest ratio of benefit to effort in the repo.**
- **License** — Pure MIT, no data touched.
- **Cheap starting point:** this repo runs *two* label systems. `worldOverlay` uses the arbiter, but the bundled-polygon layers publish through a simpler greedy screen-grid selector at `src/data/localGeojson.js:162-207` — one winner plus one surplus per grid cell, sorted by priority. That is ~45 lines and is the honest place to start (verified).

---

### Candidates thinner than they sound — said plainly

- **`src/styles/*.js` will not give you (c).** They are full-frame post-process looks — `{name, uniforms, fragmentShader}` records instantiated one stage per style. **Nothing here does clipping planes, cross-sections, or volumetric shading** — the things strata actually needs. What *is* transplantable is the surrounding convention: a uniform registry with `default/min/max/label` that simultaneously generates UI sliders and serializes into the share link via an allowlisted token table. **Afternoon** for that convention; the strata rendering itself is not in this repo (verified: src/styles/thermal.js:14-23; src/ui.js:2946-2982; src/sharelink.js:55-86).
- **`src/cctvFocusPolicy.js` is not the provenance pattern** — all 81 lines are camera-ownership arbitration. **The "estimated until you calibrate it" pattern you want is elsewhere and is genuinely good:** a three-state trust badge — `'calibrated'` (a human saved it), `'curated'` (hand-authored catalog entry), `'raw-prior'` (everything else) — derived by a pure function, with an explicit note that **the old fabricated confidence score was retired rather than kept**. Open-data rows *"stay RAW PRIOR until a human manually calibrates them"* (verified: src/data/cctv.js:765-784, :1146-1149). The store keeps value + provenance + `savedAt` (`:148-149`). For **(a)** that maps to "surveyed / recorded / inferred" per tract almost one-to-one. **Afternoon** for the badge derivation and store shape; the direct-manipulation gizmo behind it is a separate **heavy** item.
- **`src/scenes/director.js` is real; the recipes are not reusable.** The director has genuine substance — cancellation tokens threaded through every await, abort-on-supersede so an in-flight layer transition is actually cancelled rather than disowned, localStorage persistence with a schema version and a migration. The recipes are hardcoded whole-globe camera paths for social clips. **Steal the shot model and the cancellation discipline; write your own paths. Weekend** (verified: src/scenes/director.js:729-732, :915, :1026-1042; src/scenes/recipes.js:6-30).
- **`worldOverlayAllocation.worker.mjs` is not a web worker** — no `postMessage`, no `onmessage`. It is a Node child-process harness reporting GC-bracketed per-frame heap deltas. What it *actually is* — a per-frame allocation-budget regression test with published budgets per workload profile — is itself worth stealing for **(c)**, where a strata mesh will make you care about per-frame allocation. **Weekend** to stand up your own (verified: grep over the file; src/overlays/worldOverlayAllocation.test.mjs:31-40).

---

## 9. LICENSE CAUTIONS

### 9a. What MIT covers

The grant is unmodified MIT, `Copyright (c) 2026 Bilawal Sidhu` (verified: LICENSE:3-10). The file then narrows what "the Software" means, after the `---` at LICENSE:23:

> *"NOTE ON THIRD-PARTY DATA AND ASSETS — THE MIT LICENSE ABOVE COVERS THE SOURCE CODE ONLY."* (verified: LICENSE:25-26)

> *"The datasets bundled under src/data/local_data/, and all data fetched from third-party providers at runtime, are owned by their respective sources and are NOT licensed under MIT."* (verified: LICENSE:28-30)

> *"The 3D models under public/models/ are also third-party works and are NOT licensed under MIT."* (verified: LICENSE:51-52)

- **Covered:** `src/**/*.js`, `vite.config.js`, `index.html`, `style.css`, `scripts/`, `tools/`, docs prose.
- **Not covered:** everything under `src/data/local_data/`, `public/models/`, `docs/media/`, and **every byte fetched at runtime**.
- **`package.json:6` declares `"license": "MIT"` with no carve-out field.** A downstream consumer reading only package metadata gets no warning at all (verified).
- Inbound: *"By contributing, you agree your contributions are licensed under the project's MIT License"* — no CLA, no DCO (verified: CONTRIBUTING.md:59).

### 9b. Hard blockers — remove or re-source before any commercial ship

**🔴 TeleGeography submarine cables — NonCommercial.** The only source in the tree that forbids commercial use outright.

> *"the data may not be used commercially. If you use this project commercially, **remove these files** (`cable-geo.json`, `landing-point-geo.json`) or obtain a commercial license from TeleGeography."* (verified: telegeography_submarine_cables/README.md:14-16)

Removal is genuinely one folder plus three references: `src/data/local_data/telegeography_submarine_cables/`, the import at `src/data/localLayers.js:3`, the module `src/data/telegeographySubmarineCables.js`, and the credit at `src/data/dataCredits.js:171-177`. Whether the Vite build stays green after deletion is **unverified** — the loader uses `new URL(…, import.meta.url)`, which Vite statically analyzes and emits as an asset.

**🔴 OpenSky Network — non-commercial, and operational use may require a written agreement.** *"Its license is non-commercial, and operational use of the REST API in a live product can require a prior written agreement with OpenSky — even for non-profit/government use."* (verified: DATA_SOURCES.md notes). Re-sourcing means contacting OpenSky, or leaning on adsb.lol — which is ODbL and commercially usable but explicitly not equivalent coverage: *"regional observed context, not worldwide completeness"*, capped at 250 nm.

**🔴 Google News RSS — personal, noncommercial use only.** *"commercial deployments must disable/replace it or obtain separate permission."* GDELT is already wired as the fallback and permits commercial use with citation, so this is a config change rather than new integration work (verified: DATA_SOURCES.md:52).

**🔴 `docs/media/` — all-rights-reserved with a narrow permission.** See §9e.

### 9c. Encumbrances — commercial use permitted, with binding conditions

| Source | Blocker | What compliance costs |
|---|---|---|
| Google Map Tiles / Places / Geocoding | Proprietary ToS + own key + **no caching** + mandatory visible attribution | *"Google Maps Content (tiles, geocodes, places) may not be cached, stored, rehosted, or committed"*; the "Google" credit must stay visible |
| Datacenters + Dams (**ODbL 1.0**) | Attribution + **database share-alike** | Keep "© OpenStreetMap contributors" with the copyright link; ship modified databases under ODbL |
| adsb.lol, Overpass, Nominatim, OSM tiles, FOSSGIS OSRM (**ODbL 1.0**) | Same, across five live feeds | Nominatim adds a ≤1 req/s policy |
| TomTom Traffic | Proprietary, BYOK, quota governor, attribution-on-display | *"the application default is a safety limit, not a promise of free quota"* |
| TfL Open Data | **Attribution REQUIRED**, verbatim | "Powered by TfL Open Data. Contains OS data © Crown copyright and database rights" |
| Open-Meteo (**CC BY 4.0**) | Attribution **with an adjacent link** | Rendered inline beside the values, not only in the popover |
| Re:Earth Terrain (**CC BY 4.0**) | Attribution | Credit the mesh and the geoid |
| **AISStream.io** | **No license grant at all** | *"Free, beta, no formal ToS."* **No ToS is not the same as a permissive ToS** — this is unquantified risk for a commercial ship, and a free beta can change terms unilaterally |
| Radio Browser (**PDDL 1.0**) | Directory is public domain; **broadcaster stream terms are not** | Direct playback exposes the listener's IP to the broadcaster |
| Launch Library 2 | Rate cap + "don't forward without added value" | 15 unauthenticated calls/hour |

**Clean for commercial use, no re-sourcing:** CelesTrak, USGS, NASA FIRMS (CC0), Natural Earth (public domain), DataSF (PDDL 1.0), GDELT, and all nine `.glb` models (CC BY 4.0).

### 9d. The share-alike traps, specifically

**ODbL 1.0 — and the trap is already sprung.**

- It infects **the data and any derived database**, not the MIT code. *"ODbL's share-alike applies to the data / derived database, not this MIT-licensed code — the two coexist."* (verified: DATA_SOURCES.md:79)
- **But the bundled copies are not raw OSM.** They carry a privacy transform removing contact-oriented tags, and DATA_SOURCES.md names the consequence: *"the resulting derived databases remain under ODbL 1.0."* (verified: :81-85). **This repo is already publicly distributing modified derived databases under ODbL, and every downstream fork inherits that obligation whether or not it touches the files again.**
- Two provenance gaps weaken an ODbL defense: `datacenters/README.md:17-18` admits *"The original extraction date and query were not recorded alongside this snapshot"*; `dams/README.md:3` gives filters and a count but no date and no source URL.

**CC BY-NC-SA 3.0 — two independent hooks.** The **NC** hook is the ship-stopper. The **SA** hook bites separately: any adaptation of the cable geometry — a simplified pack, a reprojection, a tiled version, a merged cable+landing dataset — must remain CC BY-NC-SA 3.0. **You cannot launder the NC clause by re-deriving the geometry** (verified: telegeography README:17-18).

**No share-alike anywhere else.** CC BY 4.0 is attribution-only. PDDL, CC0 and public domain carry no downstream obligation.

### 9e. The 3D models — per-file verdict

All nine `.glb` files are Sketchfab works under **CC BY 4.0**, which *"permits sharing and adaptation, including commercial use, provided appropriate credit is retained, the license is linked, and modifications are identified"* (verified: public/models/README.md:19-21).

`airplane.glb` · `jet.glb` · `ship.glb` · `bell206.glb` · `c172.glb` · `citation2.glb` · `mq9.glb` · `b789.glb` · `atr72.glb` — **all safe to ship**, each with a named creator, source link, license link, and an unusually specific modification notice (verified: public/models/README.md:9-17; nine files on disk, nine table rows, no undocumented model).

Three conditions to keep the grant alive:

- **Credit must travel with the binaries.** `public/models/README.md` is the only artifact carrying attribution, and Vite's default `publicDir` copies it into `dist/models/` on build. **Do not "clean" it out** (inferred: from the absent `publicDir` override plus Vite's documented default — not verified by running a build).
- **There is no in-app model credit.** `DATA_CREDITS` covers 22 data sources and **zero models** (verified: src/data/dataCredits.js:28-178). A shipped product whose only attribution surface is the in-app popover would not surface the CC BY 4.0 model credits at all.
- **Keep the modification notices verbatim** — they are exactly what CC BY 4.0's "identify modifications" clause wants.

### 9f. `docs/media` — explicitly not MIT

> *"Copyright © Bilawal Sidhu. These files are not covered by the project's MIT License. Permission is limited to their inclusion and redistribution with this repository and its project documentation. No permission is granted for standalone reuse or modification. Commercial reuse outside this repository requires separate permission."* (verified: docs/media/README.md:25, :38)

And the nested-rights warning, which is the part people miss:

> *"Bilawal Sidhu's ownership and permission cover the captures and his likeness; they do not replace the terms of Google Maps Platform or any displayed data provider. Keep the visible attribution intact."* (verified: docs/media/README.md:27)

**Practical consequence:** a commercial fork may keep the GIFs *in its documentation of this project*. It may not lift one into a landing page, an ad, a pitch deck, or a different product's README.

### 9g. Where LICENSE itself is wrong

- **It never mentions `docs/media/`**, even though docs/media/README.md and README.md both state those files are not MIT-covered (verified).
- **`LICENSE:39-40` still carves out a bundled NASA FIRMS snapshot that was deleted 2026-07-16** (verified: DATA_SOURCES.md:96; no such folder on disk).
- **DATA_SOURCES.md:59 says each of the 5 bundled datasets is carved out "in LICENSE"; LICENSE names only 3.** Natural Earth and DataSF are absent — both public domain, so low stakes, but the claim is false as written (verified).
- **`tools/README.md` carries no license or terms language at all**, for scripts that write Google Map Tiles and Street View imagery to `output/*.png` — in direct tension with *"may not be cached, stored, rehosted, or committed."* Gitignoring the output prevents *committing* but not *storing* (verified: tools/README.md:3-12; .gitignore:7).

### 9h. Minimum viable commercial-ship checklist

1. Delete `src/data/local_data/telegeography_submarine_cables/` plus the import at `localLayers.js:3`, the module, and the credit — or buy the TeleGeography license.
2. Replace or license OpenSky; verify the ODbL adsb.lol fallback covers your need despite its 250 nm bound.
3. Disable the Google News RSS path; let GDELT serve headlines with its required citation and link.
4. Keep every ODbL attribution, and be prepared to publish modified datacenter/dam databases under ODbL.
5. Keep `public/models/README.md` in the shipped bundle; add model credits to `DATA_CREDITS` so CC BY 4.0 attribution reaches the running UI.
6. Keep `#cesium-credits` and the "Data attribution" popover visible in **all** modes, including clean-view and recording.
7. Reuse nothing from `docs/media/` outside this repo's own documentation.
8. Decide whether the committed `© TomTom` fixture and the unreferenced 713 KB `dams.geojson` twin are worth carrying.

---

## 10. OPEN QUESTIONS

Only the eight that you have to answer. Everything answerable from the repo is answered above.

1. **Is the land-title parcel geometry a live system-of-record you can query, or a periodic export?** The live-target handoff pattern (pattern 2) is only worth the port if a tract's status can genuinely change between sending a link and opening it. Against a nightly export it is dead weight and the simple share-link codec is enough.

2. **Does any of the three shapes have to run without a Google Maps key, or offline?** The app hard-aborts without it, and the photorealistic mesh is the surface the mesh-floor sampler probes — so an air-gapped title office or a mine-site deployment loses both the basemap and the height stack's top tier at once. If yes, the whole altitude chapter has to be re-planned around DEM only.

3. **For the evidence-first board: who is the accountable reader?** An analyst who can act on a DEGRADED chip, or a court/regulator/insurer where "we displayed a stale number" is liability? That single answer decides whether the six-word feed-state enum is sufficient, or whether every rendered datum needs its own retained provenance record with a timestamp you can produce later.

4. **Is any of the three a commercial product?** The MIT grant excludes all bundled data and runtime feeds. TeleGeography must be deleted for commercial use; OpenSky and the Google News cockpit source are NonCommercial too. Only you can say whether "commercial" applies.

5. **What vertical datum do your seam surfaces and borehole logs actually carry** — a named geoid model, a local mine datum, or an undocumented one? The geoid/DEM/mesh stack is transplantable only if the answer is convertible. If the logs are on an unrecorded local datum, this repo solves none of that problem and the first work item is datum reconciliation, not rendering.

6. **Does the strata product need true volumetric rendering** — clipping planes, arbitrary cross-sections, per-unit shading through a solid? Nothing in this repo does that. If you need sections, that is a build-from-scratch subsystem and the repo's value to shape (c) drops to the height stack and the label arbiter alone.

7. **Which shape ships first, and is this repo the codebase or the reference implementation?** Forking means owning a 455 KB `src/ui.js` and a 7,383-line `vite.config.js` with no CI and no production server mode. Lifting the eight named patterns into a new project means owning less and rebuilding the shell. That is a resourcing decision, not a technical one.

8. **Are you keeping the no-framework, vanilla-JS constraint?** Every pattern worth stealing here is dependency-free pure JS and survives either way — but the UI layer, the panel system, and the toggle-panel rendering are hand-rolled DOM that does not.

---

## INVENTORY

**Files read: 171** of 428 tracked files, cited with 1,500 distinct `file:line` pointers across 160 paths plus 11 whole-file references. An adversarial audit of every pointer found **zero nonexistent paths, zero out-of-range lines, and zero wrong-file citations**; the only defects were 19 pointers landing on a blank line adjacent to their subject.

**Files NOT FOUND: 33**, in three classes.

*Named by a document, absent from the tree — real defects (14):* `src/iconOrientation.js` (README.md:253; the file is at `src/data/iconOrientation.js`) · the bundled NASA FIRMS snapshot still carved out at LICENSE:39-40 (deleted 2026-07-16) · `.npmrc` (so the `engines` range README.md:62 and CONTRIBUTING.md:7 call "enforced" is not) · an `npm run dev:fresh` script (six documents instruct `./scripts/dev-fresh.sh`; no alias exists) · `.env.example` entries for `LL2_API_TOKEN`, `TFL_APP_KEY`, `OPENSKY_USERNAME`, `OPENSKY_PASSWORD`, `AISSTREAM_URL`, and the four `CCTV_CALTRANS_*`/`CCTV_TFL_*` vars · DATA_SOURCES.md entries for `api.adsbdb.com`, FOSSGIS OSRM, and the Google Street View Static API · a provenance entry for `firms-viirs-noaa20-sample.csv` · the Natural Earth curation script (`natural_earth/README.md:21`: "script not committed") · any license section in `tools/README.md` · any 3D-model entry in `DATA_CREDITS` · the CC BY-NC-SA string in the TeleGeography `source.json` · any findings document defining H10/H11 (they exist only in a QA script header at `scripts/qa-attribution-b12.mjs:5-8`).

*Absent by design or convention (16):* `.github/` (**no CI of any kind**) · `server/` · `Dockerfile` · `docker-compose.yml` · `eslint.config.js`/`.eslintrc` · `.prettierrc` · `.editorconfig` · `tsconfig.json` · `vitest.config.js`/`jest.config.js` · `.nvmrc` · `CODE_OF_CONDUCT.md` · `Makefile` · `yarn.lock`/`pnpm-lock.yaml` · `CLAUDE.md`/`.claude/` · any Windows setup doc · any `.ps1`/`.cmd`/`.bat`.

*Runtime directories, correctly gitignored (3):* `.env` · `.gev-logs/` · `.gev-cache/`.

Every count the README asserts — 13 layers, 28 voice tools, ~800 cameras, 4,351 datacenters, 704 dams, 712 cables, 1,917 landing points, 17 GIFs, 750 radio stations, $2/$5 — **verifies against the tree**. The gaps are not missing files. They are a `.env.example` frozen at CCTV caps of 36/48 while the code ships 250/900, two env flags no line of code reads, a `LICENSE` carving out a dataset deleted in July while omitting `docs/media/` entirely, an empty `config/cctv_sources.austin.json`, an orphan `config/cctv_sources.shinjuku.json`, a `package.json` at 0.1.0 against a CHANGELOG at 0.7.0 with four separate `[Unreleased]` headings — and no continuous integration at all.
