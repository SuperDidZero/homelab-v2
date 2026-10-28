# Build log

The build day by day. Dates are 2026. Only the homelab itself is in here.

**Before this log starts:** the ISP router was already in bridge mode behind the main router, the switch was in place, Proxmox was installed on the mini PC, and the DNS box was running Pi-hole, Unbound, the dashboard, the uptime monitor and the auto-updater.

## September

- **08:** Rack shelves arrived.
- **09:** NAS and drives arrived. Drives installed, RAID 1 mirror created, address reserved on the router. Alerting wired to a chat bot. Uptime monitor extended to cover the dashboard, itself, the hypervisor, the router and the NAS. Dashboard filled in with real services instead of the example content. Old tablet set up as a fullscreen dashboard display.
- **10:** RAID sync finished. Linux VM created on the hypervisor. Media server installed, NAS share mounted on the VM, libraries created.
- **11:** Torrent client installed as a system service, saving straight to the NAS.
- **12:** Rack frame arrived. Rack built, cabled and working.
- **15:** Smart plug with energy monitoring put in front of the UPS: the whole rack draws about 30 W. The swap caused a power blip and the NAS stayed off. Auto power-on enabled.
- **21:** Packet capture exercise on the home network (TLS handshakes, service discovery broadcasts, ARP). Written up on the [Wireshark page](wireshark-investigation.md).
- **28:** Full health check, all healthy. QoS set on the router at about 90% of the measured line speed, with priority for the gaming and streaming devices. A larger blocklist added to Pi-hole: blocked queries went from about 3% to about 35%.
- **30:** Hypervisor dropped off the network. Network chip offload fix applied and made permanent. Main VM set to start at boot.

## October

- **03:** Update round across the hypervisor, NAS, router, DNS box and VM. Two stale addresses from the old network found and fixed.
- **04:** The big day.
  - Mesh VPN set up with the hypervisor as subnet router, tested from a phone on mobile data.
  - Media server shared with friends: their own non-admin accounts, and a VPN share that reaches that one port only.
  - Second Pi-hole built in a container, loaded from the first one's export, handed out by the router as secondary DNS.
  - Nightly backup of the DNS box to the NAS.
  - Web server container built. Address conflict found and fixed.
  - Personal cloud built with its data on the NAS. Phone app connected.
  - Dashboard customised: title, theme, logo, background, live numbers on the tiles.
  - Proxmox backups to the NAS, first restore test passed on a throwaway copy, weekly job created for every machine.
  - Router settings exported into the personal cloud.
  - Help desk installed, default accounts disabled, first ticket closed.
- **05:** This repo published. The old `personal-hardware-lab` repo archived.
- **06:** First full weekly backup job confirmed: all four machines present. Nightly DNS box backup confirmed running on its own for three nights. Backup gaps closed: hypervisor host settings, the daily-driver's home folder, and a USB stick on the NAS for the NAS's own settings.
- **07:** Restore test of the DNS box's nightly backup passed: a throwaway uptime monitor started from the backup showed all 12 monitors. Done over SSH from the Mini deck, which also updated the main VM. Windows laptop hardened. PowerShell toolkit written and published. Remote control set up between the laptop and the daily-driver, and the laptop added to the mesh VPN. Firewall lab built: OPNsense VM, a lab network that exists only inside the hypervisor, and a first rule that lets the lab reach the internet but not the home network. Repo brought up to date: backups rewritten, new docs for the personal cloud, the web server, the help desk and remote support, the incidents log and this build log.
- **08:** LoRa radio for the Mini deck assembled and set up: Meshtastic, region EU_868, paired to the phone. It shipped with firmware too old for the current app, so it was updated from a laptop in the browser first. Details on the [Mini deck page](mini-deck.md). The public website added to the uptime monitor, with a certificate expiry warning, and given a tile on the dashboard. The [AI operator](ai-operator.md) designed: one small container to coordinate several cloud AI agents, cut down from a first plan that wanted a whole cluster. Not built yet.
- **22:** The LoRa radio moved from the Mini deck to the rack as an always-on Meshtastic home node: new 868 MHz antenna on a pigtail standing above the frame, small plastic case on a front post. Bluetooth pairing code changed from the default and position sharing turned off. USB network adapter plugged into the hypervisor for the planned repair bench network. Details on the [Mini deck page](mini-deck.md).
- **22 (afternoon):** Repair bench network live: the USB Ethernet adapter is now the lab bridge's port, so a machine plugged into it gets internet through the firewall and cannot reach the home network. Tested from a laptop. Details on the [firewall lab page](firewall-lab.md).
- **22 (later):** Power benchmark taken from the smart plug in front of the UPS: about 34 W average for the whole rack. Recorded on the [hardware page](hardware.md).
- **22 (evening):** NAS backup drive upgraded from the 3.7 GB stick to a 64 GB drive, now copying every backup folder plus the personal cloud. NAS settings export redone into the personal cloud. Details on the [backups page](backups.md).
- **23:** The Meshtastic home node heard no other nodes. A second radio (Cardputer Mesh Kit, now the portable one) proved both radios work: the location is the problem, not the hardware. Home node confirmed working on the global MQTT server (map reporting on, blurred position), position sharing off on the portable radio. The portable Cardputer also got a multi-firmware launcher so it can switch between Meshtastic and other tools from its own menu. Details on the [Mini deck page](mini-deck.md).
- **27:** Security round for the October PHP flaws (CVE-2026-109618 and CVE-2026-109619, fixed in PHP 8.5.12). Found PHP in three places: the personal cloud, the help desk and the lab firewall's web page. VM backed up first, then the personal cloud moved to the patched image, the help desk pinned to its current version's patched image (pulling `latest` would have jumped a major version), and the firewall updated to OPNsense 26.7.7 from its console. Each one confirmed on PHP 8.5.12 from inside the running container or VM. Both Pi-holes are v6, which has no PHP. NAS and router firmware already current. Same session: the main VM's Ubuntu updates (including Docker) installed and the VM rebooted, and both Pi-holes updated to Core 6.5 / FTL 6.8 / Web 6.6.1.
- **24-26:** Meshtastic home node moved onto home Wi-Fi so it publishes without the phone, pointed at the official global MQTT server, fixed position corrected, and the Lisbon regional channel added. Details on the [Mini deck page](mini-deck.md).
- **28:** Mini deck clip finished: two phone clamps joined by a GoPro-style hinge, with a rubber band spacer so the phone folds like a laptop screen. The rack's dual USB charger for the LoRa node and dashboard tablet is in place. Details on the [Mini deck page](mini-deck.md).
- **28 (afternoon):** Photo library set up: Immich in Docker on the main VM, photos in a folder on the NAS through the existing media mount, database on the VM's own disk, a random database password, storage template by year and month. Phone app backing up the camera roll over Wi-Fi only. Then given an uptime monitor on its health endpoint and a dashboard tile with live counts from a read-only API key. Done over SSH from the laptop. Details on the [Immich page](immich.md).
