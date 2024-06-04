
# arch-btw

Automation and documentation for setting up (my) Arch Linux workstation.

The goal of this project is for me to keep a record of my workstation configuration and provide a quick way to bring it back to that state from nothing.

To guard against configuration drift, I'll run through the whole process on a regular basis, including formatting my boot drive and reinstalling the operating system.

It is unlikely that someone would want to copy this setup exactly, but it might provide some examples or inspiration.

## OS Installation

For a full, in-depth guide, refer mainly to the [Official Installation Guide](https://wiki.archlinux.org/title/Installation_guide) up to the point of installing packages.

### Getting Started

1. Ensure system is configured to boot in UEFI mode
2. Prepare installation media and boot into live environment
3. Verify the boot mode with `cat /sys/firmware/efi/fw_platform_size`
    - `64`, or maybe `32`, should be returned - if not, DOUBLE CHECK THE BOOT MODE
5. Connect to the internet. If using a wireless connection:
    1. Run `iwctl`
    2. Get your device name from `device list`
    3. Scan for networks with `station {device-name} scan` (returns nothing)
    4. Get network names with `station {device-name} get-networks`
    5. Connect to a network with `station {device-name} connect {SSID}`
  
### Configure Boot Drive

For now, just set up a boot drive. It could be helpful to disconnect auxiliary drives ahead of time if there is any question about which is which.

1. List attached disks with `fdisk -l` or `lsblk` and determine device label
    - `/dev/sdx` will be used here as a placeholder
2. Be very sure you select the correct drive - all data will be destroyed
3. Open the device in fdisk with `fdisk /dev/sdx`
    1. Use `g` to create an empty GPT partition table
    2. Use `n` to create a new partition
        - This will just be used for the boot loader and can be small, 1-4 GiB at most
        - The first partition can start at the default `2048`
        - As a shortcut for end location, use the format `+nG`, where `n` is the desired size in GiB
    3. Use `t` to change the partition type to `EF` for EFI
    4. Use `n` to create another partition
        - This will be used for the file system and should fill the rest of the free space
        - The default partition type should be fine
    5. Use `w` to write the changes
4. Format the boot partition in FAT32 with `mkfs.fat -F 32 /dev/sdx1`
5. Format the root partition in EXT4 with `mkfs.ext4 /dev/sdx2`
6. Mount the root partition with `mount /dev/sdx2 /mnt`
7. Mount the boot partition with `mount --mkdir /dev/sdx1 /mnt/boot`

### Install Initial Packages

Follow the format `pacstrap -K /mnt x y z` with each of the necessary packages below:

Core packages:
- `base`
- `base-devel`
- `linux`
- `linux-firmware`

Other firmware:
- `amd-ucode` or `intel-ucode`

Network utilities:
- `networkmanager`
- `dhclient`
- `wpa_supplicant`

Boot loader:
- `grub`
- `efibootmgr`

Utilities needed for setup:
- `vi`
- `wget`
- `git`
- `ansible`

### Finish Install

1. Generate fstab with `genfstab -U /mnt > /mnt/etc/fstab`
2. Change root into the new system with `arch-chroot /mnt`
3. Set the timezone with `ln -sf /usr/share/zoneinfo/America/Denver /etc/localtime`
4. Generate /etc/adjtime with `hwclock --systohc`
5. Uncomment `en_US.UTF-8 UTF-8` from `/etc/locale.gen` with `vi` or `sed -i '/s/#en_US.UTF-8/en_US.UTF-8/g' /etc/locale.gen`
6. Generate locales with `locale-gen`
7. Create locale.conf and set LANG `echo "LANG=en_US.UTF-8" > /etc/locale.conf`
8. Create and set hostname `echo "somehostname" > /etc/hostname`
9. Set root password with `passwd`
10. Install grub with `grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB`
11. Generate grub config with `grub-mkconfig -o /boot/grub/grub.cfg`
12. Enable network manager with `systemctl enable NetworkManager.service`
13. Clone this repository with `git clone https://github.com/yamlinson/arch-btw.git /opt/git/arch-btw`
14. Exit chroot with `exit`
15. Unmount volumes with `umount -R /mnt`
16. `reboot`

### Post-Install

- If using wireless network, connect with `nmcli --ask device wifi connect SSID`

## Ansible

1. `cd` to `/opt/git/arch-btw`
2. Modify `hosts.yml` with appropriate variable values
3. Install required collections/roles with `ansible-galaxy install -r requirements.yml`
4. Initialize a non-root admin user with `ansible-playbook init.yml`
5. Set passwords and change to the non-root admin account
6. Run the full playbook with `ansible-playbook main.yml --ask-become-pass`


