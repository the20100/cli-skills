---
name: youtube-data
version: 1.0.0
description: Use when the user wants to search YouTube, get video/channel/playlist/comment data, or manage playlist items using the youtube-data CLI. Trigger on requests like "search for videos about X", "get info about this video", "list videos in this playlist", "get channel stats", "show comments on this video", "add video to playlist", etc.
---

# YouTube Data CLI

The `youtube-data` CLI searches and manages YouTube data via the YouTube Data API v3.

Run commands with the Bash tool. Find the binary by running `which youtube-data` or look in the standard PATH. The binary name is `youtube-data`.

**If the `youtube-data` binary is not found**, install it first:

```bash
# Check if available
which youtube-data

# If not found, clone, build, install, then delete the source folder
git clone https://github.com/the20100/youtube-data-cli
cd youtube-data-cli
go build -o youtube-data .
mv youtube-data /usr/local/bin/
cd ..
rm -rf youtube-data-cli

# If found, check if the last version is installed
youtube-data update
```

**Authentication:**

- **API key** (required for all read commands): `youtube-data auth set-key <api-key>` or set `YOUTUBE_API_KEY` env var.
- **OAuth token** (required for write operations like adding/removing playlist items): `youtube-data auth set-oauth-token <token>` or set `YOUTUBE_OAUTH_TOKEN` env var.

Get your API key at: https://console.developers.google.com/
  1. Enable the YouTube Data API v3
  2. Credentials → Create credentials → API key

Credentials are stored in:
- macOS: `~/Library/Application Support/youtube-data/config.json`
- Linux: `~/.config/youtube-data/config.json`
- Windows: `%AppData%\youtube-data\config.json`

---

## Environment variables

**API Key (read-only access)** — the following env var names are accepted (first non-empty wins):

| Variable | Notes |
|----------|-------|
| `YOUTUBE_API_KEY` | Primary (canonical) |
| `YOUTUBE_KEY` | Short form |
| `YOUTUBE_API` | Without "_KEY" suffix |
| `YOUTUBE_DATA_API_KEY` | Full data API form |
| `GOOGLE_API_KEY_YOUTUBE` | Google-prefixed form |
| `API_KEY_YOUTUBE` | Reversed prefix |
| `YOUTUBE_PK` | Public key shorthand |
| `YOUTUBE_PUBLIC` | Public key long form |

**OAuth Token (required for write operations)** — the following env var names are accepted:

| Variable | Notes |
|----------|-------|
| `YOUTUBE_OAUTH_TOKEN` | Primary (canonical) |
| `YOUTUBE_ACCESS_TOKEN` | Access token form |
| `YOUTUBE_TOKEN` | Short form |
| `YOUTUBE_BEARER_TOKEN` | Bearer token form |
| `YOUTUBE_SECRET` | Secret form |
| `YOUTUBE_SK` | Secret key shorthand (suffix) |
| `YOUTUBE_API_SECRET` | API secret form |

OAuth token takes priority over API key when both are set.

---

## Global flags (apply to every command)

| Flag | Description |
|------|-------------|
| `--json` | Force JSON output |
| `--pretty` | Force pretty-printed JSON output (implies --json) |

Output is **auto-detected**: JSON when piped, human-readable tables in terminal.

---

## update

Update the binary to the latest version. Requires `git` and `go`.

```bash
youtube-data update
```

---

## info

Show binary location, config path, and active credential status.

```bash
youtube-data info
```

---

## auth

```bash
youtube-data auth set-key <api-key>          # Save API key to config file
youtube-data auth set-oauth-token <token>    # Save OAuth token (for write operations)
youtube-data auth status                     # Show current auth status
youtube-data auth logout                     # Remove all saved credentials
```

---

## search

### `search list [query]`

Search YouTube for videos, channels, or playlists.

```bash
youtube-data search list "golang tutorials"
youtube-data search list "tech news" --type video --order date --max-results 20
youtube-data search list --channel-id UCVHFbw7woebKtfvug_hvrig --type video
youtube-data search list "music" --type playlist --json
```

| Flag | Description |
|------|-------------|
| `--type` | Filter: `video`, `channel`, `playlist` (default: all) |
| `--channel-id` | Restrict to a specific channel |
| `--order` | Sort: `relevance`, `date`, `rating`, `title`, `viewCount`, `videoCount` |
| `--language` | Relevance language (e.g. `en`, `fr`) |
| `--max-results` | Results count (1–50, default: 25) |
| `--page-token` | Pagination token from previous response |

---

## videos

### `videos get <video-id>`

Get full details of a video (snippet, statistics, contentDetails, status).

```bash
youtube-data videos get dQw4w9WgXcQ
youtube-data videos get dQw4w9WgXcQ --pretty
youtube-data videos get dQw4w9WgXcQ --parts snippet,statistics
```

| Flag | Description |
|------|-------------|
| `--parts` | Comma-separated parts (default: `snippet,statistics,contentDetails,status`) |

### `videos list`

List videos by IDs or chart.

```bash
youtube-data videos list --ids dQw4w9WgXcQ,9bZkp7q19f0
youtube-data videos list --chart mostPopular --max-results 10
youtube-data videos list --chart mostPopular --json | jq '.[].id'
```

| Flag | Description |
|------|-------------|
| `--ids` | Comma-separated video IDs |
| `--chart` | Chart: `mostPopular` |
| `--channel-id` | Filter by channel ID |
| `--parts` | Comma-separated parts (default: `snippet,statistics`) |
| `--max-results` | Results count (default: 25) |
| `--page-token` | Pagination token |

---

## channels

### `channels get [channel-id]`

