# MLB data systems and endpoint catalog

This repository maps public Baseball Savant, MLB Stats API, Gameday, and MLB video interfaces, and summarizes what first-party sources say about MLB's data systems.

For endpoint inventories and schemas, start with [AGENTS.md](AGENTS.md). The catalogs distinguish documented endpoints, source-discovered routes, directly observed responses, and access that remains unknown. The [public data evidence note](MLB_PUBLIC_DATA_EVIDENCE.md) records a bounded REA/browser investigation and response hashes from November 10, 2026.

## How MLB's data moves

The diagrams below combine dated MLB/Google Cloud reports with public observations. They show reported roles, not a live deployment map. Labels use [ASD-STE100](https://www.asd-ste100.org/) principles: short terms, clear actions, and separate known and unknown links. MLB product names remain unchanged. Solid arrows show a flow that a source describes. Dotted arrows show a link that remains uncertain. The sources do not expose private schemas, bucket names, or unrestricted storage access. [Sources 1–6](#sources)

### Capture, storage, and distribution

```mermaid
flowchart LR
  subgraph field[Ballpark edge path: 2025 report]
    cameras[Tracking systems<br/>player movement data]
    operators[Operator apps<br/>event metadata]
    amq[AMQ message queue]
    injection[Injection services]
    edgevalkey[Valkey edge cache<br/>in Google Distributed Cloud]
    cloudservices[Google Cloud services]
    cameras -->|send tracking data| amq
    operators -->|send metadata| amq
    amq -->|send messages| injection
    injection -->|read and write| edgevalkey
    injection -->|write data| cloudservices
  end

  subgraph stores[Reported data stores]
    game[Game records<br/>Cloud SQL for PostgreSQL<br/>2022 report]
    pose[Player pose data<br/>Bigtable<br/>2022 report]
    videoFeed[Camera video feeds]
    clips[Clipped camera feeds<br/>Cloud Storage<br/>2020 report]
    filmArchive[Film Room archive]
    googleCloud[Google Cloud<br/>storage service not named]
    flow[Nightly Dataflow job<br/>2022 report]
    archive[Analytics copies<br/>Cloud Storage and BigQuery]
    game -->|copy at night| flow
    pose -->|copy at night| flow
    flow -->|write copies| archive
  end
  videoFeed -->|2020 report: clip and save for search| clips
  filmArchive -->|hosted in 2022 report| googleCloud

  subgraph gumbo[Live GUMBO path: 2025 report figure]
    ingest[Ballpark ingestion apps]
    eastapps[GCP apps<br/>East region]
    east[GUMBO cache<br/>Valkey; figure says Redis]
    eventjob[opjob-event<br/>replication job]
    central[GUMBO cache<br/>Valkey; figure says Redis]
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
  end

  subgraph readthrough[Other stats and roster data: 2025 report]
    databases[Databases]
    regular[Regular Valkey cluster]
    heavy[Heavy Valkey cluster]
    apps[MLB apps and scoreboards]
    apps -->|request data| regular
    apps -->|request data| heavy
    regular -->|read through on cache miss| databases
    heavy -->|read through on cache miss| databases
    databases -->|populate cache| regular
    databases -->|populate cache| heavy
    regular -->|return data| apps
    heavy -->|return data| apps
  end

  subgraph public[Public paths observed in this project]
    stats[Stats API<br/>game and pitch data]
    savant[Baseball Savant<br/>Statcast CSV]
    csv[Pitch-level rows]
    content[Game content and schedule<br/>metadata responses]
    video[MP4 and HLS clip URLs]
    epg[Broadcast and audio EPG<br/>metadata only]
    audio[MLB Audio<br/>stream path unknown]
    tv[MLB.TV Media Center]
    stats --> content
    content --> video
    stats --> epg
    savant -->|return| csv
    epg -.->|playback route not observed| audio
    tv -.->|stream request not observed| audio
  end

  subgraph planned[Google Cloud case study]
    mediacdn[Media CDN]
    live[Planned live game streaming]
    mediacdn -->|deliver| live
  end
```

The 2025 article describes three Valkey uses. At the ballpark, an AMQ queue sends tracking data and operator metadata to injection services. Those services read and write a local Valkey cache. They also write data to Google Cloud services. For live games, GUMBO is a JSON record of game state. Regional apps read and write GUMBO caches. An `opjob-event` job replicates data from East to Central. A CDN serves `getLiveGumbo`; the article does not name the CDN product. For other data, regular and heavy Valkey clusters use read-through caching. MLB names stats, rosters, standings, and schedules as cached data. These links are from the article's [ballpark figure](https://storage.googleapis.com/gweb-cloudblog-publish/images/1_3QpqGX2.max-800x800.png) and [GUMBO figure](https://storage.googleapis.com/gweb-cloudblog-publish/images/2_QNOyrE2.max-1600x1600.png). The GUMBO figure labels its caches "redis gumbo"; the article text names Memorystore for Valkey. The source does not state if the CDN in the figure is Google Media CDN. The API and clip paths below are separate public observations. [Sources 1–6](#sources)

### Storage and analytical copies

```mermaid
flowchart TD
  subgraph eventdata[Game and tracking data: 2022 report]
    event[Game events and operator tags]
    track[Player pose records]
    event -->|store game data| sql[Cloud SQL<br/>PostgreSQL]
    track -->|store pose data| bt[Bigtable]
  end

  subgraph batchcopy[Nightly copy: 2022 report]
    flow[Dataflow job]
    sql -->|read| flow
    bt -->|read| flow
    flow -->|write analytical files| gcs[Cloud Storage<br/>analytics copy]
    flow -->|write tables| bq[BigQuery]
    bq -->|support analysis| clubs[Club analytics]
  end

  subgraph videodata[Video clip storage: 2020 report]
    feed[Camera video feeds]
    cuts[Selected video clips]
    videoStore[Cloud Storage<br/>video search data]
    feed -->|create clips| cuts
    cuts -->|save clips| videoStore
  end

  api[MLB APIs<br/>2024 report]
  statcast[Statcast data]
  teams[MLB teams and products]
  statcast -->|send through API| api
  api -->|provide data| teams
```

The 2020 article says MLB saved clipped camera feeds in Cloud Storage for search. The 2022 article names Cloud SQL for game data and Bigtable for pose data. It also describes nightly Dataflow copies to Cloud Storage and BigQuery. These sources do not name retention rules, table layouts, or bucket paths. They do not say if video and analytical files use the same storage. MLB's 2024 account says Statcast data reaches club BigQuery environments through APIs. [Sources 1–3](#sources)

### Public services and what they expose

```mermaid
flowchart LR
  subgraph clients[Public client pages]
    gd[Gameday]
    bs[Baseball Savant]
    film[Film Room]
    tv[MLB.TV Media Center]
  end

  subgraph gameapi[Game and Statcast services]
    stats[Stats API<br/>game and pitch data]
    bdfed[BDFed<br/>game aggregation]
    scout[Scout Insights<br/>tested routes need a token]
    csv[Statcast CSV]
    gd -->|request game data| stats
    gd -->|request game feed| bdfed
    gd -->|request protected insights| scout
    bs -->|request pitch rows| csv
  end

  subgraph mediaapi[Video metadata services]
    fastball[Fastball GraphQL<br/>Film Room queries]
    content[Stats API game content<br/>editorial and clip metadata]
    search[Search results and topics]
    playback[MP4 and HLS clip URLs]
    film -->|send GraphQL queries| fastball
    fastball -.->|response body not captured| search
    stats -->|return game content| content
    content -->|include clip links| playback
  end

  subgraph audioapi[Broadcast and audio]
    schedule[Stats API schedule<br/>broadcast and EPG metadata]
    epg[Radio and TV feed IDs<br/>mediaId and contentId]
    stream[Audio stream route<br/>not observed]
    player[MLB Audio player<br/>subscriber feature]
    stats -->|return schedule| schedule
    schedule -->|include feed metadata| epg
    epg -.->|resolver not found| stream
    stream -.->|public path unknown| player
  end

  stats -.->|some tested analytics routes need a token| restricted[Restricted analytics data]
```

This diagram combines repository observations with dated reports. Gameday uses Stats API and BDFed services. Tested Scout routes required a bearer token. On November 10, 2026, the game-content response included clip URLs. The schedule response included radio and TV EPG IDs. MLB Help says MLB Audio is a subscriber feature. The IDs are metadata. They are not stream URLs. These observations do not show access to raw camera recordings, full broadcasts, or private databases. See the [evidence note](MLB_PUBLIC_DATA_EVIDENCE.md).

## How to interpret the architecture

- MLB's engineering sources describe different dates and levels of detail. The 2020–2022 articles describe the stores and data copies; MLB's 2025 account adds the Valkey caching layer. These reports do not prove that every component remains deployed exactly as described.
- A Savant export, Stats API response, GraphQL result, playback URL, and internal storage object are different representations. An identifier match in one observed game does not guarantee universal coverage or a complete event-to-video join.
- MLB's published player-tracking point counts differ across dates and descriptions. Do not treat the public pitch-level exports as the complete camera or pose stream.
- Publicly readable documentation or a schema does not mean every operation or underlying dataset is publicly accessible. Authentication and response outcomes are recorded per request in the inventories.
- Public sources identify no storage layout for the audio stream itself, no direct access to MLB's operational databases, and no public raw-camera bucket. The sampled HLS/MP4 resources are selected clips, not full broadcasts.
- Google Cloud's case study names Media CDN for live streaming; it does not establish that the observed highlight host uses that product.

## Sources

1. Ben Jedlovec, MLB Technology Blog, July 20, 2020, [Introducing Statcast 2020: Hawk-Eye and Google Cloud](https://technology.mlblogs.com/introducing-statcast-2020-hawk-eye-and-google-cloud-a5f5c20321b8). The camera, PostgreSQL, Stats API distribution, and clipped-video statements are historical descriptions from this article.
2. Priyanka Vergadia with MLB's Rob Engel, Google Cloud, April 7, 2022, [Teaming up with MLB for game-changing sports analytics](https://cloud.google.com/blog/products/data-analytics/mlb-pitches-new-data-uses-with-google-cloud-services). Describes stadium processing, manual context tagging, Cloud SQL, Bigtable, Dataflow, Cloud Storage, BigQuery, and product feeds.
3. Rob Engel, MLB VP of Software Engineering, Google Cloud, June 13, 2024, [AI at Bat: How Major League Baseball is surfacing stats faster with AI](https://cloud.google.com/transform/mlb-statcast-ai-fan-experience-team-analytics). Describes near-real-time APIs and distribution to MLB products, clubs, scoreboards, and broadcast partners.
4. Rahul Joshi, Principal Software Engineer, MLB, Google Cloud, August 4, 2025, [How MLB keeps fans connected to the game – one cache hit at a time](https://cloud.google.com/blog/products/databases/mlb-delivers-real-time-data-to-fans). Describes ballpark buffering, two-region GUMBO, and read-through caches. The article includes [a ballpark diagram](https://storage.googleapis.com/gweb-cloudblog-publish/images/1_3QpqGX2.max-800x800.png) and [a GUMBO diagram](https://storage.googleapis.com/gweb-cloudblog-publish/images/2_QNOyrE2.max-1600x1600.png).
5. Google Cloud, October 11, 2022, [Crackerjack data: How MLB is reinventing the fan experience](https://cloud.google.com/blog/transform/mlb-reinventing-fan-experience-with-cloud-today-tomorrow-even-yesterday). Reports that Film Room was hosted on Google Cloud, with over three million searchable video assets and automated clip generation; it does not name a specific serving bucket or media origin.

6. Google Cloud, [MLB case study](https://cloud.google.com/customers/major-league-baseball). Lists Media CDN among the products used in MLB's cloud portfolio and describes planned live-stream delivery; it does not identify the delivery service for the public clip host observed here.

MLB/MLBAM [terms of use](https://www.mlb.com/official-information/terms-of-use) apply. Repository observations are documentation, not a licensing determination or a guarantee of stable service behavior.
