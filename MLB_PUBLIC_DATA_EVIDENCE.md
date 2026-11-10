# MLB public data and media evidence

Captured November 10, 2026. This note joins dated first-party engineering descriptions to bounded, unauthenticated observations of one historical game (`gamePk=716463`, September 23, 2023). The game payload is historical; the HTTP requests were made on the capture date. It is not a map of MLB's private production systems.

## Evidence by data class

| Data class | MLB says | Publicly observed | What remains unknown |
| --- | --- | --- | --- |
| Live game and derived stats | The 2022 account places game data in Cloud SQL for PostgreSQL and says nightly Dataflow jobs copy from Cloud SQL and Bigtable to Cloud Storage and BigQuery. MLB's 2025 account describes three Valkey uses: ballpark message buffering; multi-region GUMBO reads; and read-through caches for stats, rosters, standings, and schedules. The article figures show AMQ and injection services at the ballpark, GUMBO caches in East and Central regions, and a CDN in the Stats API path. | The public Stats API live feed returned 200 JSON (913,821 bytes) with play-by-play and pitch details, including pitch speed, location, movement, spin, and release values. Its response advertised a 10-second public cache lifetime and Google/Varnish proxy hops. | The public response does not expose database queries, table schemas, cache keys, replication internals, or which store supplied this particular historical response. HTTP proxy headers do not identify the database. The article's CDN figure does not name its product. |
| Statcast and player tracking | MLB's 2022 account identifies Bigtable for player positional pose data. Its 2025 account describes telemetry with 29 body points at 30 fps. Earlier articles describe other point counts and tracking configurations, so treat those figures as dated descriptions. | A bounded Savant CSV request returned a small pitch-level sample. The Stats API feed also exposes selected pitch-level motion and location values. | These exports do not establish access to the full Hawk-Eye camera stream, complete pose time series, raw tracking storage, or a complete camera-to-pitch join. |
| Video | MLB's 2020 account says clipped camera feeds were saved in Cloud Storage for video search. A 2022 Google Cloud article reports that Film Room was hosted on Google Cloud, with over three million assets and automatically generated, searchable edits. A Google Cloud case study lists Media CDN in MLB's cloud portfolio and describes live streaming as a planned use. | Game content returned labeled clips and MP4/HLS playback URLs. The HLS master playlist was public and listed variant playlists. A public 29.5-second MP4 probe identified H.264 video and AAC audio at 1280×720. Film Room's public page sent GET requests to `https://fastball-gateway.mlb.com/graphql`; the capture recorded 200 responses for `Search`, `getTopic`, `getContainers`, `getSuggestedSearches`, and `getUserInfo`. GraphQL response bodies were disabled in the browser capture. | The CDN hostname and `UploadServer` response header do not establish the serving bucket or origin. The observations do not expose full-game archive objects, raw camera feeds, retention rules, or the current Film Room object store. Media CDN is not proven to serve the sampled clip. |
| Audio and broadcast metadata | MLB's help pages describe MLB Audio as a subscriber feature. The published architecture articles reviewed here do not identify an audio storage or streaming origin. | Schedule hydration returned TV/radio station entries and MLBTV-Audio EPG records with `mediaId`, `contentId`, feed type, language, and media state. The response did not include an audio playback URL. | The audio resolver, stream host, codecs, object store, retention, and the meaning of per-feed availability flags remain unknown. An EPG ID or `availableForStreaming` value is not a public stream URL or proof that playback is anonymous. |
| Historical and editorial media | Google Cloud's 2022 article reports more than 60 PB of historical MLB archives and says Film Room's searchable video library spans decades. The separate 2020 Statcast article documents a specific clipped-video Cloud Storage role. | The public game-content response separates editorial metadata from playable clip resources. | The archive-wide 60 PB figure does not specify that all data is video or that it resides in one store. No public bucket names, object layout, or retention policy were found. |

## Public and reported data paths

