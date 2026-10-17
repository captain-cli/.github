# Fluid Hifi Mini

Fluid Hifi Mini is an Electron + React desktop client for controlling the Crispy Disco / Fluid Hifi audio engine. It provides a compact mini-player, a jukebox-style browser view, playback controls, queue visibility, volume control, drag-to-seek, and a local file browser that supports regular folders and symbolic-link media folders.

The app is designed as a decoupled client: the UI does not own playback directly. It talks to the backend audio service over HTTP through the Electron main process and keeps the interface synchronized by polling the playback snapshot endpoint.

---

## Features

### Mini Mode

- Compact playback view
- Current artist, album, and track title display
- Play / pause / stop controls
- Next / previous track controls
- Repeat and shuffle controls
- Volume slider with drag support
- Track progress display
- Drag-to-seek progress bar
- Space bar shortcut for play / pause
- Frameless Electron window controls

### Jukebox Mode

- Larger playback and browsing layout
- Backend player queue display
- File browser / library view toggle
- Playback controls shared with mini mode
- Current track details and progress
- Styled jukebox scrollbars
- Designed to support future database-driven library APIs

### File Browser

- Browse local folders through Electron / Node.js
- Open media files through native dialogs
- Open playlist files
- Remember last opened directory
- Supports symbolic-link folders as directories
- Select files for playback or queueing
- Play Selected plays the latest selected audio file
- Queue Selected queues multiple selected audio files
- Double-click an audio file to play it immediately
- Double-click a directory to open it
- Clear/reset selections and imported playlists
- Audio and playlist file type detection

---

## Tech Stack

- Electron
- React
- Vite
- Node.js
- Lucide React
- Crispy Disco / Fluid Hifi C++ BASS audio backend

---

## Project Structure

```text
fluid-hifi-mini-preview/
├── electron/
│   ├── main.cjs
│   └── preload.cjs
├── src/
│   ├── App.jsx
│   ├── App.css
│   ├── components/
│   │   ├── ControlStrip.jsx
│   │   ├── FileBrowser.jsx
│   │   ├── JukeboxMode.jsx
│   │   ├── MiniMode.jsx
│   │   ├── PlaybackModes.jsx
│   │   ├── ProgressBar.jsx
│   │   ├── TrayPreview.jsx
│   │   └── WindowControls.jsx
│   ├── services/
│   │   └── crispyAudioClient.js
│   └── state/
│       └── playerStore.js
├── package.json
└── README.md
```

---

## Requirements

- Node.js
- npm
- Electron runtime dependencies for your OS
- Running Crispy Disco audio backend service
- Access to the backend audio HTTP port

The backend audio service must be running before the UI can control playback.

---

## Installation

```bash
npm install
```

Do not commit `node_modules/`. This repo should ignore dependency and build output folders.

---

## Development

Start the Electron/Vite development environment:

```bash
npm run electron:dev
```

If the Electron preload or main process changes, do a hard restart:

```bash
pkill -9 -f electron
pkill -9 -f vite
npm run electron:dev
```

---

## Configuration

The frontend audio client defaults to:

```text
http://<current-host>:8080
```

Override the backend audio URL with:

```bash
VITE_AUDIO_HTTP_BASE_URL=http://192.168.1.245:3005 npm run electron:dev
```

Use the correct host and port for the Crispy Disco audio backend.

---

## Electron Bridge

The renderer accesses backend functionality through:

```js
window.fluidHifi
```

Useful DevTools checks:

```js
window.fluidHifi
Object.keys(window.fluidHifi.audio)
await window.fluidHifi.audio.getPlaybackSnapshot()
```

Examples:

```js
await window.fluidHifi.audio.setVolume(40)
await window.fluidHifi.audio.seekPlayback(90)
await window.fluidHifi.audio.getPlayerQueue()
```

If a method is missing after editing `preload.cjs`, hard restart Electron.

---

## Playback Snapshot

The UI is synchronized from:

```http
GET /getPlaybackSnapshot
```

Expected response shape:

