# Network

## Topology

```
ISP ONT/router (bridge mode) → main router (DHCP, QoS, DNS handed out to clients) → unmanaged switch → everything else
```

The ISP-provided router is kept only because some ISPs don't allow it to be fully removed from the line; putting it in bridge mode on one LAN port hands all real routing, DHCP and firewalling to a normal consumer router behind it. If your ISP lets you skip this step entirely, do that instead, it's one less device to manage.

## DNS and ad-blocking

- Primary DNS: Pi-hole + Unbound on its own small box (see [hardware.md](hardware.md)). Unbound is a local recursive resolver, so DNS lookups aren't sent to a third-party DNS provider at all once it's warmed up.
- Secondary DNS: a second Pi-hole instance, running as a lightweight container on the hypervisor host, loaded from the first instance's exported configuration (Pi-hole calls this "Teleporter"). The router hands out both addresses to DHCP clients, primary first, secondary as fallback, so a reboot or maintenance window on the primary box doesn't take down name resolution for the house.
- A maintained public blocklist (e.g. one of the HaGeZi lists) is loaded in addition to Pi-hole's defaults. Expect the "percentage blocked" number to jump a lot once a real list is loaded, that's normal, not a sign something's broken.
- **Gotcha:** if a Pi-hole instance's own upstream DNS setting still points at an old network's address after you re-IP anything, it will silently keep failing lookups. Check Settings → DNS → upstream servers any time you change your subnet.

## Remote access

- A mesh VPN (Tailscale or similar) is used for remote access rather than opening ports on the router. One node runs on the hypervisor host as a subnet router for the whole LAN (so your own authenticated devices can reach anything on the home network from anywhere); a second, separate node runs on just the one VM that hosts the media/cloud services, with an access policy that only exposes that single port to anyone you explicitly invite.
- This two-tier setup means your own devices get full access, while a friend you share something with (e.g. a media server) only ever reaches that one service, never the rest of the home network.
- Nothing on the router opens a way in from the internet: UPnP is off on the main router (an AX10, since 2026-11-25), there are no port forwards and DMZ is off. Remote access is through Tailscale only, with the subnet router on the Proxmox host.
- The AX10's own VPN servers are disabled too: PPTP, L2TP and WireGuard were turned off, and OpenVPN was already off. The exposure check finished on 2026-11-25: nothing faces the internet, and Tailscale is the only remote route.

## QoS

- Most consumer routers have a QoS page that lets you prioritize specific devices by MAC address. Worth doing for anything latency-sensitive (a gaming PC, a console) sharing the connection with large transfers (torrents, backups, a media server transcoding).

## Address planning

- Keep a running list of every static IP and MAC address reservation you hand out, and check it before adding a new one. It's easy to accidentally hand a new device the same address an existing smart-home gadget already has, which causes confusing intermittent failures on whichever device loses the fight.
- Example addressing scheme used in this build (replace with your own range): router at `.1`, DHCP pool `.2`–`.253`, with everything that needs a stable address (hypervisor host, NAS, DNS boxes, any server containers/VMs) given a DHCP *reservation* rather than a hardcoded static IP, so it still shows up correctly in the router's own client list.
