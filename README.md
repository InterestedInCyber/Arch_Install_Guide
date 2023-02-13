# Arch Linux Installation Guide

This guide follows the [Official Arch Linux Installation Guide](https://wiki.archlinux.org/title/Installation_guide) with some additions of my own personal setup and install mixed in.

**Important:** Depending on your system, the installation may vary, I will provide any alternatives should your hardware be any different.

### Installation

> 1. Boot up Arch Linux [UEFI Mode]

> 2. Set the Console Keyboard Layout
```bash
# List all Available Layouts:
$ ls /usr/share/kbd/keymaps/**/*.map.gz

# Set the Keyboard Layout. For example to set a UK Layout:
$ loadkeys uk.map.gz
```

> 3. Partitioning the Disks
```bash
# Identifying Block Devices [/dev/sda] [/dev/nvme0n1]
$ fdisk -l

# Modifying Partition Tables
$ fdisk /dev/the_disk_to_partition

# Create a GPT Layout
$ g

# New EFI Partition
$ n

# Choose Default Partition Number
# Choose Default First Sector

# Enter Partition Size
$ +1G

# New Swap Partition
$ n

# Choose Default Partition Number
# Choose Default First Sector

# Enter Partition Size. I Suggested size is half of your RAM
$ +16G

# New Root Partition
$ n

# Choose Default Partition Number
# Choose Default First Sector
# Choose The Remaining Space Left

# List the Type List
$ l

# View Created Partitions
$ p

# Change EFI Partition from 'Linux Filesystem' to 'EFI System'
$ t

# Select the First Partition
$ 1

# Select 'EFI System' from the type list
$ 1

# Change Swap Partition from 'Linux Filesystem' to 'Linux Swap'
$ t

# Select the Second Partition
$ 2

# Select 'Linux Swap' from the type list
$ 19

# Write the Changes
$ w
```

> 4. Formatting Previously Created Partitions
```bash
# Creating EFI File System
$ mkfs.fat -F32 /dev/efi_system_partition

# Creating Swap File System
$ mkswap /dev/swap_partition

# Enabling Swap
$ swapon /dev/swap_partition

# Creating BTRFS File System
$ mkfs.btrfs /dev/root_partition
```

> 5. Mounting File Systems
```bash
# Mount Root Partition to '/mnt' Directory
$ mount /dev/root_partition /mnt

# Mount and Create Boot Partition
$ mount --mkdir /dev/efi_system_partition /mnt/boot
```

> 6. Operating System Installation

Package List:
- [base](https://archlinux.org/packages/core/any/base/) (Minimal Package Set for Arch Install)
- [base-devel](https://archlinux.org/packages/core/any/base-devel/) (Basic Build Tools for Arch Linux Packages)
- [linux](https://archlinux.org/packages/core/any/base-devel/) (Linux Kernel and Modules)
- [linux-firmware](https://archlinux.org/packages/core/any/linux-firmware/) (Firmware Files for Arch Linux)
- [linux-hardened](https://archlinux.org/packages/extra/x86_64/linux-hardened/) (Security Hardened Linux Kernel and Modules)
- [btrfs-progs](https://archlinux.org/packages/core/x86_64/btrfs-progs/) (BTRFS File System Utilities)
- [sudo](https://archlinux.org/packages/core/x86_64/btrfs-progs/) (Allows User to Run Root Priviledged Commands)
- [nano](https://archlinux.org/packages/core/x86_64/nano/) (Simple Command Line Text Editor)
- [networkmanager](https://archlinux.org/packages/extra/x86_64/networkmanager/) (Network Connection Manager & User Applications)
- [efibootmgr](https://archlinux.org/packages/core/x86_64/efibootmgr/) (Application to Modify EFI Boot Manager)
- [mtools](https://archlinux.org/packages/extra/x86_64/mtools/) (Utilities to Access MS-DOS Disks)
- [dosfstools](https://archlinux.org/packages/core/x86_64/dosfstools/) (DOS File System Utilities)
- [os-prober](https://archlinux.org/packages/community/x86_64/os-prober/) (Utility to Detect other Operating Systems)
- [git](https://archlinux.org/packages/extra/x86_64/git/) (Software Distribution and Pull Request Command Line Tool)
- [net-tools](https://archlinux.org/packages/core/x86_64/net-tools/) (Linux Networking Tools)
- [neofetch](https://archlinux.org/packages/community/any/neofetch/) (Optional Command Line System Information Display Application)
 
```bash
# Installing ALL Necessary Packages
$ pacstrap -K /mnt base base-devel linux linux-firmware linux-hardened btrfs-progs sudo nano networkmanager efibootmgr mtools dosfstools os-prober git net-tools neofetch
```

> 7. Generating FSTAB
```bash
# Defining File Systems with UUID Instead of Drive Paths [/dev/sda] [/dev/nvme0n1]
$ genfstab -U /mnt >> /mnt/etc/fstab
```

> 8. Chroot
```bash
# Changing root into the new system
$ arch-chroot /mnt
```

> 9. Time Zone
```bash
# Setting the time zone
$ ln -sf /usr/share/zoneinfo/Region/City /etc/localtime

# Syncing to the Hardware Clock
$ hwclock --systohc
```

> 10. Locales
```bash
# Uncomment the appropriate language for the Console (For me its en_GB.UTF-8)
# Then Run the Following Command to Set the Language
$ locale-gen

# Create / Edit the 'locale.conf' File
$ nano /etc/locale.conf

# Add the Following Line According to your Locale Set Previously
$ LANG=en_GB.UTF-8

# Save and Exit
$ CTRL-S and CTRL-X

# Create / Edit the 'vconsole.conf'
$ nano /etc/vconsole.conf

# Add the Following Line According to your Keymap Set in Step 2 (Only the first part of the map so 'uk' for me)
$ KEYMAP=uk

# Save and Exit
$ CTRL-S and CTRL-X
```

> 11. Network Configuration
```bash
# Create the Hostname File
$ nano /etc/hostname

# Add a Hostname (This is up to you, 'archvbox' is my example)
$ archvbox

# Save and Exit
$ CTRL-S and CTRL-X

# Edit hosts file
$ nano /etc/hosts

# /etc/hosts
$ 172.0.0.1   localhost
$ ::1         localhost
$ 172.0.1.1   archvbox.localdomain    archvbox

# Save and Exit
$ CTRL-S and CTRL-X

# Enable Network Manager Service
$ systemctl enable NetworkManager.service
```

> 12. User Creation and User Modification
```bash
# Creating A User (archuser is my example)
$ useradd -m archuser

# Creating A Password for the User
$ passwd 'enter a password here'

# Modifying the user to have sudo access
$ usermod -aG wheel archuser

# Editing the sudoers file
$ nano /etc/sudoers

# Uncomment %wheel (NANO ONLY)
$ CTRL-W "%wheel"
$ Uncomment the line it takes you to which has '% wheel'

# Save and Exit
$ CTRL-S and CTRL-X
```

> 13. Installing Systemd-Boot & Associated File Editing and Creation
```bash
# Installing the Boot Loader
$ bootctl --path=/boot install

# Loader Configuration
$ nano /boot/loader/loader.conf

# loader.conf
$ default   arch.conf
$ timeout   4

# Save and Exit
$ CTRL-S and CTRL-X

# Adding Loader Entries
$ nano /boot/loader/entries/arch.conf

# arch.conf
$ title   Arch Linux
$ linux   /vmlinuz-linux
$ initrd  /initramfs-linux.img
$ options root="/dev/root_partition" rw

# Save and Exit
$ CTRL-S and CTRL-X
```

> 14. Installing CPU Microcode

Install one of the following packages:

- [amd-ucode](https://archlinux.org/packages/core/any/amd-ucode/) (AMD Processors)
- [intel-ucode](https://archlinux.org/packages/extra/any/intel-ucode/) (Intel Processors)

```bash
# Microcode for AMD Processor
$ pacman -S amd-ucode

# Microcode for Intel Processor
$ pacman -S intel-ucode
```

> 15. Unmounting and Rebooting 
```bash
# Exit the chroot environment
$ CTRL-D or exit

# Unmount '/mnt'
$ umount -R /mnt

# Restart the Machine
$ reboot
```

> Post Installation Reference to the [Official Arch Linux General Recommendations](https://wiki.archlinux.org/title/General_recommendations)
