# Arch Linux Installation on Banana Pi M2 Zero (Headless Setup)

## Overview
This README provides instructions for installing Arch Linux on a Banana Pi M2 Zero using an SD card without a display. It involves setting up the system using QEMU user-mode emulation and `chroot`, and includes steps for accessing the system via the serial debug port.

## Prerequisites
- A Linux host system with Arch Linux
- QEMU user-mode emulation package
- `binfmt_misc` support in the kernel
- An SD card with at least 16GB of storage
- A serial-to-USB adapter if you plan on using the serial debug port

## Instructions

### Step 1: Update System and Install Packages
```bash
sudo pacman -Syu
sudo pacman -S u-boot-tools dosfstools qemu-arch-extra
```

### Step 2: Enable ARM Binary Translation
This step involves configuring `binfmt_misc` to recognize and translate ARM binaries so that they can be executed on an x86_64 system using QEMU.

```bash
sudo modprobe binfmt_misc
echo ':arm:M::\x7fELF\x01\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x02\x00\x28\x00:\xff\xff\xff\xff\xff\xff\xff\x00\xff\xff\xff\xff\xff\xff\xff\xff\xfe\xff\xff\xff:/usr/bin/qemu-arm-static:' | sudo tee /proc/sys/fs/binfmt_misc/register
```

### Explanation of `binfmt_misc` Commands
- `modprobe binfmt_misc` enables the `binfmt_misc` module, allowing the kernel to support additional binary formats.
- The `echo` command registers the ARM binary format with the kernel. It specifies the magic bytes that identify an ARM binary and associates it with the `qemu-arm-static` emulator.

### Step 3: Prepare the SD Card
```bash
lsblk # Identify the SD card device name (e.g., /dev/mmcblk0)
sudo dd if=/dev/zero of=/dev/mmcblk0 bs=1M count=8
sudo fdisk /dev/mmcblk0 # Create a new partition table and a primary partition
sudo mkfs.ext4 /dev/mmcblk0p1
sudo mkdir /mnt/banana
sudo mount /dev/mmcblk0p1 /mnt/banana
```

### Step 4: Install Arch Linux ARM
```bash
wget http://os.archlinuxarm.org/os/ArchLinuxARM-armv7-latest.tar.gz
sudo bsdtar -xpf ArchLinuxARM-armv7-latest.tar.gz -C /mnt/banana
```

### Step 5: Configure the System with QEMU and `chroot`
Copy QEMU ARM static binary to the mounted partition:
```bash
sudo cp /usr/bin/qemu-arm-static /mnt/banana/usr/bin/
```

`chroot` into the system:
```bash
sudo arch-chroot /mnt/banana
```

### Step 6: Set Up Boot Script
```bash
cd /boot
mkimage -A arm -O linux -T script -C none -n "Boot.scr" -d boot.cmd boot.scr
```

### Step 7: Flash U-Boot to the SD Card
```bash
sudo dd if=u-boot-sunxi-with-spl.bin of=/dev/mmcblk0 bs=1024 seek=8
```

### Step 8: Accessing the Terminal via Serial Debug Port
To access the terminal through the Banana Pi's serial debug port, connect the serial-to-USB adapter to the debug pins on the Banana Pi. Use a terminal emulator like `screen` or `minicom` on your host system to interact with the Banana Pi:

```bash
screen /dev/ttyUSB0 115200
```

This command will open a terminal session to the Banana Pi, replace `/dev/ttyUSB0` with the actual device your serial adapter is recognized as.

### Step 9: First Boot and System Configuration
After inserting the SD card into the Banana Pi and starting it up, connect via SSH or the serial debug port using the following default credentials:

- Default user: `alarm`
- Password: `alarm`
- Default root user: `root`
- Password: `root`

Initialize the keyring and update the system:
```bash
pacman-key --init
pacman-key --populate archlinuxarm
pacman -Syu
```

Install and configure essential services like network managers.

### Step 10: Clean Up
Before unmounting the SD card:
```bash
sudo umount /mnt/banana
```

## Conclusion
You should now have a working Arch Linux installation on your Banana Pi M2 Zero. This guide omits setting up peripherals and focuses on the initial installation and configuration. For further customization and peripheral setup, consult the Arch Linux ARM community and documentation.


Remember to replace `/dev/mmcblk0` with the actual device name of your SD