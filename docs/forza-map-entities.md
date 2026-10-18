# Forza Horizon: map entities and progress

[Back to the contents](../README.md) · [All messages](forza-horizon.md)

The `mapentities` message lists every item on the map with the player's progress. One item:

```
"name": [ shown, "Type", value, ... ]
```

- **shown** (first value, every type): the item is on the player's map, i.e. its area has been
  discovered (or, for a barn find, its rumour heard). `false` = the phone app did not show it.
- **Type**: one of the seven types below.
- **value(s)**: depend on the type.

The meanings were checked against the game's own statistics screen ("Discoveries") on a save where
almost everything was done:

| Game statistics screen | From `mapentities` |
| --- | --- |
| Barn finds found: **7 / 9** | 7 BarnFind items with value `true` |
| Discount signs smashed: **89 / 100** | 89 Flyer items with value `true` |
| Speed traps set off: **8 / 9** | 8 AverageSpeed items with a record above 0 |

## The types

| Type | Count | Values | Meaning |
| --- | --- | --- | --- |
| `BarnFind` | 9 | `found` (bool) | The rare car in this barn has been found |
| `Flyer` | 100 | `smashed` (bool) | The discount sign has been smashed (smashed signs disappear from the game's map) |
| `SpeedCamera` | 22 | `record` (number) | The player's best speed through this camera; 0 = not set off yet |
| `AverageSpeed` | 9 | `record` (number) | The player's best average speed through this zone; 0 = not done yet |
| `Events` | 88 | 4 values, see below | Races, showcases, nemesis races, plane races |
| `GasStation` | 10 | a number | Gas stations (fast travel points). The number was 10000 for all; *likely* the fast travel price |
| `RaceHub` | 3 | a number | The street race hubs. Values seen: 0, 10, 5 (*unknown*) |

The record units (km/h or mph) are *unknown*.

### `Events`

```
"FR02": [true, "Events", false, 1, 0, false]
```

| Position | Meaning |
| --- | --- |
| 1 | shown |
| 2 | `"Events"` |
| 3 | a flag; mostly `false` (*unknown*, possibly "new") |
| 4 | **best finishing place**: `1` = won, `8` = finished 8th, `-1` = not raced yet. The game's map draws a won race with a gold cup |
| 5 | a number, mostly 0 (one event had 250) (*unknown*) |
| 6 | a flag, mostly `false` (*unknown*) |

## Names

Item names are the game's own ids:

| Type | Names | Where the position is |
| --- | --- | --- |
| BarnFind | `barnfind_01` ... `barnfind_09` | the game's `barnfinds.xml` pairs each number with its barn's trigger object and hint circle |
| Flyer | `flyer_001` ... `flyer_100` | the game's world objects |
| SpeedCamera | `speed_camera_NN` | the game's world objects |
| AverageSpeed | `average_speed_NN` | the game's world objects |
| Events | `FRnn` (festival races), `NRnn` (nemesis), `EXHIBITION_*`, `PLANE_RACE_*`, `HEADLINE_EVENT` | the game's world objects; the display name comes from the game database's `Events` table (its `HorizonEventID`), through the game's text table |
| GasStation | `GasStation_01` ... `_10` | the game's world objects (written there as `GASSTATION_001` ...) |
| RaceHub | `eventhub001` ... `003` | |

The message carries **no positions**; the app had its own copy of where everything is.

Race type: circuit or sprint is not in the message either. In the game database, a race with more
than one lap (`Races.NumLaps` > 1) is a circuit.
