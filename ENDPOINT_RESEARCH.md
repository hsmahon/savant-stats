# Endpoint discovery and verification

Initial checks on **2026-11-09** used anonymous, read-only HTTPS requests. The table below preserves the initial historical-game probes. The subsequent broader audits are documented in [Stats API](STATSAPI_AUDIT.md), [Savant](SAVANT_AUDIT.md), [Film Room/media](MEDIA_AUDIT.md) and [Gameday](GAMEDAY_AUDIT.md); those inventories supersede the initial discovery-only status where further verification succeeded. Neither pass establishes every private backend route or every season's coverage. No credentials were supplied, full games downloaded, private database queries attempted, or authentication boundaries crossed. Raw responses and inspection utilities were kept outside the repository; the evidence below records identifiers, request parameters, response shapes and outcomes.

## Primary-source discovery

MLB's [2020 Hawk-Eye article](https://technology.mlblogs.com/introducing-statcast-2020-hawk-eye-and-google-cloud-a5f5c20321b8) points to Stats API distribution and video search backed by Cloud Storage. It does not publish new API paths or bucket names. The [architecture reference](MLB_DATA_ARCHITECTURE.md) follows that article and additional MLB engineering posts, plus first-party Google Cloud accounts.

Live first-party application inspection provided concrete route leads:

