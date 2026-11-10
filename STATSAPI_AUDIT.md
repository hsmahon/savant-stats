# Stats API discovery audit

Reviewed **2026-11-09**. The authoritative snapshot is [statsapi_openapi.json](statsapi_openapi.json); [statsapi_route_inventory.yaml](statsapi_route_inventory.yaml) indexes every published operation, parameter, declared response schema, access check and supplementary lead.

## Coverage and provenance

The public [documentation shell](https://docs.statsapi.mlb.com/) explicitly references a main bundle and runtime. The runtime lists **137 public application chunks**; all were retrieved successfully. Their metadata identifies **110 documentation pages**, including release notes, tracking specifications, authentication, video resources and game data. The inventory retains each page's permalink and source asset URL.

The API-reference asset [`726afaaf.538fc284.js`](https://docs.statsapi.mlb.com/assets/js/726afaaf.538fc284.js) embeds a JSON OpenAPI document. Its parsed snapshot contains **200 paths / 200 GET operations and 533 component schemas**. All operations are cataloged with full published parameter metadata and declared response schema references; references resolve into the companion snapshot. Source and snapshot SHA-256 hashes are recorded in the inventory. The document declares OpenAPI 3.1 and info version 2.0.0; HTTP route versions are separately v1 and v1.1. These are different version concepts.

The public live resource [`https://statsapi.mlb.com/v3/api-docs`](https://statsapi.mlb.com/v3/api-docs) also returned **200 JSON** (573,729 bytes). It contains the same 200 paths and 533 component schemas. All components and all parameter metadata excluding descriptions match the bundled snapshot. The bundled version enriches summaries, descriptions and some parameter descriptions. The inventory records the live-response hash and comparison; the checked-in snapshot retains those documentation annotations.

Searching all chunks and specification text additionally found **12 document-only operations**: two legacy tracking GET routes, five POST operations, and five WebSocket subscription forms. These are kept separate because they are not operations in the embedded OpenAPI map. The full reference snapshot is data, not retained JavaScript.

A [community SDK](https://github.com/toddrob99/MLB-StatsAPI/blob/master/statsapi/endpoints.py) supplied **64 parameterized route patterns** as comparison leads. They are retained separately with their original path-building rules and query names, not presented as 64 additional verified endpoints. Several describe operations already present in the official specification; generic `{type}` placeholders are not grounds to invent new paths.

## Anonymous response checks

**160 distinct OpenAPI operations** were checked, spanning **all 34 documented tag families**: **129 returned anonymous JSON**, **22 returned authentication HTML with HTTP 401**, **7 returned JSON errors**, and **2 returned non-JSON 404 responses**. Each observation records the exact request URL, content type, status and response shape in the inventory. Queries used one known historical game (`716463`), player (`592450`), historical seasons and bounded dates/limits where applicable. A successful empty response is not proof that a dataset has records for every game.

The [official authentication topic](https://docs.statsapi.mlb.com/getting-started/authentication-authorization) distinguishes unsupported public usage from licensed Statcast access. The application's documentation UI uses an authentication gate, while referenced public assets contain documentation. No login, credential submission, token generation, authorization change or bypass was attempted. Reading a published schema does not demonstrate access to its underlying dataset.

The specification declares authentication for many routes that nevertheless returned anonymous JSON. Accordingly, **declared security and observed access are separate fields**. The absence of an operation-level security block also does not guarantee unrestricted production access.

### Newly useful public surfaces

| Family | Observed capabilities |
| --- | --- |
| Game metrics | `/game/{gamePk}/withMetrics` returns a Gumbo feed with `contextMetrics` fields attached to plays/events. `/game/{gamePk}/contextMetrics` returns game context, field-direction sacrifice-fly probabilities and win probabilities. |
| Search configuration | `/stats/search/config`, `/params`, `/stats`, `/groupByTypes` under `/stats/search` expose search-related metadata. Returning configuration does not prove access to the underlying `/stats/search` query. |
| Tracking/video catalogs | Tracking versions/vendors/software/owners, coaching video types, resolution types, color-space types and Statcast position types are discoverable taxonomy resources. They do not themselves return video bytes or pose trajectories. |
| Baseball reference | Uniforms by game/team; team alumni/coaches/history interfaces; player awards; milestones; draft/prospects; postseason schedules; jobs; broadcast catalogs and other reference families expand the earlier quick catalog. |

The game-metrics paths returned data for the tested game. Their field names alone do not establish numeric units. In particular, observed play `contextMetrics.xWoba` values such as `7.6` and `75.1` should not be silently interpreted as conventional expected-wOBA values.

### Authentication boundaries and response failures

| Published operation | Observed result |
| --- | --- |
| `/api/v1/stats/metrics` | 401 Okta login HTML |
| `/api/v1/teams/{teamId}/history` | 404 Object not found |
| `/api/v1/stats/analytics/stolenBaseProbability` | 401 Okta login HTML |
| `/api/v1/stats/analytics/sprayChart` | 401 Okta login HTML |
| `/api/v1/stats/analytics/outsAboveAverage` | 401 Okta login HTML |
| `/api/v1/schedule/trackingEvents` | 401 Okta login HTML |
| `/api/v1/review` | 500 Internal error occurred |
| `/api/v1/people/{personId}/measuredHeights` | 401 Okta login HTML |
| `/api/v1/homeRunDerby` | 500 Internal error occurred |
| `/api/v1/homeRunDerby/bracket` | 500 Internal error occurred |
| `/api/v1/homeRunDerby/pool` | 500 Internal error occurred |
| `/api/v1/homeRunDerby/mixed` | 500 Internal error occurred |
| `/api/v1/game/{game_pk}/feed/color` | 404  |
| `/api/v1/game/{game_pk}/feed/color/timestamps` | 404  |
| `/api/v1/game/{gamePk}/{playId}/analytics/skeletalData/files` | 401 Okta login HTML |
| `/api/v1/game/{gamePk}/{playId}/analytics/biomechanics/{positionId}` | 401 Okta login HTML |
| `/api/v1/game/{gamePk}/{guid}/homeRunBallparks` | 404 Object not found |
| `/api/v1/game/{gamePk}/{guid}/contextMetrics` | 401 Okta login HTML |
| `/api/v1/game/{gamePk}/{guid}/contextMetricsAverages` | 401 Okta login HTML |
| `/api/v1/game/{gamePk}/{guid}/analytics` | 401 Okta login HTML |
| `/api/v1/game/{gamePk}/guids` | 401 Okta login HTML |
| `/api/v1/game/lastPitch` | 401 Okta login HTML |
| `/api/v1/batTracking/game/{gamePk}/{playId}` | 401 Okta login HTML |
| `/api/v1/analytics/guids` | 401 Okta login HTML |
| `/api/v1/analytics/game` | 401 Okta login HTML |
| `/api/v1/weather/venues/{venueId}/basic` | 401 Okta login HTML |
| `/api/v1/weather/venues/{venueId}/full` | 401 Okta login HTML |
| `/api/v1/weather/game/{gamePk}/{playId}` | 401 Okta login HTML |
| `/api/v1/deployment/info` | 401 Okta login HTML |
| `/api/v1/streaks` | 401 Okta login HTML |
| `/api/v1/props/play/predictions` | 401 Okta login HTML |

A 401 establishes an authentication boundary for that request, not the unavailability of the operation to licensed clients. A 404 or 500 on one historical query does not disprove a published route: data coverage, additional inputs or obsolete documentation may explain it. For example, testing a Home Run Derby collection with only `season=2023` returned 500; no fabricated derby game identifier was substituted.

## Operations outside the OpenAPI map

- **Legacy tracking reads:** `/game/{gamePk}/{GUID}/analytics/raw` and `/analytics/parsed` appear in the [raw](https://docs.statsapi.mlb.com/tracking-data/tracking-data-raw) and [parsed](https://docs.statsapi.mlb.com/tracking-data/tracking-data-parsed) topics. Zero-width spaces in rendered links were normalized in the catalog. These documented shapes are distinct from the current formatted `/analytics` operation.
- **Video hydration:** the [video URLs topic](https://docs.statsapi.mlb.com/media-resources/video-urls) documents `gameModeId=2&hydrate=analytics(video)` on game GUIDs. That is another representation of a documented route, not a new public video API. The game GUID endpoint returned 401 in the anonymous test.
- **Live subscriptions:** [Gumbo over WebSockets](https://docs.statsapi.mlb.com/game-data/gumbo-over-websockets) documents a game subscription and push IDs for associated feed/diff fetches. The [connection guide](https://docs.statsapi.mlb.com/game-data/mlb-live-data-websocket-connection-guide) documents `/ws/v1` with token, game/venue filters, data types, packet types and measurement system. Current [live tracking documentation](https://docs.statsapi.mlb.com/tracking-data/live-tracking-data) also shows three legacy Stitch feed forms. None was connected; token-bearing examples remain documentation only.
- **Administrative writes:** [v3.9.0 release notes](https://docs.statsapi.mlb.com/release-notes/2025-08-05-v3.9) mention `POST /game/{gamePk}` and `/slim` for cache updates. [v3.54.0](https://docs.statsapi.mlb.com/release-notes/v3.54.0) mentions `POST /authorization/evaluate`. The authentication topic describes token and refresh POSTs. These five POST operations were recorded but never invoked; no request-body schema was invented where the source lacked one.

## Reconciliation and stopping boundary

The inherited quick catalog contained route problems. The documented canonical forms are `/homeRunDerby/{gamePk}` (not `/homerundby`), `/streaks` (not `/stats/streaks`), `/jobs/umpires` (not `/umpires`) and `/highLow/{highLowType}` (not an unqualified `/highLow`). `/schedule/games`, `/people/currentGameStats`, `/people/gameLog` and `/game/{gamePk}/decisions` were not operations in the extracted reference; absence here is not a universal claim about every historical or internal API. The root quick catalog records its reconciliation separately.

The SDK's color-feed diff pattern and stats-streaks pattern are secondary leads, not equivalents automatically validated by nearby canonical routes. The official specification's own descriptions also contain malformed sample URLs, case discrepancies and alternate placeholder notations. These are listed as source anomalies instead of being promoted to extra operations.

The finite source frontier examined comprises the shell, both referenced application bundles, every runtime-listed chunk, all extracted document topics, the full embedded API reference and all 64 SDK route patterns. Production API calls are deliberately bounded representatives, not exhaustive parameter combinations or a crawl of all games/players. Non-production servers listed in the source were not probed. Documentation content discovered behind neither login nor credentials can still describe licensed interfaces.

Remaining limitations: 40 OpenAPI operations have their complete published interface cataloged but no direct request in this audit; WebSockets and all POST operations are source-only; privileged response schemas are documentation contracts, not validated payloads. No source establishes an exhaustive list of private MLB services. Other first-party application surfaces are covered separately by the Gameday, Savant and video audits.

### Supplementary checks

| Source-derived route | HTTP | Observation |
| --- | --- | --- |
| `/api/v1/game/{gamePk}/{GUID}/analytics/raw` | 401 | authentication_required |
| `/api/v1/game/{gamePk}/{GUID}/analytics/parsed` | 401 | authentication_required |
| `/api/v1/game/{gamePk}/feed/color/diffPatch` | 404 | json_error |
| `/api/v1/stats/streaks` | 404 | json_error |
| `/v3/api-docs` | 200 | anonymous_json; full JSON response validated |
