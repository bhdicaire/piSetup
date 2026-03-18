# ![Logo](docs/header.png "Logo")

![GitHub Stars](https://img.shields.io/github/stars/bhdicaire/piSetup?style=flat-square&logoColor=186ADE&labelColor=3E5462&color=C25100)
![GitHub forks](https://img.shields.io/github/forks/bhdicaire/piSetup?style=flat-square&logoColor=186ADE&labelColor=3E5462&color=C25100)
![GitHub Last Commit](https://img.shields.io/github/last-commit/bhdicaire/piSetup?style=flat-square&logoColor=186ADE&labelColor=3E5462&color=C25100)
![GitHub licence](https://img.shields.io/github/license/bhdicaire/piSetup?style=flat-square&logoColor=186ADE&labelColor=3E5462&color=C25100)

An [Ansible](https://www.redhat.com/en/ansible-collaborative) playbook for configuring one or more [Raspberry Pi](https://www.raspberrypi.com/products/) devices running [Raspberry Pi OS Lite](https://www.raspberrypi.com/software/operating-systems/) (64-bit).

> **Why does this exist?** Setting up a Pi from scratch involves a lot of steps that are easy to forget. This playbook automates the configuration so that every device ends up in a known, reproducible state — whether it's your first Pi or your fifth.

## What it configures
←
→
**Common** (`roles/common`)

- Generate UTF-8 locale
- Install common dependencies not included with [Raspberry Pi OS Lite](https://www.raspberrypi.com/software/operating-systems/)
- Configure timezone ← `group_vars/all/infrastructure.yml`
- Configure hostname ← `host_vars/<hostname>.yml`, you can use `host_vars/pi-01.yml` as template
- Create an `ops` user, install your SSH public key, and grant passwordless sudo
- Disable root login and password-based SSH authentication

**Network** (`roles/network`)

- Disable IPv6
- Configure a static IPv4 address on a VLAN-tagged interface ← `host_vars/<hostname>.yml`

**Print Server** (`roles/printServer`)

- Install [CUPS](https://www.cups.org/) and [Avahi](https://avahi.org/) for network printer discovery
- Configure [Brother P-Touch](https://www.brother.ca/en/computer-connected-label-makers/c/pr-labelling-cpuConnected) and [Dymo](https://www.dymo.ca/label-makers-printers/labelwriter-label-printers/) label printer drivers

**Dotfiles** (`roles/chezmoi`)

- Bootstrap [chezmoi](https://www.chezmoi.io/) and apply personal configuration from your [dotFiles repository](https://github.com/bhdicaire/dotFiles)

## :rocket: Quick Start

### 1 — Flash and boot the Pi

Follow the [manual setup guide](docs/process.md) to flash the OS, set credentials, and get the device on the network. Come back here once you can SSH in.

### 2 — Configure the inventory

Edit `inventory.ini` to match your device's hostname or IP address:

```ini
[pi_nodes]
pi-01 ansible_host=192.168.1.50   # ← change this
pi-02 ansible_host=192.168.1.51   # ← change this
```

### 3 — Set global and host-specific variables

Copy the infrastructure vars file and adjust it for your ecosystem:
```bash
cp group_vars/all/infrastructure.yml group_vars/all/ecosystem.yml
```
Edit the new file to set the correct values. Refer to the comments inside the file for guidance.


Copy the example host vars file and adjust it for your device:

```bash
cp host_vars/pi-01.yml host_vars/<your-hostname>.yml
```

Edit the new file to set the correct static IP, VLAN interface, and any other device-specific values. Refer to the comments inside the file for guidance.

### 4 — Enable the roles you need

Open `piSetup.yml` and uncomment the roles you want to apply:

```yaml
- name: Raspberry Pi Configuration
  hosts: pi_nodes
  roles:
    - common        # Always recommended
    - network       # Static IP + disable IPv6
    # - printServer # CUPS + label printer support
    # - chezmoi     # Personal dotfiles
```

### 5 — Run the playbook

```bash
ansible-playbook piSetup.yml
```

For the very first run, Ansible needs to authenticate with a password before SSH key-based access is configured:

```bash
ansible-playbook piSetup.yml --ask-pass
```

You can also limit execution to a single host or a specific role using tags:

```bash
ansible-playbook piSetup.yml --limit pi-01
ansible-playbook piSetup.yml --tags network
```

## Project structure

```
piSetup/
├── ansible.cfg            # Ansible defaults (SSH, privilege escalation)
├── inventory.ini          # List of Pi devices
├── piSetup.yml            # Main playbook
├── group_vars/all/        # Variables shared across all hosts
├── host_vars/             # Per-device overrides (IP, VLAN interface, etc.)
├── roles/
│   ├── common/            # Base system configuration
│   ├── network/           # Static IP and IPv6 hardening
│   ├── printServer/       # CUPS + label printers
│   └── chezmoi/           # Dotfiles bootstrap
└── docs/
    ├── process.md         # Manual setup guide (flash, hardware, firmware)
    └── troubleshooting.md # LED codes, common errors, diagnostic commands
```

## Documentation

- [Manual guide](docs/guide.md) — step-by-step guide to flash the OS, optimize hardware, and update firmware before running Ansible
- [Troubleshooting](docs/troubleshooting.md) — LED status codes, SSH errors, SD card health, and common issues

## Contributing

[Suggestions](https://github.com/bhdicaire/piSetup/issues) and pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change. Refer to the [contribution guidelines](.github/CONTRIBUTING.md) and adhere to this project's [code of conduct](.github/CODE_OF_CONDUCT.md).

## License

*piSetup* by Benoît H. Dicaire is shared under the [MIT license](LICENSE).
