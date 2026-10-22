# Hardware

## Rack

- Open-frame 2-post desktop rack, 8U, plus four shallow (8"/203mm) shelves.
- **Lesson:** an enclosed cabinet rack almost got bought first. Shelf depth is what actually governs whether your gear fits, the rack frame itself barely matters. An open-frame design also means there's no real "floor," so anything deep (a tower UPS, for instance) has to sit on the ground underneath the rack rather than on a shelf.
- Shelf layout, top to bottom: router + NAS on top (so the NAS's height has no shelf above it) → switch + DNS box + hypervisor host together → two shelves of headroom plus a rack-mount power strip. Nothing else is planned for it; it's sized to exactly what's running.

## Hypervisor host

- A small-form-factor "tiny" business PC (Lenovo ThinkCentre M720q class: 6-core i5, 16GB RAM, 512GB SSD, integrated GPU, Wi-Fi/Bluetooth), bought used.
- Runs Proxmox VE. See [proxmox.md](proxmox.md) for what's on it and a BIOS gotcha that blocks VMs from starting at all on this class of machine.
- Picked over several other tiny/SFF candidates (HP EliteDesk 800 G3, Lenovo M73/M715q/M700/M900, HP ProDesk 400 G6, a Chromebox) mainly on specs-per-euro and known Proxmox compatibility.

## NAS

- 2-bay NAS (UGREEN NASync DH2300 class: 4GB RAM, 8-core ARM, vendor NAS OS), two 3TB drives in RAID 1 (mirror), ~2.7TB usable.
- Picked specifically because its software stack supports what was needed (SMB/CIFS, a usable storage manager) without also needing a container runtime on the NAS itself, the container workloads live on the Proxmox host instead, the NAS is pure storage.
- **Lesson:** after a power interruption, the NAS fully powered itself off and stayed off, while every other device on the network rode through the blip and came back on its own. NAS vendors often default to "stay off until a human presses the button" as a safety behavior. Turn on "auto power-on when power is restored" (usually under Control Panel → Hardware/Power) the day you set it up, not after the first outage finds it for you.

## DNS / ad-blocking box

- A small single-board computer (Raspberry Pi class, 2GB RAM) running Pi-hole + Unbound, at capacity and deliberately not expanded further, it's a DNS appliance, not a general-purpose box.
- A second, independent Pi-hole instance runs as a container on the Proxmox host as a secondary DNS server, loaded from the first one's exported blocklist/config. If the primary box is down or being rebooted, DNS for the whole house doesn't go with it.

## UPS

- A line-interactive UPS (~600VA/360W class), open-bottom tower design.
- Measured steady-state draw for the whole rack (router, switch, DNS box, hypervisor, NAS) is well under 10% of the UPS's rated capacity, healthy headroom, no load concerns.
- Sits on the floor underneath the rack, not on a shelf (see the rack note above on shelf depth).

## Power benchmark

A smart plug with energy monitoring sits between the wall and the UPS, so it measures everything the UPS feeds. Reading on 22 October 2026, mid-afternoon:

| Measure | Value |
|---|---|
| Draw right now | 34 W |
| Energy since midnight (14.1 h) | 0.474 kWh |
| Average over those 14.1 h | about 34 W |
| Voltage | 232.6 V |
| Current | 0.194 A |

What was running: router, switch, DNS box, hypervisor with its usual VMs and containers, NAS, the dashboard tablet and the LoRa home node. The lab firewall VM was on for part of the day for the repair bench test.

The average matters more than the live number, because it smooths out spikes. At about 34 W the rack uses roughly 0.8 kWh a day, 25 kWh a month and 300 kWh a year. Compared with the about 30 W measured in September, the extra devices since then added only a few watts.

Worth re-reading this after any hardware change, as a before and after.

## What's deliberately *not* in this repo's scope

A separate "daily driver" single-board computer, a DIY electronics build, and an Arduino learning track all live alongside this homelab in real life, but they're independent projects with their own goals and aren't part of this write-up.
