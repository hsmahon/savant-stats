# Baseball Savant endpoint audit

Reviewed **2026-11-09**. This audit expands the initial hand-picked checks into a source and response inventory. The machine-readable results, source URLs, source hashes, route expressions, query names and individual probe outcomes are in [savant_route_inventory.yaml](savant_route_inventory.yaml).

## How discovery was bounded

The crawl began with every distinct route in first-party Savant navigation, plus concrete Aaron Judge and Gerrit Cole player pages and game `716463`. It followed additional application links and relevant routes referenced in the functional sources. The covered families include MLB/minors/WBC search, every linked leaderboard, player pages, gamefeed, scoreboard, positioning and other visuals, Illustrator, replay, ABS, comparison tools, pitch mix, HR Derby and Savant games.

For each HTML page, the audit fetched linked Savant application bundles from MLB's own build host, then recursively followed literal local module imports. Sources and cached responses remained outside the repository. The YAML source manifest records URLs, HTTP status, media type and SHA-256 hashes; character offsets identify request expressions in minified bundles. Monitoring/advertising services, generic framework URLs, images, fonts and 3D model assets are excluded from the endpoint inventory. Two dynamically imported telemetry recorder/profiler modules were deliberately excluded from further traversal.

Navigation, source references and response verification are distinct evidence levels. A source-discovered route is not automatically public, documented, current or data-bearing. The inventory includes application pages and aliases alongside actual data operations, but marks their kinds separately. `coverage` reports the retrieval and probe totals. The final source-only records identify operations not exercised.

Final coverage: **129 page requests, 219 linked/imported modules, 108 read probes and 214 distinct route templates**. The inventory distinguishes 120 application-page records, 87 data/fragment records, one legacy CSV record, one dynamic image renderer, one navigation redirect and four state-changing operations. All fixed navigation links and relevant literal functional imports in this recorded frontier were processed; two telemetry modules were excluded.

## Newly documented route families

| Family | Source-discovered interfaces | What responses establish |
| --- | --- | --- |
| Player lookup and header services | `/player/search`, `/player/search-all`, `/player/search-statcast`, `/savant/api/v1/trending-players`, random-player redirect | Public lookup/trending JSON observed; random-player route is a navigation reference, not verified JSON. |
| Player detail and charts | `/player-services/{service}`, `/player/full_spray`, `/player/slices`, `/player/random-video`, chart routes under `/player/`, `/player-viz/lookup` | Mix of JSON, HTML fragments and JSON wrapping HTML tables. Formats are not interchangeable. |
| Leaderboard drawers | `/leaderboard/services/abs/{entity}`, catcher framing/blocking/throwing, baserunning, basestealing/running-game, pitcher-running-game, first-base receiving, swing timing | Several drawers return play-level JSON well beyond top-level aggregates. Source retains serialized page query/filter construction. |
| Pose-related viewer services | `/leaderboard/services/scoops-skeletal/{play_key}`, `/leaderboard/services/scoops-skeletal/player` | Source references established. A per-play probe returned `{"data":null}`; public pose availability was **not** demonstrated. |
| Gamefeed auxiliaries | `/gamefeed/pitcher-yearly-averages`, `/gamefeed/x-parks/{gamePk}/{playId}`, `/gamefeed/services/abs/{...}`, `/gf`, `/schedule`, `/scoreboard-data` | Public pitcher averages, schedule and scoreboard JSON observed; tested x-parks response had empty `hr`/`not` arrays. |
| Pitch and bat visualization data | `/app/pitch-data/{pitcherId}`, `/app/archetype/{pitcherId}`, `/pitchmix/pitchers`, `/pitchmix/league`, `/visuals/bat-speed-distribution/{playerId}`, league distribution | Archetype is CSV; pitch-mix and bat-speed distributions returned JSON. The full pitcher pitch-data route was source-discovered without downloading its complete history. |
| Positioning and zones | `/visuals/position_data`, `/player-services/zones`, `/team-services/zones` | Public positioning/zone JSON observed; request type selects different entities and shapes. |
| Search and chart feeds | `/feed`, legacy `/statcast_search/csv`, `/statcast_field`, `/statcast_catch_probability`, `/statcast_hit_probability` | Some responses depend on flags/query mode. The initial narrow `/feed` probe was empty. A second probe using the current frontend's `warehouse=true` mode returned 20 pitch records for Aaron Judge on June 1, 2025. The legacy CSV probe returned a download with a header only. |
| Other applications | `/derby-data`, `/game-strategy-explorer` JSON mode, weighted-score and spin-direction API routes, historical visual data | Formats, filters and coverage vary. Correctly shaped Strategy Explorer input returned JSON `[]`; it does not establish a populated result for that game situation. |
| State-changing operations | `POST /evp/add`, `POST /utils/url-shortener`, GET score-save routes for prospect/pitch games | Documented from explicit source methods, without execution. A GET method does not imply the operation is read-only. |

