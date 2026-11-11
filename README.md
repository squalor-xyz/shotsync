# shotsync

Copies screenshots between the laptop and one or more VMs so agents running
there can read them. Uses plain `ssh` + `rsync`; nothing is ever deleted.

```
shotsync push [target|all]   ~/Pictures/shotsync/  -> <target>:<remote_dir>/
shotsync pull [target|all]   <target>:<remote_dir>/ -> ~/Pictures/shotsync/from-<target>/
shotsync targets             list targets (first = default)
```

With no target name, the default (first) target is used.

## Requirements

- **Laptop:** `bash`, `ssh`, `rsync`, and a `readlink` that supports `-f`
  (Linux, or recent macOS). The screenshot-folder step in Setup is macOS-only.
- **Each VM:** `rsync` must be installed there too. rsync runs on both ends,
  so a VM without it will fail the push or pull.
- SSH access to each VM. Key-based auth is recommended: a push opens two SSH
  connections per target, so password auth would prompt twice.

## Setup (once)

1. `cp targets.example targets`, then add your VMs to `targets`, one per line
   (`targets` is git-ignored, so your hostnames stay local):

   ```
   # name   ssh-host    remote_dir
   vm1      myvm        ~/shots
   gpu      gpu-box     /data/shots
   ```

   `ssh-host` is what you'd type in `ssh <host>`. `remote_dir` is a full path
   on the VM: absolute (`/data/shots`) or under the VM's home (`~/shots`).
   Fields can't contain spaces.

2. Put `shotsync` on your PATH, e.g. (pick a dir that's already on PATH):

   ```sh
   # from this folder:
   ln -s "$PWD/shotsync" ~/.local/bin/shotsync
   ```

   The symlink is fine: the script finds `targets` next to its real location.

3. Create the local folder and point macOS screenshots at it:

   ```sh
   mkdir -p ~/Pictures/shotsync
   defaults write com.apple.screencapture location ~/Pictures/shotsync
   killall SystemUIServer
   ```

   To revert to the default (Desktop):

   ```sh
   defaults delete com.apple.screencapture location
   killall SystemUIServer
   ```

## Use

1. Take a screenshot with `Cmd+Shift+4` (region) or `Cmd+Shift+5`.
   The clipboard variants (with `Ctrl`) don't save a file.
2. `shotsync push` (or `shotsync push gpu`, or `shotsync push all`).
3. Paste the printed path into the agent's prompt, e.g.
   `look at "~/shots/Screenshot ... .png": the toolbar spacing is off`.
   Keep the quotes: macOS screenshot names contain spaces.

To fetch images an agent produced on a VM: `shotsync pull vm1`. They land in
`~/Pictures/shotsync/from-vm1/`. `from-*` folders are never pushed back out.

## Notes

- Override the local folder with `SHOTSYNC_DIR=/some/dir shotsync push`.
- Neither side is ever pruned. Clear old shots yourself in both places.
- `push all` / `pull all` go through targets in file order and stop at the
  first one that fails (unreachable host, missing rsync, bad `targets` line).
  Targets after it are skipped. Fix the failing one or run the rest by name.

## License

MIT. See [LICENSE](LICENSE).
