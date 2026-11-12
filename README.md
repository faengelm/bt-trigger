# BT Trigger

**Runs an iPhone Shortcut when a two-wire contact closes.**

BT Trigger is a dry-contact sensor for iPhone Shortcuts. Connect a switch, relay,
reed switch or pressure pad to a small ESP32 board. When the contact closes, the
board connects to your iPhone over Bluetooth, and a Shortcuts automation runs.

It works like a dry-contact sensor for Apple Home. The difference is where the
signal goes. A Home sensor reports open or closed to a home hub. BT Trigger talks
straight to the iPhone, so the Shortcuts app can do anything a Shortcut can:
send a message, play a sound, start a timer, run a Home scene, call a web service.

No home hub, no Wi-Fi, no app to install.

### [Set it up from your browser →](https://faengelm.github.io/bt-trigger/)

---

## How it works

1. **The contact closes.**
2. **The board connects** to your iPhone over Bluetooth.
3. **Your Shortcut runs**, from an automation that says "When BT Trigger 1 connects".

When the contact opens again, the board disconnects. A second automation
("When BT Trigger 1 disconnects") can react to that too.

To the iPhone, the board looks like a Bluetooth mouse that never moves. Connecting
is the signal: it never types, clicks or moves anything.

## What you need

- **An original ESP32 board**, such as an "ESP32 DevKit" or "ESP32-WROOM-32"
  (about $3–5). Newer chips (ESP32-S3, C3, C6) will not work: they lack the kind of
  Bluetooth that Shortcuts can use as a trigger.
- **A USB power supply** for the board, and a data USB cable for setup.
- **A passive contact**: a switch, relay, reed switch, push button or pressure pad
  with two wires.
- **An iPhone** with the Shortcuts app.
- **A computer with Chrome or Edge**, once, to install the firmware.

## Setting it up

1. **Install.** Plug the board into your computer, open the
   [installer page](https://faengelm.github.io/bt-trigger/) and click **Connect**.
   Leave the input pin on **4**, check the name (for example "BT Trigger 1"), then
   click **Install**.
2. **Pair.** Power the board.
   - **iPhone:** go to **Settings → Accessibility → Touch → AssistiveTouch**, turn it
     on, then open **Devices → Bluetooth Devices** and tap your board's name.
     iPhones only list Bluetooth mice there.
   - **iPad:** skip AssistiveTouch. Open **Settings → Bluetooth** and tap your
     board's name.
3. **Automate.** In **Shortcuts → Automation → +**, choose **Bluetooth**, pick your
   board, choose **Is Connected**, then **Run Immediately**, and add your actions.
4. **Connect the contact** (below), close it, and your Shortcut runs.

## Connecting a contact

Connect the contact's two wires to **D4** and **GND** on the board. Either wire can
go to either pin.

```
   ESP32 board
  ┌────────────┐
  │  D4        ├─────────┐
  │            │         ╱   switch, relay, reed
  │  GND       ├─────────┘   or pressure pad
  └────────────┘
```

- **Passive contacts only.** Do not connect anything that supplies a voltage. The
  board's pins take 3.3 V at most, and more can damage it.
- **No extra parts needed.** The board's internal resistor holds the pin up until
  the contact closes.
- **Bouncy or noisy contacts:** raise **Debounce** in the installer so short
  glitches are ignored.
- **Quick open-close events** (like a door that swings shut): set **Keep connected
  after release** to a few seconds, so the iPhone has time to connect.

## What the light means

The board's blue LED shows what it is doing:

| Light | Meaning |
|---|---|
| **Off** | Normal. The contact is open and the board is waiting. |
| **Fast blinking** | No iPhone is paired yet. It is ready to pair. |
| **A short flash every 1.5 seconds** | The contact is closed and the board is looking for the iPhone. |
| **Steady on** | Connected. Your automation has run. |
| **Very fast flicker** | The BOOT button was held for 5 seconds and the board is forgetting its phones. |

## Using several boards

Give each board its own name. The installer suggests **BT Trigger 1**, then
**BT Trigger 2**, and so on, counting in the browser you use for setup. Each name
gets its own automation in Shortcuts, so each contact can run a different Shortcut.

## Changing settings later

Plug the board into the computer and open the installer again. It reads the
board's current settings. Change what you need and click **Save settings only**:
the iPhone stays paired.

To start over, hold the board's **BOOT** button for 5 seconds, or reinstall with
**Forget paired phones** ticked. Then remove the old entry on the iPhone and pair
again.

## Troubleshooting

- **The board doesn't appear on the iPhone:** look under AssistiveTouch →
  Devices → Bluetooth Devices, not Settings → Bluetooth. (An iPad lists it in
  Settings → Bluetooth.) Turn Bluetooth off on other
  iPhones or iPads that were paired with it before.
- **Closing the contact does nothing:** check the wires are on D4 and GND, and that
  the installer shows input pin 4. Watch the light: it should start flashing when the
  contact closes.
- **The Shortcut runs but asks first:** edit the automation and choose
  **Run Immediately**.
- **See what the board is doing:** the installer's **Show live device log** button
  prints its messages.

## For developers

The firmware uses ESP-IDF 5.3.1 through PlatformIO. Build it with:

```bash
pio run -e esp32
```

ESP-IDF won't build from a folder whose path contains a space.

```bash
python scripts/make_release.py
```

That builds the merged image and manifest in `firmware/`, which the installer serves.

| Path | What it holds |
|---|---|
| `src/main.c` | Reading the contact, BOOT-button reset, status light |
| `src/transport_classic.c` | The Bluetooth mouse and reconnecting to the iPhone |
| `src/config.c` | Reads the settings written by the installer |
| `docs/` | The installer website (GitHub Pages) |
| `firmware/` | The ready-to-flash image the installer uses |
| `test/host/` | Checks that the installer and firmware agree on the settings format |

## Credits and license

Based on [esp32-bt-trigger](https://github.com/Mattboxx/esp32-bt-trigger) by
Mattboxx. This version adds iPhone pairing through AssistiveTouch, a fresh
Bluetooth address whenever the board forgets its phones (so an old iPad can't grab
it back), no stray connection at power-up in contact mode, and numbered names for
several boards.

MIT license, see [LICENSE](LICENSE). The installer includes
[esptool-js](https://github.com/espressif/esptool-js) (Apache-2.0).
Not affiliated with Apple or Espressif.