```mermaid
flowchart LR
  subgraph reported[Reported paths: dated sources]
    sensors[Tracking systems<br/>player movement data]
    tags[Operator apps<br/>event metadata]
    amq[AMQ queue<br/>2025 report figure]
    injection[Injection services]
    edgecache[Valkey edge cache<br/>Google Distributed Cloud]
    cloud[Google Cloud services]
    sensors -->|send tracking data| amq
    tags -->|send metadata| amq
    amq -->|send messages| injection
    edgecache -->|read| injection
    injection -->|write| edgecache
    injection -->|write data| cloud

    game[Game records] -->|store| sql[Cloud SQL<br/>PostgreSQL, 2022]
    pose[Player pose records] -->|store| bt[Bigtable, 2022]
    sql -->|nightly read| df[Dataflow, 2022]
    bt -->|nightly read| df
    df -->|write analytics copies| lake[Cloud Storage and BigQuery]
    clips[Clipped camera feeds] -->|save for search, 2020| clipstore[Cloud Storage]
    history[Film Room archive] -->|hosted on Google Cloud, 2022| google[Google Cloud<br/>exact store unknown]
    ingest[Ballpark ingestion apps]
    eastapps[GCP apps<br/>East region]
    east[GUMBO cache<br/>East region]
    eventjob[opjob-event]
    central[GUMBO cache<br/>Central region]
    centralapps[GCP apps<br/>Central region]
    eastapi[Stats API<br/>East region]
    centralapi[Stats API<br/>Central region]
    cdn[CDN<br/>product not named]
    users[Fans and internal users]
    ingest -->|POST data| eastapps
    eastapps -->|write| east
    east -->|read| eastapps
    east -->|send updates| eventjob
    eventjob -->|replicate data| central
    central -->|read| centralapps
    centralapps -->|write| central
    eastapps -->|provide game data| eastapi
    centralapps -->|provide game data| centralapi
    eastapi --> cdn
    centralapi --> cdn
    cdn -->|serve getLiveGumbo| users
    db[Database<br/>specific store not named]
    regular[Regular Valkey cluster]
    heavy[Heavy Valkey cluster]
    apps[Apps and clients]
    apps -->|request stats and roster data| regular
    apps -->|request data| heavy
    regular -->|read on cache miss| db
    heavy -->|read on cache miss| db
    db -->|populate cache| regular
    db -->|populate cache| heavy
    regular -->|return data| apps
    heavy -->|return data| apps
  end

  subgraph observed[Observed public paths: Nov 10, 2026]
    stats[Stats API]
    live[GET /api/v1.1/game/{gamePk}/feed/live<br/>play and pitch JSON]
    schedule[GET /api/v1/schedule<br/>date and hydrate parameters]
    gamecontent[GET /api/v1/game/{gamePk}/content<br/>editorial and clip JSON]
    csvsite[Baseball Savant]
    csv[GET /statcast_search/csv<br/>pitch-level rows]
    film[Film Room page]
    fastball[Fastball GraphQL<br/>Search and topic operations]
    clipurls[MP4 and HLS clip URLs]
    delivery[mlb-cuts-diamond.mlb.com<br/>sampled public clip host]
    hls[HLS master playlist<br/>variant list and codecs]
    mp4[MP4 clip<br/>H.264 video and AAC audio]
    audio[MLB Audio<br/>stream origin unknown]
    epg[EPG feed IDs<br/>no playback URL observed]
    stats -->|GET live feed| live
    stats -->|GET schedule| schedule
    stats -->|GET game content| gamecontent
    csvsite -->|GET CSV query| csv
    film -->|GET GraphQL requests, 200| fastball
    gamecontent -->|return links| clipurls
    clipurls -->|point to| delivery
    delivery -->|return playlist| hls
    delivery -->|return clip| mp4
    schedule -->|return IDs| epg
    epg -.->|audio resolver not observed| audio
  end
  eastapi -.->|same region as public request unknown| stats
  centralapi -.->|same region as public request unknown| stats
```

The solid arrows show flows that a source describes. The dotted arrows show links that remain unknown. The reports describe services from different years. The browser and HTTP records show requests from November 10, 2026. They do not prove that the sampled Stats API request used a specific GUMBO region, Cloud SQL, or Bigtable. They do not prove that the clip host reads from the reported Cloud Storage bucket. The EPG response did not show an audio resolver. The 2025 article calls the cache service Memorystore for Valkey. Its GUMBO figure labels the cache "redis gumbo." Its CDN figure does not name a product. Do not assume that this is Google Media CDN.

