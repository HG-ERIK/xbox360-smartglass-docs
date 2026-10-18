# Forza Horizon: every SmartGlass message

[Back to the contents](../README.md)

Forza Horizon (2012), title update 12. The phone app drew a live map of Colorado: the car, the GPS
route, and every race, barn find, sign and camera the player had found. All of it came from these
messages. Examples are real answers from the game.

- [Console → phone](#console--phone): [`gameState`](#gamestate) · [`carpos`](#carpos) ·
  [`mapentities`](#mapentities) · [`navdata` / `navclear`](#navdata--navclear)
- [Phone → console](#phone--console): [`sessioninfo`](#sessioninfo) · [`setroute`](#setroute) ·
  [`mapfilter`](#mapfilter) · [asking again](#asking-for-a-message-again)
- [Timing summary](#timing-summary)

---

## Console → phone

### `gameState`

**What it is for:** tells the app what the player is doing, so it can show or hide things (for example,
"set route" only works in free roam).

**When:** whenever one of the flags changes (the game keeps the last values it sent and compares), and
when the phone asks for it.

```json
{"gameState":{"initialized":true,"initialExperience":false,"freeRoam":true,"inRace":false,
  "inChallenge":false,"canSetRoute":true,"canBuyMap":true,"multiplayer":false,"isRally":false}}
```

| Field | Meaning |
| --- | --- |
| `initialized` | The game's world is loaded (false on menus and loading screens) |
| `initialExperience` | The game's opening drive at the start of a new save (*likely*) |
| `freeRoam` | Driving freely in the open world |
| `inRace` | In a race |
| `inChallenge` | In a challenge (*likely*: showcases, free roam challenges) |
| `canSetRoute` | The GPS can be set now (what the phone's "set route" relies on) |
| `canBuyMap` | The treasure map (shows all barn finds etc.) can be bought (*likely*) |
| `multiplayer` | Online |
| `isRally` | In the rally part of the game (the Rally Expansion) (*likely*) |

### `carpos`

**What it is for:** the moving car on the phone's map.

**When:** every 200 ms (the message's own interval). The position inside it is refreshed by the game
only about every **0.6 s**; in between the same position is sent again. A smooth marker has to glide
between changes (and a speed worked out from two messages that are the same is 0, not the real speed).

```json
{"carpos":{"pos":[-2356.79833984375,8.64318752288818,-2587.36840820313],
           "dir":[-0.690914452075958,-0.0233793072402477,0.722605109214783]}}
```

| Field | Meaning |
| --- | --- |
| `pos` | Position in metres: x (east), y (height), z (**north**) |
| `dir` | The direction the car faces, a unit vector in the same axes. Heading clockwise from north: `atan2(dir.x, dir.z)` |

There is no speed in the message; it has to be worked out from the change in position.

### `mapentities`

**What it is for:** every item on the map, with the player's progress (found, smashed, won, records).
This is how the app knew what to show: undiscovered items stay hidden, smashed signs disappear.

**When:** only when the phone asks for it (by sending `mapentities`). The game then sends all items,
**10 per message**, until the list is done. In a full save there are 143 items.

```json
{"mapentities":{"barnfind_01":[true,"BarnFind",true],
                "speed_camera_01":[true,"SpeedCamera",96.0063171386719],
                "FR02":[true,"Events",false,1,0,false],
                "GasStation_04":[true,"GasStation",10000]}}
```

Each item: `name: [shown on the map, type, value...]`. All types and values are described in
[Map entities and progress](forza-map-entities.md).

### `navdata` / `navclear`

**What it is for:** the game's own GPS route, so the phone draws exactly the line the game's GPS
shows (not a route of its own).

**When:** when the route changes (set on the console or by the phone, or planned again after the car
left it). The route comes as road nodes, **100 per message**:

```json
{"navdata":{"routeid":12,"totalnodes":424,"startindex":0,"pagenodes":100,"pageindex":0,"numpages":5,
            "nodes":[8603,5364,2849,2702, ...]}}
```

| Field | Meaning |
| --- | --- |
| `routeid` | The route's number; a new number = a new route (also after the game plans again) |
| `totalnodes` | How many nodes the whole route has |
| `numpages` | How many `navdata` messages carry it |
| `pageindex` | Which one this is (0, 1, ...) |
| `startindex` | The first node's position in the route (`pageindex` × 100) |
| `pagenodes` | Nodes in this message (100, the last one fewer) |
| `nodes` | Road node **ids** from the game's road network file (`media\tracks\colorado\colorado.nav`); each id has a position there. Nodes are about 20 m apart |

The nodes run from one end of the route to the other; the end away from the car is the destination.

When there is no route, the game sends `{"navclear": ...}` instead.

---

## Phone → console

### `sessioninfo`

**What it is for:** the phone introduces itself when it connects.

```json
{"sessioninfo":{"experience":"...","browserInfo":"...",
  "deviceInfo":{"hasHaptic":true,"hasLocation":true,"hasGyroscope":true,"hasAccelerometer":true}}}
```

The game reads `experience`, `browserInfo` and the four `deviceInfo` flags and keeps them per phone.
What it then does with them is *unknown*.

### `setroute`

**What it is for:** "set route" on the phone. The game plans a route along its roads to the point and
makes it its GPS route, exactly as if it was chosen on the game's own map. The new route then comes
back as `navdata`.

The point is a map position in metres (x east, z north); the game snaps it to the nearest road.

### `mapfilter`

**What it is for:** the phone's filter buttons. They change what the **game's own map** on the TV
shows. The message is a list of filter names; each named filter is switched on, the rest off, and
`All` switches on everything:

| Filter | Shows |
| --- | --- |
| `All` | everything |
| `New` | new items (*likely* not seen yet) |
| `NotWon` | events not won |
| `Rivals` | rivals |
| `FestivalEvents` | festival events |
| `Showcases` | showcase events |
| `NemesisRaces` | nemesis races |
| `Outposts` | outposts (*likely* the gas stations, which are the fast travel points) |
| `StreetRaceHubs` | street race hubs |
| `SpeedCameras` | speed cameras |
| `Flyers` | discount signs |
| `BarnFinds` | barn finds |

### Asking for a message again

Sending a message's own name (`"gameState"`, `"mapentities"`) makes the game send it again. For
`mapentities` this is the only way to get it.

---

## Timing summary

| Message | Sent | Size |
| --- | --- | --- |
| `gameState` | on change / on request | one message |
| `carpos` | every 200 ms (position changes about every 0.6 s) | one message |
| `mapentities` | on request | 10 items per message (143 items: 15 messages) |
| `navdata` | on route change | 100 nodes per message (a 424-node route: 5 messages) |
| `navclear` | when the route is removed | one message |
