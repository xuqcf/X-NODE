# Linux Networking & Interface Naming

## 1. Why Interface Names Changed (`wlan0` → `wlp9s0`)

In the past, Linux assigned network names sequentially based on discovery order (`eth0`, `wlan0`, `eth1`). Because systems now boot in parallel, the order of discovery is not guaranteed—a card might be `wlan0` on one boot and `wlan1` on the next.

### Predictable Network Interface Names

Modern Linux (via `systemd` and `udev`) uses **Predictable Network Interface Names**. These names are based on the hardware's physical location on your motherboard or its firmware index.

- **Stability:** The name stays the same even if you add/remove other hardware or update drivers.
    
- **Format:**
    
    - **Prefixes:** `en` (Ethernet), `wl` (WLAN/Wireless), `ww` (WWAN/Mobile).
        
    - **Suffixes:** Indicate physical location (e.g., `p9s0` = PCI bus 9, slot 0).
        
- **The "New" Name:** `wlp9s0` tells you exactly where that card sits on your motherboard.
    

---

## 2. How Networking Works in Arch

Arch Linux networking is managed by a combination of the kernel (which talks to the hardware) and user-space tools (which configure the connection).

### The Networking Stack

1. **Kernel/Driver:** Detects the hardware and registers it (e.g., `rtw89_8852be` for your Realtek card).
    
2. **udev:** Detects the device and assigns a "predictable name" based on its hardware path.
    
3. **NetworkManager:** The daemon that "claims" the interface. It monitors for network availability, handles credentials (secrets), and maintains routing tables.
    

### Why Troubleshooting Often Involves "Deleting"

NetworkManager stores connection "profiles" (stored in `/etc/NetworkManager/system-connections/`). If your hardware name changes from `wlan0` to `wlp9s0`, the old profile for `wlan0` becomes a **"ghost profile."** It tries to manage a device that doesn't exist, which leads to:

- **"Secrets required but not provided"** errors.
    
- **Hanging/Loading loops** (as it waits for a device to appear).
    
- **Dormant/Down states** (as it tries to force old settings onto a new interface).
    

---

## 3. CLI Networking Toolkit

When things go wrong, these are your standard diagnostic tools:

|**Command**|**Purpose**|
|---|---|
|`ip link`|Shows all network interfaces and their current status (UP/DOWN/DORMANT).|
|`nmcli connection show`|Lists all saved network profiles.|
|`nmcli connection delete [name]`|Removes a stale/corrupt profile.|
|`nmcli --ask dev wifi connect [SSID]`|Forces a fresh connection and prompts for a password.|
|`dmesg|grep [driver_name]`|

---

## 4. Troubleshooting Workflow

If your Wi-Fi fails to connect:

1. **Check the Link:** Run `ip link` to verify the current interface name.
    
2. **Verify Status:** Is the interface `DOWN`? If so, try `sudo ip link set [name] up`.
    
3. **Clear Ghosts:** Delete any connection profiles that refer to old names or show errors using `nmcli connection delete`.
    
4. **Force Re-auth:** Use `nmcli --ask` to bypass cached, corrupted secrets.
    

