# Troubleshooting

A collection of diagnostic techniques and known issues for Raspberry Pi 4 running Raspberry Pi OS Lite.

## LED Status Codes

The Pi communicates hardware and boot status through its two on-board LEDs without needing a display.

### Red LED (Power)

| State | Meaning |
|---|---|
| Solid | Receiving stable 5 V (above 4.63 V). Normal. |
| Flashing | Under-voltage warning. The power supply cannot provide enough current. Try a better cable or a 5 V / 3 A supply. |
| Off | No power, or a brownout has occurred. |

### Green LED (Activity)

| State | Meaning |
|---|---|
| Solid (first second of boot) | Normal — the Pi is reading the SD card during boot. |
| Solid (persists after ~5 s) | Cannot read the SD card. Re-seat the card or re-flash. |
| Irregular rapid blinking | Normal operation — SD card or EEPROM read/write activity. |
| 4 blinks (rhythmic) | `start.elf` not found. SD card is empty, corrupted, or the wrong format. |
| 7 blinks (rhythmic) | `kernel.img` not found. OS is corrupted — re-flash. |
| 8 blinks (rhythmic) | SDRAM failure. Hardware issue. |



## SSH Connection Issues

### Permission denied (publickey)

Your SSH client is trying key-based authentication before password authentication. Force password auth explicitly:

```bash
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no myuser@hostname.local
```

### Host key conflict after re-imaging

If you re-flashed the SD card or changed the hostname, the old host key in `~/.ssh/known_hosts` will cause a rejection:

```bash
ssh-keygen -R hostname.local
```

### Slow SSH connections

Check that `GSSAPIAuthentication` is disabled on your control machine's SSH config. In `~/.ssh/config`:

```
Host pi-*
    GSSAPIAuthentication no
    IdentityAgent none   # only if you want to bypass 1Password/agent for these hosts
```



## Ansible Failures

### `sudo: a password is required`

The `ops` user does not yet have passwordless sudo configured (i.e., the `common` role has not run yet). Use `--ask-become-pass` on the first run:

```bash
ansible-playbook piSetup.yml --ask-pass --ask-become-pass
```

### `Permission denied (publickey,password)`

SSH key injection hasn't happened yet. Run with password auth:

```bash
ansible-playbook piSetup.yml --ask-pass
```

### Host is unreachable

Verify the IP in `inventory.ini` matches the actual address:

```bash
ansible pi_nodes -m ping
```

Check connectivity manually: `ping hostname.local` or `arp -a | grep <mac-prefix>`

### `interpreter_python` warnings

The `ansible.cfg` sets `interpreter_python = auto_silent` which suppresses these for Debian. If you see them, confirm you are running the playbook from the repo root so `ansible.cfg` is picked up.



## Network Issues

### Static IP not taking effect

After the `network` role runs, networking must be restarted or the device rebooted. The role handles this with a handler, but if you ran tasks manually:

```bash
sudo systemctl restart networking
```

Verify the assigned address:

```bash
ip addr show eth0
```

### mDNS not resolving `hostname.local`

Avahi must be running on the Pi:

```bash
sudo systemctl status avahi-daemon
```

If it is not installed: `sudo apt install avahi-daemon`

On the control machine, confirm mDNS discovery is working:

```bash
avahi-browse -a    # macOS: dns-sd -B _ssh._tcp
```



## Print Server (CUPS)

### List connected USB printers

Run this on the Pi to identify attached printers before running the `printServer` role:

```bash
lpinfo -v | grep usb
```

### CUPS web interface not reachable

The interface listens on port 631. Verify the service is running:

```bash
sudo systemctl status cups
```

Check that CUPS is configured to listen on all interfaces (not just `localhost`):

```bash
grep Listen /etc/cups/cupsd.conf
```

It should include `Listen 0.0.0.0:631` or the Pi's specific IP. Restart after any config changes: `sudo systemctl restart cups`

### Avahi not advertising printers

```bash
sudo systemctl status avahi-daemon
```

Printers shared in CUPS with `Browsing On` are advertised automatically via Avahi. If clients cannot see them, check that UDP 5353 is open between the subnets (Avahi uses mDNS multicast).



## SD Card Health

### Check write-cycle health (if supported)

```bash
cat /sys/block/mmcblk0/device/life_time
```

Returns two hex values in 10 % increments if the card supports it (e.g. `0x01 0x02` = partition A: 0–10 % worn, partition B: 10–20 % worn). Most Class 10 consumer cards do not expose this.

### Check for filesystem errors flagged by the kernel

```bash
dmesg | grep -i mmcblk
```

Watch for `I/O error`, `Read-only file system`, or `Card stuck in recovery` — these indicate a worn or failing card.

### Check disk usage

Cards near full capacity wear faster because the controller has fewer free cells for write-leveling:

```bash
df -h
```

Aim to keep utilization below ~80 % for longevity. A 32 GB card kept at 15 % utilization will outlast one running at 90 % by a wide margin.


## Temperature

Monitor live temperature:

```bash
watch -n 2 vcgencmd measure_temp
```

| Reading | Assessment |
|---|---|
| 35–45 °C idle | Excellent — passive cooling is sufficient |
| 45–60 °C idle | Acceptable — consider better airflow |
| 60–70 °C under load | Normal with good passive cooling |
| > 80 °C | Throttling will occur — add active cooling |

The Pi 4 throttles the CPU when the temperature exceeds 80 °C and hard-shuts at 85 °C.
