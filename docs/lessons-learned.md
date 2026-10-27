# Lessons learned

## Principles that guided the build

- **Separation of concerns**: a device or VM that's for experimenting stays separate from one that's for something the household actually depends on. Blow up the experimental one as much as you want.
- **Shelf depth, not rack width, is the real constraint** on an open-frame rack. Measure your gear's depth before buying shelves.
- **A backup you haven't restored from is a guess.** Always restore to a throwaway copy, confirm it works, then delete the copy, never test a restore on the live machine.
- **Pulling a new image is not proof you're patched.** After a security update, check the version from inside the running container (for example `php -v`). Early rebuilds can still ship the old, vulnerable version.
- **Pin image tags on anything with a database.** `latest` moves, and on a security fix it can carry a major upgrade in with it. Pin the version you run, change it deliberately.
- **Track your own IP/MAC inventory.** It costs nothing to keep a running list and check it before handing out a new address; it costs an afternoon of confused debugging when a new device collides with an existing one.
- **Write down what's actually done vs. still planned.** It's tempting to describe a homelab as further along than it is, especially when writing about it publicly. Being specific about what's finished and what's still an open item makes the whole write-up more useful (and more trustworthy) than a highlight reel.
- **If local pricing/retailer pages render via JavaScript, don't trust an automated fetch of the raw price**, verify with a real screenshot or a direct visit. This applies to a lot of regional retailer sites, not just one.

## Hardware ruled out (and why it doesn't need re-litigating)

A handful of small-form-factor PC candidates were seriously considered as the hypervisor host and ruled out mainly on specs-per-euro and known Proxmox compatibility once the final choice was confirmed working. If you're shopping for the same kind of box, the short version is: almost any recent "tiny"-class business PC with 16GB+ RAM and a 6+ core CPU will do this job, the brand matters less than checking VT-x/virtualization support and NIC compatibility (see the e1000e note in [proxmox.md](proxmox.md)) before you buy.

## The loop this build follows

Learn the thing → build it → actually use it day to day → automate the parts that are repetitive → document it properly (this repo) → improve it. Nothing gets sold or shared outward until it's gone through that whole loop at least once.
