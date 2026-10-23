# Side project: the Mini deck

An old phone and a phone-sized keyboard, turned into a pocket terminal for the homelab. Built for fun out of things that were already in a drawer.

## What it is

- **Phone:** a Samsung Galaxy A03s, the oldest spare phone in the house. Factory reset, no accounts on it, stock Android left as it is.
- **Keyboard:** a Rii i4 mini Bluetooth keyboard with a touchpad, almost exactly the same size as the phone.
- The phone is locked to landscape and set to dark mode, so the pair reads as one small computer.

## What is on it

Everything comes from F-Droid, so the phone never needs an account:

- **Termux** for a real Linux terminal
- **Jellyfin** for the media server
- **WiFiAnalyzer** and **Port Authority** as a small network toolkit
- **NewTube** for video
- **Meshtastic** to talk to the LoRa home node (see below)
- A home screen shortcut to the homelab dashboard

No offline maps or offline encyclopedia on this one. Another device in the house already does that job, and this one has a different purpose.

## What it is for

- A pocket terminal and command line practice pad
- Checking the homelab from the sofa: the dashboard, a quick port scan, which Wi-Fi channel is crowded
- Media in bed
- Logging in to the homelab's servers over SSH, straight from Termux (installed with `pkg install openssh`)

## Three things worth knowing

- **The keyboard would not show up in Bluetooth at first.** It ships in 2.4G dongle mode (green light). It only appears on the phone after switching it to Bluetooth mode (blue light).
- **The keyboard layout is set on the phone, not on the keyboard.** Adding the matching language under the on-screen keyboard's input languages made the symbols land where the keycaps say.
- **The @ key would not type in Termux**, which gets in the way of `ssh user@host`. The same command without it: `ssh -l user host`.

## The LoRa radio

The Mini deck started with its own radio: a Seeed XIAO ESP32S3 with a Wio-SX1262 board, running Meshtastic. It pairs to the phone over Bluetooth, so the phone is the radio's screen and keyboard. It now lives on the homelab rack as an always-on home node (see below). Text messages with no mobile network and no internet.

- **Node name:** Minideck, short name MND
- **Region:** EU_868
- **Role:** Client
- **App:** Meshtastic from F-Droid, like everything else on the phone. No account needed.

### Putting it together

The kit is two small boards and two antennas. The order matters:

1. The small 2.4 GHz antenna snaps onto the XIAO first. Its connector is covered once the boards are joined.
2. The radio board presses onto the XIAO by the small board-to-board connector, and by nothing else. The XIAO's pin headers stay empty. They are only there for breadboard use.
3. The LoRa antenna snaps onto the radio board.
4. Only then does it get power. A LoRa radio should never be powered with no antenna attached.

### The one real problem: old firmware

The kit is sold as pre-flashed, and it is. But it shipped with Meshtastic 2.5.2, and the current phone app could not finish loading the radio's settings from firmware that old. The app found the radio, accepted the pairing code, then sat in a loop of connecting and reconnecting.

The fix was a firmware update from a laptop, all in the browser:

1. `client.meshtastic.org` over a USB cable showed the firmware version, which is how the cause was found.
2. `flasher.meshtastic.org`, device "Seeed XIAO S3", newest release in the stable group, full erase and install.
3. Unplug and replug once the flash finishes. The board does not always restart by itself.
4. Back in `client.meshtastic.org`: Settings, LoRa, set the region, save. The radio does not transmit until a region is set.
5. On the phone, remove the old Bluetooth pairing and pair again. The update gives the board a fresh identity.

Things that went wrong on the way, and what they meant:

- **"The port is already open"**: another browser tab still had the board, or the flash button got a second click.
- **"Failed to set control signals"**: the flasher restarted the board into install mode, the board dropped off USB for a moment, and the flasher lost it. Trying again got through. The fallback is holding the XIAO's B button while plugging it in, which on this kit means lifting the radio board off first, because it covers the button.
- **A USB-C to USB-C cable from the phone showed no device.** Not every cable carries data.
- **The stable firmware group is labelled "Beta" on the flasher.** That is the stable line. Alpha and nightly are the ones to leave alone.
- **After the update the light is solid green and no longer blinks.** That is normal on the new firmware.

### From pocket radio to home node

The first plan was a portable radio: power bank, keyboard, phone and radio as four parts that came apart. In practice a radio that is only switched on now and then hears almost nobody. Meshtastic nodes only announce themselves every so often, so the node list fills up over hours, not minutes.

