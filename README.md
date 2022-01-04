# olivia-party - ansible-driven automated configuration of kiosk soc/sbc hardware

This project implements ansible roles + playbooks that automate the configuration of an intel-based SoC/SBC/computer to become a touch-screen kiosk that boots directly into a web application run by the Chromium web browser.

The project was developed with an intel-based [UP Board](https://up-board.org/) running [Ubuntu Server 20.04 LTS](https://releases.ubuntu.com/20.04/) connected to an ASUS touchscreen that uses device name "ELON Touchscreen".

Other hardware and alternative architectures such as ARM may be supported with the installation + configuration of additional packages.

The project board is mated to a USB joystick interface and runs the "Olivia Party" accessibility project. The kiosk provides a custom hardware interface (including a touchscreen, joystick, and physical buttons) to a custom web application designed for a user with severe physical disabilities.

The roles and playbooks are designed to be easily extensible to support a wide range of potential applications.

The code is released under an open-source license. Use at your own risk!

## Features

- Kiosks boot into Chromium running in kiosk-mode plus a number of cli tweaks that "lock" the user into the app
- Minimal footprint consisting of xserver + Chromium that omits graphical environments such as gdm3 to save on resources
- Kiosks support USB input devices, audio output, and text-to-speech

## Dependencies

This project is intended for a linux/unix environment; Windows users may run the playbooks in a linux distro powered by WSL2.

Control Machine Dependencies:

- python3
- ansible

Target Machine Dependencies:

- intel-powered hardware running a clean install of Ubuntu 20.04 server with openssh
- user with sudo privileges that can authenticate over ssh via password authentication

## Running the Project Playbook

### Inventory

Revise `inventory/hosts` to specify your target board(s): replace the list under the `[intelboards]` section with the IP address(es) or FQDN(s) of your target device(s).

All target devices must be accessible by your host/control machine over the network.

Targets are assumed to be running a clean install of Ubuntu 20.04 server (no GUI), including an ssh server configured to support password authentication by the user that ansible will use to connect to each target.

Replace the `ansible_user` and `ansible_become_password` under the `[intelboards:vars]` sections with the credentials of a user that exists on the target(s) and has root privileges via sudo.

### Run the Playbook

Review the playbooks (`playbooks/`) and roles they invoke (`roles/`) to understand what they do prior to executing them. Revise them as appropriate to suit the needs of your project.

Use the `ansible-playbook` command to execute playbooks:

```sh
ansible-playbook kiosk.yml
```
