# MLB and Baseball Savant data reference

Endpoint discovery and data architecture research for Baseball Savant, Statcast and the MLB Stats API. This repository contains documentation and YAML catalogs.

Start with [endpoint research and verification results](ENDPOINT_RESEARCH.md) and [MLB data storage and distribution](MLB_DATA_ARCHITECTURE.md).

## Expanded endpoint audit

The expanded audit records source coverage, interface definitions and live access results in these files:

| Surface | Captured coverage | Report and interfaces |
| --- | --- | --- |
| Stats API | All 200 published GET operations, 533 schemas, 12 supplementary documented operations; 160 direct operation probes | [Audit](STATSAPI_AUDIT.md), [inventory](statsapi_route_inventory.yaml), [OpenAPI snapshot](statsapi_openapi.json) |
| Savant | 214 method/path templates across 129 page and 219 module requests; 108 read probes | [Audit](SAVANT_AUDIT.md), [inventory](savant_route_inventory.yaml) |
| Film Room/media | 28 Fastball schema root fields, 22 client document variants, five REST/configuration routes; 15 read-query names verified | [Audit](MEDIA_AUDIT.md), [operations](mlb_media_operations.yaml), [GraphQL schema](mlb_fastball_schema.json) |
| Gameday services | 36 application templates plus a color-feed reference; 13 read-only HTTP probes | [Audit](GAMEDAY_AUDIT.md), [inventory](gameday_route_inventory.yaml) |

The current public [Stats API OpenAPI endpoint](https://statsapi.mlb.com/v3/api-docs) returns a machine-readable specification. The audit compares it with the captured documentation snapshot; a publicly readable specification does not imply anonymous access to every listed operation.

Savant's total includes 120 application-page routes, 87 data/fragment routes, one legacy CSV route, one dynamic image route, one redirect and four source-only mutations. GraphQL root fields, client document variants and HTTP endpoints are different units. The inventories overlap; their counts must not be added as distinct backend APIs.

The original catalogs below remain concise references. The expanded inventories carry source locations, full discovery coverage and per-operation access results. Counts distinguish documented operations, frontend templates and live probes; they are not interchangeable.

The [private-service access report](PRIVATE_MLB_SERVICES.md) explains which services are publicly documented, which routes worked anonymously, and what remains restricted or unclear.

## Catalogs

| Catalog | Contents |
| --- | --- |
| [Savant endpoints](savant_endpoints.yaml) | Search, gamefeed, player lookup, Illustrator, replay and play video viewers |
| [MLB Stats API](statsapi_endpoints.yaml) | Game data, media metadata, live-feed updates, statistics and code lookups |
| [Media services](mlb_media_endpoints.yaml) | Playback resources and first-party video-service discovery leads |
| [Leaderboards](savant_leaderboards.yaml) | Leaderboard paths, parameters and inherited field metadata |
| [CSV fields](baseballsavant_csv_fields.yaml) | Search parameters and aggregate-output field reference |
| [Game status](statsapi_game_status.yaml) | Game status code reference |
| [Request limits](savant_rate_limits.yaml) | Current official Stats API rate policy; Savant limits not published in reviewed docs; historical community reports separated |

## What was verified

Checks on **2026-11-09** confirmed public JSON for game content, the v1.1 live feed, feed timestamps, context metrics, Savant gamefeed, player lookup, Illustrator stats and replay breakdowns. Game content includes highlight playback metadata. A play ID from the live feed resolved through Savant's video viewer to an MP4 resource; its HEAD response was `200 video/mp4`. A highlight HLS master playlist returned `200` and an `#EXTM3U` header. These checks used a historical game and do not establish universal season or play coverage. [Evidence](ENDPOINT_RESEARCH.md).

The tested play-analytics route returned `401` with an Okta login page. The historical Swagger URL returned `200` HTML for a Docusaurus documentation shell rather than a JSON specification. Authentication is endpoint-specific; public access to selected routes does not establish access to every tracking or media dataset. [Evidence](ENDPOINT_RESEARCH.md).

The public game-content API exposes archive feed identifiers, and the corresponding MLB.TV URL returns an anonymous HTML player shell. Neither response exposed a full-game stream URL. MLB describes archive availability as subscriber access; see the [broadcast and Hawk-Eye access audit](MEDIA_AUDIT.md#anonymous-access-to-full-game-broadcasts-and-hawk-eye-recordings) for the bounded probe and caveats.

## Reading the catalogs

Each newly verified route records its check date, response type and evidence reference. Entries without verification are inherited references or explicitly marked discovery leads. An HTML application, JSON endpoint, video resource and internal storage component are different kinds of surface. Hostnames found in application configuration are leads until an actual operation is verified.

The concise catalogs do not replace the captured official Stats API specification. Inherited field lists, leaderboard availability dates and query options have not all been revalidated. Request-limit status and evidence are recorded separately in [savant_rate_limits.yaml](savant_rate_limits.yaml). The CSV field list describes an aggregate search context and should not be treated as the complete pitch-level Statcast schema; consult MLB's [CSV documentation](https://baseballsavant.mlb.com/csv-docs).

MLB's engineering articles describe PostgreSQL, Bigtable, Cloud Storage and BigQuery at different dates. The [architecture reference](MLB_DATA_ARCHITECTURE.md) separates those historical accounts from observed public responses and identifies unresolved questions.

MLB/MLBAM copyright and [terms of use](https://www.mlb.com/official-information/terms-of-use) apply. Access checks are observations, not licensing conclusions or stable service guarantees.

## Verification

All YAML catalogs parse without duplicate keys; local document links resolve. The Stats API inventory contains exactly the 200 operation keys in the captured specification, and its 1,627 internal schema references resolve. Snapshot hashes and source references are recorded and checked. No JavaScript application, package manifest, examples directory or runnable script samples are retained.

Coverage is complete for the explicitly audited source sets. It does not establish all private MLB services, every dynamic/feature-gated route or every parameter combination. Restricted interfaces, disabled introspection, failed requests and unprobed operations remain visible in the inventories.
