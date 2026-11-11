# Publicly reported storage views

These are service-level views from published MLB and Google Cloud descriptions. They are not table definitions or a current deployment map.

| Service | Publicly described data role | Schema or table view available here |
| --- | --- | --- |
| Cloud SQL for PostgreSQL | 2022 MLB/Google Cloud reporting places game data in Cloud SQL. | No database name, table name, column, index, or DDL was published in the sources reviewed. |
| Bigtable | 2022 reporting places player pose or positional data in Bigtable. | No table ID, row-key format, column family, cell layout, or retention policy was published. |
| Memorystore for Valkey | The 2025 report describes ballpark message buffering, multi-region GUMBO reads, and read-through caches for stats and roster data. | No cache key, value schema, TTL, or cluster configuration was published. The GUMBO article describes the JSON record's meaning but does not publish its full JSON schema. |
| Cloud Storage | 2020 reporting describes clipped camera feeds saved for video search. 2022 reporting describes nightly analytical copies. | No bucket name, object key layout, object metadata schema, lifecycle rule, or retention policy was published. The reports do not show that video and analytics objects share a bucket. |
| BigQuery | 2022 reporting describes analytical copies in BigQuery. 2024 reporting describes Statcast delivery to club BigQuery environments through APIs. | No dataset, table, field, partition, or view definition was published in the sources reviewed. |

The phrases “game data,” “pose data,” and “analytical copies” describe data roles. They do not identify physical tables. The public architecture figures do not establish table layouts or object paths.
