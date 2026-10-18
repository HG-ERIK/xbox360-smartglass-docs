# How SmartGlass works on the Xbox 360

[Back to the contents](../README.md)

## The short version

A SmartGlass-enabled game carries its own small SmartGlass library. The library opens a connection to
the phone, and the game sends and receives **JSON messages** through it. The phone app showed a web
page (the game's "companion experience") that drew whatever the game sent: in Forza Horizon, a map
of Colorado with the player's car and progress.

```
 game code  ──builds JSON──▶  SmartGlass library (xbc.lib, inside the game)  ──encrypted──▶  phone app
 (one object per message)        TCP on the home network, or a cloud relay                 (web page)
```

## The library inside the game

The library is linked into the game itself (not the system). In Forza Horizon its class names are
still in the program: `CXbcImpl`, `CXLrcImpl`, `CXLrcListener`, `CXLrcClient`, `CXLrcTcpTransport`,
`CXLrcCloudTransport`. What they show:

| Part | What it does |
| --- | --- |
| `CXLrcListener` | Listens for the phone on the local network (TCP) |
| `CXLrcTcpTransport` | A direct connection to one phone on the same network |
| `CXLrcCloudTransport` | The same through Microsoft's cloud, when phone and console are not on one network |
| `CXLrcClient` | One connected phone (the SDK allows up to 4 at a time) |
| `CXbcImpl` | The part the game uses: start, send a JSON message, get events |

What the game's code uses (names as in the Xbox 360 SDK):

| Function | What it does |
| --- | --- |
| `XbcInitialize(callback, state)` | Starts the library; the game gets its events through the callback |
| `XbcSendJSON(delivery, client, writer, state)` | Sends one JSON message (built with XJSON) to one phone or to all |
| `XbcDoWork()` | Keeps the library going (called every frame) |

The events the game gets: a phone connected, a phone disconnected, a message was sent, a message
arrived, accelerometer / gyroscope / touch data arrived. A message is at most 1 KB, which is why big
lists (map items, the route) come in pages.

## Encryption, and why the old app can't connect any more

The messages are **encrypted**: the transport works out each message's encrypted length before
sending (`GetEncryptMessageLength` / `GetDecryptMessageLength` in the library). The keys come from
pairing the phone with the console, which went through Microsoft's SmartGlass service (the console's
Xbox Live side, `XamLrc*`). That service has been switched off, so:

- the original phone app can no longer pair with an Xbox 360, and
- listening on the network gives only encrypted bytes.

The messages themselves still exist inside the game: the game builds every one of them in plain JSON
before the library encrypts it. That is where they are read today (see
[Reading the messages on a modded console](reading-on-rgh.md)).

## How a game builds and sends its messages (Forza Horizon)

Forza Horizon has a `SmartGlassComponent` with one class per message:

| Class | Message key | Direction |
| --- | --- | --- |
| `CSmartGlassSessionInfo` | `sessioninfo` | phone → console |
| `CSmartGlassGameState` | `gameState` | both (sent; the phone can ask for it) |
| `CSmartGlassActivities` | `mapentities` | both (sent on request) |
| `CSmartGlassMapFilter` | `mapfilter` | phone → console |
| `CSmartGlassMapRoute` | `navdata` / `navclear`, `setroute` | both |
| `CSmartGlassCar` | `carpos` | console → phone |

Every class has the same shape: a method that says whether it has something new, a method that
writes its JSON, and a method that reads a message from the phone. The game keeps a queue: for each
queued message it makes a fresh XJSON writer, lets the message object write itself into it, hands the
writer to `XbcSendJSON`, and closes it.

Messages from the phone go to every object; each one checks whether the message's key is its own
(`"mapentities"`, `"setroute"`, ...) and acts on it.

## JSON shape

Every message is one object with one key, the message name:

```json
{"gameState":{"initialized":true,"freeRoam":true,"inRace":false, ...}}
```

Lists are JSON arrays, numbers are written in full (e.g. `-2356.79833984375`), and text is UTF-16
inside the game (plain ASCII in practice).
