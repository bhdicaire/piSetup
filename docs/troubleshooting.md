# Troubleshooting

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

## Hostname

When you use the Raspberry Pi Imager to pre-configure a hostname, it doesn't just write to /etc/hostname once. On Debian 13, it uses a tool called cloud-init. Every time the Pi boots, it checks `/boot/firmware/user-data`, and override `/etc/hostname`. Refer to `/etc/cloud/cloud.cfg` for more details.

You should disable cloud-init to stop running on subsequent boots: `sudo touch /etc/cloud/cloud-init.disabled` and set the current session hostname `sudo hostnamectl set-hostname pi-01`.

## Printer Server
1. Check the Common UNIX Printing System (CUPS) 
  * Check the service status: `sudo systemctl status cups` and errors `sudo journalctl -u cups -n 50 --no-pager`
  * Is it listening to the network or just the localhost ? `sudo netstat -tulpn | grep 631`
  * Can you see  the "Home" page of CUPS with your preferred web browser: `http://172.30.80.22:631`
2. Check printers
  * Check the local USB printers: `lpinfo -v | grep usb`
  * Check the "Sharing" Status: `lpstat -p -d`
  * Check if sharing is globally enabled: `cupsctl | grep remote`. If you see _remote_admin=1 and _remote_any=1, run: `sudo cupsctl --remote-any --share-printers`
3. Check Avahi (Bonjour)
  * Check the service status:`sudo systemctl status avahi-daemon`
  * If it's active, does it see the CUPS services: `avahi-browse -rt _ipp._tcp`
  * Can you see the printers via Bonjour on your Mac ? `dns-sd -B _ipp._tcp .` or with avahi tools on Mac `avahi-browse -rt _ipp._tcp`
4. Is there a firewall on the print server?  
  * Check ufw: `sudo ufw status`
  * Check nftables (e.g., the Debian 13 default): `sudo nft list ruleset | grep 631`
  * Check your /etc/cups/cupsd.conf. It needs to explicitly allow your network.
5. Try to kick the service back to life
  * `sudo systemctl enable cups`
  * `sudo systemctl restart cups


INgredients:
Common
* Generate UTF-8 Locale
* Install Sudo and dependencies
* Configure timezone and host name
* Create Ops user, copy SSH public key, allow ops passwordless sudo
*  Harden SSH — disable root login and password auth
Install chezmoi binary & dotFiles

Network
 * Disable IPv6
 * Configure IPv4 static address on VLAN 80 Interface

PrintServer
* Install CUPS and Avahi to be available on 172.30.*.*
* Configure CUPS with PTouch and Dymo label printers
