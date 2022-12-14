# \[ olivia-party \] op-ansible-device-automation

Automate the configuration and deployment of _OP Accessibility Devices_ (OPADs): hackable web kiosks that provide a foundaton for creating customized tools and toys for users with physical disabilities.

This repo is part of the Olivia Party (OP) Accessibility Project.

The [Ansible](https://www.ansible.com/)-driven automations found in this repo will transform commodity PC's and low-cost boards into linux-based devices with user interfaces provided via local web applications run by the [Chromium](https://chromium.org) web browser.

Many parts of this project are readily transferable to commercial applications. You are free to use and modify the code as you wish under the terms of the permissive Apache 2.0 open source license. Use at your own risk!

## Feature Summary

OPAD operates like a web kiosk and boots directly into the web application that serves as its UI (you can also boot directly to the URL of any website).

- Connect touchscreens, joysticks, keyboards, and custom-built input devices via USB
- Out-of-the-box support for audio and text-to-speech, including the HTML5 speech synthesis API
- Create hardware-accelerated 3D interfaces and experiences using WebGL and the latest browser capabilities
- Take advantage of broad support for Bluetooth, WiFi, and other options for connectivity thanks to Ubuntu Linux

Special considerations:

- Touchscreen users are "locked" into the interface: users cannot smudge or swipe out nor can they activate menus from the browser or OS
- The combination of xserver + Chromium that runs the UI has a smaller footprint and requires less resources vs. heavier-weight linux graphical environments like gdm3

You are free to customize the configuration to suit your use case.

## Supported Hardware

The ansible playbooks as-implemented should broadly support intel-based devices running a clean server installation of the free linux-based [Ubuntu](https://ubuntu.com/) 20.04 LTS or 22.04 LTS operating system.

Most modern touch screens are supported by Ubuntu running on PC. In most cases, no special drivers are required to use typical touch screens with video input and USB output. The OS will recognize touchscreens connected via USB as pointing devices and will treat them similar to a USB mouse.

Display orientation, resolution, and more can be changed by customizing the x server configuration. Search for resources related to xorg.conf for more information, and refer to xserver role in `roles/xserver`. Be careful to ensure your configuration reflects the capabilities of your display as it is possible to cause damage to your hardware in certain cases.

Additional architectures, hardware profiles, and/or software can be supported with additional customization of the ansible roles and playbooks in this repo.

### Tested Hardware Profiles

The ansible playbook in this repo has been executed successfully on the following hardware.

Computers:

- Lenovo ThinkCentre Tiny PC with Intel i5-6500T CPU
- [UP Board](https://up-board.org/) x86 single-board-computer (SBC) with Intel Atom CPU

Input devices:

- ASUS touchscreen w/ HDMI + USB interfaces
  - This particular display was identified by the OS as "ELON Touchscreen" and runs at 1366x768 resolution
- Generic USB external audio adapter with advertised support for Linux
- Generic USB joystick/gamepad controller with an arcade-style Joystick and buttons
- Generic USB mouse + keyboard (optional)
- Low-cost USB WiFi dongle with advertised support for Linux

## About Ansible

Ansible is a popular free and open-source IT automation platform that enables hardware configurations to be defined in a declarative and clean yaml-based syntax that delivers the benefit of self-documenting code.

The Ansible software can be installed Win/Mac/Linux computers to enable them to serve as a _control node_ capable of executing playbooks against an _inventory_ of target devices. Ansible connects to the target devices using [ssh](https://en.wikipedia.org/wiki/Secure_Shell).

Ansible and its ecosystem of powerful modules are [well-documented](https://docs.ansible.com/). Refer to the [Getting Started](https://docs.ansible.com/ansible/latest/user_guide/index.html#getting-started) guide to learn core concepts and how to install it on your system.

## Running the Project

Follow the steps in this section to deploy your own _OP Accessibility Device_ / web kiosk.

> Security Note / Disclaimer: this project assumes that the kiosk will be deployed in a physically private and secure environment on a private internal network. Public-facing, internet-facing, or otherwise production-grade device deployments are not part of the scope of this project. Use at your own risk!

### Prerequisites

#### On the control machine

The _control machine_ is the "puppetmaster": the laptop or desktop that will run the ansible playbook against your target _hosts_ (devices).

The control machine is assumed to be running a modern linux/unix distribution or MacOS. PC's running Windows are advised to run the project in linux via WSL2, Microsoft's official _Windows Subsystem for Linux_. Ubuntu is recommended.

Ensure the following prerequisites are met:

- python3 is installed (e.g. confirm `which python3` and `python3 --version` returns output)
- [Ansible](https://docs.ansible.com) is installed per its documentation
- Verify that you can successfully execute `ansible --version` in your Terminal
- Clone or download the source code from this repo
- You have an SSH client installed
- You have generated an SSH key pair that you can use to connect to your target devices

#### On target device(s)

- Plug in all required USB devices and peripherals to the device
- Install a fresh server install image (often referred to as _minimal_) of the [Ubuntu 20.04 LTS](https://releases.ubuntu.com/20.04/) or [Ubuntu 22.04 LTS](https://releases.ubuntu.com/20.04/) operating system
  - Ensure that an ssh server (e.g. OpenSSH) is installed
  - Create a non-root user account that has root privileges via `sudo` (this can often be done during the install process)
  - Confirm that you can connect to the target device as your non-root user via password or key-based authentication from your control machine
- Ensure the control machine and targets are connected to the same network and that the control machine can connect to them

### Ansible Inventory

Revise `inventory/hosts` to specify your target board(s). Replace the list under the `[intelboards]` section with the IP address(es) or FQDN(s) of your target device(s).

Targets are assumed to be running a clean install of Ubuntu server (no GUI), including an ssh server configured to support password authentication by the user that ansible will use to connect to each target.

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

## Managing Displays & Configuration

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

In most cases you should not make revisions to files or folders under `/usr/lib`.

### Troubleshooting Displays

With so many makes and models of displays, there are a lot of things that could go wrong. This section provides some tips to help you if the x server fails to start, exits prematurely, or if there are issues with the picture.

When the `~/launch-browser.sh` script runs, it saves the output of useful troubleshooting commands to log files in the: `~/kiosk-debug/` folder.

- `~/kiosk-debug/screens.debug.log`: `xrandr` output showing connected displays + ports
- `~/kiosk-debug/xinput-list.debug.log`: `xinput` output showing detected input devices for x server

If the kiosk fails to boot into chromium with the target app/website running full-screen at the proper resolution, and/or drops back to the Terminal CLI, the log for the x session can be found in:

`~/.local/share/xorg/Xorg.0.log`

The `lshw` (list hardware) command can provide more information about your graphics card:

```sh
sudo lshw -c display
```

For more troubleshooting information, check the xorg docs including key articles such as: <https://xorg-team.pages.debian.net/xorg/howto/use-xrandr.html>.

### Troubleshooting TV's

If you are using a flat-panel TV as a display and the alignment of the picture is off, e.g. a portion of the picture is cut off on the left and right sides, make sure to check through your TV's settings for possible configuration options that might correct the situation.

There is a huge amount of variance between TV makes and models. It is recommended that you search the web + forums for more information specific to your device.

An example of a device-specific setting related to an alignment issue with a 55" Samsung Smart TV manufactured in 2012:

- "Screen Resolution" setting was changed from "16:9" to "Screen Fit" to resolve a "sides slightly cut off" issue

## License

Released under Apache 2.0 license by Kevin Firko (@firxworx) - hello@firxworx.com
