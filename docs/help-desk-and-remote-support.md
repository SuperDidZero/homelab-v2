# Help desk and remote support

Two pieces that turn the homelab from "servers at home" into something closer to a small IT setup.

## Help desk (GLPI)

- GLPI runs as a Docker Compose stack (the app plus a MySQL database) on the same VM as the media server and the personal cloud, on its own port.
- The compose file pins exact image versions for both the app and the database, not `latest`. Pulling `latest` for a security fix would have dragged in a new major version along with it. Updating means changing those two tags on purpose, after reading the release notes.
- First job after install: sign in, change the admin password, and disable the default accounts it ships with. A fresh GLPI comes with several well-known logins, and leaving them active is the classic mistake.
- Monitored and on the dashboard.
- The first ticket was a real one: the web server's address conflict from [incidents.md](incidents.md), opened, solved and closed.

What it is for: the uptime monitor already records *when* something went down and came back, automatically. It does not record why, or what was done about it. A ticket holds the problem, the steps taken and the solution, so the same fault three weeks later takes two minutes instead of an afternoon.

In practice the homelab's own incidents now live in [incidents.md](incidents.md), and the ticket system is used for customer jobs from the PC build and repair business the homelab belongs to.

## Remote support

The goal: sit at one machine and fix another, first inside the house and then from anywhere.

- **RustDesk** is installed on the Windows laptop and on the Linux daily-driver. The machine being controlled shows an ID and a one-time password, and the other machine connects with those. Nothing to configure on the router.
- **The mesh VPN** is installed on the laptop as well, so it is reachable from outside the house without opening any ports. See [network.md](network.md) for how the VPN is laid out.

## Hardening the Windows laptop first

The laptop was tidied up before it was made remotely reachable:

- A restore point before touching anything.
- All Windows Security protections on, updates installed, UAC on.
- No extra user accounts, unused apps removed, startup trimmed, sign-in required on wake, Find my device on.

The checks were then turned into three small PowerShell scripts (a health report, a hardening check and a temp cleanup), which live in their own repo: [powershell-toolkit](https://github.com/superdidotwitch-jpg/powershell-toolkit).
