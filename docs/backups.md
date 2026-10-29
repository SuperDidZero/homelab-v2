# Backups

## The model

Everything backs up to the NAS, once. A USB drive plugged into the NAS covers the NAS's own settings and the smallest, most important backups. Since October 2026 the NAS also sends a nightly copy of everything to a small rented storage box in a data centre in Germany, so a fire or theft at home no longer takes the backups with it. That copy includes the weekly machine backups too, so the earlier plan for a bigger drive kept at a friend's house was dropped.

## What gets backed up

| What | How | When | Kept |
|---|---|---|---|
| Every VM and container on the hypervisor | Proxmox backup job to the NAS over SMB/CIFS (a Proxmox storage target with content type "Backup") | Weekly, Sunday 01:00 | Last 3 |
| The hypervisor host's own settings | Small nightly job on the host, copied to the NAS | Nightly, 02:30 | Recent copies |
| DNS box (Pi-hole export plus the app data for the dashboard, the uptime monitor and Unbound) | Cron job on the box: Pi-hole Teleporter export plus a tar of the app data, copied to the NAS | Nightly, 03:30 | Newest 14 |
| Daily-driver single-board computer (home folder) | Copy to the NAS. An hourly check runs it only when the machine is on the home network | Daily, when at home | Newest 7 |
| The personal cloud plus every backup folder above | NAS backup task to a 64 GB USB drive that stays plugged into the NAS | Nightly, 04:30 | Incremental |
| Everything on the NAS (all shared and personal folders, about 215 GB) | NAS backup task over rsync, tunnelled through SSH, to a 1 TB Hetzner Storage Box in Germany. See [Off-site copy](#off-site-copy) | Nightly, 05:30 | 30 versions, plus 10 daily snapshots on the box |
| Router configuration | Manual export of the router's settings file, uploaded to the personal cloud | After router changes | Latest |

Notes on a few of these:

- The Proxmox job's selection mode is "All" on purpose, so a newly created VM or container is included automatically instead of being silently left out until someone remembers to add it.
- The uptime monitor is paused for a few seconds while the DNS box backup runs, so its database is copied in a consistent state.
- The times are staggered so that each job's output already exists when the next one picks it up: machines, then host settings, then the DNS box, then the USB drive, then the off-site copy.
- The router export and the NAS settings export are manual snapshots. They go stale the moment something changes, so they get redone after any change to the router or the NAS.

## The USB drive, and what did not fit

The first version used the only spare drive available: a 3.7 GB stick that used to be the Proxmox installer. It only had room for the personal cloud, the DNS box backups and the host settings.

In October 2026 it was replaced with a 64 GB drive, and the backup grew with it. It now copies the whole backups folder (the DNS box, the host settings and the daily-driver's home folder) and the personal cloud, where documents now live. The NAS settings export is saved inside the personal cloud, so it rides along too.

Two lessons from the swap:

- **An existing backup task's destination can't be changed** on this NAS. The fix was a new task pointing at the new drive, run once by hand, then the old task deleted and the old stick unplugged.
- **Check folder sizes before ticking them.** The weekly machine backups came to 49.2 GB, which would fill a 64 GB drive within a couple of runs once the incremental versions pile up. They stay on the NAS only until a bigger separate drive takes over, together with the media library.

Two things got in the way while setting it up:

- **The NAS's app store said the NAS was not connected to the internet**, so the backup app could not be installed. The NAS was using the Pi-hole box for DNS like every other device. Setting the NAS's DNS to manual with public resolvers fixed it straight away, and it now stays that way. An infrastructure device that has to reach its vendor for apps and updates is a reasonable thing to leave off the ad-blocker.
- **"Backup between storage pools" refused** with a message that the device only has one storage pool. A two-bay NAS in RAID 1 is one pool. The option that works is a backup task with the external USB drive as the destination. (With the USB drive plugged in, the same option later worked fine, since the drive counts as a second place to back up to.)

## Off-site copy

Added 29 October 2026. Everything above lived in one room; this is the copy that survives losing the room.

**Where it goes:** a Hetzner Storage Box (the 1 TB size, a few euros a month) in their Falkenstein data centre. Plain storage reached over standard protocols, nothing to run on it.

**How it is locked down:**

- Only SSH is switched on. File sharing (SMB) and WebDAV are off, so there is one door instead of three.
- The NAS does not get the box's main login. It gets a **sub-account** that can only see one folder. If the NAS were ever compromised, the attacker could not reach anything else on the box.
- **Daily snapshots** on the box, keeping 10. The NAS cannot delete them; only the account owner can, from the provider's web console. That is the protection if ransomware scrambles the NAS and the scrambled files get backed up over the good ones.
- **Delete protection** on, so the box cannot be removed by an accidental click.
- The provider account has two-step login. The two passwords live in a password manager, nowhere else.
- The first SSH connection's host fingerprint was checked against the one the provider publishes before saying yes.

**The NAS side:** the built-in backup app, "Backup this NAS" to a remote rsync server, with transmission encryption on (that turns it into rsync over SSH, port 23 on this provider). Multi-version backup, keeping the last 30. It runs at 05:30, after every other nightly job has landed on the NAS.

**Snags worth knowing:**

- **rsync over SSH does work from this NAS to this provider.** Several forum threads from owners of the same NAS brand say they could not get rsync to connect and ended up recommending WebDAV instead. Here it connected first time with these settings: server address of the **sub-account** (not the main account), transmission encryption **on**, port **23**, the box's SSH support and external reachability switched on, server path left on the home directory.
- **The destination picker said "No Data".** The connection worked, but a brand new box is empty and the picker can only choose an existing folder; it has no "new folder" button. Fix: log in once with `sftp -P 23` as the sub-account, `mkdir` a folder, log out, press Refresh. Other owners of the same NAS brand hit the exact same wall on forums without a posted answer.
- **The first sftp login was dropped** after accepting the fingerprint. Not a password problem: the server only allows a short time to log in, and checking the fingerprint took longer than that. Running the same command again worked.
- **Encryption at rest is weak on this NAS app.** The transfer is encrypted, but the app gives no option to set your own password for the stored backup, and community write-ups report it uses one fixed, publicly known password for every backup archive it creates. Acceptable here for now. Proper client-side encryption with a password only the owner holds is the next improvement.

**Watching it:** a TCP port check on the box's port 23 in the uptime monitor (every 5 minutes, alerts to Telegram), and a dashboard tile that reads the box's status, space used, backup data and snapshot size from the provider's API with a **read-only** token limited to that one project.

**Knowing when a nightly run fails.** The NAS's own software has no email or push alerts, and its backup app has no failure notification setting, so a failed run would go unnoticed. The workaround is "no news is bad news": the uptime monitor has a **Push** monitor that expects a check-in at least every 26 hours. A small script on the DNS box runs at 09:00, asks the provider's API how much backup data is on the box, and compares it with the day before. If the number changed, the nightly run wrote something, so the script checks in. If the backup failed, got stuck or the NAS was off, the number stays the same, nobody checks in, and the monitor alerts on Telegram. It compares "changed" rather than "grew" because once the version limit is reached, old versions are pruned and the total can shrink on a good night.

## Restore testing

The rule used here: **never test a restore on the live machine**, especially anything the rest of the house depends on (DNS being the obvious one). Instead:

1. Restore the backup as a *new*, throwaway VM or container with a different ID.
2. Confirm it actually comes up and serves what it is supposed to.
3. Destroy the throwaway copy. The real machine was never touched.

This has been done twice.

**A Proxmox machine backup.** Done with the web server container: its backup was restored under a new ID, the copy picked up its own address from DHCP, it served the same page and images as the original, and then it was destroyed. A restore to a new ID takes a couple of minutes in Proxmox and it is the only way to know a backup is good before it is needed for real.

**The DNS box's nightly backup.** The box is a physical single-board computer, not a VM, so there is no restore-to-new-ID button for it. The test instead:

1. On a different machine that already runs Docker, unpack last night's archive from the NAS into a scratch folder.
2. Start a throwaway uptime monitor container from the unpacked data, on a spare port.
3. Open it in a browser and log in with the real account.
4. Delete the container and the scratch folder.

Result: all 12 monitors came back, with their history, showing as up. The throwaway copy was a newer major version than the one on the box, so it spent about three minutes converting the old database first. That turned out to be a useful second finding: the backup is good enough to restore onto a newer version, not only the identical one.

The Pi-hole half of that backup (the Teleporter export) had already been proven when the second Pi-hole was built from the same kind of export.

## Honest open items

- The photo library ([Immich](immich.md)) stores its photos on the NAS and dumps its database there nightly. Until the second copy below exists, the phone keeps its own originals as the other copy.
- RAID 1 is not a backup. The off-site copy now covers losing the NAS or the room, including the machine backups and the media.
- The off-site copy has not had its restore test yet. It comes after the first full upload finishes.