```json
{
  "state": "playing",
  "isPlaying": true,
  "isPaused": false,
  "isStopped": false,
  "bassActiveCode": 1,
  "artist": "Aaliyah",
  "album": "One in a Million",
  "title": "Hot Like Fire",
  "duration": 263.27,
  "position": 35.75,
  "remaining": 227.52,
  "progressPercent": 13.58,
  "volume": 25,
  "muted": false,
  "repeatState": false,
  "repeatStyle": 0,
  "repeatMode": "off",
  "shuffleState": false,
  "shuffleStyle": 0,
  "shuffleMode": "off",
  "playerQueueMode": 0,
  "playMode": "normal"
}
```

The UI treats this endpoint as the source of truth for playback state, track metadata, timer/progress, volume/mute, repeat mode, shuffle mode, and player mode.

---

## Command-Line Backend Testing

Replace host and port as needed.

### Snapshot

```bash
curl --max-time 2 -s http://192.168.1.245:3005/getPlaybackSnapshot | jq
```

### Play File

```bash
curl -i -X POST http://192.168.1.245:3005/playFile \
  -H "Content-Type: application/json" \
  -d '{"File":"/home/lahjeq/Music/Phil Collins/Phil Collins - Do You Remember.mp3","ResetQueue":true}'
```

### Pause

```bash
curl -i -X POST http://192.168.1.245:3005/pausePlayback \
  -H "Content-Type: application/json" \
  -d '{"Command":"PAUSE","PauseState":true}'
```

### Resume

```bash
curl -i -X POST http://192.168.1.245:3005/pausePlayback \
  -H "Content-Type: application/json" \
  -d '{"Command":"PAUSE","PauseState":false}'
```

### Stop

```bash
curl -i -X POST http://192.168.1.245:3005/stopPlayback \
  -H "Content-Type: application/json" \
  -d '{"Command":"STOP","PlaybackRunning":false}'
```

### Seek

```bash
curl -i -X POST http://192.168.1.245:3005/seekPlayback \
  -H "Content-Type: application/json" \
  -d '{"Command":"SEEKPLAYBACK","PositionSeconds":60}'
```

Verify seek:

```bash
curl --max-time 2 -s http://192.168.1.245:3005/getPlaybackSnapshot \
  | jq '.state, .position, .duration, .progressPercent'
```

### Set Volume

```bash
curl -i -X POST http://192.168.1.245:3005/setVolume \
  -H "Content-Type: application/json" \
  -d '{"Command":"SETVOLUME","Volume":40}'
```

### Mute / Unmute

```bash
curl -i -X POST http://192.168.1.245:3005/setMuteState \
  -H "Content-Type: application/json" \
  -d '{"Command":"MUTE","MuteState":true}'

curl -i -X POST http://192.168.1.245:3005/setMuteState \
  -H "Content-Type: application/json" \
  -d '{"Command":"MUTE","MuteState":false}'
```

### Repeat Modes

```bash
# Off
curl -i -X POST http://192.168.1.245:3005/setRepeatStyle \
  -H "Content-Type: application/json" \
  -d '{"Command":"REPEATSTYLE","RepeatStyle":0}'

# Repeat one
curl -i -X POST http://192.168.1.245:3005/setRepeatStyle \
  -H "Content-Type: application/json" \
  -d '{"Command":"REPEATSTYLE","RepeatStyle":1}'

# Repeat all
curl -i -X POST http://192.168.1.245:3005/setRepeatStyle \
  -H "Content-Type: application/json" \
  -d '{"Command":"REPEATSTYLE","RepeatStyle":2}'
```

### Shuffle Modes

```bash
# Off
curl -i -X POST http://192.168.1.245:3005/setShuffleStyle \
  -H "Content-Type: application/json" \
  -d '{"Command":"SHUFFLESTYLE","ShuffleStyle":0}'

# Legacy
curl -i -X POST http://192.168.1.245:3005/setShuffleStyle \
  -H "Content-Type: application/json" \
  -d '{"Command":"SHUFFLESTYLE","ShuffleStyle":1}'

# No repeat
curl -i -X POST http://192.168.1.245:3005/setShuffleStyle \
  -H "Content-Type: application/json" \
  -d '{"Command":"SHUFFLESTYLE","ShuffleStyle":2}'
```

