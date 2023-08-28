# Raspberry PI Setup

I prefer a headless setup remotely via the command line interface without attaching a display, a keyboard and a mouse.

As usual, this document is intended to my future self to help me debug my network ecosystem and install additional Pi.

## Preparation is key to success

Being prepared is better than being lucky. Don't get fancy, build and test the unit with a display, keyboard, a trusty SD card and a regular power supply. You can save tons of times, wondering what's wrong with your setup.

Always use an ethernet cable, life is too short to mess with WIFI – especially for a network controller. Ensure that you have a sudo enabled account named pi with raspberry as password. You can thank me later.

Download the operating system: [Raspberry Pi OS Lite (64 bits)](https://www.raspberrypi.com/software/operating-systems).

Install the following items on your Mac:
1. [BalenaEtcher](https://etcher.balena.io/) to flash the OS images to the SD cards
  *  `brew install --cask balenaetcher`
2. GNU Core Utilities
    * `brew install coreutils`
2. [NMap](https://nmap.org/), a network mapper to identify the allocated DHCP IP address used by the Raspberry
    * `brew install nmap`

The Raspberry Pi OS is written to a partition with an ext4 journaling file system. This is a native file system for Linux operating systems thus `bootfs` and `rootfs` partitions will be mounted automatically in read / write mode when you insert the flashed Micro SD card.

Handling an EXT4 partition with MAC is no fun, you'll need a Linux virtual machine.

Flash the OS image to a SD with [BalenaEtcher](https://etcher.balena.io/) or DD `sudo gdd if=raspberryOS.img of=/dev/rdisk4 status=progress bs=16M`.

Stick with the official supported operating system: Raspberry Pi OS also known as Raspbian. The current release is [Bullseye](https://www.raspberrypi.com/software/operating-systems/).

Boot your linux virtual machine and insert the the flashed SD card. Execute the following items to complete the headless setup:
 1. SSH is not enabled by default, place an empty file named ssh
   * `touch /volumes/bootfs/ssh`
 2. There is no user account, place a file with a single line of text with the username:encrypted password
   * `vi /volumes/bootfs/userconf`
   * content is `pi:$6$RDYHLMpEOHsozpZH$Hxb9ROGzwLLUdrJAesUofoHt0loATgOoBl4EE58grohrx2CEvfBZvRAMZ1UQ7AcRV/E1L8nVQwuPiUCfNTKdK/`
   * use `echo 'anotherPassword' | openssl passwd -6 -stdin`to create a strong password although you'll change that later in the process

Shutdown the virtual machine and eject the SD card.

## Initial Setup

Insert the flashed SD card and boot the Raspberry Pi. The lights should be on the device including the network interface card, on the Ethernet PoE Splitter, and the network port on the PoE Switch. If this is not the case, something is wrong. Check the connections and run the usual troubleshooting.

It's time to identify the Raspberry's IP with NMap on your subnet:
 * `sudo nmap -sn 192.168.86.0/24`
2. Connect via SSH, the username is pi and the password is raspberry:
 * `ssh pi@192.168.86.76` or `ssh pi@raspberrypi.local` if you're lucky

If you don't remember the account password or something went wrong while pasting my encrypted password earlier in the process – initiate plan B.
 * Power down and pull the SD card out from your Pi
 * Boot your linux virtual machine and insert the the flashed SD card
 * `vi /volumes/rootfs/passwd
 * Find the line that begins with `pi:x:1000:1000...`
 * Get rid of the x; leave the colons on either side. This will eliminate the need for a password
 * Save the file
 * Shutdown the virtual machine and eject the SD card
 * Put the SD card back in the Pi and boot

Either go back to activity #1 to connect via SSH or connect a keyboard and monitor to connect in terminal mode. The username is still pi and the password now empty.

### Configuration on First Boot

Modify settings according to my opinionated preferences, all the current settings can be found in /volumes/bootfs/overlays/README`:
   * `vi /volumes/bootfs/config.txt`
   * Go at the bottom before `[all]` and add two lines to disable bluetooth and wireless network
     * `dtoverlay=disable-bt`
     * `dtoverlay=disable-wifi`
   * Go at the bottom after `[all]` and add a line to reduce the allocated video memory after all there is no graphical user interface (GUI)
     * `gpu_ mem=16`

Several settings are easier to change via Raspi Config, you can move through the menus using the keyboard arrows and use the `Enter` or`Back` buttons, you can also use [enter] or [escape] keys:
 * `sudo raspi-config`
 *  Select 1. System Options
   *  Select S4 Hostname to change the name of this Raspberry Pi on the network to prevent name conflicts
 *  Select 4. Performance Options
   *  Select P2 GPU Memory to reduce the allocated video memory after all there is no graphical user interface (GUI)
   * Pick 16, if you record videos at high resolution pick 256
 *  Select 5. Localisation Options
   * Choose to set your (`L1 Locale`), `L2 Timezone`, and `L3 Keyboard` if needed
   *  Select L4 Wlan Country to ensure that your wireless connection work properly
     * Pick Canada
 *  Select 6. Advanced Options
   *  Select A1 Expand Filesystem to be able to use all the space of the memory card
 * Click Finish and then `yes` to reboot when asked

### Additional Configuration

You have now enabled stuff and can continue the setup. Refer to the [vendor documentation](https://www.raspberrypi.com/documentation/computers/getting-started.html) or [Stack Overflow](https://raspberrypi.stackexchange.com/) if something does not jive.

Connect to Raspberry PI and complete the following items:
1. Update the list of packages and then update the packages currently installed to the latest version:
 * `sudo apt-get update && sudo apt-get upgrade -y`
2. Install VIM:
 * `sudo apt-get install vim -y`
3. Check if SSH is enabled
 *  service --status-all
 * If not, run `sudo service ssh start`
4. Check the configuration:
 * `raspinfo` for an overview
 * `netstat -nr` for network interfaces
5. Static IP adress
 * ifconfig eth0 192.168.45.12 netmask 255.255.255.0

 interface eth0
metric 300
static ip_address=192.168.0.40/24
static routers=192.168.0.1
static domain_name_servers=192.168.0.1

interface wlan0
metric 200

6. reboot
 * `sudo reboot`

Connect to Raspberry PI and complete the following items:
1. Change Pi's password
 * passwd pi
2. Change Pi's default group
 * sudo chgrp -R users pi
3. Add a user
 * sudo adduser bhdicaire  --ingroup users
 * sudo adduser bhdicaire  sudo
 * Not able to add a user to multiple existing groups:
   * `sudo adduser bhdicaire dialout cdrom audio video plugdev games users input render netdev spi i2c gpio`
4. Delete or rename a user if required
 * `sudo deluser --remove-home <username>`
 * `sudo rename-user`

https://raspberrypi.stackexchange.com/questions/70214/what-does-each-of-the-default-groups-on-the-raspberry-pi-do

The default user account own the UID 1000 and is a member many groups to support all the use cases. You don't need to provide the same membership for all users account.

| Group| Description|
| :---  | :--- |
| adm|Allows access to log files in /var/log and using xconsole|
| audio|Allows access to audio devices like microphones and soundcards|
| cdrom|Allows access to optical drives|
| Dialout|Allows access to serial ports/modem reconfiguration|
| games|Games usually use this group to write high score files|
| gpio|Allows GPIO pin access|
| i2c|Allows I2C access, usually generated after installing i2c-tools|
| input|Allows access to /dev/input/mice|
| lpadmin|Enable CUPS printer administration|
| netdev|Enables access to network interfaces|
| pi|User-specific group, a group is usually created for each new user|
| plugdev|Enables access to external storage devices|
| spi|Enables access to the SPI bus|
| sudo|Enables sudo access|
| users|Allows access to /opt/vc/src/hello_pi|
| video|Allows access to videocard's frame buffer and webcam – required to run startx (e.g., XWindows)|
