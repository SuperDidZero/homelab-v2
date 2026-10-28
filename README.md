# Homelab v2

A documented self-hosted homelab build: virtualization, a NAS, network-wide ad-blocking with redundant DNS, monitoring, a personal cloud, a help desk, remote access and backups. All of it runs on consumer-grade secondhand and retail hardware, no enterprise gear.

Built and documented by SuperDidZero (PC Build PT). This is the second iteration of the homelab. The first, `personal-hardware-lab`, is archived. This one is written up properly so the choices, the mistakes and the fixes are all somewhere other people can actually use them.

All IP addresses, hostnames, API keys and passwords in these docs are **placeholders**. Swap them for your own network's values; nothing here will work by copy-pasting the addresses as-is.

## What's actually in here

| Doc | Covers |
|---|---|
| [docs/hardware.md](docs/hardware.md) | The physical build: rack, hypervisor host, NAS, Pi-hole box, UPS, and why each thing was picked |
| [docs/network.md](docs/network.md) | Topology, DNS/ad-blocking with Pi-hole (primary + secondary), remote access over Tailscale |
| [docs/proxmox.md](docs/proxmox.md) | The Proxmox VE host: what runs where, and the two hardware/config gotchas that actually caused outages |
| [docs/firewall-lab.md](docs/firewall-lab.md) | An OPNsense firewall VM with its own lab network, and a first rule that keeps the lab out of the home network |
| [docs/web-server.md](docs/web-server.md) | A small nginx container, and what it ended up being useful for |
| [docs/nextcloud.md](docs/nextcloud.md) | The personal cloud: Nextcloud in Docker with its data on the NAS |
| [docs/immich.md](docs/immich.md) | The photo library: Immich in Docker, photos on the NAS, database kept off the network share, phone backing up over Wi-Fi |
| [docs/help-desk-and-remote-support.md](docs/help-desk-and-remote-support.md) | A self-hosted ticketing system, and remote control of a Windows laptop from inside and outside the house |
| [docs/wireshark-investigation.md](docs/wireshark-investigation.md) | A packet capture of the home network: what HTTPS still gives away, a NAS asking the router to open ports, and a device map built from traffic alone |
| [docs/backups.md](docs/backups.md) | What gets backed up, how, how often, and how restores are actually tested |
| [docs/dashboard-and-monitoring.md](docs/dashboard-and-monitoring.md) | Homepage dashboard + Uptime Kuma + alerting, so there's one screen that shows if anything's down |
| [docs/mini-deck.md](docs/mini-deck.md) | Side project: an old phone and a tiny keyboard as a pocket terminal for the homelab, plus the LoRa radio that became the rack's Meshtastic home node |
| [docs/ai-operator.md](docs/ai-operator.md) | Planned: one small container that coordinates several cloud AI agents, with a task queue, a log and human approval before anything happens |
| [docs/incidents.md](docs/incidents.md) | Every outage and odd failure so far, in date order: symptom, cause, fix |
| [docs/build-log.md](docs/build-log.md) | The build day by day, from the first shelf to the current state |
| [docs/lessons-learned.md](docs/lessons-learned.md) | What got ruled out and why, and the principles that guided the build |
| [photos/](photos/README.md) | Pictures of the real build |

## Rough shape of the thing

```mermaid
flowchart LR
  ISP[ISP router<br/>bridge mode] --> RTR[Main router<br/>DHCP + QoS]
  RTR --> SW[Switch]
  SW --> HV[Hypervisor host<br/>Proxmox VE]
  SW --> NAS[NAS<br/>RAID 1]
  SW --> DNS1[Pi-hole #1<br/>+ Unbound]
  HV --> VM1[VM: media + cloud + help desk<br/>Jellyfin, qBittorrent, Nextcloud, GLPI, Immich]
  HV --> CT1[Container: web server]
  HV --> CT2[Container: Pi-hole #2<br/>secondary DNS]
  HV -. backups .-> NAS
  DNS1 -. nightly backup .-> NAS
  NAS -. nightly .-> USB[USB stick<br/>NAS settings + small backups]
  VPN([Mesh VPN<br/>subnet router]) -.-> HV
```

## Why this is public

The point isn't that anyone needs this exact stack. Most homelab write-ups either stop at "I installed Proxmox" or only show the finished, working version. This one keeps the actual failures in: a NIC that silently hung under load, a dashboard that refused outside connections until one environment variable was added, a container update tool that broke because its upstream project was abandoned, a NAS that claimed it had no internet. Those are the parts that are actually useful to someone hitting the same wall.

## Still open

Written down on purpose, so the repo doesn't read as more finished than it is:

- There is no full second copy of the NAS outside the NAS. Media and the machine backups exist in one place only.
- VLANs on the firewall lab and a home automation VM are planned and not started.
- The AI operator is designed and not built. Its page describes the plan, not a working system.

## Related

- [command-console](https://github.com/SuperDidZero/command-console): a sci-fi dashboard UI built as a front end for a personal daily brief.
- [powershell-toolkit](https://github.com/SuperDidZero/powershell-toolkit): the small Windows health, hardening-check and cleanup scripts used on the laptop mentioned in the remote support doc.
- [esp32-mp3-player](https://github.com/SuperDidZero/esp32-mp3-player): the DIY electronics build that lives next to this homelab.

## License

MIT. Do whatever you want with it.
