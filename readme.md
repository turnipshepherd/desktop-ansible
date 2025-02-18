# Desktop Ansible

I found this repo from [this post](https://old.reddit.com/r/archlinux/comments/rox2f5/i_created_an_ansible_role_to_wipe_a_drive_and/)
and discovered it matched my current bash scripted install almost exactly. Current goals with the project is to make it more general
and finish some of the objects on the original author's todo list. 

# TODO: 
- remove a lot of the original authors hard coded information, such as
  - passwords
  - ip addresses
  - locale
  - etc if i find more
- use reflector instead of rankmirror
- abstract grub to do efi and mbr for virtual machine install
- does the original make sure pacman-init has finished?
- does the original do a secure disk wipe?
- automatic snapshotting (adapt from existing bash script)
  - https://wiki.archlinux.org/title/Snapper
- enable grub-btrfs for snapshot boot (adapt from existing bash script)
  - https://archlinux.org/packages/?name=grub-btrfs

- disable QoW for QEMU images
  - https://wiki.archlinux.org/title/btrfs#Disabling_CoW => specifically on a certain directory
- enable scrubbing
  - https://wiki.archlinux.org/title/btrfs#Scrub

# Original Readme

Ansible playbook to install my personal computer from scratch (including the partitioning and bootloader config. (uses full disk)
Initial install heavily based of: https://github.com/jsf9k/ansible-arch-install

## iso-install

Boot the host you want to manage with an arch iso. run the following commands:

1. `passwd` ("root" is used for the arch iso password, can be changed in the inventory files)
2. `systemctl start sshd`
3. `ip a` (to get the host ip address, change this in the inventory file)

Change the variables you want under group_vars or the inventory file. (depending if the variable is needed both in the postinstall playbook and the iso_install playbook)
- `install_drive`: drive that will be wiped and used for installing arch. 2 partitions will be made, one for boot and one root partition.
- `user`:
  - `name`: user to create that is not root
  - `password`: encrypted user password, can be clear text as well but this is not advised
  - `pub_key_location`: public key to add to host so we can run the common playbook after that
- `encryption`: set encryption with luks (luks1 as luks2 is not supported with grub)
  - `enabled`: true or false
  - `password`: encryption password, a random key will be generated for system load so you only have to enter your encryption password once
- `btrfs`: have the option to use btrfs as filesystem (includes subvolumes)
  - `enabled`: true or false
  - `subvols`: list containing subvol name and map location
    - `sub`: i.e. @log
    - `path`: i.e. var/log
- `manual_partitions`: override default wipe behaviour (both options necessary, you may overwrite an existing boot partition if wanted)
  - `boot`: set custom boot partition location (i.e. nvme0n1p1)
  - `root`: set custom root partition locatoin (i.e. nvme0n1p3)
- `hostname`: sets hostname for networking
- `timezone`: timezone to set on the host

Run the playbook:
`ansible-playbook -i production.yaml iso-installation.yaml`

*remarks*:
- currently locale is hard set
- currently pacman rankmirrors is using central europe as location
- grub is set to efi

## common
this relies on ansible-aur from the following repo: https://github.com/kewlfft/ansible-aur

This should be runnable on most of the arch installs (works on arm as well). Items set in `iso-install` can be overridden for hosts that are maybe not provisioned with Ansible.

### Running on Arch ARM
When running on arch arm use the following method to run
1) change the setting in `/etc/ssh/sshd_config` to `PermitRootLogin yes`
(this will be undone in our playbook during the user setup)
2) restart sshd `systemctl restart sshd`
3) run: `ansible-playbook -i production.yaml common.yaml -e "ansible_user=alarm" --ask-pass --ask-become-pass --become-method su --tags arm_setup`

## WSL Prerequisites

Run before ansible:
- https://github.com/yuk7/ArchWSL/wiki/How-to-Setup
- pacman -S openssh
- systemctl start sshd