All interfaces in this table have corresponding source references in the inventory; each verification statement is limited to the recorded request. The `/statcast_search/csv` legacy URL is inherited from the existing catalog and independently probed, rather than falsely attributed to a literal in the current frontend bundle.

## Useful data relationships and limits

The first-base-receiving drawer returns event metadata and derived trajectory values. For the tested player, fields included `game_pk`, `play_id`, `play_key`, `has_skeletal`, `catch_x/y/z`, trajectory coefficients and a `play_url` pointing to `research.mlb.com`. Those links identify a separate research surface; their presence does not establish access to it. Some records set `has_skeletal: true`, but a matching per-play skeletal request returned `data: null`. Do not treat that flag or the route name as evidence of a publicly downloadable full skeletal dataset.

The player random-video endpoint returned JSON with `link`, `title` and `gameDate`; `link` points to an MP4 on `sporty-clips.mlb.com`. The probe supplied `videoType=home_run` but the returned title described a foul ball. That input's accepted enum/filter behavior therefore remains unverified. Likewise, list endpoints may return league-wide or current-default results despite player/season parameters being present. A successful HTTP status does not prove every supplied filter was honored.

Player-service HTML fragments are data-bearing presentation responses. For example, arm strength, bat tracking, catching and ABS services can return HTML intended to be inserted into a player tab, while `/player-services/stats` returns JSON whose properties contain table HTML. These should not be described as uniform raw Statcast JSON APIs.

The pitch-mix pitcher response contains many rows and is substantially larger than league or single-player responses. Even a single-player response can be large: `/player-viz/lookup` returned about 17.8 MB for the tested pitcher/season, while `/visuals/pitch-plinko-data` returned 3,281 rows. The preview also references `/player/visuals/pitches` as a dynamically rendered image URL; it is cataloged separately from static assets and JSON data. Most probes used one historical game, one player, one play or one season. No full games/media streams, private databases or authentication-gated datasets were downloaded.

## Leaderboard CSV checks

The audit sent one `csv=true` format probe to each discovered leaderboard page in the tested frontier, with `year=2025`. These are format checks, not proof that every board accepts `year` or that it filtered to that season. The actual request and returned format are attached to each page record.

Across 50 leaderboard format checks, **39 returned `text/csv`**, **10 returned HTML**, and **1 returned 404**. The HTML results were fielding-run-value, hot-stove, rolling, statcast-park-factors, weighted-score, swing-timing-top-performers, top-performers, the `catcher_framing` alias, statcast-venue and the `top` alias. The source-referenced bat-tracking acceleration route returned 404. Alternate download mechanisms may exist; this test does not show that a board has no export capability.

Thus neither “all leaderboards support `csv=true`” nor “HTTP 200 means CSV” is supported. CSV observations record parsed headers and row counts in addition to Content-Type. Page and format records retain aliases and failures so they do not silently disappear from the inventory.

## Interpretation and coverage boundary

- Route counts include HTML applications and aliases, data/fragment operations, one legacy CSV route, a navigation redirect and source-only state-changing operations. Use the `kind` field before interpreting a total as a count of data APIs.
- Source query names come from literal request strings or nearby `URLSearchParams`/AJAX `data` construction. They are observed names, not a verified schema of accepted, required or default parameters. Probe-only inputs remain under each observation.
- Dynamic query forwarding is recorded explicitly where a drawer passes the page's query string. The source expressions preserve path variables whose semantics are not confidently named.
- An empty array, `data: null`, HTTP error, CSV header or HTML shell is retained as that result. None establishes full data coverage, permissions or a stable service guarantee.
- This inventory exhausts the explicitly recorded public source frontier, not MLB's server routes, authenticated research tools, feature flags never exposed to this session, dynamically computed nonliteral module imports, or every possible query/season/league combination. The two player pages are representatives; infinitely many player URLs are not counted as distinct endpoints.

First-party entry points: [Savant home](https://baseballsavant.mlb.com/), [Statcast search](https://baseballsavant.mlb.com/statcast_search), [Aaron Judge player page](https://baseballsavant.mlb.com/savant-player/aaron-judge-592450), [gamefeed](https://baseballsavant.mlb.com/gamefeed?gamePk=716463), [Illustrator](https://baseballsavant.mlb.com/illustrator), [First Base Receiving](https://baseballsavant.mlb.com/leaderboard/first-base-scoops-receiving), [visuals](https://baseballsavant.mlb.com/visuals), [replay](https://baseballsavant.mlb.com/replay). Specific versioned bundle URLs are retained in the YAML rather than treated as stable API paths.
