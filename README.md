# BT Trigger

**Runs an iPhone Shortcut when a two-wire contact closes.**

BT Trigger is a dry-contact sensor for iPhone Shortcuts. Plug a switch, relay,
reed switch or pressure pad into its 3.5mm socket. When the contact closes, it
connects to your iPhone over Bluetooth, and a Shortcuts automation runs.

It works like a dry-contact sensor for Apple Home, but instead of reporting to a
home hub it talks straight to the iPhone. The Shortcuts app can then do anything a
Shortcut can: send a message, play a sound, start a timer, run a Home scene, call
a web service. No hub, no Wi-Fi, no app to install.

Based on [esp32-bt-trigger](https://github.com/Mattboxx/esp32-bt-trigger) by
Mattboxx. See [Credits](#credits-and-license).

---

## Bought a BT Trigger?

It arrives programmed and ready to use. The
**[user guide](https://faengelm.github.io/bt-trigger/)** walks through it. In short:

1. **Power it** from any USB power adapter. Its name is on the label, for example
   "BT Trigger 1".
2. **Pair it once.**
   - **iPhone:** Settings → Accessibility → Touch → AssistiveTouch, turn it on,
     then Devices → Bluetooth Devices, and tap the name.
   - **iPad:** Settings → Bluetooth, and tap the name.
3. **Make the automation:** Shortcuts → Automation → + → Bluetooth → your BT
   Trigger → Is Connected → Run Immediately → add your actions.
4. **Plug your contact** into the 3.5mm socket and close it.

Only plug in **passive contacts**: no headphones, audio cables, or anything that
supplies a voltage.

---

## Build your own

Experienced DIY builders can program and wire their own unit. The
**[DIY page](https://faengelm.github.io/bt-trigger/diy.html)** has a browser-based
installer, so you don't need to install any tools.

### Parts

- **An original ESP32 board**, such as an "ESP32 DevKit" or "ESP32-WROOM-32"
  (about $3–5). ESP32-S3, C3 and C6 won't work: they only have Bluetooth LE, and
  Shortcuts can't use those accessories as triggers.
- **A 3.5mm mono socket** for the contact.
- Recommended: **a 1 kΩ resistor and a 100 nF capacitor** to protect the input.
- A data USB cable, and Chrome or Edge on a computer for programming.

### Wiring

```
  D4  ──┬──[ 1 kΩ ]──── jack tip
        │
      100 nF
        │
  GND ──┴──────────────── jack sleeve
```

The firmware reads D4 with the ESP32's internal pull-up, and treats LOW as closed,
so the contact only has to connect D4 to GND. The resistor limits the current if
someone plugs in something that supplies a voltage. The capacitor filters noise
on long leads. For a quick bench test you can skip both and use a jumper wire.

### Programming

1. Plug the board into your computer and open the
   [DIY page](https://faengelm.github.io/bt-trigger/diy.html).
2. Click **Connect** and choose the board's port.
3. Check the Bluetooth name. The page suggests **BT Trigger 1**, then **BT Trigger 2**
   for the next board, counting in that browser.
4. Click **Install firmware + settings**.

Then pair and automate as in [Bought a BT Trigger?](#bought-a-bt-trigger) above.

### Status light

The board's blue LED (GPIO2) shows what it is doing:

| Light | Meaning |
|---|---|
| Off | Normal. The contact is open and the board is waiting. |
| Fast blinking | No iPhone is paired yet. It is ready to pair. |
| A short flash every 1.5 s | The contact is closed and it is looking for the iPhone. |
| Steady on | Connected. The automation has run. |
| Very fast flicker | BOOT is being held: it is forgetting its phones. |

### Reset

Once the board is running, hold **BOOT** for 5 seconds. (Holding it while the
board powers up starts the ESP32's programming mode instead.) The board forgets
its phones and picks a new Bluetooth address, so devices that paired before can't
grab it back. You can also reinstall from the DIY page with **Forget paired phones**
ticked. Afterwards, remove the old entry on each iPhone or iPad and pair again.

### Building from source

The firmware uses ESP-IDF 5.3.1 through PlatformIO:

```bash
pio run -e esp32
```

```bash
python scripts/make_release.py
```

The first command builds the firmware. The second makes the merged image and
manifest in `firmware/`, which the DIY page serves. ESP-IDF won't build from a
folder whose path contains a space.

| Path | What it holds |
|---|---|
| `src/main.c` | Reading the contact, BOOT-button reset, status light |
| `src/transport_classic.c` | The Bluetooth mouse and reconnecting to the iPhone |
| `src/config.c` | Reads the settings written by the installer |
| `docs/index.html` | The user guide (GitHub Pages) |
| `docs/diy.html` | The DIY page with the browser installer |
| `firmware/` | The ready-to-flash image the installer uses |
| `test/host/` | Checks that the installer and firmware agree on the settings format |

---

## Credits and license

BT Trigger is based on
**[esp32-bt-trigger](https://github.com/Mattboxx/esp32-bt-trigger) by Mattboxx**,
which came up with the core idea: an ESP32 that poses as a Bluetooth mouse, so that
connecting to the iPhone triggers a Shortcuts automation. Thank you.

This version changes:

- **Focus:** it's a dry-contact sensor, with a 3.5mm socket on D4 and GND.
- **iPhone pairing** goes through AssistiveTouch.
- **Forgetting phones** gives the board a new Bluetooth address, so an old iPad
  can't re-pair it.
- **No stray connection** at power-up.
- **Numbered names** for several units.

MIT license, see [LICENSE](LICENSE). The DIY installer includes
[esptool-js](https://github.com/espressif/esptool-js) (Apache-2.0).
Not affiliated with Apple or Espressif. iPhone and Shortcuts are trademarks of
Apple Inc.