### Queue

```bash
# Get queue
curl --max-time 2 -s http://192.168.1.245:3005/getTracksInPlayerQueue | jq

# Add one track
curl -i -X POST http://192.168.1.245:3005/addTrackToPlayerQueue \
  -H "Content-Type: application/json" \
  -d '{"Command":"ADDFILETOPLAYERQUEUE","FilePath":"/path/to/song.mp3"}'

# Add multiple tracks
curl -i -X POST http://192.168.1.245:3005/addTracksToPlayerQueue \
  -H "Content-Type: application/json" \
  -d '{"Command":"ADDTRACKSTOQUEUE","Tracks":["/path/to/song-1.mp3","/path/to/song-2.mp3"]}'

# Clear queue
curl -i -X POST http://192.168.1.245:3005/clearPlayerQueue \
  -H "Content-Type: application/json" \
  -d '{"Command":"CLEARPLAYERQUEUE"}'
```

### Next / Previous Track

```bash
curl -i -X POST http://192.168.1.245:3005/skipTrack \
  -H "Content-Type: application/json" \
  -d '{"Command":"SKIPTRACK"}'

curl -i -X POST http://192.168.1.245:3005/previousTrack \
  -H "Content-Type: application/json" \
  -d '{"Command":"PREVIOUSTRACK"}'
```

---

## Common Troubleshooting

### `window.fluidHifi.audio.someMethod is not a function`

The preload bridge is stale or missing a method.

```bash
pkill -9 -f electron
pkill -9 -f vite
npm run electron:dev
```

Then check:

```js
Object.keys(window.fluidHifi.audio)
```

### Progress bar moves but title does not update

Check snapshot metadata:

```bash
curl --max-time 2 -s http://192.168.1.245:3005/getPlaybackSnapshot | jq '.artist, .album, .title'
```

If metadata is correct in snapshot but wrong in UI, inspect reducer hydration in `playerStore.js`.

### Volume snaps back to zero

Check snapshot volume:

```bash
curl --max-time 2 -s http://192.168.1.245:3005/getPlaybackSnapshot | jq '.volume, .muted'
```

Volume should be a sane `0..100` number.

### Symlink folders show as files

Electron file browsing must use `fs.lstat()` plus `fs.stat()` for symbolic links:

```js
const linkStats = await fs.lstat(fullPath);

if (linkStats.isSymbolicLink()) {
  targetStats = await fs.stat(fullPath);
}
```

---

## Git Notes

The repository should ignore generated and dependency folders:

```text
node_modules/
dist/
build/
release/
.vite/
```

Useful checks:

```bash
git check-ignore -v node_modules/electron/dist/electron
git ls-files | grep '^node_modules/' | head
git status
```

---

## Future Direction

The next refactor target is the library API layer, not the audio command API layer.

Planned library API areas:

```text
GET    /libraries
GET    /libraries/:libraryId
GET    /libraries/:libraryId/artists
GET    /libraries/:libraryId/albums
GET    /libraries/:libraryId/tracks
GET    /libraries/:libraryId/items
POST   /libraries/:libraryId/scan
GET    /libraries/:libraryId/scan-status
GET    /library/tracks/:trackId/playback
GET    /library/albums/:albumId/playback
```

The goal is for the jukebox/database mode to consume library-aware media objects with:

```text
libraryId
mediaType
title/name
filePath
symbolicPath
duration
artist
album
scan status
```

---

## Current Stable Features

- Snapshot polling
- Timer/progress updates
- Drag-to-seek
- Play/pause synchronization
- Volume bridge
- Next/previous queue navigation
- File browser Play Selected
- Double-click file playback
- Space-bar play/pause
- Symlink directory browsing
- Clear/reset selections
