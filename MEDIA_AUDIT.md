# MLB media operation audit

Checked **2026-11-09**. [The operation inventory](mlb_media_operations.yaml) records discovered interfaces, variable declarations, source locations and probe outcomes. This audit replaces the earlier hostname-only Film Room leads with concrete operations and captures the publicly exposed Fastball GraphQL schema. It does not establish every private MLB API or a complete schema for Data Graph or CMS.

## Coverage and method

The first-party [Film Room page](https://www.mlb.com/video) dynamically loads a [3.67 MB application bundle](https://builds.mlbstatic.com/mlb.com/sections/video/builds/3f19968e7502c244bf346238f2fbab4907a655a0_1788358291/scripts/build/index.bundle.js). Every query/mutation document in its application operation block was classified: **19 server document variants representing 17 operation names**, plus **35 client-only Apollo cache operations**. Three server variants share the name `Search` with different selections. The bundle’s publicly referenced [source map](https://builds.mlbstatic.com/mlb.com/sections/video/builds/3f19968e7502c244bf346238f2fbab4907a655a0_1788358291/scripts/build/index.bundle.js.map) returned 200 and contains 2,188 original source entries. Its query/mutation modules and local network resolvers confirm this classification; original source names and line numbers are recorded in the inventory. A source scan found no numeric asynchronous chunk imports in this bundle; this is a bounded statement about this deployment, not all MLB clients.

The first-party [Gameday bundle](https://prod-gameday.mlbstatic.com/app-mlb/5.61.0-mlb.1/gd.min.js) adds `getGamesByGamePks` and `getPlayerStories`. Film Room's local location resolver adds the remote `UserQuery`. Together, the inventory contains **20 distinct server operation names / 22 document variants**, plus **five REST/configuration operations**. Bundle SHA-256 digests and character offsets identify the exact inspected deployment.

Browser network observation covered Film Room home and an ordinary search for `Aaron Judge home runs`. The search interface navigated to `/video/?q=Aaron+Judge+home+runs&qt=FREETEXT`, returned 782 results at the check time, and issued `getQueryInfo` plus `Search` requests. These counts are observations, not stable dataset totals. Follow-up requests used query documents and parameter values from first-party client code. Video-byte, playlist and segment requests were blocked in this browser audit; previous media-resource checks remain in [the original endpoint evidence](ENDPOINT_RESEARCH.md).

## Standard schema discovery

A normal anonymous read-only `IntrospectionQuery` against **POST `https://fastball-gateway.mlb.com/graphql`** returned **200 JSON**. The [schema snapshot](mlb_fastball_schema.json) retains the standard introspection result: **26 query root fields, two mutation root fields, no subscription root, and 107 type definitions**, including object fields, argument/default values, input objects, enums, directives and deprecation metadata. Its SHA-256 digest and every root interface are recorded in `schema_checks` and `schema_root_interfaces` in [the inventory](mlb_media_operations.yaml). This is the declared public schema at check time, not proof that every resolver allows anonymous data access.

The schema exposes **11 root fields beyond the inspected client documents**. These were documented from authoritative metadata without invoking their resolvers:

| Root field | Result type | Principal arguments |
| --- | --- | --- |
| `types` | `[SearchFieldTypeMetadata]` | None |
| `commonFields` | `[SearchFieldValue]` | `plays: [String]!`, language preference |
| `people` | `[PlayerInfo]` | `ids: [Int]` |
| `teams` | `[TeamInfo]` | `ids: [Int]` |
| `getReelBySlug` | `ReelResponse` | `slug: String!`, optional user ID |
| `story` | `Story` | `search: String!`, `searchType: StorySearchType!`, pagination/language/preview |
| `userStory` | `Story` | `startDate: String!`, user ID, experience, end date, language/limit |
| `playerStory` | `Story` | `playerId: Int!`, experience, dates, language/limit |
| `sections` | `SectionResponse` | `ids: [Int]` |
| `sectionTypes` | `SectionTypeResponse` | `ids: [Int]` |
| `sectionTypeFields` | `SectionTypeFieldResponse` | `sectionTypeId: Int!` |

Schema defaults can differ from a client's variable defaults; both are retained separately. Schema metadata includes additional arguments on existing roots, such as content groups/search types, topic team/session context and story date ranges. Full nested interfaces are available in the JSON snapshot rather than inferred from a single frontend's selections.

The same standard metadata request against **POST `https://data-graph.mlb.com/graphql`** returned HTTP 200 with GraphQL error **`INTROSPECTION_DISABLED`**, message “introspection has been disabled.” Schema inspection stopped there; source-defined queries remain available as described below. The configured **`https://graphql.cms.mlbinfra.com/graphql`** hostname failed DNS resolution, so no HTTP or schema result was obtained; this is an availability limitation, not evidence that the service does not exist. No alternate path or authentication bypass was attempted.

## Concrete network operations

Except where stated, GraphQL reads below use anonymous **GET `https://fastball-gateway.mlb.com/graphql`**, with `query`, `operationName` and JSON-encoded `variables`. Variable declarations, exact source-defined selection documents and required fragment definitions are retained as declarative interface metadata in the YAML inventory. These are not JavaScript wrappers, runnable request examples or a complete server schema. Append referenced fragments exactly once and remove Apollo `@client` selections before server transport. Source-only mutations remain unexecuted. Apollo `@client` selections are resolved locally and removed from transported queries; they are not additional server endpoints.

| Operation | Function | Verification |
| --- | --- | --- |
| `Search` (three document variants) | Structured or free-text search, pagination, language/content/feed preferences | 200 JSON with media metadata. Browser verified `FREETEXT`; a bounded structured condensed-game query also succeeded. |
| `getQueryInfo` | Parse/validate search expression; optional count and parsed graph | 200 JSON, tested structured expression `valid=true`; browser free-text parsing also observed. |
| `getFields` | Filter field names, types, values, ranges and units | 200 JSON using application-configured IDs. An unfiltered lookup returned 193 field definitions, retained in the inventory. Empty/null selector lists act as unfiltered selectors. |
| `clipQuery` | Resolve media IDs/slugs to playback metadata | 200 JSON for slug `blue-jays-vs-rays-highlights-x6166`, `idType=SLUG`, `withUser=false`. |
| `getTopic` | Topic playlist and pagination | 200 JSON for `most-popular`, including playlist media and model metadata. |
| `getGames` | Per-game condensed media, linescore, teams and decisions | 200 JSON for `716463`. |
| `getGameStatus` | Game state/date | 200 JSON `FINAL` and game date for `716463`. |
| `getContentTagDisplayValues` | Content-tag labels | 200 JSON mapping `condensed-game` to its display name. |
| `getSuggestedSearches` | Natural-language search suggestions | 200 JSON string list, observed in browser and bounded direct probe. |
| `getContainers` | Carousel/layout definitions and stored queries | 200 JSON for `Homepage V1`. |
| `getUserInfo` | Followed players/teams and favorite team | Application-generated anonymous `userId=""` request returned empty/default values. No other user identifiers tried. |
| `getUserSuggestions` | Personalized/contextual recommendations | 200 JSON empty list for `userId=anonymous`, country-only location and `VIDEO_MULTIPLE_PREFERENCES`. This verifies input acceptance, not recommendation completeness. |
| `getSuggestedStories` | Related editorial/story suggestions | HTTP 200 with a GraphQL resolver error for the tested slug. Do not treat as a successful data result. |
| `getReels` | User reel listing and owner display metadata | Source-defined; not called. User-specific access rules unverified. |
| `ValidateReel` | Validate a structured reel definition | Client document and introspected `ReelInput` interface retained; no validation payload constructed or resolver called. |
| `CreateReel` / `DeleteReel` | Save/delete reels | Server mutations found in source; **not executed**. |
| `getPlayerStories` | Player-highlight story availability | Fastball GET `/graphql/`, 200 JSON `valid=false` for player `592450`; source default experience `PLAYER_HIGHLIGHTS`. |
| `getGamesByGamePks` | Game video content, recap and related articles | Data Graph **POST `https://data-graph.mlb.com/graphql`**, 200 JSON, game `716463`. Source GET and persisted-query GET forms returned 400 empty bodies in this environment. |
| `UserQuery` | Current request geolocation | Data Graph **POST `/graphql`**, 200 JSON with `user.geoDetection` fields. Location values deliberately omitted. No bearer token supplied. |

GraphQL queries sent with POST above were read-only `query` operations. This does not authorize or imply execution of mutations.

## Additional REST and configuration routes

| Route | Source and inputs | Outcome |
| --- | --- | --- |
| `GET https://typeahead.mlb.com/api/v1/typeahead/suggestions/{input}` | Film Room search hook appends encoded text; optional `q` serializes suggestion options | 200 JSON for `Judge`, returning Aaron Judge `playerId=592450`, `teamId=147`. Full search phrase returned empty suggestion arrays. |
| `GET https://www.mlbstatic.com/mlb.com/video/config/mlb-vpp-aws/runtime_config.json` | Embedded `externalConfigLocation` (HTTPS equivalent fetched) | 200 JSON configuration overlay, `app.runTimeConfig=prod`. |
| `GET https://bdfed.stitch.mlbinfra.com/bdfed/contentful/dictionary/mlb-filmroom/{lang}-US?dictionaryKeys=filmroom` | Embedded `dictionaryEndpoint`; `lang=en` | 200 JSON UI dictionary. This is localization/configuration, not baseball data. |
| `GET https://dapi.mlbinfra.com/v2/content/en-us/videos/vizcast-{gamePk}` | Savant gamefeed client references `fields.playbackScenarios[0].location`; see [Savant audit](SAVANT_AUDIT.md) | 404 for historical game `716463`. Current-season coverage and successful Vizcast metadata access remain unestablished. |

The browser also called `GET https://terra.mlb.com/` and received request-location classification JSON, consistent with the source-map `getIsEU` resolver. This supports media/region behavior and is categorized separately from baseball event data. Returned location values are not retained.

## New evidence for event-to-video joins

Data Graph `getGamesByGamePks` uses `gamePks`, `locale=EN_US`, `contentSource=MLB`, recap tag `game-recap` and related tag `storytype-article`. The source defines a persisted-query form with hash `e12a5c2692406844d5f15f52ce7754208f53da81867de7d3e9e773611f4a040a`, but only POST with the full source-defined query succeeded in this audit.

For game `716463`, its video content response contained **25 video records**. The operation aliases `playGuid` to `guid` and `playbackScenarios` to `playbacks`. **16 returned `guid` values matched event `playId` values in the Stats API live feed**. The returned `playIdRef` values did not supply additional matches in this response. One concrete match:

| Identifier | Returned value |
| --- | --- |
| Game | `716463` |
| Editorial clip slug | `jordan-romano-in-play-run-s-to-josh-lowe` |
| Data Graph `guid` / source `playGuid` | `ec8bdeff-ca06-4ab5-8a37-1d2a72fa014a` |
| Stats API event `playId` | `ec8bdeff-ca06-4ab5-8a37-1d2a72fa014a` |

This supplies an observed editorial-media-to-event mapping that the original game-content slug inventory alone did not establish. Recaps, condensed games and some edited clips had no matching GUID. The 16 matches are evidence for this response, not a promise that every play or camera feed has a public clip.

## Boundaries and unresolved surfaces

All Film Room application GraphQL documents found in the inspected operation block are recorded, including source-only user/reel operations and local cache operations. Data-bearing read operations were exercised where the client supplied sufficient ordinary inputs. `getSuggestedStories` errors, historical Vizcast absence, untested user-specific operations, and unexecuted mutations are explicitly retained rather than omitted from coverage counts.

The Film Room configuration also names `https://graphql.cms.mlbinfra.com/graphql` and `https://content-service.mlb.com`. No concrete operation against those hosts was found in the inspected Film Room application documents or observed home/search network session. CMS remains a configuration lead whose hostname failed DNS during the standard schema check; Content Service remains a configuration lead without a source-defined path to probe. Neither is presented as a verified video API in [the earlier media-host inventory](mlb_media_endpoints.yaml). This audit does not exhaust every MLB application, private GraphQL resolver, authenticated club video system, subscription playback entitlement service, camera-angle archive or raw Hawk-Eye asset.

The public routes return curated media metadata and CDN URLs. They do not establish a public raw-storage bucket. Requesting a playback URL from metadata does not by itself establish permission to redistribute the footage. Incidental advertisement, audience measurement, consent and telemetry requests were excluded from the baseball/media-data inventory.

## Anonymous access to full-game broadcasts and Hawk-Eye recordings

Checked **2026-11-09** with no MLB account, cookies or authorization header. The anonymous Stats API request `GET https://statsapi.mlb.com/api/v1/game/716463/content` returns `media.epg` entries for the two MLB.TV archive feeds (home `BSSUN` and away `Sportsnet`). Each entry exposes a `contentId`, `mediaId`, `mediaFeedType`, `mediaState=MEDIA_ARCHIVE` and a `freeGame=true` flag for this 2023 game. Those are catalog/playback identifiers and availability metadata; they are not video URLs. The same response's editorial `playbacks` are different: they point to discrete, edited clips such as MP4 or HLS highlights.

The corresponding ordinary player route, `https://www.mlb.com/tv/g716463/v091d48fa-d854-4e39-9884-5df1cbe2e5ba`, returned HTTP 200 anonymously, but only as a roughly 1.2 MB MLB.TV HTML application shell. It embedded the game/content identifiers and playback settings; it contained no `.m3u8` or MP4 media URL. The anonymously fetched player bundles delegate to federated modules, so this check did not resolve a playable stream or prove the feed itself was accessible. No login, entitlement, token, signed URL or other access control was bypassed.

MLB's current [Terms of Use](https://www.mlb.com/official-information/terms-of-use) describe archived-game availability for MLB.TV subscribers, and its [MLB.TV help page](https://www.mlb.com/live-stream-games/help-center/subscription-access-how-to-watch-mlbtv-on-android-mobile-devices) says subscribers can access live out-of-market games and game archives, subject to blackouts. Therefore this probe found public archive metadata and a public player shell, but **did not establish anonymous access to a complete broadcast**. The `freeGame` marker for this historical entry is a metadata flag and does not by itself prove that the stream currently plays without an account.

No raw Hawk-Eye camera recording or full-game camera feed URL appeared in the checked public game-content response or player page. MLB's engineering article describes Hawk-Eye camera inputs as powering Statcast tracking data; the media-search evidence in this audit covers curated clips and broadcasts surfaced through the separate MLB.TV player. Neither the existence of camera/video storage nor its use inside MLB's systems establishes a public download endpoint. Public access verified here is limited to selected editorial video clips whose response metadata supplies MP4/HLS playback URLs. MLB's [engineering overview](https://technology.mlblogs.com/introducing-statcast-2020-hawk-eye-and-google-cloud-a5f5c20321b8) and [subscriber terms](https://www.mlb.com/official-information/terms-of-use) are useful context, but are not endpoint documentation or authorization to retrieve raw recordings.
