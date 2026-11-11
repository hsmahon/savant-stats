# Video, audio, and broadcast views

## Video

| View | Observed structure | Limit |
| --- | --- | --- |
| Stats API game content | JSON metadata includes editorial records and clip references. Some references contain MP4 or HLS URLs. | This is a bounded response sample. It does not define every game-content response. |
| HLS | The sampled master playlist uses `#EXTM3U`, lists variant playlists, resolutions, bit rates, and codec strings. | Only the master playlist was downloaded. The variants and media segments were not downloaded. |
| MP4 | FFprobe identified the sampled public clip as H.264 video at 1280×720 with AAC audio. | The clip was 29.546 seconds. The video file was not saved. This is a clip profile, not a full-game profile. |
| Film Room Fastball GraphQL | [The introspection snapshot](fastball_graphql_schema.json) gives the GraphQL type system. The page capture observed `Search`, `getTopic`, `getContainers`, `getSuggestedSearches`, and `getUserInfo` requests with HTTP 200 responses. | Network response bodies were disabled in the browser capture. The capture did not play a video or watch a full game. |

## Audio and broadcast

The observed Stats API schedule response contains broadcast and electronic program guide (EPG) metadata. The sample includes `mediaId`, `contentId`, feed type, language, and media state. It does not include an audio playback URL.

An EPG ID identifies metadata about a program or feed. It is not a stream URL and does not establish anonymous playback. The audio resolver, stream format, and storage schema remain unknown from this evidence.
