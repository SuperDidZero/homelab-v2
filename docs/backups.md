# Backups

## The model

Everything backs up to the NAS, once. A USB drive plugged into the NAS covers the NAS's own settings and the smallest, most important backups. No redundant second copies of things that are already on the NAS, for now. A full second copy on a bigger separate drive is the next step and is not done yet.

## What gets backed up

| What | How | When | Kept |
|---|---|---|---|
| Every VM and container on the hypervisor | Proxmox backup job to the NAS over SMB/CIFS (a Proxmox storage target with content type "Backup") | Weekly, Sunday 01:00 | Last 3 |
| The hypervisor host's own settings | Small nightly job on the host, copied to the NAS | Nightly, 02:30 | Recent copies |
| DNS box (Pi-hole export plus the app data for the dashboard, the uptime monitor and Unbound) | Cron job on the box: Pi-hole Teleporter export plus a tar of the app data, copied to the NAS | Nightly, 03:30 | Newest 14 |
| Daily-driver single-board computer (home folder) | Copy to the NAS. An hourly check runs it only when the machine is on the home network | Daily, when at home | Newest 7 |
| The personal cloud plus every backup folder above | NAS backup task to a 64 GB USB drive that stays plugged into the NAS | Nightly, 04:30 | Incremental |
| Router configuration | Manual export of the router's settings file, uploaded to the personal cloud | After router changes | Latest |

Notes on a few of these:

- The Proxmox job's selection mode is "All" on purpose, so a newly created VM or container is included automatically instead of being silently left out until someone remembers to add it.
- The uptime monitor is paused for a few seconds while the DNS box backup runs, so its database is copied in a consistent state.
- The times are staggered so that each job's output already exists when the next one picks it up: machines, then host settings, then the DNS box, then the USB drive.
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

- RAID 1 is not a backup, and the NAS is still a single location. A full second copy on a separate drive, including the machine backups and the media, is planned and not done.
