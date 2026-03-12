# 🍎 The Pro-Tender

**macOS Compatibility Layer for Debian 13 (Trixie)**

> *One script. Your Debian box shows up in every Mac's Finder sidebar — with a Mac Pro icon — like it was always there.*

`The Pro-Tender` is a zero-config automation suite that bridges Debian 13 and macOS. It installs and configures a full Apple compatibility stack: `vfs_fruit` for native SMB metadata handling, Avahi for Bonjour broadcasting, Netatalk for AFP and hardware identity, and shell-level safeguards to keep Mac metadata intact from the CLI.

---

## ✨ What it does

| Layer | Technology | Effect on your Mac |
|---|---|---|
| **Hardware Spoofing** | Avahi `_device-info._tcp` + `fruit:model` | Finder renders the box with the "Cheese Grater" Mac Pro icon |
| **SMB Apple Layer** | Samba `vfs_fruit` + `streams_xattr` | Finder tags, colours, and comments persist on the share |
| **Bonjour Discovery** | `avahi-daemon` XML service files | Box appears automatically in the Finder sidebar under Locations |
| **AFP Discovery** | `netatalk` | Legacy AFP broadcasting + hardware icon fallback |
| **Character Safety** | Samba `catia` module | Filenames with `:` `"` `'` translate cleanly between macOS and Linux |
| **Windows Discovery** | `wsdd2` | Visible to Windows clients on the same network (bonus) |
| **CLI Safety** | Shell aliases in `.bashrc` / `.zshrc` | `cp` and `rsync` preserve xattrs so Mac tags survive terminal use |
| **Metadata Auditing** | `mac-check` utility | Scan `/appleshare` for active `com.apple.*` metadata at any time |

---

## 🚀 Quick Start

Tested on **Debian 13 (Trixie)** bare-metal. Run as root or with `sudo`.

```bash
git clone https://github.com/tux-tech/The-Pro-Tender.git
cd The-Pro-Tender
sudo bash mac-layer-setup.sh
```

That's it. The script will ask you to set one Samba password — the credentials your Macs use when Finder prompts for login. Everything else is automatic.

---

## 🔌 Connecting from macOS

After the script completes (~2 minutes), your Debian box should appear in the **Finder sidebar → Locations** within 30 seconds via Bonjour.

**Manual connect (if it doesn't appear automatically):**

```
Finder → Go → Connect to Server (⌘K)
smb://oculus.local/AppleShare
```

Replace `oculus` with your Debian machine's actual hostname — the script detects and uses this automatically.

**Login:** Use the username and Samba password you set during installation. Tick *"Remember this password"* in Keychain to make it seamless going forward.

---

## 📁 The Share

The script creates a single root-level share directory:

```
/appleshare
```

All your Macs read and write here. Files created from macOS retain their Finder tags, colour labels, and comments — stored as Linux Extended Attributes (`com.apple.*` xattrs) rather than cluttering the directory with `._` files.

### Adding an external drive later

Mount your drive to `/appleshare` via `/etc/fstab`. Format it as **Ext4 or Btrfs** — both support xattrs natively, so Mac tags survive. Your existing Mac bookmarks and shortcuts won't need updating.

```bash
# Example /etc/fstab entry (replace UUID with yours from `blkid`)
UUID=xxxx-xxxx  /appleshare  ext4  defaults,user_xattr  0  2
```

---

## 🔧 What the script installs

The script runs seven stages:

1. **Package Provisioning** — Samba, `samba-common-bin`, `avahi-daemon`, `avahi-utils`, `libnss-mdns`, `netatalk`, `attr`, `acl`, `wsdd2`
2. **Filesystem Provisioning** — Creates `/appleshare` with `chmod 2775` (Setgid bit ensures group-ownership inheritance for all network-created files)
3. **Samba Orchestration** — Writes a clean `smb.conf` with `vfs_fruit`, `catia`, and `streams_xattr` in the correct load order
4. **Bonjour Service Definitions** — Generates `/etc/avahi/services/` XML files for SMB, SSH, AFP, and `_device-info` (hardware icon)
5. **Netatalk Configuration** — Configures AFP framework for legacy discovery and hardware identity broadcasting
6. **Credential Setup** — Adds your user to the Samba password database (independent of your Linux system password)
7. **Service Daemonisation** — Enables and starts `smbd`, `nmbd`, `avahi-daemon`, `netatalk` as systemd units

---

## 🔍 Verification & Utilities

### `mac-check`

Installed system-wide. Scans the share for files carrying active `com.apple.*` metadata — useful for confirming tags survived a copy or move operation.

```bash
mac-check                  # scans /appleshare by default
mac-check /some/other/dir  # scan any path
```

### `testparm`

Verify Samba parsed `smb.conf` correctly:

```bash
testparm
```

### `smbstatus`

See which Macs are currently connected and what files they have open:

```bash
smbstatus
```

### `avahi-browse`

Confirm your services are broadcasting on the network:

```bash
avahi-browse -a
```

---

## 🐚 Shell Safety Aliases

The script appends these to your `.bashrc` / `.zshrc` so Mac metadata is never accidentally stripped when working in the terminal:

```bash
alias cp='cp -a'          # preserves xattrs, permissions, and timestamps
alias rsync='rsync -aX'   # preserves extended attributes during sync
```

These are silent and non-breaking — if a file has no Mac metadata, the flags simply find nothing to copy and carry on.

> **Note:** If you copy files to a filesystem that doesn't support xattrs (FAT32, old NFS mounts), you'll see a soft warning like `failed to preserve extended attributes: Operation not supported`. The file data copies fine — Linux is just telling you the destination can't store the tags.

---

## ⚙️ Samba Config Reference

The generated `smb.conf` uses these key Apple-specific parameters:

| Parameter | Value | Purpose |
|---|---|---|
| `vfs objects` | `catia fruit streams_xattr` | Load Apple modules in dependency order |
| `fruit:aapl` | `yes` | Enable Apple SMB2/3 extensions |
| `fruit:metadata` | `stream` | Store metadata in alternate data streams (cleanest method) |
| `fruit:model` | `MacPro7,1` | Hardware identity — sets Finder icon |
| `fruit:copyfile` | `yes` | Server-side copy (fast duplication without re-uploading) |
| `fruit:posix_rename` | `yes` | POSIX rename semantics (prevents duplicate-file bugs) |
| `fruit:time machine` | `no` | Set to `yes` on any share to enable Time Machine backups |

---

## 🕰 Time Machine (Optional)

To turn `/appleshare` (or a sub-directory) into a Time Machine target, edit `/etc/samba/smb.conf` and change one line in the `[AppleShare]` block:

```ini
fruit:time machine = yes
```

Then restart Samba:

```bash
sudo systemctl restart smbd
```

On your Mac, go to **System Settings → Time Machine → Add Backup Disk** and select the share.

---

## 📋 Requirements

- Debian 13 (Trixie) — bare metal or VM
- Root / sudo access
- Network interface on the same subnet as your Macs
- Filesystem formatted as **Ext4, Btrfs, or ZFS** (xattr support required)

---

## ⚖️ License

MIT License 

---

## 🙏 Background

Built from notes accumulated while running a Debian 13 box alongside a macOS-heavy workflow. The goal: zero friction between Linux and Apple — no manual IP bookkeeping, no metadata loss, no `._` file pollution. One script, then forget about it.
