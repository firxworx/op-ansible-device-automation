# \[ olivia-party \] olivia-hardware-ansible

## Automated configuration of touchscreen kiosks

The [Ansible](https://www.ansible.com/) roles + playbooks in this repository automate the configuration of commodity PC/SoC/SBC hardware to become hackable and extensible browser-based touchscreen kiosks.

The code is well-commented and released under open source license so that it may be extended in support of a wide range of potential use-cases.

Features:

- Chromium is started in kiosk-mode alongside a number of cli tweaks that "lock" the user into the web app
- Minimal footprint install consisting of xserver + Chromium; graphical environments such as gdm3 are omitted to save on resources
- The configured device supports USB input devices (joysticks, keyboards, etc), audio output, and text-to-speech

A number of blog articles and forum threads exist that detail how to configure linux-based SBC's such as Raspberry Pi to become web kiosks. This project implements a recipe that works, and thanks to ansible, it is executable via a single command.

## Introduction

This repository is home to the documentation and configuration of hardware devices related to the [OliviaParty](https://olivia.party) project. The project aims to enable the creation of cost-effective custom accessibility solutions for users with disabilities.

One of the project's working hardware prototypes is a kiosk-like device that enables a user with severe physical disabilities to interact with a custom web application using inputs that include a touchscreen, joystick, and buttons.

The OliviaParty device boots directly into a custom web application (URL) run by the open-source [Chromium](https://www.chromium.org/) web browser started in _kiosk mode_.

A number of additional configuration tweaks are implemented that disable various features and behaviours of Chromium that might interfere with the kiosk use-case. For example, the _swipe to go forward or back_ feature is disabled because it risks disrupting gesture-support implemented by web apps, and because it increases the risk that a user can "break out" of the kiosk UI.

## Supported Hardware

The ansible playbooks as-implemented should broadly support intel-based devices running a clean server installation of the free linux-based [Ubuntu](https://ubuntu.com/) 20.04 LTS or 22.04 LTS operating system.

Additional architectures, hardware profiles, and/or software can be supported with additional customization.

### Project Hardware Profile

The specific hardware run by this project is as follows:

- [UP Board](https://up-board.org/) x86 single-board-computer (SBC) powered by Intel Atom
- ASUS touchscreen w/ HDMI + USB interfaces that identifies itself to the OS as "ELON Touchscreen" and runs at 1366x768 resolution
- Generic USB external audio adapter with advertised support for Linux
- Generic USB joystick/gamepad controller with an arcade-style Joystick and buttons
- Generic USB mouse + keyboard (optional)

No special drivers are required to use this particular touch screen. The OS recognizes it as a pointing device when connected via USB (similar to a USB mouse). The touchscreen is used is in landscape orientation. Portrait orientation can be supported by further customizing the x server configuration.

## About Ansible

Ansible is a popular free and open-source IT automation platform that enables hardware configurations to be defined in a clean and declarative yaml-based syntax that provides the benefit of self-documenting code.

The Ansible software can be installed Win/Mac/Linux computers to enable them to serve as a _control node_ capable of executing playbooks against an _inventory_ of target devices. Ansible connects to the target devices using [ssh](https://en.wikipedia.org/wiki/Secure_Shell).

Ansible and its ecosystem of powerful modules are [well-documented](https://docs.ansible.com/). Refer to the [Getting Started](https://docs.ansible.com/ansible/latest/user_guide/index.html#getting-started) guide to learn core concepts and how to install it on your system.

## Running the Project

Follow the steps below to deploy your own touchscreen kiosk.

> Security Note / Disclaimer: this project assumes that the kiosk will be deployed in a physically private and secure environment on a private internal network. Public-facing, internet-facing, or otherwise production-grade device deployments are not part of the scope of this project. Use at your own risk.

### Prerequisites

#### On your control machine

This project assumes the control machine is a linux/unix system or MacOS. Windows PC's are supported via WSL2, Microsoft's official _Windows Subsystem for Linux_.

- Ensure that python3 is installed (`which python3` should return output)
- Install [Ansible](https://docs.ansible.com)
- Verify that you can successfully execute `ansible --version` in your Terminal (command line) app
- Clone or download the source code from this repo
- Ensure that you have SSH installed
  - Generate an SSH key pair if you wish to connect to your target devices using key-based authentication

#### On your target devices

- Plug in all required USB hardware devices/peripherals to the device
- Install a fresh server (or _minimal_) install image of the [Ubuntu 20.04 LTS](https://releases.ubuntu.com/20.04/) or [Ubuntu 22.04 LTS](https://releases.ubuntu.com/20.04/) operating system
  - Ensure that an ssh server (OpenSSH) is installed
  - Create a non-root user account that has root privileges via `sudo` (this can be done during the install process)
  - Ensure the user can authenticate over ssh via password authentication (or configure key-based authentication with your control machine)
- Ensure the control machine can connect to your target device(s) via the network

### Ansible Inventory

Revise `inventory/hosts` to specify your target board(s). Replace the list under the `[intelboards]` section with the IP address(es) or FQDN(s) of your target device(s).

Targets are assumed to be running a clean install of Ubuntu 20.04 server (no GUI), including an ssh server configured to support password authentication by the user that ansible will use to connect to each target.

Replace the `ansible_user` and `ansible_become_password` under the `[intelboards:vars]` sections with the credentials of a user that exists on the target(s) and has root privileges via sudo.

Take care not to commit any sensitive credentials into git. You may wish to implement key-based authentication and additional sudo configuration to eliminate the need for specifying `ansible_become_password`.

### Customize Roles & Scripts

Review the playbooks (`playbooks/`) and roles they invoke (`roles/`) to understand what they do prior to executing them. Revise them as appropriate to suit the needs of your hardware and project requirements.

Key script templates are located in `roles/kiosks/templates`.

The `launch-browser.sh.j2` template produces the bash script that launches the chromium web browser. If you are running different hardware vs. the _Project Hardware Profile_ detailed above it is likely that you will need to modify this script.

### Run the Playbook

Use the `ansible-playbook` command to execute the kiosk playbook:

```sh
ansible-playbook kiosk.yml
```

## Adding Displays

System configuration files for x can be referenced at the following paths:

```sh
# note: this file may not exist on your system - this is OK
cat /usr/lib/X11/xorg.conf

# conf files in this folder are used by x
cd /usr/share/X11/xorg.conf.d
```

Host-specific overrides to the system configuration should be added to:

```sh
/etc/X11/xorg.conf.d
```

Generally you should not make revisions to folders or files under `/usr/lib`.

## Troubleshooting Displays

When the `~/launch-browser.sh` script runs, it saves the output of useful troubleshooting commands to log files in the: `~/kiosk-debug/` folder.

- `~/kiosk-debug/screens.debug.log`: `xrandr` output showing connected displays + ports
- `~/kiosk-debug/xinput-list.debug.log`: `xinput` output showing detected input devices for x server

If the kiosk fails to boot into chromium with the target app/website running full-screen at the proper resolution, and/or drops back to the Terminal CLI, the log for the x session can be found in:

`~/.local/share/xorg/Xorg.0.log`

The `lshw` (list hardware) command can provide more information about your graphics card:

```sh
sudo lshw -c display
```

### Troubleshooting Flat-Panel TV's + Large Displays

If alignment of the picture is off (e.g. a small portion of the picture is cut off on the left and right sides) and you are using a flat-panel TV as a display, check your TV's settings for possible configuration points that might fix the alignment.

Due to the huge variety of makes + models, it is recommended that you search the web + forums for more information particular to your device.

As an example of a setting that was changed on a 55" Samsung Smart TV that was manufactured in 2012:

- "Screen Resolution" setting was changed from "16:9" to "Screen Fit" to fix "cut off"

## License

Released under Apache 2.0 license by Kevin Firko (@firxworx) - hello@firxworx.com

Use at your own risk!
