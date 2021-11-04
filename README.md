# Btrfs Autosnap

- A POSIX shell script that creates a new snapshot, then removes expired snapshots of btrfs subvolumes with a single-line command
- Author: snoopy3476@outlook.com


## Description

Refresh the list of btrfs (read-only) snapshots for the target subvolumes given through cmdline arguments.

Here, "refreshing snapshots" means following jobs:
- CREATE a new btrfs snapshot for a subvolume, only if there is some changes after the latest snapshot
- Then REMOVE old snapshots, if each snapshot is both:
  - Older than `$SNAP_EXPIRATION_DAYS` days
  - `n`-th latest, where `n > $SNAP_MIN_COUNT` (That is, latest `$SNAP_MIN_COUNT` snapshots are always preserved even if expired)

By running this script everyday (using crontab, etc.), you can maintain enough number of snapshots without ending up with no free space left, because of unlimited snapshot creation.


## Usage

`$ btrfs-autosnap [-t <snap_expiration_days>] [-n <snap_min_count>] <subvol-path-1> [subvol-path-2] ...`


## Snapshot path

If refresh a snapshot subvolume with the following path:
- `<PARENT_DIR_PATH>/<SUBVOL_DIR_NAME>`

each snapshot will be stored with the following name:
- `<SUBVOL_DIR_NAME>_<SNAPSHOT_TIME>`
  - `SNAPSHOT_TIME`: `date +%Y.%m.%d-%H:%M:%S`

inside the snapshot list dir with the following path:
- `<PARENT_DIR_PATH>/.@snapshots_<SUBVOL_DIR_NAME>/`


## Examples

### Run the script

- Refresh snapshots of `./subvol` and `./subvol_2` with default config
  - `$ sudo ./btrfs-autosnap ./subvol ./subvol_2`

- Refresh snapshots of `/path/to/subvol` with `SNAP_EXPIRATION_DAYS=60`, and `SNAP_MIN_COUNT=5`
  - `$ sudo ./btrfs-autosnap -t60 -n5 /path/to/subvol`

- Refresh snapshots of `./subvol`, `./subvol_2` with `SNAP_EXPIRATION_DAYS=(default)`, `SNAP_MIN_COUNT=30`
  - `$ sudo ./btrfs-autosnap -n30 ./subvol ./subvol2`


### Run the script everyday

- /etc/cron.daily/btrfs-autosnap-daily
```
/path/to/the/script/btrfs-autosnap <SUBVOL_PATH_1> <SUBVOL_PATH_2> ...
```


### Show previous versions (snapshot list) on Windows explorer remotely, when a btrfs subvolume is shared with samba 

- /etc/samba/smbd.conf
```
...

vfs objects = shadow_copy2 btrfs acl_xattr
shadow:snapdir = <PARENT_DIR_PATH>/.@snapshots_<SUBVOL_DIR_NAME>
shadow:basedir = <PARENT_DIR_PATH>/<SUBVOL_DIR_NAME>
shadow:format = @<SUBVOL_DIR_NAME>_%Y.%m.%d-%H:%M:%S
shadow:localtime = yes

...
```
