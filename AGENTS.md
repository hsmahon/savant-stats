# MLB endpoint catalog index

Use this page to find the endpoint inventories. The linked YAML/JSON catalogs are the detailed records; they include source-discovered routes and leads as well as operations verified by request. Check each record's method, `kind`, source, and verification/access fields before treating a URL as a working data endpoint.

## Endpoint inventories

| Surface | Inventory | Supporting reference |
| --- | --- | --- |
| MLB Stats API | [Published operations, response schemas, and access observations](api-routes/statsapi_route_inventory.yaml); [compact endpoint catalog](api-routes/statsapi_endpoints.yaml) | [OpenAPI snapshot](api-routes/statsapi_openapi.json); [game status codes](api-routes/statsapi_game_status.yaml); [observed response schemas](schemas/statsapi_observed_response_schemas.json) |
| Baseball Savant and Statcast | [Discovered route and module inventory](api-routes/savant_route_inventory.yaml); [compact endpoint catalog](api-routes/savant_endpoints.yaml); [leaderboards](api-routes/savant_leaderboards.yaml) | [CSV parameters and aggregate fields](api-routes/baseballsavant_csv_fields.yaml); [request-limit notes](api-routes/savant_rate_limits.yaml) |
| Gameday application | [Production route templates and probe results](api-routes/gameday_route_inventory.yaml) | Includes Stats API, BDFed, CMS, WebSocket, and Scout service references discovered in the client; [observed JSON shapes](schemas/gameday_observed_response_shapes.json). |
| Film Room and video services | [GraphQL operations, REST/configuration routes, and access results](api-routes/mlb_media_operations.yaml); [service and playback-resource index](api-routes/mlb_media_endpoints.yaml) | [Fastball GraphQL schema snapshot](schemas/fastball_graphql_schema.json); [video/audio response views](schemas/media_service_views.md) |

## Reading the records

All YAML route catalogs and the Stats API OpenAPI file are in [api-routes](api-routes/). Formal GraphQL, sample-derived JSON, and documented storage views are in [schemas](schemas/README.md). No private database table definitions are claimed.

- The Stats API OpenAPI file describes published operations. Its operation inventory records direct probes separately; documented security and observed access are distinct facts.
- Savant and Gameday inventories include application templates, aliases, data routes, and source-only operations. Use each entry's `kind` or status before counting it as a data API.
- Media records distinguish client GraphQL documents, server schema fields, REST/configuration routes, returned playback URLs, and client-only cache operations.
- `source_discovered` means a route or operation appeared in first-party client code. It does not prove that the route is current, anonymous, or data-bearing.
- `verified` describes the recorded request and date only. Errors, empty payloads, and authentication responses are retained as observations, not universal availability claims.

The inventories cover their recorded source sets; they do not claim to enumerate every private, authenticated, dynamic, or unpublished MLB service. For MLB's reported storage and distribution architecture, see [README.md](README.md); for the bounded November 10, 2026 public evidence and artifact hashes, see [MLB_PUBLIC_DATA_EVIDENCE.md](MLB_PUBLIC_DATA_EVIDENCE.md).
