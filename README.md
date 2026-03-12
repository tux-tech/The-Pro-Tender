# 🍎 The Pro-Tender
**A macOS Compatibility Layer for Debian 13 (Trixie)**

`The Pro-Tender` is an automation script that configures a Debian 13 (Trixie) system to act as a native peer on a macOS network. By integrating Apple-specific SMB extensions and mDNS service broadcasting, it allows Linux servers to provide the same metadata support and discovery features as a genuine Mac server.

---

## 🚀 Key Features

* **Hardware Identity Spoofing:** Configures Avahi/mDNS to broadcast a `MacPro7,1` identifier, forcing macOS to display the server with the native Mac Pro icon in Finder.
* **Apple VFS Integration:** Optimizes the Samba `vfs_fruit` module for Apple’s SMB2/3 extensions, enabling fast directory enumeration and native file handling.
* **Metadata Persistence:** Uses `streams_xattr` to store Finder tags, colors, and comments in filesystem Extended Attributes, preventing `.DS_Store` clutter.
* **Network Visibility:** Provides out-of-the-box support for **Bonjour (Avahi)** and **WS-Discovery (wsdd2)** for cross-platform visibility on both macOS and Windows.
* **Character Mapping:** Utilizes `catia` to transparently map illegal filename characters between POSIX and macOS standards.

---

## 🛠 Technical Architecture

| Component | Role | Description |
| :--- | :--- | :--- |
| **Samba 4.x** | Protocol | Configured with `catia fruit streams_xattr` VFS objects. |
| **Avahi** | Discovery | Broadcasts `_smb._tcp`, `_afpovertcp._tcp`, and `_device-info`. |
| **Netatalk** | Icon Support | Facilitates the AFP framework required for hardware icon spoofing. |
| **wsdd2** | Windows Link | Ensures visibility for Windows clients via Web Services Dynamic Discovery. |
| **Xattr/ACL** | Filesystem | Manages Apple Double metadata and fine-grained permissions. |
