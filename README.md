# ![Logo](docs/header.png "Logo")

![GitHub Stars](https://img.shields.io/github/stars/bhdicaire/repositoryTemplate?style=flat-square&logoColor=186ADE&labelColor=3E5462&color=C25100)
![GitHub forks](https://img.shields.io/github/forks/bhdicaire/repositoryTemplate?style=flat-square&logoColor=186ADE&labelColor=3E5462&color=C25100)
![GitHub Last Commit](https://img.shields.io/github/last-commit/bhdicaire/repositoryTemplate?style=flat-square&logoColor=186ADE&labelColor=3E5462&color=C25100)
![GitHub licence](https://img.shields.io/github/license/bhdicaire/repositoryTemplate?style=flat-square&logoColor=186ADE&labelColor=3E5462&color=C25100)

This is an [ansible](https://www.redhat.com/en/ansible-collaborative) playbook for configuring a [Raspberry Pi](https://www.raspberrypi.com/products/)  device.

[Raspberry Pi OS Lite](https://www.raspberrypi.com/software/operating-systems/) is an ARM64 port of Debian that _excludes a desktop environment_ to remain lightweight. It utilizes specific, non-free proprietary firmware found in packages such as `raspberrypi-bootloader` and `firmware-brcm80211` to manage the GPU, Wi-Fi, and Bluetooth. Consequently, the firmware-linux-free package is not required.

## What problem does it solve and why is it useful?

Setup one or several devices with easy-to-understand instructions that automate the installation and configuration to ensure that _everything_ is configured properly.

Obviously, I don't remember _everything_, refer to my [manual documented process](docs/process.md). 

## :rocket: Setup
<details>
<summary>Flashing the operating system to microSD</summary>

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
<summary>Ansible playbook</summary>
`ansible-playbook piSetup.yml`

</details>
### [Suggestions](https://github.com/bhdicaire/piSetup/issues) and pull requests are welcome :grin:

For major changes, please open an issue first to discuss what you would like to change. Refer to the [contribution guidelines](.github/CONTRIBUTING.md) and adhere to this [project's code of conduct](./.github/CODE_OF_CONDUCT.md).

## _piSetup_ by Benoît H. Dicaire is shared with an [MIT license](https://github.com/bhdicaire/piSetup/raw/main/LICENSE).
