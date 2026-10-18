# Xbox 360 SmartGlass: what the console and games send

Documentation of the **Xbox 360** side of SmartGlass (2012-2016): how a game talks to the SmartGlass
phone/tablet app, and exactly which messages one game, **Forza Horizon** (title update 12), sends and
receives. Found by reverse engineering on a modded (RGH) console with the game running; every message
format below was read back from the game itself.

The Xbox One protocol is a different thing and is documented elsewhere (the OpenXbox project).
Nothing here covers it.

> Status: what is written here was checked on a real console unless it is marked *unknown* or
> *likely*. Addresses are for Forza Horizon title update 12 (title ID `4D5309C9`) only.

## Contents

| Page | What it is about |
| --- | --- |
| [How SmartGlass works on the 360](docs/architecture.md) | The pieces: the game's SmartGlass library, the connection, the encryption, why the old app can't connect any more |
| [Forza Horizon messages](docs/forza-horizon.md) | Every message the game sends and receives: what it is for, when and how often it is sent, every field, real examples |
| [Map entities and progress](docs/forza-map-entities.md) | The `mapentities` message in detail: the 7 kinds of map item and what their values mean (checked against the game's own statistics screen) |
| [Reading the messages on a modded console](docs/reading-on-rgh.md) | How to get these messages today without the phone app, the game's functions and addresses, and the traps (crashes) found on the way |
| [Dashboard](docs/dashboard.md) | What is known (little) about the system side |

## The messages at a glance (Forza Horizon)

Console → phone:

| Message | What it is for | When it is sent |
| --- | --- | --- |
| `gameState` | What the player is doing: free roam, race, challenge, rally, online; whether a route can be set | When one of its flags changes, or when the phone asks |
| `carpos` | The player's car: position and direction | Every 200 ms (the position itself is refreshed about every 0.6 s) |
| `mapentities` | Every map item with the player's progress: barn finds, discount signs, speed cameras, races, gas stations... | When the phone asks; 10 items per message |
| `navdata` | The game's GPS route, as a list of road nodes | When the route changes; 100 nodes per message |
| `navclear` | "There is no route" | When the route is removed |

Phone → console:

| Message | What it does |
| --- | --- |
| `sessioninfo` | The phone introduces itself: its sensors (location, gyroscope, accelerometer, vibration), browser, experience |
| `setroute` | Sets the game's GPS route to a point on the map (the game plans it along its roads) |
| `mapfilter` | Chooses which kinds of item the **game's own map** shows (12 filters) |
| `gameState` | Asks for the game state again |
| `mapentities` | Asks for the map items (the game then sends them page by page) |

## Words used here

- **Title update (TU)**: a game patch. Forza Horizon got its SmartGlass support in title update 12;
  the disc version alone has none.
- **RGH**: a hardware mod that lets the console run unsigned code; needed to look inside a running game.
- **XJSON**: Microsoft's small JSON reader/writer library in the Xbox 360 SDK; the game builds every
  message with it.

## License

The text of this documentation: CC BY 4.0. No game files, game text or Microsoft code are included.
