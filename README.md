# eos_grub_loader

## What
A Linux/Grub2 boot loader for EOS (alternative to aboot). This does NOT include EOS, you will need to download EOS from arista.com with your account.

## Why
Aboot is not easy to install on x86/x86_64 and has limitations such as lack of ACPI support, EFI, etc. Aboot also decompresses the root file system to memory. Eos_grub_loader is based on Grub2 which is the most popular Linux boot loader and is very well supported. Eos_grub_loader decompresses the root file system to disk (on each boot) hence memory is preserved. A switch usually has a low write flash medium. The expectation is that eos_grub_loader is installed on a system with a disk (preferably SSD). Disk space is also less costly on cloud infrastructure them memory. 

## How
Eos_grub_loader installs a Linux kernel, a small initrd, and Grub2 to disk (it comes in a bootable installer). On eos_grub_loader boot, the file system /mnt/flash is mounted (it looks for a disk with the label persist). Assuming that autoinit is not disabled (settings file on /mnt/flash) then the system proceeds to the next stage. A new filesystem is created on the disk with the label bootimage and mounted to /bootimage. If a previous filesystem existed it will be destroyed. The file /mnt/flash/boot-config is consulted to determine the desired SWI. The SquashFS file (rootfs-i386.sqsh) is extracted from the SWI and decompressed to /bootimage. A static settings file and tool(s) are installed to /bootimage. A fstab entry is installed to /bootimage/etc/fstab for /mnt/flash. Finally, the EOS kernel is loaded and executed. At this point eos_grub_loader dies and its memory is reclaimed by the EOS kernel. If any failure occurs or if autoinit is disabled the user can manage eos_grub_loader with the bash shell (no credentials required).

## Download
A binary bootable installer can be downloaded from https://github.com/aristanetworks/eos_boot_loader/releases/download/v.1/installer.iso

## Tools
### configure.sh
The configure.sh script is available from the installer only. This script requires one arg, the disk to install the boot_loader. The configure.sh script will generate the script install.sh in the same directory.

1. configure.sh
2. install.sh

### Autoinit
The autoinit (/usr/bin/autoinit) script is available in the boot_loader and EOS (not the installer). It can be executed from the bash shell or from the EOS CLI. The script is used to enable/disable autoboot, show the status of autoboot and if ran inside the boot_loader it can be used to manually boot the next stage.

1. No Arg    - status
2. 'enable'  - enable autoinit
3. 'disable' - disables autoinit
4. 'boot' - boots the next stage (only available in the boot_loader)

## Build
### Instructions
1. [ -f /usr/bin/mock ] || sudo dnf -y install mock
2. cd ~/workspace; mkdir eos_grub_loader; cd eos_grub_loader
3. cat /etc/mock/fedora-24-i386.cfg | sed 's/fedora-24-i386/eos_grub_loader/g' > mock.cfg
4. mock -r mock.cfg --init
5. mock -r mock.cfg --install dnf git vim
6. ln -s /var/lib/mock/eos_grub_loader/root (Optional)
7. mock -r mock.cfg --shell
8. cd
9. git clone https://github.com/aristanetworks/eos_boot_loader.git
10. cd eos_grub_loader
11. ./build.sh
12. exit
13. mock -r mock.cfg --copyout /builddir/autoinit/dist/installer.iso .

### USB bootable
A bootable USB flash disk can be created with the command

1. sudo dd if=installer.iso of=/dev/[USB] status=progress
