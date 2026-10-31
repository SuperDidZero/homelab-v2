# Incidents

Every outage and odd failure so far, oldest first. Short on purpose: symptom, cause, fix, and what changed afterwards.

## Auto-updater crash-looping (September 2026)

- **Symptom:** the container auto-update tool on the DNS box kept restarting.
- **Cause:** its upstream project had been archived, and the Docker client bundled inside it was too old to talk to the current Docker engine. No setting could fix that.
- **Fix:** switched to an actively maintained fork with the same flags and the same socket mount.
- **Takeaway:** when a tool that used to work starts failing for no reason, check whether its project is still alive.

## Monitor reporting a healthy service as down (September 2026)

- **Symptom:** the uptime monitor showed the DNS box as down while it was clearly working.
- **Cause:** the monitor still pointed at the box's address on the previous network.
- **Fix:** corrected the monitor's target address.

## NAS stayed off after a power blip (2026-09-15)

- **Symptom:** a brief power interruption while a smart plug was being added in front of the UPS. The router, switch, DNS box and hypervisor rode through it or rebooted on their own. The NAS, the media server and the torrent client stayed down.
- **Cause:** the NAS had powered off completely and was waiting for someone to press its button. That is its default behaviour after losing power.
- **Fix:** turned on "auto power-on when power is supplied" in the NAS's power settings. Checked the RAID mirror afterwards: healthy, no damage.

## Media server down for one minute (2026-09-25)

- **Symptom:** one alert, HTTP 503 from the media server, recovered a minute later without anyone touching it.
- **Cause:** most likely the app restarting itself for an automatic update.
- **Fix:** none needed. Logged so that the next one-minute blip is recognised for what it is.

## Hypervisor dropped off the network (2026-09-30)

- **Symptom:** alerts that the hypervisor was unreachable, followed a minute later by every service on its VM. Everything not on that host stayed up. The machine was not hot. A short press of the power button shut it down cleanly and it came back within a minute. It had been streaming media for about two hours beforehand.
- **Cause:** the system log showed the onboard Intel network chip (`e1000e` driver) logging "Detected Hardware Unit Hang" every two seconds. The host kept running but could not talk to anything.
- **Fix:** turned off the TSO, GSO and GRO offload features on that interface with `ethtool`, then made it permanent with a `post-up` line in the interface config. Details in [proxmox.md](proxmox.md). Confirmed still in place after a later kernel update.
- **Also found:** the main VM was not set to start at boot, so it had to be started by hand after the restart. That is now on.

## Stale addresses left over from the old network (2026-10-03)

- **Symptom:** nothing visibly broken. Both were found while reading logs after the incident above.
- **Cause:** the hypervisor's own hosts entry and the Pi-hole's upstream DNS setting both still pointed at addresses from the previous network.
- **Fix:** corrected both in their web interfaces. The upstream now points at the local resolver on the same box.
- **Takeaway:** after changing subnets, search every device's settings for the old range. Things can keep half-working for weeks.

## Web server address conflict (2026-10-04)

- **Symptom:** the newly built web server container was sharing its address with something else.
- **Cause:** it had been given a hand-picked static address that another device on the network was already using.
- **Fix:** moved it to a free address handed out by DHCP with a reservation on the router, then updated the monitor and the dashboard tile. Logged as the first ticket in the help desk.
- **Takeaway:** check the address inventory before picking an address, every time.

## NAS reported "not connected to the internet" (2026-10-06)

- **Symptom:** the NAS's app store would not load, so the backup app could not be installed. Everything else on the NAS worked.
- **Cause:** name lookups from the NAS through the ad-blocking DNS were failing for the vendor's services, although the query log showed nothing blocked for that device.
- **Fix:** set the NAS's DNS to manual with public resolvers. It stays that way.

## Daily-driver hard freezes during map downloads (2026-10-31, root cause still open)

- **Symptom:** the daily-driver (Raspberry Pi 5, 8 GB) froze completely, then rebooted itself, several times in one afternoon. It always happened while the offline maps app (Organic Maps, desktop Flatpak) was downloading map regions. SSH stopped answering too, so the whole system hung, not just the desktop.
- **Ruled out, with how:**
  - **Power:** official Raspberry Pi power supply, monitor powered from the wall, `vcgencmd get_throttled` stayed at `0x0` while it ran.
  - **Heat:** 49 °C idle, about 60 °C under load. Throttling starts around 80 to 85 °C.
  - **Wi-Fi:** it froze on Wi-Fi and again on a wired Gigabit connection.
  - **SD card:** the Raspberry Pi Diagnostics card speed test passed.
  - **The system in general:** a one-hour full-image backup afterwards (heavy disk and network work, no maps app open) ran without a single freeze, and so did a 100-package system update.
- **What was seen:** nothing. The saved system log had no error before the restart, and kernel messages streamed live over SSH (`sudo dmesg -w` from another machine) printed nothing at the moment of the freeze. A hang that sudden, with no last message even over the network, means something low-level stopped the whole machine at once.
- **Trigger found:** it happens when the **status filter** in the maps app's download window is used (to see which region is downloading). Without touching the filter, all the downloads finished fine.
- **Root cause: not confirmed.** An app bug should only crash the app, not freeze the whole computer, so something below the app is failing. The suspects, in order:
  1. **Memory running out.** Filtering a list of thousands of map regions could use RAM faster than the system can recover, and a Pi with no memory left can lock up before it logs anything.
  2. **Graphics driver hang.** The filter redraws the whole list, and a GPU driver hang freezes the machine instantly with no message. The same day's system update brought new graphics drivers, and an earlier whole-world download on the older drivers never froze.
  3. **A kernel or firmware bug from the same update,** only hit by whatever the filter does. Less likely, since nothing else triggers it.
- **How to tell them apart (about 10 minutes):** over SSH from another machine, run `free -h -s 2` (memory every 2 seconds) and use the filter once. Memory draining to zero before the freeze points to 1. Memory flat and it still freezes points to 2 or 3, and the next step is a bug report to the Organic Maps project, with the system log from `journalctl -b -1` attached.
- **Changed afterwards:** the system log is now kept on disk. Raspberry Pi OS ships a setting that keeps it in memory only, so a crash used to wipe the evidence. A drop-in at `/etc/systemd/journald.conf.d/99-persistent.conf` with `Storage=persistent` overrides it, so after the next freeze `journalctl -b -1` shows the last moments before the restart.
- **Workaround until then:** don't use the filter in the download window.