- The [Savant video viewer](https://baseballsavant.mlb.com/sporty-videos) returns a video source when given a supported play ID.
- [Illustrator](https://baseballsavant.mlb.com/illustrator)'s linked application bundle calls `/player/search-statcast` and `/illustrator/stats`. Its player-search UI waits for at least three input characters. The bundle also contains `POST /evp/add` for saving visualizations; that mutation was **not called** and is not presented as a data-download API.
- [Replay](https://baseballsavant.mlb.com/replay)'s HTML embeds review records and its linked application bundle fetches `/leaderboard/replay/breakdown`.
- [MLB video](https://www.mlb.com/video) HTML contains service configuration for CMS GraphQL, Data Graph, Content Service, Fastball Gateway and typeahead. These were initially **configuration-discovered leads** in [the media inventory](mlb_media_endpoints.yaml). The [expanded media audit](MEDIA_AUDIT.md) now records concrete GraphQL documents and verification of search/video operations, while distinguishing local client operations from remote queries.
- Savant navigation links to `/leaderboard/first-base-scoops-receiving` and uses `/leaderboard/batted-ball` (correcting the inherited `battled-ball` typo). The new leaderboard's CSV export and availability dates remain unverified.

These frontend bundles were inspected remotely or in temporary files; no JavaScript is retained in this repository.

## Probe results

The table's probe names match catalog `verification.probe` values. GET was used except for the separately noted MP4 HEAD check. HTTP 200 is recorded alongside the actual response type to distinguish data, applications, fallbacks and HTML shells.

| Probe | Request path and parameters | Observed result |
| --- | --- | --- |
| `content` | Stats API v1 `/game/716463/content` | 200 JSON object: `editorial`, `media`, `highlights`, `summary`, `gameNotes` |
| `feed` | Stats API v1.1 `/game/716463/feed/live` | 200 JSON object: `gamePk`, `metaData`, `gameData`, `liveData` |
| `timestamps` | Stats API v1.1 `/game/716463/feed/live/timestamps` | 200 JSON array of `YYYYMMDD_HHMMSS` strings |
| `diff` | Stats API v1.1 `/game/716463/feed/live/diffPatch`, `startTimecode=20230923_201000`, `endTimecode=20230923_201100` | 200 JSON full-feed object; `metaData.logicalEvents` contains `fullUpdate` |
| `diff_valid` | Same diffPatch route, `startTimecode=20230923_231745`, `endTimecode=20230923_231815` (both returned by timestamps) | 200 JSON full-feed fallback again; patch-array behavior not demonstrated |
| `context` | Stats API v1 `/game/716463/contextMetrics` | 200 JSON: `game`, three field-direction sac-fly probabilities and home/away win probabilities |
| `lookup` | Stats API v1 `/pitchTypes` | 200 JSON array of pitch codes/descriptions |
| `metrics` | Stats API v1 `/metrics` | 200 JSON array with `metricId`, `name`, and where applicable `group`, `unit` |
| `stats_types` | Stats API v1 `/statTypes` | 200 JSON array of stat types |
| `groups` | Stats API v1 `/statGroups` | 200 JSON array of stat groups |
| `teams` | Stats API v1 `/teams`, `sportId=1` | 200 JSON `teams` array; used to replace inherited incorrect team IDs |
| `people_stats` | Stats API v1 `/people/592450/stats`, `stats=gameLog`, `group=hitting`, `season=2023` | 200 JSON `stats[].splits` containing game logs |
| `people_gamelog` | Stats API v1 `/people/592450/gameLog`, `season=2023` | 404 JSON error served as `text/plain`; retain as a failed inherited route with working alternative |
| `decisions` | Stats API v1 `/game/716463/decisions` | 404 JSON error served as `text/plain`; decisions exist in feed `liveData.decisions` |
| `analytics` | Stats API v1 `/game/716463/fced30ac-7099-4337-a52f-9bc983932b31/analytics`, `hydrate=analytics(hit,pitch,ball,metrics,video,positions,diagram)` | 401 HTML “Please Login” with Okta widget; data schema and hydrate semantics unverified |
| `swagger` | `https://statsapi.mlb.com/docs/game/swagger.json` | 301 to `https://docs.statsapi.mlb.com`, then 200 HTML Docusaurus documentation shell, **not** Swagger JSON |
| `gf` | Savant `/gf`, `game_pk=716463` | 200 JSON game object with `home_pitchers`, `away_pitchers`, `players`, `scoreboard`, `boxscore`; not a top-level pitch array |
| `sporty_play` | Savant `/sporty-videos`, `playId=fced30ac-7099-4337-a52f-9bc983932b31` | 200 HTML containing a `video/mp4` source on `sporty-clips.mlb.com` |
| `video` | Savant `/sporty-videos` without playId | 200 HTML “No Video Found”; not proof of video availability |
| `player_search` | Savant `/player/search-statcast`, `search=Judge`, empty `minors` | 200 JSON array; Aaron Judge `player_id=592450`, `org_id=147` |
| `illustrator_stats` | Savant `/illustrator/stats`, `playerId=592450` | 200 JSON `stats` array, career hitting stat group |
| `replay` | Savant `/replay` | 200 HTML application with embedded review records |
| `replay_breakdown` | Savant `/leaderboard/replay/breakdown`, `year=2025`, `type=challenging` | 200 JSON: `overturned`, `challengingGroup`, `challengedGroup`, `challengeType`, `teams`, flags |
| `illustrator` | Savant `/illustrator` | 200 HTML application; linked bundle supplied lookup-route leads |
| `sav_statcast` | Savant `/statcast_search` | 200 HTML application; search interface, not CSV data |
| `film`, `filmroom` | MLB `/video` and `/video/search` | 200 HTML; no backing video-search operation verified |

Stats API requests above use `https://statsapi.mlb.com/api/{version}`; Savant requests use `https://baseballsavant.mlb.com`. Tested parameter values document observations, not required/default values for every endpoint.

## Proven video and data relationships

Game `716463` is the September 23, 2023 Toronto–Tampa Bay game. Its feed identifies the teams as Toronto `141` and Tampa Bay `139`. In the fetched responses:

1. `liveData.plays.allPlays[].playEvents[].playId` contains 342 distinct IDs. Savant's `home_pitchers` and `away_pitchers` maps contain pitch lists with 340 distinct `play_id` values. All 340 match feed event IDs. This establishes a join for this game, not guaranteed coverage for all feeds or all event types.
2. Feed play ID `fced30ac-7099-4337-a52f-9bc983932b31` resolves in the Savant viewer to the returned MP4 source below. HEAD returned **200**, `Content-Type: video/mp4`, `Content-Length: 3957776`. Media bytes were not downloaded for this check.
3. Game content's `highlights.highlights.items[]` includes clip ID `blue-jays-vs-rays-highlights-x6166` and `playbacks[]` entries named `mp4Avc` and `hlsCloud`. The returned HLS master playlist below was fetched: **200**, `application/vnd.apple.mpegurl`, beginning `#EXTM3U`. It references stream variants; segments and full playback were not fetched.

| Returned media resource | Source |
| --- | --- |
| [Tested play MP4](https://sporty-clips.mlb.com/MkI3am9fWGw0TUFRPT1fVUZJRUFGY0VCMUVBRDFJTFh3QUFBZ1ZSQUFBQVVRY0FDd2RXQ1FzR0NBQlhVZ0ZY.mp4) | [Play viewer](https://baseballsavant.mlb.com/sporty-videos?playId=fced30ac-7099-4337-a52f-9bc983932b31) |
| [Tested highlight HLS master](https://mlb-cuts-diamond.mlb.com/FORGE/2023/2023-09/23/5d472b2f-b2617938-3c241167-csvm-diamondx64-asset.m3u8) | [Game content JSON](https://statsapi.mlb.com/api/v1/game/716463/content) |

A `gamePk` groups game records. MLBAM person/team IDs identify participants. A feed event's `playId`, Savant's pitch `play_id`, an editorial clip slug and an opaque media asset path are distinct identifiers. The successful feed-to-Savant match does not establish a universal editorial clip-to-play mapping. Read returned identifiers and playback URLs; do not synthesize CDN filenames or assume each pitch has a highlight.

## Access boundaries and remaining leads

The tested analytics route requires authentication. Its hydration names are a probe input, not verified evidence that those datasets are accessible or that all requested fields exist. The historical Swagger URL redirected to `https://docs.statsapi.mlb.com` and did not return a machine-readable specification. The expanded audit subsequently found a working public specification at [`/v3/api-docs`](https://statsapi.mlb.com/v3/api-docs); see [its recorded comparison](STATSAPI_AUDIT.md). The destination shell links a main bundle whose authentication wrapper allows `/login` and `/okta/callback` as public paths and gates other pages; an authenticated documentation session was not attempted. Club/partner distribution and private storage are described only from the primary sources in the architecture document.

The expanded audits now cover the Film Room query documents, the complete captured Stats API specification and the reachable Savant application-module frontier. Remaining limits include authenticated data retrieval, coverage across every season/game, dynamically constructed or unpublished routes and private correction/retention behavior. No current public bucket or complete public pose-data export was established. These probes did not measure a request-rate or truncation threshold; current published policy is separated by service in [savant_rate_limits.yaml](savant_rate_limits.yaml), including the Stats API's documented rate rule.