So the radio moved to the rack and became an always-on home node. The radio does all the listening, relaying and remembering by itself. The phone is just a window into it, and the Mini deck is back to being a phone and a keyboard. A second, truly portable radio came later (see below), and the home node gives it something to reach back to.

How it sits on the rack:

- **Case:** a small clear plastic case, clipped to a front rack post. Plastic, not metal, because metal blocks the signal.
- **Antenna:** a short 868 MHz SMA antenna on a U.FL to SMA pigtail, standing up above the top of the frame. Metal right next to an antenna weakens it, so it pokes out above the rack rather than sitting beside the post. Plain SMA, not RP-SMA: they look the same and do not fit each other.
- **The other antenna stays.** The XIAO's small 2.4 GHz antenna is for Bluetooth and Wi-Fi. The 868 MHz antennas are the wrong band for it, so they only ever go on the radio board.
- **Boards:** a sticky foam pad between the two boards now holds them together.
- **Power:** USB only. It draws about as much as a phone charger on standby.

### Locking it down

A node that is always on can be reached by anyone in Bluetooth range, so two settings changed:

- **Bluetooth pairing code:** off the default `123456` and on to a fixed code of my own (Radio configuration, Bluetooth, Pairing mode: Fixed PIN). A phone that was already paired keeps working; any new device needs the new code.
- **No position sent:** the radio has no GPS, so any position it shares comes from the phone. The phone's location permission for the Meshtastic app is turned off, Smart position is off and GPS mode is set to Disabled. On the LongFast channel itself, Position enabled is switched off too, so nothing is shared even if a location source turns up later. A home node should not broadcast where home is.

### "No nodes around me": the two-radio test

After a night on the rack the node list still showed only the home node. Channel utilisation sat at 0.0% and test messages on LongFast came back "failed to deliver to mesh". It looked like broken hardware, so the usual suspects were checked first: the U.FL pigtail reseated on the radio board, region EU_868 and the LongFast preset confirmed, and a test on the windowsill with the antenna vertical. Still nothing.

What settled it was a second radio. The portable one arrived the same day: an M5Stack Cardputer Mesh Kit (Cardputer-Adv plus a Cap LoRa-1262 board with an SX1262 radio and GPS), which ships with Meshtastic already installed. With both radios across the room from each other, each saw the other and messages went through in both directions.

So both radios work. The home node is fine; nothing else is in range of a first-floor window in a narrow street of stone buildings. Lessons:

- **A second radio is the fastest hardware test.** If two nodes can talk to each other, neither is broken.
- **0.0% channel utilisation is not a fault.** With nobody in range, there is simply nothing to hear.
- **"Failed to deliver to mesh" on a public channel** only means no other node repeated the message back. With no neighbours, every message fails.
- **The map is not a test.** It only shows nodes that share their location, and many people turn that off.
- **Height beats everything.** LoRa wants a roof or a top floor and open sky, not a street-level window.

### The portable radio

The Cardputer Mesh Kit is now the radio that goes out of the house, so the home node can stay on the rack. It was set up from the same phone app over Bluetooth:

- Region EU_868, LongFast, role Client
- Bluetooth pairing on a fixed code of my own
- GPS off, and Position enabled off on the LongFast channel. This one has real GPS hardware, so it matters more here
- Uplink and downlink left off: it stays a radio-only node, so anything on its node list was actually heard over the air

### Seeing the wider mesh over the internet

The home node links to the global Meshtastic MQTT server through the phone's internet connection ("Proxy to client"), so the node keeps its Bluetooth. Uplink and downlink are on for the LongFast channel, and the connection tested as reachable. This shows the rest of the mesh even when no radio is in range, and lets the node appear on public maps. Nodes that arrive this way are marked as coming over MQTT, not heard by radio.

Map reporting is on, so the node publishes itself to those maps once an hour. The position it shares is a deliberately blurred fixed point in central Lisbon, a few kilometres wide, never the real address, since a home node should not broadcast where home is.

### Later, maybe

- Putting the node on home Wi-Fi instead of Bluetooth, so it can be watched like the rest of the homelab. On this board that turns Bluetooth off, and the network connection has no password, so it would stay on the home network and the VPN only, never open to the internet, with MQTT off.

## Planned

- **A custom clip** that joins the phone and the keyboard into one unit. It has to be hinged, come apart quickly, leave both USB-C ports free, and stay clear of the keyboard's top buttons, battery cover and dongle slot. Nothing ready-made fits, so it will be made by hand or printed. Not designed yet.
