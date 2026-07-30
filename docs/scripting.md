# Scripting interface (`hey` / MCP)

Brube2000 exposes a native **BeOS scripting suite**, so other tools can read the
player's state and drive playback. Haiku's built-in `hey` command talks to it
directly, and because the [Pippo](https://github.com/atomozero/Pippo) MCP server
controls apps through `hey`, an MCP-compatible LLM can control Brube2000 too.

- **Application signature:** `application/x-vnd.brube2000`
- **Suite:** `suite/vnd.brube2000-player`

## Authorization

Remote control is **off by default**, and split into two independent **channels**
so a person at the Terminal and a programmatic client can be trusted differently.
Set both in **App → Account & Settings → General**:

- **hey** — scripting from Haiku's `hey` command-line tool.
- **MCP** — everything else (the Pippo server, another app, an LLM).

The channel is decided from the sender's executable (which an app can't spoof):
messages from the `hey` binary use the *hey* level, all others use the *MCP*
level. Each channel has three levels (persisted to
`~/config/settings/Brube2000/remotecontrol-hey` and `…-mcp`):

| Level | `get` (read) | `set` / `do` (control) |
|-------|:---:|:---:|
| **Off** (default) | ✗ | ✗ |
| **Read-only** | ✓ | ✗ |
| **Full** | ✓ | ✓ |

A request that isn't allowed returns `B_NOT_ALLOWED` naming the channel and the
level needed, e.g.:

```
$ hey application/x-vnd.brube2000 get State
Brube2000 hey remote control is off — enable it in Account & Settings → General. (error 0x8000000f)
```

## Properties

| Property | Type | `get` | `set` | Notes |
|----------|------|:---:|:---:|-------|
| `State` | string | ✓ | | `idle` / `logging-in` / `loading` / `playing` / `paused` / `ended` / `error` |
| `Title` | string | ✓ | | current track title (empty when nothing is playing) |
| `Artist` | string | ✓ | | current track artist |
| `Album` | string | ✓ | | current track album |
| `Volume` | float | ✓ | ✓ | `0.0`–`1.0` (drives the same mixer channel as the slider) |
| `Position` | double | ✓ | ✓ | seconds; setting it seeks |
| `Duration` | double | ✓ | | seconds |
| `Repeat` | bool | ✓ | ✓ | |
| `Shuffle` | bool | ✓ | ✓ | |
| `Track` | string | | ✓ | set to a URI (`spotify:track:…` / `tidal:track:…`) to load and play it |

Read values come back in the reply's `result` field.

## Commands (`do`)

| Command | Effect |
|---------|--------|
| `Play` | resume if paused, otherwise (re)start the current queue entry |
| `Pause` | pause if playing |
| `Stop` | stop |
| `Next` | skip to the next track |
| `Previous` | skip to the previous track |

## Examples with `hey`

```sh
# Discover what the app supports
hey application/x-vnd.brube2000 getsuites

# Read state
hey application/x-vnd.brube2000 get State
hey application/x-vnd.brube2000 get Title
hey application/x-vnd.brube2000 get Artist
hey application/x-vnd.brube2000 get Position

# Control (needs the "Full" level)
hey application/x-vnd.brube2000 set Volume to 0.4
hey application/x-vnd.brube2000 set Shuffle to true
hey application/x-vnd.brube2000 do Pause
hey application/x-vnd.brube2000 do Next

# Load and play a specific track
hey application/x-vnd.brube2000 set Track to "spotify:track:1301WleyT98MSxVHPZCA6M"
```

A successful `get` prints a `B_REPLY` with the value:

```
$ hey application/x-vnd.brube2000 get Volume
Reply BMessage(B_REPLY):
   "result" (B_FLOAT_TYPE) : 0.400
```

A successful `set` / `do` replies with `error = 0`.

## MCP / LLM usage (Pippo)

[Pippo](https://github.com/atomozero/Pippo) is a native MCP server for Haiku that
turns `hey`-scriptable apps into LLM tools. With Brube2000 set to **Full** and
Pippo running, an MCP client can:

1. call `getsuites` to discover the `suite/vnd.brube2000-player` properties, then
2. issue `get` / `set` / `do` requests,

so prompts like *"pause the music"*, *"skip this track"* or *"turn it down to
30%"* map onto the commands above. Nothing is exposed until you raise the level
in Settings, so remote control is always opt-in.

## Notes

- Playing a track (`do Play` on a stopped queue, or `set Track`) still requires
  being signed in to the backend; otherwise the player goes to the `error`
  state, it does not play.
- The scripting handler runs on the application thread and calls the thread-safe
  `Player` API directly, so requests never block the UI.
