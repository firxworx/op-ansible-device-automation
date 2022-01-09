# olivia-party - ansible-driven automated configuration of kiosk PC/SoC/SBC hardware

This project implements [ansible](https://www.ansible.com/) roles + playbooks that automate the configuration of intel-based PC/SoC/SBC's running [Ubuntu](https://ubuntu.com/) 20.04 to become touch-screen kiosks that boot directly to a web app / URL run by the [Chromium](https://www.chromium.org/) web browser in kiosk mode.

The code was created as part of the [OliviaParty](https://olivia.party) project. It is designed to be easily extensible and customizable to support a wide range of potential applications. It is well-commented and the ansible devops automation platform is [well-documented](https://docs.ansible.com/).

Alternative OS's, architectures, displays, and peripherals can be supported with the installation + configuration of additional packages.

The code is released under the Apache 2.0 open-source license. Use it at your own risk!

## Project Details

[OliviaParty](https://olivia.party) enables a user with severe physical disabilities to interact with a custom web application using touchscreen, joystick, and/or physical button inputs.

The ansible automation was developed for an intel-based [UP Board](https://up-board.org/) single-board-computer (SBC) running a server install image of [Ubuntu 20.04 LTS](https://releases.ubuntu.com/20.04/).

The kiosk setup is completed by an ASUS touchscreen that runs at 1366x768 resolution and identifies as "ELON Touchscreen", a generic USB joystick (gamepad), and a USB external audio adapter.

## Features

- Chromium is started in kiosk-mode alongside a number of cli tweaks that "lock" the user into the web app
- Minimal footprint install consisting of xserver + Chromium; graphical environments such as gdm3 are omitted to save on resources
- The configured device supports USB input devices, audio output, and text-to-speech

## Dependencies

This project is intended for a linux/unix environment. Windows users can run this project using WSL2 to run a linux environment.

Control Machine Dependencies:

- python3
- ansible

Target Machine Dependencies/Requirements:

- intel-powered hardware running a clean install of Ubuntu 20.04 server with openssh
- user with sudo privileges that is able to authenticate over ssh via password authentication

## Running the Project Playbook

The default configuration assumes deployment on a private and secured internal network. It is not appropriately secure for public-facing or production deployments. Use at your own risk.

### Inventory

Revise `inventory/hosts` to specify your target board(s): replace the list under the `[intelboards]` section with the IP address(es) or FQDN(s) of your target device(s). Target device(s) must be accessible by your host/control machine over the network.

Targets are assumed to be running a clean install of Ubuntu 20.04 server (no GUI), including an ssh server configured to support password authentication by the user that ansible will use to connect to each target.

Replace the `ansible_user` and `ansible_become_password` under the `[intelboards:vars]` sections with the credentials of a user that exists on the target(s) and has root privileges via sudo.

Take care not to commit any sensitive credentials into git. You may wish to implement key-based authentication and additional sudo configuration to eliminate the need for specifying `ansible_become_password`.

### Customize Roles & Scripts

Review the playbooks (`playbooks/`) and roles they invoke (`roles/`) to understand what they do prior to executing them. Revise them as appropriate to suit the needs of your hardware and project requirements.

Key script templates are located in `roles/kiosks/templates`.

The `launch-browser.sh.j2` template is for the bash script that launches the chromium web browser. It is likely that you will need to modify this script if you are running different hardware.

### Run the Playbook

Use the `ansible-playbook` command to execute the kiosk playbook:

```sh
ansible-playbook kiosk.yml
```
