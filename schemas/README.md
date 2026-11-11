# Service schemas and data views

This folder contains machine-readable schemas and bounded response-shape views. It does not contain MLB's private database DDL. A response shape from one request is not a complete service contract.

## Service map

| Service or data class | Schema or view | What the file establishes |
| --- | --- | --- |
| MLB Stats API | [Published OpenAPI file](../api-routes/statsapi_openapi.json) | Documented HTTP operations and their published request and response descriptions. It is not a database schema. |
| MLB Stats API | [Observed JSON response schemas](statsapi_observed_response_schemas.json) | Partial schemas derived from one bounded live-feed, game-content, and schedule response. The evidence note gives request details and hashes. |
| Baseball Savant | [CSV fields and parameters](../api-routes/baseballsavant_csv_fields.yaml) | Documented and observed CSV columns and query fields. This is a field catalog, not a JSON API schema. |
| Gameday, BDFed, CMS, Scout | [Observed JSON response shapes](gameday_observed_response_shapes.json) | Top-level keys from the HTTP observations in the Gameday route inventory. It includes error and authentication response shapes. It does not define nested types. |
| Film Room and Fastball | [Fastball GraphQL introspection snapshot](fastball_graphql_schema.json) | The observed GraphQL type system and root operations. It does not prove that every resolver is public or works. |
| Video playback | [Media response views](media_service_views.md) | Observed HLS and MP4 formats and their known limits. No full-game storage schema was found. |
| Audio and broadcast | [Media response views](media_service_views.md) and the Stats API sample schema | Observed EPG metadata fields. No audio stream schema or playback resolver was found. |
| Cloud SQL, Bigtable, Cloud Storage, BigQuery, Valkey | [Storage service views](storage_service_views.md) | Publicly reported data roles only. No table, row-key, bucket, or cache-key schema was published in the sources reviewed. |

The route catalogs are in [api-routes](../api-routes/). Their `source`, `kind`, `verification`, and `access` fields separate documented routes from client leads and direct observations.
