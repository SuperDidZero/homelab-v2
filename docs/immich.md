# Photo library (Immich)

A self-hosted replacement for Google Photos: Immich, with every photo and video stored on the NAS and the phone backing itself up over Wi-Fi. No paid cloud service.

## How it is put together

- Immich runs as a Docker Compose stack on the same general-purpose Linux VM that hosts the media server and the personal cloud. The stack is four containers: the server, the machine learning service (face and object recognition), a small cache (Valkey) and a PostgreSQL database.
- It uses the two files Immich publishes with every release, `docker-compose.yml` and `example.env` (saved as `.env`), downloaded straight from the latest GitHub release into a folder of its own on the VM. The compose file is used as shipped; only `.env` is edited.
- It is published on Immich's default port on that VM. Nothing is opened on the router: away from home it is reached over the mesh VPN, the same way as every other service here.

## Where the files go, and the one rule that matters

The `.env` file has two locations in it, and they are deliberately in different places:

- **`UPLOAD_LOCATION`** (the photos and videos) points at a folder on the NAS. The VM already had the NAS's media share mounted over SMB/CIFS for the media server, so Immich simply uses a folder inside that existing mount. No new mount was needed. Delete the VM and the photos are still on the NAS.
- **`DB_DATA_LOCATION`** (the database) stays on the VM's own disk. Immich's own config file says it in plain words: network shares are not supported for the database. PostgreSQL over SMB is a good way to end up with a corrupted database.

The other changes in `.env`:

- `TZ` set to the local time zone, so photo dates come out right.
- `DB_PASSWORD` changed from the default to a long random password, letters and numbers only, as the file asks. Nothing outside the stack ever uses it, but the default is the same for everyone.

Before installing, the VM's memory was raised to give the machine learning container room, after checking how much the host had spare for the other machines.

## First-run setup choices

- **Admin account** created in the browser on first visit.
- **Google Cast** left off: it loads files from Google every time the page opens, and nothing here casts to a TV.
- **Storage template** turned on with a year/month layout, so the files on the NAS sit in dated folders with readable names instead of random IDs. They stay usable even without Immich.

## The phone

- Immich app installed, pointed at the VM's address on the home Wi-Fi, logged in.
- Backup on for the camera album.
- Cellular data off for both photos and videos: uploads only happen on Wi-Fi.
- The phone's battery setting for the app set to unrestricted. Without that, the phone puts the app to sleep and background uploads quietly stop.
- Away from home it keeps working as long as the mesh VPN is on, because the hypervisor routes the home subnet for the VPN.

## After it was running

Like every other service here, it is not counted as done until it is watched and visible:

- **Uptime monitor:** checks Immich's own health endpoint (`/api/server/ping`) rather than the front page, so a green light means the server itself is answering. Added to the home status page and wired to the same alerting as the rest.
- **Dashboard tile:** in the Media group next to the media server, with a live status dot and Homepage's built-in Immich widget showing users, photos, videos and storage used. The widget uses its own API key, created with only the statistics permission, so the dashboard can read the counts but could not see or delete a single photo if the key leaked.
- The tile was slotted into the existing Media group with `sed`, after that group's last line, instead of appended to the end of the file, which would have put it under whichever group happens to be last.

## Backups

- Immich dumps its own database every night into a `backups` folder inside the upload location, so those dumps land on the NAS.
- The whole VM, database included, is in the weekly hypervisor backup to the NAS.
- The photos themselves are on the NAS's RAID 1 pair, which is not a backup. Until the planned second copy on a separate drive exists, the phone's own originals are the second copy, so they are not being deleted from the phone yet. See the [backups page](backups.md).
