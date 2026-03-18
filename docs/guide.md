# Manual Setup Guide

This guide walks through everything that needs to happen *before* you run the Ansible playbook: flashing the OS, connecting to the Pi, optimizing the hardware, and updating the firmware.

[Raspberry Pi OS Lite](https://www.raspberrypi.com/software/operating-systems/) is a 64-bit ARM port of Debian that excludes a desktop environment to remain lightweight. It uses proprietary firmware packages (`raspberrypi-bootloader`, `firmware-brcm80211`) to manage the GPU, Wi-Fi, and Bluetooth — `firmware-linux-free` is not required.

```
     ,g$$$$$$$$$$$$$$$P.       ops@RaspberryPI
   ,g$$P""       """Y$$.".     OS: Debian GNU/Linux 13 (trixie) aarch64
  ,$$P'              `$$$.     Host: Raspberry Pi 4 Model B Rev 1.5
',$$P       ,ggs.     `$$b:    Kernel: Linux 6.12.62+rpt-rpi-v8
`d$$'     ,$P"'   .    $$$     Packages: 649 (dpkg)
 $$P      d$'     ,    $$P     Shell: zsh 5.9
 $$:      $$.   -    ,d$$'     CPU: BCM2711 (4) @ 1.80 GHz
 $$;      Y$b._   _,d$P'       Memory: 187 MiB / 7.69 GiB
 Y$$.    `.`"Y$$$$P"'          Disk (/): 4.22 GiB / 28.79 GiB - ext4
 `$$b      "-.__               Local IP (eth0): 172.30.30.125/24
  `Y$$b                        Locale: en_CA.UTF-8
```

---

## Step 1 — Flash the OS

Use the [Raspberry Pi Imager](https://www.raspberrypi.com/software/) for a streamlined installation.

**Hardware:** A Class 10 microSD card larger than 16 GB is recommended.[^1] Cards larger than 2 TB are not supported due to MBR limitations.

In the Imager, configure the following before writing:

| Setting | Value |
|---|---|
| Device | Raspberry Pi 4 Model B 8 GB |
| OS | Raspberry Pi OS (other) → Raspberry Pi OS Lite (64-bit) |
| Hostname | Single word: letters, numbers, hyphens only. Reachable as `hostname.local` via mDNS. |
| Locale / Timezone | e.g. `America/Toronto` |
| Keyboard | `us` |
| Username / Password | Set a unique ops username and strong password |
| SSH | Enable with **password authentication** |
| Wi-Fi | Configure only if no Ethernet cable is available |
| Raspberry Pi Connect | Disable[^2] |

> **On FQDNs:** Use a simple single-word hostname unless you specifically need SSL certificates via Let's Encrypt or are mirroring a datacenter DNS structure. A FQDN is not required for normal home lab use.

Insert the card, connect power. A solid red LED means the Pi is receiving stable 5V and booting.

**Power options:**

- USB-C power supply (5 V / 3 A recommended)
- PoE injector (IEEE 802.3af/at)
- PoE splitter converting Ethernet to 5 V USB-C

---

## Step 2 — First Login

Connect via Ethernet, then SSH in using password authentication:

```bash
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no myuser@hostname.local
```

The explicit flags are needed because your SSH client will otherwise try all available keys first, resulting in `Permission denied (publickey)` before you ever get a password prompt.

If you get a host key conflict after re-imaging the card:

```bash
ssh-keygen -R hostname.local
```

Alternatively, add a temporary entry to `~/.ssh/config` on your control machine:

```
Host pi-init
    HostName hostname.local
    User myuser
    PreferredAuthentications password
    PubkeyAuthentication no
```

---

## Step 3 — Hardware Optimization

For headless operation, reduce power consumption and heat by disabling on-board Wi-Fi and Bluetooth (assuming you are using Ethernet).

```bash
ssh myuser@hostname.local
sudo vi /boot/firmware/config.txt
```

Replace the content with:

```ini
# Core Settings
arm_64bit=1
arm_boost=1
disable_overscan=1

# Audio (comment out if not needed)
dtparam=audio=on

# Headless Optimizations
gpu_mem=16
# dtoverlay=vc4-kms-v3d
# camera_auto_detect=1
# display_auto_detect=1

# Disable Wi-Fi and Bluetooth (Ethernet only)
dtoverlay=disable-wifi
dtoverlay=disable-bt

[cm4]
otg_mode=1

[all]
enable_uart=1
```

Reboot: `sudo reboot`

---

## Step 4 — Firmware Update

```bash
ssh myuser@hostname.local
sudo rpi-eeprom-update
```

If an update is available, apply it:

```bash
sudo rpi-eeprom-update -a
sudo reboot
```

---

## Step 5 — System Fine-Tuning

```bash
ssh myuser@hostname.local
sudo raspi-config
```

Key settings to review:

- **8 Update** — ensure `raspi-config` itself is current
- **1 System Options → S4 Hostname** — change hostname if required
- **5 Localisation Options → L4 WLAN Country** — required for Wi-Fi regulatory compliance[^4]
- **6 Advanced Options → A1 Expand Filesystem** — the Imager usually handles this automatically
- **4 Performance Options → P3 Fan** — configure GPIO fan behaviour if using a case fan
- **6 Advanced Options → A12 Logging** — consider `Volatile` (RAM) to reduce SD card write wear:

| Mode | Storage | Pros | Cons |
|---|---|---|---|
| Default | SD Card | Persistent across reboots | Write wear on the card |
| Volatile | RAM | Zero card wear; very fast | Logs lost on reboot |
| Persistent | SD Card (optimized) | Survives reboots | Still causes some wear |
| None | Disabled | Zero disk usage | Cannot troubleshoot crashes |

Monitor temperature in real time:

```bash
watch -n 2 vcgencmd measure_temp
```

Healthy idle: 35–45 °C. Under load with passive cooling: stay below 70 °C.

Check system info: `fastfetch`

---

## Optional — Static IPv4 Address

If you prefer to set a static IP directly on the Pi rather than relying on DHCP (the Ansible `network` role handles this, but here is the manual equivalent):

Edit `/etc/network/interfaces.d/eth0`:

```
auto eth0
iface eth0 inet static
    address 172.30.80.22
    netmask 255.255.255.0
    gateway 172.30.80.1
    dns-nameservers 172.30.50.2
```

Restart networking: `sudo systemctl restart networking`

---

## Optional — Ghostty Terminal Compatibility

If you use [Ghostty](https://ghostty.org/) as your terminal emulator, the Pi will receive `TERM=xterm-ghostty` over SSH and may respond with `Error opening terminal: xterm-ghostty`.

The cleanest fix is to export `TERM=xterm-256color` in your shell config on the Pi (`~/.zshrc` or `~/.bashrc`):

```bash
export TERM=xterm-256color
```

Alternatively, stop your local machine from forwarding the variable by editing `~/.ssh/config` or `/etc/ssh/ssh_config` and commenting out `SendEnv LANG LC_*`.

If you want to keep Ghostty-specific features on the Pi, upload the terminfo entry from your local machine (not inside an SSH session):

```bash
infocmp -x xterm-ghostty | ssh myuser@hostname.local -- tic -x -
```

---

## SD Card Health

Standard microSD cards lack S.M.A.R.T. controllers. Some higher-end cards (SanDisk, Kingston) expose a limited health indicator via sysfs:

```bash
cat /sys/block/mmcblk0/device/life_time
```

Two hex values in 10 % increments are returned if supported (e.g. `0x01 0x01` = 0–10 % worn). On most Class 10 cards this file is absent or empty.

Cards are designed to fail-safe into read-only mode when write cycles are exhausted. Check for kernel-level errors:

```bash
dmesg | grep -i mmcblk
```

Look for `I/O error`, `Read-only file system`, or `Card stuck in recovery`.

Keeping the card at low utilization (e.g. 15 % on a 32 GB card) maximizes write-leveling headroom and extends lifespan significantly. Check current usage: `df -h`

---

## What's Next

Once the Pi is reachable via SSH with key-based authentication, return to the [README](../README.md) and run the Ansible playbook.

If you run into issues, see [docs/troubleshooting.md](troubleshooting.md).

---

## Footnotes

[^1]: A case with heatsinks or a fan is strongly recommended — the Pi 4 runs hot. Example: [Miuzei Aluminium Raspberry Pi Case with Passive Cooling](https://www.amazon.ca/Miuzei-Raspberry-Aluminum-Conductive-Compatible/dp/B08LVRTYPD).
[^2]: Raspberry Pi Connect requires a Raspberry Pi account and cloud connectivity. Disable it for offline or private network deployments.
[^3]: See the [dotFiles README](https://github.com/bhdicaire/dotFiles/blob/main/README.md) for details on `run_once_before_install-packages.sh`.
[^4]: The WLAN country setting is required for Wi-Fi regulatory compliance and stability — even if you are not using Wi-Fi on this device.
