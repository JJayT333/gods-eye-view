# Running this fork on a Mac

Personal setup notes for this fork. The `mac-setup` branch is upstream `main`
plus this folder and `.claude/launch.json` (Claude's preview-pane config).
Nothing here is needed to run the app; it all starts without keys.

## 1. Get the code

- Install Git and Node.js **24.x (24.14.0 or later) or 26.x**. The setup doctor
  warns about Node 25, which is end-of-life.
- Clone the fork, switch to this branch, and add the original project as `upstream`:

```sh
git clone https://github.com/JJayT333/gods-eye-view.git
cd gods-eye-view
git switch mac-setup
git remote add upstream https://github.com/bilawalsidhu/gods-eye-view.git
```

- Cloning a public repo needs no login. Pushing does: `gh auth login`, or an SSH key.
- Pushes that carry upstream's CI workflow files are refused for a `gh` token
  without the `workflow` scope. Push over SSH, or run `gh auth refresh -s workflow`.

## 2. Install and run

```sh
npm ci
npm run doctor
./scripts/dev-fresh.sh
```

- Open **http://localhost:4173**.
- `dev-fresh.sh` clears the Vite cache, reads any keys from the macOS Keychain,
  and serves on port 4173 unless `PORT` is set.
- Plain `npm run dev` also works, but without `PORT=4173` in `.env` it falls back
  to Vite's default port, 5173. Use whatever address the server prints.

## 3. Accounts and keys

None are required. Each one switches on one more capability.

| Account                         | Unlocks                                     | Cost                                                   | POWER UP field | Keychain service / account    |
| ------------------------------- | ------------------------------------------- | ------------------------------------------------------ | -------------- | ----------------------------- |
| AISStream                       | Live ships                                  | Free                                                   | AISSTREAM      | `aisstream-api` / `api-key`   |
| NASA FIRMS                      | Live active fires                           | Free                                                   | NASA FIRMS     | `firms-map` / `map-key`       |
| TomTom                          | Real live traffic (keyless is a simulation) | Free tier                                              | TOMTOM         | `tomtom-api` / `api-key`      |
| Google Cloud                    | Photorealistic 3D and Google address search | Metered, needs billing                                 | GOOGLE MAPS    | `google-maps-api` / `api-key` |
| OpenAI (optional)               | Voice control and the AI HUD summary        | Pay per use; the app warns at $2, ends a session at $5 | OPENAI         | `openai-api` / `api-key`      |
| Cesium ion (Google alternative) | The same 3D tiles plus world terrain        | Free for eligible personal, non-commercial use         | CESIUM ION     | `cesium-ion` / `token`        |

Sign-up pages: https://aisstream.io, https://firms.modaps.eosdis.nasa.gov/api/map_key/,
https://developer.tomtom.com, https://console.cloud.google.com,
https://platform.openai.com/api-keys, https://ion.cesium.com/tokens.

OpenSky and Launch Library already work without an account; one only raises
their request limits.

Two places to keep keys on the Mac:

- **POWER UP** (bottom-right chip) > Provider Settings > paste > SAVE KEYS.
  Keys land in the repo's `.env`, which Git ignores, and the app restarts itself.
- **macOS Keychain** (the stronger option, per the README), then start with
  `./scripts/dev-fresh.sh`. The command prompts for the key:

```sh
security add-generic-password -U -s "aisstream-api" -a "api-key" -w
```

Keychain keys show in POWER UP as "configured externally" (read-only there).

## 4. Google Maps key (photorealistic 3D)

1. https://console.cloud.google.com > create a project.
2. Billing: link a billing account. Google requires it even inside the free allowance.
3. APIs & Services > Library: enable **Map Tiles API** and **Geocoding API**.
4. APIs & Services > Credentials > Create credentials > API key.
5. Edit the key:
   - Application restrictions: Websites. Add `http://localhost:4173/*` and
     `http://127.0.0.1:4173/*`, plus the same two with `5173` if you use plain `npm run dev`.
   - API restrictions: restrict the key to Map Tiles API and Geocoding API.
6. Spend protection: create a budget under Billing > Budgets & alerts (it only
   emails), and lower the per-day quotas under APIs & Services > Map Tiles API >
   Quotas (that is the real cap).
7. Paste it into POWER UP > GOOGLE MAPS, or the Keychain as `google-maps-api` / `api-key`.

The README says the first 1,000 Photorealistic 3D Tiles sessions a month are
currently free; confirm on Google's pricing page.

Optional server key for nearby places and the Street View camera fallback: a
second key restricted to **Places API (New)** and **Street View Static API**. POWER UP
hides this field, so add it to `.env` as `GOOGLE_MAPS_SERVER_API_KEY=...`. See SECURITY.md.

## 5. Antennas (optional)

No antenna is needed. Every layer works over the internet.

Hardware only matters for the **Local ADS-B** layer, which draws aircraft your
own receiver hears. They usually appear ahead of the public Flights layer, which
renders about 30 seconds behind. Full details: `docs/LOCAL-RECEIVERS.md`.

- **Simplest:** one plain RTL-SDR USB dongle with a 1090 MHz antenna, opened by
  the browser (Radio panel > Local RTL-SDR card). The doc names desktop Chrome or
  Edge; nothing else to install.
- **978 MHz UAT, or two bands at once:** run decoders (dump1090-fa, readsb or tar1090
  for 1090; dump978-fa with skyaware978 for 978) and point `LOCAL_RECEIVER_FEEDS`
  in `.env` at their `aircraft.json`. The doc has a macOS example and mentions
  dual-band boards such as the Nooelec FlyCatcher, and Raspberry Pi feeders.
- A dongle can be opened by only one program at a time.

## 6. Use it from Claude (MCP)

From `docs/MCP_SETUP.md`:

```sh
npm run build:panel   # the in-conversation globe; rerun after app changes
```

Claude Desktop: Settings > Developer > Edit Config, then add:

```json
{
  "mcpServers": {
    "gods-eye-view": {
      "command": "/full/path/from/which/node",
      "args": [
        "/full/path/to/gods-eye-view/server/mcp/stdio.js",
        "--api-base",
        "http://localhost:4173"
      ]
    }
  }
}
```

- Use absolute paths; desktop apps do not load your shell profile.
  `which node` prints the Node path.
- The doc's example uses port 5173; use the port your server prints.
- Fully quit and reopen Claude Desktop, start a new chat (not the Code tab), and
  ask: "Show San Diego in God's Eye View with military flights on."
- Claude Code: `claude mcp add gods-eye-view -- node /full/path/to/gods-eye-view/server/mcp/stdio.js --api-base http://localhost:4173`

## 7. Staying current

`main` on the fork mirrors upstream; work happens on branches like this one.

```sh
git fetch upstream
git switch main && git merge --ff-only upstream/main && git push origin main
git switch mac-setup && git rebase main && git push --force-with-lease origin mac-setup
npm ci                # when package-lock.json changed
npm run build:panel   # when using the MCP globe
```
