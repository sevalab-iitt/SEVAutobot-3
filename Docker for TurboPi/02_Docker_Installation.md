<img width="490" height="478" alt="image" src="https://github.com/user-attachments/assets/ef3991a0-b996-40a7-a55d-2d382ffd4622" />

<img width="699" height="96" alt="image" src="https://github.com/user-attachments/assets/b907710f-d50b-4d18-9a5c-9e5b679a94f5" />


As we want to install docker, docker image might take all the storagr in an instant so we'll clear up some space.

Run these commands
```
df -h
sudo du -h --max-depth=1 / | sort -h
du -h --max-depth=1 ~ | sort -h
```


Well instead of microsd card i am using pendrive 
<img width="764" height="237" alt="image" src="https://github.com/user-attachments/assets/4f87cf0d-19df-42a3-afa6-789fc5113b09" />

As you can see the pendrive is currently in FAT32 format so we have to format it to EXT4 
here is why...
# File Systems: EXT4 vs. FAT32

A quick reference guide on the full forms, use cases, and differences between the EXT4 and FAT32 file systems.

---

## 1. EXT4 (Fourth Extended Filesystem)

*   **Full Form:** Fourth Extended Filesystem
*   **Uses:**
    *   **Default Linux Filesystem:** The standard file system for most modern Linux distributions (e.g., Ubuntu, Debian, Fedora).
    *   **Android Devices:** Used internally by the Android OS for system and data partitions.
    *   **Network Attached Storage (NAS):** Commonly used in Linux-based NAS drives.
    *   **High-Performance Storage:** Ideal for servers and workstations requiring journaling (to prevent data corruption) and large file support.

---

## 2. FAT32 (File Allocation Table 32)

*   **Full Form:** File Allocation Table 32-bit
*   **Uses:**
    *   **Universal Compatibility:** The go-to format for cross-platform file sharing, as it is supported by Windows, macOS, Linux, and gaming consoles.
    *   **Removable Storage:** Widely used for USB flash drives and SD cards.
    *   **Legacy Devices:** Used in digital cameras, MP3 players, smart TVs, and older gaming systems.

---

## 3. Comparison Chart

| Feature | EXT4 (Fourth Extended Filesystem) | FAT32 (File Allocation Table 32) |
| :--- | :--- | :--- |
| **Primary OS** | Linux, Android | Windows, macOS, Linux, & Legacy Devices |
| **Max File Size** | 16 TiB | 4 GB |
| **Max Volume Size** | 1 EiB | 8 TB |
| **Journaling** | **Yes** (Prevents data corruption) | **No** (Prone to corruption if unplugged during write) |
| **Security & Permissions** | **High** (Native Linux security, encryption) | **Low** (No file-level permissions) |
| **Performance** | **High** (Efficient allocation, resists fragmentation) | **Moderate** (Prone to fragmentation) |
| **Compatibility** | **Limited** (Requires third-party drivers on Windows/macOS) | **Universal** (Native read/write on almost all devices) |
| **Best For** | Linux system drives, internal storage, NAS | USB drives, SD cards, cross-platform file transfers |
