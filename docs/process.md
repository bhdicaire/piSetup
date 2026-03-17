# Manual Setup

[Raspberry Pi OS Lite](https://www.raspberrypi.com/software/operating-systems/) is an ARM64 port of Debian that excludes a desktop environment to remain lightweight. It utilizes specific, non-free proprietary firmware found in packages such as `raspberrypi-bootloader` and `firmware-brcm80211` to manage the GPU, Wi-Fi, and Bluetooth. Consequently, the firmware-linux-free package is not required.

~~~
        _,met$$$$$gg.          ops@RaspberryPI
     ,g$$$$$$$$$$$$$$$P.       -------
   ,g$$P""       """Y$$.".     OS: Debian GNU/Linux 13 (trixie) aarch64
  ,$$P'              `$$$.     Host: Raspberry Pi 4 Model B Rev 1.5
',$$P       ,ggs.     `$$b:    Kernel: Linux 6.12.62+rpt-rpi-v8
`d$$'     ,$P"'   .    $$$     Uptime: 15 mins
 $$P      d$'     ,    $$P     Packages: 649 (dpkg)
 $$:      $$.   -    ,d$$'     Shell: zsh 5.9
 $$;      Y$b._   _,d$P'       Terminal: /dev/pts/0
 Y$$.    `.`"Y$$$$P"'          CPU: BCM2711 (4) @ 1.80 GHz
 `$$b      "-.__               Memory: 187.08 MiB / 7.69 GiB (2%)
  `Y$$b                        Swap: 0 B / 2.00 GiB (0%)
   `Y$$.                       Disk (/): 4.22 GiB / 28.79 GiB (15%) - ext4
     `$$b.                     Local IP (eth0): 172.30.30.125/24
       `Y$$b.                  Locale: en_CA.UTF-8
         `"Y$b._
~~~
## Setup
<details>
<summary>1. Flashing the operating system to microSD</summary>

1. Hardware Recommendations: A Class 10 microSD card larger than 16GB is recommended. Note that capacities above 2TB are not supported due to Master Boot Record (MBR) limitations. Use the [Raspberry Pi Imager](https://www.raspberrypi.com/software/) for a streamlined installation.

    * Device: Select `Raspberry Pi 4 Model B 8GB`[^1]
    * OS: Select `Raspberry Pi OS (other)` > `Raspberry Pi OS Lite (64-bit)`
    * Customization Settings:
      * Hostname: Set a single-word name using only letters, numbers, and hyphens. You can reach it at hostname.local, thanks to mDNS/Avahi, which is built into the OS
        * Avoid using a Fully Qualified Domain Name (FQDN) unless you have specific SSL requirements:
          * If you get an SSL certificate (via Let's Encrypt), having the FQDN set can make some automated tools slightly easier to configure
          * If you are mimicking a data center structure where every machine must belong to a specific sub-domain, especially with Public Certificates
      * Localization: 
        * Capital city: `Ottawa (Canada)`
        * Time zone: `America/Toronto`
        * Keyboard layout: `us`      
      * Credentials: Set a unique username and password
      * Connectivity: Configure Wi-Fi: SSID/password if required and enable SSH with password authentication
      * [Raspberry Pi Connect](https://www.raspberrypi.com/software/connect/)[^3]: Disable.
2. Hardware Connection: Insert the card and connect the power supply. A steady red LED indicates power, and the device will boot immediately.

    * Power Options:
      * An USB-C Power Supply (5V 3A recommended)
      * POE Injector (IEEE 802.3af/at)
      * PoE splitter that converts Ethernet to 5V USB-C    
3. Initial Access:

    * Connect via a 1GbE network cable, a micro-HDMI display and keyboard
    * Alternatively, connect via SSH using: `ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no user@hostname`
      * This explicitly instructs the SSH client to use Password Authentication first. If you have existing SSH keys on your computer, your computer will try to use those keys to log into the Raspberry Pi automatically. You'll get a "Permission denied (publickey)" error before ever giving you a chance to type a password.
      * You may need to run ssh-keygen -R hostname.local on your main computer to clear the old SSH key if you change the device hostname or IP address
</details>

<details>
<summary>2. Initial Pi setup</summary>

1. Login to the PI: `ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no myuser@hostname`
2. Install [Chezmoi](https://www.chezmoi.io/) via the [dotFiles repository](https://github.com/bhdicaire/dotFiles)[^4] to apply personal configurations: `sh -c "$(curl -fsLS get.chezmoi.io)" -- -b ~/.config/bin init --apply bhdicaire`
3. Upload your public key if it's not in the GitHub repository: `cat ~/.ssh/ops.pub | ssh myuser@hostname "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_key"`
    * Alternatively, you can add a host entry in your local SSH config: `vi /.ssh/config`
```bash
Host pi
    HostName hostname.local
    User myuser
    PreferredAuthentications password
    PubkeyAuthentication no
```
4. Log out of your SSH session: `exit`
</details>
<details>
<summary>3. Hardware Optimization</summary>

1. Login to the PI: `ssh myuser@hostname`
2. Hardware Optimization, edit `/boot/firmware/config.txt` to reduce power consumption and heat for headless configurations. This includes disabling on-board Wi-Fi and Bluetooth. Replace the current content: `sudo vi /boot/firmware/config.txt`
``` bash
# Core Settings
arm_64bit=1
arm_boost=1
disable_overscan=1

# Audio (Keep on if you need sound, otherwise comment out)
dtparam=audio=on

# Headless Optimizations
gpu_mem=16
# dtoverlay=vc4-kms-v3d
# camera_auto_detect=1
# display_auto_detect=1

# Hardware Disables (Assuming Ethernet usage)
dtoverlay=disable-wifi
dtoverlay=disable-bt

[cm4]
otg_mode=1

[all]
enable_uart=1
```

3. Restart the device: `sudo reboot`
</details>
<details>
<summary>4. Firmware Update</summary>

1. Login to the PI: `ssh myuser@hostname`
2. Firmware Updates, check the EEPROM status using `sudo rpi-eeprom-update` and apply updates if necessary `sudo rpi-eeprom-update -a`.
3. Restart the device if you updated the firmware: `sudo reboot`
</details>
<details>
<summary>5. System Fine-Tuning</summary>

1. Login to the PI: `ssh myuser@hostname`
2. System Fine-Tuning with `sudo raspi-config`:
    * Ensure the tool itself is current: `8. update`
    * Change the hostname if required
    * Set your Timezone and WLAN Country[^5] if required : `5 Localisation Options` > `4 WLAN Country`
    * Expand the Filesystem, although the Imager usually do it: `6. Advanced Options` > `A1 Expand`
    * Set behaviour of GPIO case fan: `4 Performance` > `P3 Fan`
    * Set storage location for logs: `6. Advanced Options` > `A12 logging`> `2 Volatile`
| Mode | Storage Type | Pros| Cons|
|:--|:--|:--|:--|:--|
| Default|SD Card|Persistent across reboots; logs are never lost.|High _write wear_ on microSD cards which can lead to card failure over time
| Volatile|RAM|Extremely fast; zero wear on the SD card.|All logs are deleted when the Pi is rebooted or loses power.
| Persistent|SD Card (Optimized)|Keeps logs through reboots.|Still causes physical wear to the SD card though often handled more efficiently
| None|Disabled|Zero disk usage.|Impossible to troubleshoot system crashes or errors after they happen
3. Temperature Monitoring with `watch -n 2 vcgencmd measure_temp` to measure in real-time
    * Idle temperatures between 35°C–45°C are excellent
    * Under load, temperatures should remain below 70°C if the passive cooling case is performing correctly
4. Check the current system information with `fastfetch`

Standard microSD cards usually lack the S.M.A.R.T. (Self-Monitoring, Analysis, and Reporting Technology) controllers found in larger drives. Some high-end SD cards like those from SanDisk or Kingston have proprietary ways to report health. If your card supports it, you can check the sysfs interface: `cat /sys/block/mmcblk0/device/life_time`
  * If this returns two values (e.g., 0x01 0x01), it represents a range of health in 10% increments
  * On most consumer-grade _Class 10_ cards, this file often doesn't exist or returns an empty value

MicroSD cards are designed to _fail-safe_ by locking themselves into Read-Only mode once their write cycles are exhausted to prevent data corruption. You can check if the kernel has flagged any such issues: `dmesg | grep -i "mmcblk`. Look for errors like `I/O error`, `Read-only file system`, or `Card stuck in recovery` indicating a worn-out card.

A card that is nearly full (e.g., 90%+) will wear out much faster than a card with plenty of free space. This is because the controller has fewer "fresh" cells to rotate through for wear leveling. Keeping a 32GB card at only 15% usage, is actually the best way to preserve its life. Check with `df -k`.
</details>

**Optional**

<details>
<summary>TCP/IP</summary>
{{< include f="unifi-static.md" >}}

{{< include f="debian-ipv6.md" >}}

<details>
<summary>Setting a Static IPv4 on Debian</summary>
{{< include f="debian-ipv4.md" >}}
</details>
</details>
<details>
<summary>Ghostty terminal emulator</summary>

Terminals applications, especially on macOS and Linux have a setting that automatically sends your local environment variables to the remote server. If you're using [Ghostty](ghostty.org) as your terminal emulator, it identifies itself to the Raspberry Pi as `xterm-ghostty` and you'll get: "Error opening terminal: xterm-ghostty".

My sessions stopped trying to be a Ghostty terminal and instead use the universal standard xterm-256color defined in the `~/.config/zsh/.zshrc/`.

I prefer to generate the _right_ locale setting. If you want to prevent this from happening with other future servers _without_ fixing the server side, you can tell your local computer to stop sending those variables by  editing `~/.ssh/config` or `/etc/ssh/ssh_config` and comment out the line `SendEnv LANG LC_*.` and restart the SSH service `sudo systemctl restart ssh`

If you want to keep using Ghostty’s specific advanced features such as specific keybindings, you can "upload" Ghostty's definition to the Pi from your local computer, not inside a SSH session with `infocmp -x xterm-ghostty | ssh username@hostname -- tic -x -`

</details>
<details>
<summary>Troubleshooting</summary>

The Raspberry Pi uses blinking LEDs to communicate system health status without the need for a display:
* Red LED (power indicator)
  * Solid: The system is receiving a steady 5V (specifically above 4.63V).
  * Flashing: Warning; the power supply is providing insufficient voltage (Under-voltage).
  * Off: The Pi has no power or has experienced a brownout.
* Green LED (Activity Indicator)
  * Solid: Typically occurs during the first second of boot. If it remains solid, the Pi cannot read the SD card.
  * Irregular Rapid Blinking: Normal operation; indicates the Pi is reading from or writing to the microSD card or EEPROM.
  * Regular Rhythmic Blinking: Indicates a boot failure:
    * 4 blinks: start.elf not found (SD card may be empty or corrupted).
    * 7 blinks: kernel.img not found (The OS is corrupted).
    * 8 blinks: SDRAM failure (Hardware issue).
</details>

[^1]: It's highly recommended to use a case with heatsinks or a fan, as the Pi 4 runs hot such as the [Miuzei Aluminiun Raspberry Pi Case Passive Cooling with Heatsink Cooler](https://www.amazon.ca/Miuzei-Raspberry-Aluminum-Conductive-Compatible/dp/B08LVRTYPD)
[^2]: Out-of-the-box access to your Raspberry Pi from anywhere in the world
[^3]: Refer to the [README.md](https://github.com/bhdicaire/dotFiles/blob/main/README.md) for details about the `run_once_before_install-packages.sh`
[^4]: This is vital for Wi-Fi stability, fortunately I prefer wired connection
