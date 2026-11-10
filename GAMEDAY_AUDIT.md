# Gameday application route audit

Checked **2026-11-09**. This audit complements the Stats API specification and the Savant and media inventories with the routes actually configured by the current production Gameday application.

## Source and coverage

The [historical-game Gameday page](https://www.mlb.com/gameday/716463/final) redirects to a dated game URL and exposes `appInfo.gdapp`, which identifies the [production application bundle](https://prod-gameday.mlbstatic.com/app-mlb/5.61.0-mlb.1/gd.min.js). Its literal `pathTemplate` values were extracted, assigned to their production hosts from adjacent configuration, deduplicated by exact host/template and cross-checked manually. Development and QA-only host variants were not probed. A static-resources URL explicitly linked by the page was also fetched; it contains promotion/display configuration rather than another route registry.

The [machine-readable inventory](gameday_route_inventory.yaml) retains source character offsets, source hash, placeholders, method basis and request results. Different parameter/hydration variants of the same URL path remain separate application templates, so this count must not be added to the Stats API operation count as if all were new operations.

The bundle contains **42 literal path-template occurrences**, yielding **36 unique production host/template pairs** after exact duplicates, an empty template and a QA-only query variant are excluded. One additional literal color-feed route is recorded outside the path-template registry. Its separate GET probe returned 404 with an empty body and no Content-Type header; the exact request is retained in the YAML. **12 read-only HTTP probes** were performed: **8 returned 200 JSON, 2 returned 401 JSON, 1 returned 403 XML, and 1 returned 500 JSON**.

## Findings

- **Public game aggregation:** `bdfed/gameday-content/` returns audio, TV, video, recap, summary and EPG groups. `bdfed/playMetrics/{gamePk}` returns categorized scoring plays, home runs, hard hits, strikeouts, lead changes, leverage, win-probability and ABS challenge groups for the tested flags. These BDFed application services are distinct from Stats API itself.
- **Public supporting data:** Gameday configuration, localization dictionary, matchup data, spotlight data, article lookup and the Stats API uniforms route returned JSON. A 200 result can still be empty or configuration-oriented; response keys and size are recorded without assuming coverage.
- **Restricted Scout services:** the client explicitly attaches `Authorization: Bearer {accessToken}` to `/api/v1/insights/{gamePk}/{timecode}` and `/api/v1/scout-game-briefings/{gamePk}/{timecode}` on `scout-insights-api.mlbinfra.com`. Anonymous requests returned 401. No token or authenticated request was attempted. MLB's [October 2026 engineering article](https://technology.mlblogs.com/scout-insights-automating-the-whoa-didnt-know-that-feeling-139df5300ff2) describes the product's precomputed wording and cache layers and says Gameday viewers use a free MLB account; it does not publish its API paths.
- **Live updates:** the client configures both delayed Gameday and direct WebSocket subscription paths on `wss://ws.statsapi.mlb.com`. These are source-discovered subscription routes; no persistent connection was opened. HTTPS feed requests can also target `ws.statsapi.mlb.com` in production client configuration.
- **GraphQL:** Data Graph `getGamesByGamePks` and Fastball `getPlayerStories` documents are embedded in the bundle; their verification is tracked in [the media audit](MEDIA_AUDIT.md), rather than inferring support from a hostname.
- **Errors preserved:** the historic-game team-comparison probe returned 500; the player-of-the-game winner resource returned 403. These results do not distinguish missing historical coverage from other server/access conditions.

## Template outcomes

All paths are relative to the host recorded in the YAML. Placeholders and defaults below are copied from source; they are not parameter values to send literally.

| Host | Template | Verification |
| --- | --- | --- |
| `https://statsapi.mlb.com` | `api/v1.1/game/{gid}/feed/live{differ}?language={site}{timecode}{pushUpdateId}` | source_discovered_not_probed |
| `https://statsapi.mlb.com` | `api/v1/game/{gid}/content?language={site}` | source_discovered_not_probed |
| `https://statsapi.mlb.com` | `api/v1/schedule?gamePk={gid}&language={site}&hydrate=story,xrefId,lineups,broadcasts(all),probablePitcher(note),game(tickets)&useLatestGames=true&fields=dates,games,teams,probablePitcher,note,id,dates,games,broadcasts,type,name,homeAway,isNational,dates,games,game,tickets,ticketType,ticketLinks,dates,games,lineups,homePlayers,awayPlayers,useName,lastName,primaryPosition,abbreviation,dates,games,xrefIds,xrefId,xrefType,story` | source_discovered_not_probed |
| `https://statsapi.mlb.com` | `api/v1/people?personIds={playerId}&season={yyyy}&hydrate=stats(group=hitting,type=season,season={yyyy},gameType={game_type:R})` | source_discovered_not_probed |
| `https://statsapi.mlb.com` | `api/v1/people?personIds={playerId}&season={yyyy}&hydrate=stats(group=pitching,type=season,season={yyyy},gameType={game_type:P})` | source_discovered_not_probed |
| `https://statsapi.mlb.com` | `api/v1/people?personIds={playerId}&season={yyyy}&hydrate=stats(group=hitting,type=season,season={yyyy},gameType={game_type:P})` | source_discovered_not_probed |
| `https://dapi.cms.mlbinfra.com/v2/content` | `{site}-us/stories?tags.slug=gamepk-{gid}&$limit={limit:10}&$sort=contentDate:{sort:desc}` | 200 JSON ['pagination', 'meta', 'items'] |
| `https://gd.mlb.com` | `components/game/{sportcode}/year_{yyyy}/month_{mm}/day_{dd}/gid_{gid}/media/mobile{underLang}.xml` | source_discovered_not_probed |
| `https://bdfed.stitch.mlbinfra.com` | `bdfed/contentful/dictionary/mlb-gameday/{site}-JP` | 200 JSON ['dictionary'] |
| `wss://ws.statsapi.mlb.com` | `api/v1/game/push/subscribe/gameday/{gid}` | source_discovered_not_probed |
| `wss://ws.statsapi.mlb.com` | `api/v1/game/push/subscribe/{gid}` | source_discovered_not_probed |
| `https://scout-insights-api.mlbinfra.com` | `api/v1/insights/{gamePk}/{timecode}` | 401 JSON ['message', 'details'] |
| `https://statsapi.mlb.com` | `api/v1/schedule` | source_discovered_not_probed |
| `https://bdfed.stitch.mlbinfra.com` | `bdfed/transform-mlb-postseason-mini-scoreboard` | source_discovered_not_probed |
| `https://statsapi.mlb.com` | `api/v1.1/game/{gamePk}/feed/live{differ}?language={language}{timecode}{pushUpdate}` | source_discovered_not_probed |
| `https://bdfed.stitch.mlbinfra.com` | `bdfed/gameday/{locale:en-US}?gamePk={gamePk}&config=true&appContext={appContext}&stitch_env={stitchEnv}` | 200 JSON ['teamId', 'name', 'teamNameDisplay', 'triCode', 'sportId', 'type', 'sid', 'socialFooter', 'bettingOddsSchedule', 'internalName', 'sportIds', 'leagueIds', 'disableDynamicFieldView', 'disableDynamicFieldViewDesktop', 'disableDynamicFieldViewMobile', 'disableFieldVision', 'disableFieldVisionOD', 'useFallbackFV', 'fieldVisionLiveFreeGameOnly', 'disableDynamicUniforms'] |
| `https://statsapi.mlb.com` | `api/v1/standings?leagueId={leagueIds}&season={season}&date={gameDate}&hydrate=division` | source_discovered_not_probed |
| `https://statsapi.mlb.com` | `api/v1.1/game/{gamePk}/feed/live?language={language}&hydrate=ruleSettings&fields=gameData,ruleSettings` | source_discovered_not_probed |
| `https://statsapi.mlb.com` | `api/v1/schedule?gamePk={gamePk}&language={language}&hydrate=story,xrefId,lineups,broadcasts(all),probablePitcher(note),game(content(media(epg)),tickets)&useLatestGames=true&fields=dates,games,teams,probablePitcher,note,id,dates,games,broadcasts,type,name,homeAway,language,isNational,callSign,mediaState,mediaStateCode,availableForStreaming,freeGame,mediaId,dates,games,game,tickets,ticketType,ticketLinks,dates,games,content,media,epg,dates,games,lineups,homePlayers,awayPlayers,useName,lastName,primaryPosition,abbreviation,dates,games,xrefIds,xrefId,xrefType,story` | source_discovered_not_probed |
| `https://statsapi.mlb.com` | `api/v1/people/{batterId}?hydrate=stats(group=batting,type=vsPlayerTotal,opposingPlayerId={pitcherId},season={season})` | source_discovered_not_probed |
| `https://data-graph.mlb.com` | `graphql/?operationName=getGamesByGamePks&{query}&variables={variables}` | source_discovered_not_probed |
| `https://bdfed.stitch.mlbinfra.com` | `bdfed/gameday-content/?locale={locale:en}&gamePk={gamePk}` | 200 JSON ['audio', 'tv', 'videos', 'recap', 'summary', 'epg'] |
| `https://bdfed.stitch.mlbinfra.com` | `bdfed/playMetrics/{gamePk}?keyMoments={keyMoments}&scoringPlays={scoringPlays}&homeRuns={homeRuns}&strikeouts={strikeouts}&hardHits={hardHits}&highLeverage={highLeverage}&leadChange={leadChange}&winProb={winProb}&absChallenge={absChallenge}` | 200 JSON ['scoringPlays', 'homeRuns', 'hardHits', 'strikeouts', 'leadChange', 'highLeverage', 'winProb', 'keyMoments', 'absChallenge', 'allPlayMetrics'] |
| `https://bdfed.stitch.mlbinfra.com` | `bdfed/gameday-team-comparison?homeId={homeId}&awayId={awayId}&gameType={gameType}&endDate={gameDate}` | 500 JSON ['error'] |
| `https://bdfed.stitch.mlbinfra.com` | `bdfed/spotlight-game/{locale:en-US}?gamePk={gamePk}` | 200 JSON ['flipYear', 'useDate', 'useSeason', 'useSeasonState', 'useGameType', 'limit', 'page', 'playersRequested', 'totalPages', 'maxPlayers', 'exceededMax', 'players', 'reqHeaders', 'headers', 'status'] |
| `https://fastball-gateway.mlb.com` | `graphql/?query={query}&variables={variables}` | source_discovered_not_probed |
| `https://dapi.cms.mlbinfra.com/v2/content` | `{locale}/stories?tags.slug=gamepk-{gamePk}&$limit={limit:5}&$sort=contentDate:{sort:desc}` | source_discovered_not_probed |
| `https://statsapi.mlb.com` | `api/v1/uniforms/game?gamePks={gamePk}` | 200 JSON ['copyright', 'uniforms'] |
| `https://statsapi.mlb.com` | `api/v1/people?personIds={playerId}&season={season}&hydrate=stats(group=hitting,type=season,season={season},gameType={gameType})` | source_discovered_not_probed |
| `https://bdfed.stitch.mlbinfra.com` | `bdfed/matchup/{gamePk}?statList={statList}` | 200 JSON ['probables', 'home', 'away'] |
| `https://statsapi.mlb.com` | `api/v1/schedule?` | source_discovered_not_probed |
| `https://mlbentitlementservices.mlb.com/entitlements` | `?principalType=user&principalId={uid}&includeFeatures=true` | source_discovered_not_probed |
| `https://scout-insights-api.mlbinfra.com` | `api/v1/scout-game-briefings/{gamePk}/{timecode}` | 401 JSON ['message', 'details'] |
| `https://www.mlb.com` | `apps/player-of-the-game/game/json/winners/{gamePk}.json` | 403 |
| `wss://ws.statsapi.mlb.com` | `api/v1/game/push/subscribe/gameday/{gamePk}` | source_discovered_not_probed |
| `wss://ws.statsapi.mlb.com` | `api/v1/game/push/subscribe/{gamePk}` | source_discovered_not_probed |

## Limits

This is complete for the literal path-template set in the cited bundle, not for MLB's backend or every dynamically assembled URL. Bundle presence alone does not prove a feature is enabled or that a route remains supported. The legacy XML template, entitlement query, GraphQL templates and WebSocket subscriptions are retained as source discoveries without treating them as anonymously verified REST data. No account, voting, entitlement or other mutation was performed. Historical-game responses do not establish current-season completeness.
