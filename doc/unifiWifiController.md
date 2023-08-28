# Ubiquit Unifi WIFI Controller

Your Unifi Controller should be running 24/7, it will capture network's activity, update your components and provide insight on the performance. Ubiquiti have a wide range of options for setting up a WIFI  Controller: you can purchase a Ubiquiti cloud key, run a container on your infrastructure or install the package on a computer.

The controller provide the same capability no matter the implementation chosen. It gives the user access to all of their network's activity.

This document is intended to my future self to help me debug my network ecosystem and reinstall an Unifi WIFI Controller on a Raspberry Pi.

## Acquisition list
 * Raspberry PI (Model 3B or 4B) with a case
 * MicroSD card such as the [SanDisk Ultra 64GB microSDXC UHS-I Card](https://www.amazon.ca/gp/product/B07B7QZHVF/) although you can use a 4GB card with the OS lite version
 * USB SD card reader
 * Ethernet PoE Splitter 802.3af (5V 2.4A) footnote1
   * ANVISION [Gigabit PoE Splitter 802.3af](https://www.amazon.ca/gp/product/B07PYZQBK8) with Micro USB
   * UCTRONICS [PoE Splitter 802.3at/af](https://www.amazon.ca/dp/B087FBNCN6 )with USB C

I [purchased](https://www.amazon.ca/gp/product/B07G3811BQ) a [YuanLey 8 Port Gigabit PoE Switch](https://www.yuanley.com/products/yuanley-8-port-gigabit-poe-switch,-8-poe-ports-1000mbps,-120w-8023af-at,-metal-fanless-unmanaged-plug-and-play?VariantsId=10031) early 2019 to remove several power supply from my rack and my desk. You can't beat a metal fanless unmanaged device, there is elegance in simplicity.

At first, I was using a PoE (Power over Ethernet) HAT (Hardware Attached on Top) to power the Raspberry Pi via my Ethernet network. Most cases are not designed to support accessories including an add-on board to reduce fan failure and the noise. A third party fan such as [Noctua](https://noctua.at/) is quieter and perform better although I don't need the aggravation when a passive-cooling solution works.

An Ethernet PoE Splitter is my _current solution_, this is is cheaper and more convenient. Keep in mind that you need to check the safety certifications anf the stated network performance (e.g., Gigabit) are provided. After all, PoE did result in some accidents...

## Preparation is key to success

My generic Raspberry PI installation process is [documented here](../raspberryPiSetup.md).

> Don't forget to pick the current [Raspberry Pi OS Lite (32 bits)](https://www.raspberrypi.com/software/operating-systems). The solution is not compatible with the 64 bit operating system __yet__, don't hold your breath.

Always use an ethernet cable, life is too short to mess with WIFI – especially for a network controller.

I don't use containers on __limited__ device such as the Raspberry PI 3b+. I just add the Debian repository from the manufacturer, and use the Advanced Package Tool (APT) to install all the required packages.

1. Install the headless versionJava Runtime Environment (JRE)
  * sudo apt install openjdk-11-jre-headless

During the installation process, you might get an error such as “unexpected end of file or stream” and “abort-upgrade: please reinstall the previous version.” Run `sudo apt-get clean`to delete corrupt files and download the package again.

2. Add entropy to improve the startup time of the UniFi controller software, fortunately the Pi has an Random Number Generator (RNG)
  * `sudo apt install rng-tools`
  * `sudo vi /etc/default/rng-tools`
  * Find the line that begins with `#HRNGDEVICE=/dev/hwrng`
  * Save the file
  * Start the service: `sudo systemctl restart rng-tools`

3. Add the ubiquity unfi repository and the trusted keys
 * `sudo wget -O /etc/apt/trusted.gpg.d/unifi-repo.gpg https://dl.ui.com/unifi/unifi-repo.gpg`
 * `echo 'deb https://www.ui.com/downloads/unifi/debian stable ubiquiti' | sudo tee /etc/apt/sources.list.d/100-ubnt-unifi.list`
 * `sudo apt-get update && sudo apt-get upgrade -y

4. Add the MondDB repository and the trusted keys
 * wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
 *  echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
 * `sudo apt-get update && sudo apt-get upgrade -y

5 Install MondDB
 * sudo apt-get install -y mongodb-org
 * sudo systemctl enable mongod
 * sudo systemctl start mongod

6. Install Unifi Controller a service that will automatically start at boot
 * sudo apt install unifi

Confirm my PI's current IP address using `hostname -I`to connect to the UniFi Controller Web Portal using my Mac:
 * https://192.168.86.86:8443
 * The browser will warn you that your connection is not private due to security certificate, proceed anyway
 * Use your UI SSO account or click on “Switch to Advanced Setup” and create a local account
 * Confirm backup and optimize network automatically
 * Pick un-adopted devices, If you are moving your devices from a previous controller, you will need to “forget” them first in the old controller
 * You are good to go and configure your networks, have fun!

Keep your Ubiquiti's products and your network up to date to fix bugs and boost performance.

As you know updates or __stuff__ might brick your controller and the Raspberry Pi thus backup is not an option. Everything is on the SD Card: the operating system and the Ubiquiti Unifi Wireless Controller. Refer to the [Raspberry PI Maintenance](../raspberryPiMaintenance.md) to learn about SD card backup and restore.
