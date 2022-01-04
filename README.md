# olivia-party - ansible-driven automated configuration of kiosk soc/sbc hardware

This project implements ansible roles + playbooks that can configure an intel-based SOC/SBC running Ubuntu server 20.04 as a touch-screen kiosk that boots directly into a web application run by the Chromium web browser.

The project was developed to target an intel-based UP Board running Ubuntu Server 20.04 LTS connected to an ASUS touchscreen that registers the device name "ELON Touchscreen".

The board is mated to a USB joystick interface and runs the "Olivia Party" accessibility project. The kiosk provides an accessible user interface to a user with severe physical disabilities.

## Dependencies

This project requires that ansible is installed on the host/control machine that will run the playbooks included in this repository.

## Running the Project

### Inventory

Revise `inventory/hosts` to specify your target board(s): replace the list under the `[intelboards]` section with the IP address(es) or FQDN(s) of your target device(s).

All target devices must be accessible by your host/control machine over the network.

Targets are assumed to be running a clean install of Ubuntu 20.04 server (no GUI), including an ssh server configured to support password authentication by the user that ansible will use to connect to each target.

Replace the `ansible_user` and `ansible_become_password` under the `[intelboards:vars]` sections with the credentials of a user that exists on the target(s) and has root privileges via sudo.

### Run Playbook

Review the playbooks (`playbooks/`) and roles they invoke (`roles/`) to understand what they do prior to executing them. Revise them as appropriate to suit the needs of your project.

Use the `ansible-playbook` command to execute playbooks:

```sh
ansible-playbook kiosk.yml
```
