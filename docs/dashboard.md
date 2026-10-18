# The dashboard (system) side

[Back to the contents](../README.md)

Besides the games, the console itself took part in SmartGlass: pairing a phone, and the app's
console-wide features (remote control with the phone as a controller pad, typing with the phone's
keyboard, what is playing, launching a game). That part lives in the system (`xam.xex` and the
dashboard), not in games.

**Not documented yet.** What is known so far:

| What | Known |
| --- | --- |
| Pairing and the connection keys | Done through the console's Xbox Live side (`XamLrc*` functions); Microsoft's SmartGlass service is switched off, so pairing no longer works |
| Remote control and keyboard | Handled by the system, not by games; how the phone's input reaches the running app is *unknown* |
| The dashboard's SmartGlass promo tile | The home screen's right promo tile showed a picture (a QR code) downloaded from `cms-assets.xboxservices.com`; that download now fails |

Planned: the system's own SmartGlass features (remote control and keyboard) as the console implemented
them, so a replacement app can use them the standard way.
