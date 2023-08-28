# Raspberry PI Maintenance

The Advanced Package Tool (APT) handle the installation and removal of software components on Debian-based Linux distributions such as Raspberry Pi OS.

Everything including Raspberry Pi's firmware are installed as a Debian package and updated automatically with APT. A stable computer require regular updates:
 * `sudo apt-get update && sudo apt-get upgrade -y && sudo apt-get autoremove -y` followed by `sudo reboot`
  * `sudo apt-get full-upgrade` might be required for significant update

During the installation process, you might get an error such as “unexpected end of file or stream” and “abort-upgrade: please reinstall the previous version.” Run `sudo apt-get clean`to delete corrupt files and download the package again.

## Backup and restore SD card





> https://github.com/azlux/log2ram/blob/master/README.md

Backup / restore SD card
diskutil unmountDisk disk4
sudo dd if=/dev/rdisk4 of=debian64.dmg

Or, with `gzip`, to save a substantial amount of space
sudo gdd if=/dev/rdisk4 of=sd_backup.dmg status=progress bs=16M

```
sudo dd if=/dev/rdisk1 bs=1m | gzip > /path/to/backup.gz
```

And, to copy the image back onto the SD:

```
gzip -dc /path/to/backup.gz | sudo dd of=/dev/rdisk1 bs=1m
```