Get details of a channel by ID or handle.

```bash
youtube-data channels get UCVHFbw7woebKtfvug_hvrig
youtube-data channels get --handle @Google
youtube-data channels get UCVHFbw7woebKtfvug_hvrig --pretty
```

| Flag | Description |
|------|-------------|
| `--handle` | Channel handle (e.g. `@Google`) |
| `--parts` | Comma-separated parts (default: `snippet,statistics,contentDetails`) |

The `contentDetails` part includes the **uploads playlist ID** — use it with `playlist-items list` to get all uploaded videos.

### `channels list`

List channels by IDs or handle.

```bash
youtube-data channels list --ids UCVHFbw7woebKtfvug_hvrig
youtube-data channels list --handle @Google --json
```

| Flag | Description |
|------|-------------|
| `--ids` | Comma-separated channel IDs |
| `--handle` | Channel handle (e.g. `@Google`) |
| `--parts` | Comma-separated parts (default: `snippet,statistics`) |
| `--max-results` | Results count (default: 25) |
| `--page-token` | Pagination token |

---

## playlists

### `playlists get <playlist-id>`

Get details of a playlist.

```bash
youtube-data playlists get PLbpi6ZahtOH6Ar_3GPy3ghEPnT8RU4fcf
youtube-data playlists get PLbpi6ZahtOH6Ar_3GPy3ghEPnT8RU4fcf --pretty
```

| Flag | Description |
|------|-------------|
| `--parts` | Comma-separated parts (default: `snippet,contentDetails,status`) |

### `playlists list`

List playlists for a channel or by IDs.

```bash
youtube-data playlists list --channel-id UCVHFbw7woebKtfvug_hvrig
youtube-data playlists list --channel-id UCVHFbw7woebKtfvug_hvrig --max-results 10
youtube-data playlists list --ids PLbpi6ZahtOH6Ar_3GPy3ghEPnT8RU4fcf
youtube-data playlists list --channel-id UCVHFbw7woebKtfvug_hvrig --json | jq '.[].id'
```

| Flag | Description |
|------|-------------|
| `--channel-id` | List playlists for this channel *(required if no --ids)* |
| `--ids` | Comma-separated playlist IDs |
| `--parts` | Comma-separated parts (default: `snippet,contentDetails,status`) |
| `--max-results` | Results count (default: 25) |
| `--page-token` | Pagination token |

---

## playlist-items

### `playlist-items list`

List all videos in a playlist.

```bash
youtube-data playlist-items list --playlist-id PLbpi6ZahtOH6Ar_3GPy3ghEPnT8RU4fcf
youtube-data playlist-items list --playlist-id PLbpi6ZahtOH6Ar_3GPy3ghEPnT8RU4fcf --max-results 50
youtube-data playlist-items list --playlist-id PLxxx --json | jq '.[].snippet.resourceId.videoId'
```

| Flag | Description |
|------|-------------|
| `--playlist-id` | Playlist ID *(required)* |
| `--video-id` | Filter by a specific video ID |
| `--parts` | Comma-separated parts (default: `snippet,contentDetails`) |
| `--max-results` | Results count (default: 50) |
| `--page-token` | Pagination token |

### `playlist-items add` (OAuth required)

Add a video to a playlist. Requires an OAuth token.

```bash
youtube-data playlist-items add --playlist-id PLxxx --video-id dQw4w9WgXcQ
youtube-data playlist-items add --playlist-id PLxxx --video-id dQw4w9WgXcQ --position 0
```

| Flag | Description |
|------|-------------|
| `--playlist-id` | Playlist ID *(required)* |
| `--video-id` | Video ID to add *(required)* |
| `--position` | Position in playlist (0-indexed) |

### `playlist-items delete <item-id>` (OAuth required)

Remove an item from a playlist. The item ID is the playlist item ID (not the video ID). Use `playlist-items list --json` to find item IDs.

```bash
youtube-data playlist-items list --playlist-id PLxxx --json | jq '.[0].id'
youtube-data playlist-items delete <item-id>
```

---

## comments

### `comments list`

List top-level comment threads for a video or channel.

```bash
youtube-data comments list --video-id dQw4w9WgXcQ
youtube-data comments list --video-id dQw4w9WgXcQ --order time --max-results 50
youtube-data comments list --video-id dQw4w9WgXcQ --json | jq '.[].snippet.topLevelComment.snippet.textDisplay'
```

| Flag | Description |
|------|-------------|
| `--video-id` | Comments for this video *(required if no --channel-id)* |
| `--channel-id` | All comments for a channel |
| `--order` | Sort: `time`, `relevance` (default: `time`) |
| `--parts` | Comma-separated parts (default: `snippet`) |
| `--max-results` | Results count (1–100, default: 20) |
| `--page-token` | Pagination token |

---

## Tips

- **Authentication**: All read commands only need `YOUTUBE_API_KEY`. Write commands (add/delete playlist items) additionally need `YOUTUBE_OAUTH_TOKEN`.
- **Get all uploads for a channel**: `youtube-data channels get <id> --parts contentDetails --json | jq '.contentDetails.relatedPlaylists.uploads'` → then use `playlist-items list --playlist-id <uploads-id>`.
- **Finding IDs**: use `--json | jq '.[].id'` to extract IDs for piping.
- **Pagination**: list commands print a `--page-token` hint when more results are available.
- **Output is auto-detected**: JSON when piped, tables in terminal. Use `--json` when parsing output.
- **Rate limits**: the YouTube Data API has a daily quota. Monitor usage at https://console.developers.google.com/.
- **Update**: run `youtube-data update` to pull the latest version from GitHub.