The article reports ballpark cache latency of 1–2 milliseconds and peak command volume near 11,000 per second. It reports a cross-region GUMBO availability target under two seconds. It also reports regular and heavy Valkey clusters in each region for read-through data. These are MLB's published figures. This investigation did not measure them.

## Methods

- REA CLI `rea-agents@6.5.0` ran bounded Playwright scenarios in Chromium 155.0.8059.84 for the Film Room page, one Stats API feed, and the MLB.TV Media Center. Network bodies were disabled in those captures; the returned capture files retain URLs, response statuses, resource types, and event completeness.
- The current Film Room bundle and its referenced source map were fetched as raw artifacts. REA's JavaScript application analyzer parsed the 3.67 MB bundle. It recovered static route/query candidates but exhausted the worker heap during semantic analysis, so the graph is partial and was not treated as a complete client inventory. The source map was inspected for its embedded source list and content. The Media Center main bundle was also parsed as inert JavaScript; its graph was partial and exposed no verified endpoint inventory. Static Okta and GraphQL strings were treated as leads only; no live audio request was established.
- `curl` saved bounded Stats API JSON, one Savant CSV query, and a single HLS master playlist with response headers. `jq` checked response shape and selected fields. FFprobe inspected one remote short clip; no full broadcast or bulk media was downloaded.

## Reproducible observations

Representative requests and retained response details:

| Request | Result on capture date | Evidence |
| --- | --- | --- |
| `GET https://statsapi.mlb.com/api/v1.1/game/716463/feed/live` | 200 `application/json;charset=UTF-8`, 913,821 bytes; `Cache-Control: max-age=10, public, stale-while-revalidate=30, stale-if-error=86400`; `Via: 1.1 google, 1.1 varnish, 1.1 varnish`; `X-Cache: MISS, MISS` | Body SHA-256 `ad8487b9d283c24eae8efe8fe5a11c6c2ad3062c82c196596539564eac436c9f` |
| `GET https://statsapi.mlb.com/api/v1/game/716463/content` | 200 JSON, 462,526 bytes; response includes editorial and highlight content with MP4/HLS playback URLs | Body SHA-256 `f8aa51e6d68555fefee3fc5596d0eddc4559d1d3a73a456e85cd39cdb7997270` |
| `GET https://statsapi.mlb.com/api/v1/schedule?sportId=1&date=2023-09-23&gamePk=716463&hydrate=broadcasts(all),game(content(media(epg)))` | 200 JSON, 36,942 bytes; response includes radio and television broadcast metadata plus MLBTV-Audio EPG entries | Body SHA-256 `85740589dd2c3d8dbc1975fd0db8047e3ac9fd88b46eba52eee0713d41cd9d91` |
| Savant CSV: `https://baseballsavant.mlb.com/statcast_search/csv?batters_lookup%5B%5D=592450&hfSea=2023%7C&game_date_gt=2023-09-23&game_date_lt=2023-09-23&player_type=batter&type=details&min_pitches=0` | 200 `application/download; charset=utf-8`, 1,825 bytes; one bounded batter/date query | Body SHA-256 `c1703bd38919ac5304ed7f1e0f3eb225f385b969203db316cde88adc409462e3` |
| `GET https://mlb-cuts-diamond.mlb.com/FORGE/2023/2023-09/23/5d472b2f-b2617938-3c241167-csvm-diamondx64-asset.m3u8` | 200 `application/vnd.apple.mpegurl`, 2,948 bytes; lists 384×216 through 1280×720 variants and H.264/AAC codec declarations | Playlist SHA-256 `f0708cc62f7622f529e9e5bf587c286e239a18cac9561cd8ea405b4ec5a6f8d8` |
| `HEAD https://mlb-cuts-diamond.mlb.com/FORGE/2023/2023-09/23/a4e817b9-c16b80cc-cbdaea07-csvm-diamondx64-asset_1280x720_59_4000K.mp4` | 200 `video/mp4`; FFprobe identified a 29.546-second, 1280×720 H.264 video stream and AAC audio. The video file was not saved. | FFprobe result JSON SHA-256 `602a78f697aefa363b9c1663aa18d1012246104503c54607d49205779c631ef1` |

