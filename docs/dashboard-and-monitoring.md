# Dashboard and monitoring

## The dashboard

A self-hosted dashboard app (e.g. Homepage) runs on the DNS box and acts as the single home page for the whole homelab: one tile per service, each with a live status dot and, where the underlying app exposes an API, a live number (library size, active downloads, blocked-query count, that kind of thing).

Setup notes that actually mattered:

- **Host validation**: newer versions of some self-hosted dashboards require you to explicitly list which host *and port* combinations are allowed to load the page (not just the bare IP). If the dashboard loads fine from `localhost` on the box itself but refuses connections from anywhere else on the network with a vague "invalid host" error, this is almost always it.
- Per-service API keys/tokens (for a media server, a hypervisor's own API, etc.) are created specifically for the dashboard integration, scoped to read-only where the app supports it, rather than reusing an admin account's main credentials.
- The Nextcloud widget signs in with Nextcloud's serverinfo token (the `key` field), not a username and password. Placeholder values:

  ```yaml
  widget:
    type: nextcloud
    url: http://YOUR_NEXTCLOUD_ADDRESS
    key: YOUR_SERVERINFO_TOKEN
  ```

- A kiosk-mode tablet mounted near the rack, pointed permanently at the dashboard URL, makes "is everything okay" a glance instead of opening five different admin pages.

## Monitoring and alerting

- A self-hosted uptime monitor (e.g. Uptime Kuma) runs alongside the dashboard and checks every service on a schedule, independent of the dashboard's own live-status tiles, so there's a monitor watching things even when nobody's looking at the dashboard.
- It is Uptime Kuma v2 on the rack's Raspberry Pi, migrated from v1 on 2026-11-25 with a backup of the v1 data kept. The image is pinned to an exact version instead of `latest`, and it is started with `docker run`:

  ```bash
  docker run -d --restart=always -p 3001:3001 -v uptime-kuma:/app/data --name uptime-kuma louislam/uptime-kuma:2.5.12
  ```

- The same monitor also watches the business's public website from the outside, over HTTPS, and warns when its certificate is close to expiring. It is the one check that looks at something outside the house, and it sits on the dashboard as its own tile.
- Alerts are wired to a messaging app (a dedicated bot in a chat app works well) so an outage produces an actual notification, not just a red dot waiting to be noticed.
- **Lesson learned the hard way**: a container-auto-update tool can silently start failing if its upstream project goes unmaintained and its bundled dependencies (in this case, an old bundled Docker client) stop working against current versions of the thing it's supposed to update. If a previously-working auto-updater starts crash-looping for no obvious reason, check whether its upstream project is still active before assuming it's a local config problem, switching to an actively maintained fork fixed it outright here, no config change needed.
- When a monitor for a service that moved to a new address keeps reporting it as down, check the monitor's own target address, it's easy for a monitoring tool to keep pointing at wherever a service used to live after you re-IP something.

## Keeping it honest

Every status page here reflects what's actually running, not what was planned. A few integrations (friend-facing upload links, some automation ideas) are intentionally left as "not built yet" rather than implied as done, worth the same discipline in your own write-up.
