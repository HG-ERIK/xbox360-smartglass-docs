# Reading the messages on a modded console

[Back to the contents](../README.md)

The phone app can no longer pair, and the network traffic is encrypted, but the game still builds
every message in plain JSON. On an RGH console with a system plugin (e.g. loaded by DashLaunch), the
messages can be produced on demand without any phone:

1. Make **your own copy of a message object** with the game's own constructor.
2. Let the game's own code **update** it (where the message has an update step).
3. Make an XJSON writer with the game's own function and let the object's **write** method fill it.
4. Read the text out of the writer, close the writer.

Nothing in the game is changed and no game data is read by pointer chains: everything is computed by
the game's own SmartGlass code, the same way it was for the phone.

## The game's functions (Forza Horizon, title update 12)

Title ID `4D5309C9`. These addresses are only valid for this version; check the code before calling
anything (the first words of each function and the method tables below).

### XJSON writer

| Address | Function |
| --- | --- |
| `0x82A6E000` | new writer (returns its handle) |
| `0x82A6DFB8` | close writer |
| `0x82A6E038` | begin object |
| `0x82A6E068` | end object |
| `0x82A6E340` | begin field (name), for an object value |
| `0x82A6E0B0` | begin field (name), for a plain value (*likely*) |
| `0x82A6E098` | end field |
| `0x82A6E050` | begin array |
| `0x82A6E080` | end array |
| `0x82A6E0D0` | write number (a double) |
| `0x82A6E0E8` | write bool |
| `0x82A6E360` | write a whole bool field (name + value) |

The writer keeps its text as UTF-16 at **writer + 8**, the length in characters at **writer + 0xC**.

### SmartGlass library

| Address | Function |
| --- | --- |
| `0x82A68770` | `XbcInitialize` (the game passes its event callback `0x824C7418`) |
| `0x82A68788` | `XbcSendJSON` |
| `0x824C6D98` | the game's send loop (queue → new writer → write → `XbcSendJSON` → close) |

### Message classes

Method table slots: 1 = has something new / update, 2 = send everything again, 5 = write JSON,
6 = read a message from the phone.

| Message | Constructor | Method table | Size | Update (slot 1) | Write (slot 5) |
| --- | --- | --- | --- | --- | --- |
| `gameState` | `0x824C9048` | `0x820064D8` | 0x3C | `0x824C9630` (changed?) | `0x824C9750` |
| `mapentities` | `0x824C7628` | `0x8200630C` | 0x38 | `0x824C76C8` | `0x824C8418` |
| `carpos` | `0x824C85A8` | `0x82006374` | 0xA0 | `0x824C8A08` | `0x824C8670` |
| `navdata` | `0x824C9A08` | `0x820065D4` | 0xA8 | `0x824C9D10` | `0x824C9D60` |
| `mapfilter` | `0x824CA628` | `0x82006740` | 0x2DC | | read: `0x824CAB70` |
| `sessioninfo` | `0x824CADD0` | `0x82006848` | 0x2C | | read: `0x824CB068` |

Per message:

- **`gameState`**: construct, write. One message.
- **`mapentities`**: construct, then set: entries per page at +0x08 (the game uses 10; any number
  works), "asked for" byte at +0x10 = 1, next entry at +0x14 = 0. Write repeatedly; the game clears
  the "asked for" byte after the last page.
- **`carpos`**: keep one object for the whole game session (it has a timer). Set its send interval at
  +0x90 to 0, call the update (slot 1), write.
- **`navdata`**: keep one object. Call slot 2 ("send everything again"), then slot 1 (looks at the active
  GPS route), then write until pages sent (+0x38) reaches the page count (+0x34). With no route, the
  write gives `navclear`.
- **`setroute`**: the game's handler (`0x824CA368`) plans a route with the navigation planner
  (`0x82D9E548(planner, target vector)` → route id) and makes it active (`0x82A54068(gps, route)`) if the
  GPS (`0x82A4DDA0`) has a different one. The planner and GPS objects come from the game's object table
  (`0x8269B4F8`) at the slots stored in `0x833EDC44` and `0x833EDC08`.

## Traps found on the way

These each crashed or froze the console, or silently gave wrong results:

| Trap | What happened | Fix |
| --- | --- | --- |
| **Vector stores** | The car's update writes the position with VMX vector stores, which always land on a 16-byte boundary. On a 4-byte aligned copy of the object the numbers landed beside their fields: the position read as 0 | Align message objects to 16 bytes |
| **Vector code needs a game thread** | The game's code uses VMX; a system thread has no VMX state | Call the game's code from a short-lived thread of the game's own process |
| **Handles are per process** | An event made by the plugin's system thread meant nothing to the game's thread: its "done" signal never arrived, every wait timed out | Signal with a shared variable instead |
| **Hardware registers** | While the game loads its pointers are garbage; reading one that pointed at `0x7F000000`-`0x81FFFFFF` (GPU registers, kernel) froze the whole console | Don't follow pointers into the game at all (or never into that range) |
| **The game's start** (*suspected*, not proven) | The console froze in the system's storage dialog during Forza's cloud save sync while the plugin made Live and profile calls (friends list, profile, avatar). It never froze without the plugin | Leave a starting game alone (the first minute) and ask for profile data only on the dashboard |
| **Skipping the intro** | Pressing A over and over while Forza syncs its cloud save at start froze the console, with or without any plugin | Let the intro and sync finish |

## What the game does not send

- **Speed**: work it out from two `carpos` positions that differ (the position repeats in between).
- **Positions of map items**: the app had its own copy; take them from the game's files.
- **Item names**: the game's text tables (`media\stringtables\<language>.zip`, e.g. `Events.str`).
  Their format: `LSB2` header, a count, then key (16-bit) + offset (32-bit) entries, then UTF-16
  strings. The key is the low 16 bits of the text id the game database uses (e.g. `_&202727060`).