The anonymous MP4 `HEAD` request returned 200 `video/mp4`. FFprobe inspected the remote 29.5-second clip and reported H.264 1280×720 video plus AAC audio. The clip itself was not saved. The server ignored the attempted byte-range request with HTTP 200, and curl's 64 KiB maximum stopped before writing a partial file.

### Capture and source artifact hashes

Full raw captures and downloaded first-party artifacts are kept outside the repository at `/tmp/mlb-rea-evidence/`; only the compact evidence and hashes are committed here. Capture times below are UTC file times; response `Date` headers are preserved in the raw header files.

| Artifact | Capture time (UTC) | SHA-256 |
| --- | --- | --- |
| REA Playwright capture of `https://www.mlb.com/video` (complete, 672 events) | 2026-11-10 22:18:25 | `39d2f8e3063fd9e344997b8f1bec8a14c8b3fde469f67c37a0b18ad4d61d89e4` |
| REA Playwright capture of Stats API live feed (complete, 3 events) | 2026-11-10 22:21:30 | `9dff53f026709c41ccfa13fb3f54e61377e03de5ded877a2c5f0622f98ae8ea9` |
| REA Playwright capture of MLB.TV Media Center (incomplete: in-flight events at end) | 2026-11-10 22:25:47 | `553b9b703322815eee0df072ddb2872f6352b7facab0085d982924db50521f5e` |
| Current Film Room client bundle (3,669,812 bytes) | 2026-11-10 22:26:49 | `7975624ee5d7a2e99e3d88fad3149debd346b1ae7b7d4df66b2f79568c3a1f4c` |
| Film Room source map (12,346,795 bytes; 2,188 sources with embedded content) | 2026-11-10 22:26:50 | `6e3e3c9dd13130243c7d2757491c81580bc074967c2a2bd2ec2ab3f7abe46185` |
| REA static analysis of the Film Room bundle (partial semantic graph; worker heap limit reached) | 2026-11-10 22:31:26 | `2d51bc5c9a17fb7164efcfa8c25c384b4ab081d43b385f7998c228839cc9c252` |
| Google Cloud 2025 ballpark caching figure (682 × 800 PNG) | 2026-11-10 23:15:33 | `3ff70fcf5763a6a1aa8ed2a0d5a5d579b259319d7905e29b7685404190697269` |
| Google Cloud 2025 GUMBO cache figure (1563 × 1014 PNG) | 2026-11-10 23:15:33 | `0073464969ebadf67c323b5973c703d305e6ab261410647a31edc282ddad7c6e` |
| Media Center main client bundle (1,715,658 bytes; inert static-analysis input) | 2026-11-10 22:45:06 | `1e3e7871181e736b6d6817072fad6c0bf2dc95435fbb5d14dd0c34ea01bee661` |
| Stats API live-feed JSON | 2026-11-10 22:19:57 | `ad8487b9d283c24eae8efe8fe5a11c6c2ad3062c82c196596539564eac436c9f` |
| Stats API game-content JSON | 2026-11-10 22:19:58 | `f8aa51e6d68555fefee3fc5596d0eddc4559d1d3a73a456e85cd39cdb7997270` |
| Stats API schedule / EPG JSON | 2026-11-10 22:19:58 | `85740589dd2c3d8dbc1975fd0db8047e3ac9fd88b46eba52eee0713d41cd9d91` |
| Savant CSV sample | 2026-11-10 22:30:01 | `c1703bd38919ac5304ed7f1e0f3eb225f385b969203db316cde88adc409462e3` |

REA evidence IDs: Film Room page `ev_d02acff552ad482351c0ac245770889a3443bebf6f12cee627b8288114475c03`; Stats API page `ev_c2bf45c97fac5a46c4e9918436fa4afb35f6e40d56709416e561918e5043b399`; Media Center page `ev_106099620aefb74b1f413b1505cb1eccc643a919314622504cd4d9133ccb87fc`; Film Room static bundle analysis `ev_fbc88c104cfb21e1a824a9bb069259630696b6333fe2382ffeb33b90f1adc6e0`; Media Center static bundle analysis `ev_0a0d6277930f45101481ccad2b09ccc2b59ec4319b48c347f1ab5531d25cb1fc`. The Film Room static pass recovered request/route candidates, but its semantic pass exhausted the worker heap. Media Center static analysis completed with a partial graph and no endpoint inventory. Neither static result establishes runtime execution or server storage.

## Remaining public evidence

The investigation did not exhaust the public source set. These records identify concrete follow-up work:

| Source set | Remaining work |
| --- | --- |
| [Gameday inventory](gameday_route_inventory.yaml) | 24 client routes remain marked `source_discovered_not_probed`. Probe them one at a time and record each response. |
| [Savant inventory](savant_route_inventory.yaml) | 108 source- or page-discovered rows mark a data operation as not probed. Check the route templates and parameters before probing. |
| [Media operation inventory](mlb_media_operations.yaml) | 35 client-only GraphQL operations remain source-only. Four other operations have source documents or schema evidence, but no resolver call. |
| Film Room client | The 3.67 MB bundle's REA semantic graph is partial because the worker reached its memory limit. A smaller, per-module or per-source-map analysis could recover more static links. |
| MLB.TV Media Center | The anonymous browser capture ended with requests in flight. It did not exercise an authenticated playback session. Two downloaded client chunks (`media-player.js` and `media-routes.js`) have not had separate REA analysis. |
| Stats API | The catalog includes published operations and secondary SDK leads. This evidence run directly checked only the live feed, game content, and one schedule hydration. It did not test every documented operation or parameter set. |

These are leads and gaps, not proof that each route is current or accessible. A bounded next pass can complete the 24 Gameday probes, prioritize the 108 Savant entries by unique path and data class, and split the remaining JavaScript artifacts into smaller REA analyses. An audio playback route requires a valid public playback state; a subscriber-only route is outside anonymous evidence. Private services and authenticated data remain outside this investigation.

## First-party references and limits

- Ben Jedlovec, MLB Technology Blog (2020), [Introducing Statcast 2020: Hawk-Eye and Google Cloud](https://technology.mlblogs.com/introducing-statcast-2020-hawk-eye-and-google-cloud-a5f5c20321b8).
- Priyanka Vergadia with MLB's Rob Engel, Google Cloud (2022), [Teaming up with MLB for game-changing sports analytics](https://cloud.google.com/blog/products/data-analytics/mlb-pitches-new-data-uses-with-google-cloud-services).
- Google Cloud (2022), [Crackerjack data: How MLB is reinventing the fan experience](https://cloud.google.com/blog/transform/mlb-reinventing-fan-experience-with-cloud-today-tomorrow-even-yesterday).
- Rob Engel, MLB VP of Software Engineering, Google Cloud (2024), [AI at Bat](https://cloud.google.com/transform/mlb-statcast-ai-fan-experience-team-analytics).
- Rahul Joshi, Principal Software Engineer, MLB, Google Cloud (2025), [How MLB keeps fans connected to the game](https://cloud.google.com/blog/products/databases/mlb-delivers-real-time-data-to-fans), with its [ballpark caching figure](https://storage.googleapis.com/gweb-cloudblog-publish/images/1_3QpqGX2.max-800x800.png) and [GUMBO figure](https://storage.googleapis.com/gweb-cloudblog-publish/images/2_QNOyrE2.max-1600x1600.png).
- Google Cloud, [MLB case study](https://cloud.google.com/customers/major-league-baseball), which lists Media CDN and describes its planned role in live streaming.
- MLB Help Center, [MLB.TV and MLB Audio access](https://www.mlb.com/live-stream-games/help-center/accessibility-what-is-the-best-way-for-a-screen-reader-user-to-access-mlbtv).

The public evidence identifies useful representations and some dated service roles, but not MLB's private schemas, buckets, full media retention model, or audio origin. Client bundle strings and delivery hostnames are leads; they are not proof of physical storage. Public sample access is a point-in-time observation and does not establish rights to reuse or bulk-download content.
