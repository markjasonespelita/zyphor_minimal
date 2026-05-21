# How to Create Your Own Operating System (Based on the Linux Kernel) — Zyphor Minimal Guide

This guide demonstrates how to create a very minimal Linux-based operating system using the Linux kernel, a custom initramfs, and the GRUB bootloader.

---

# Install Required Build Packages

First, install the required packages needed for building the Linux kernel and generating the bootable ISO image.

```bash
sudo apt install build-essential bc bison flex libssl-dev libelf-dev dwarves pkg-config libncurses-dev busybox-static libdw-dev grub-pc-bin xorriso cpio gzip
```

---

# Minimal (`tinyconfig`) Version

## Step 1: Clone this Repository

```bash
git clone https://github.com/markjasonespelita/zyphor_minimal
```

---

# Kernel Space

## Step 2: Clone the Linux Kernel Source Code

Navigate to the `kernel` directory inside the `zyphor_minimal` project folder and clone the Linux kernel source code.

```bash
cd zyphor_minimal/kernel

git clone --depth=1 https://github.com/torvalds/linux
```

---

## Step 3: Create a Tiny Kernel Configuration

Navigate into the Linux source directory and generate a minimal kernel configuration.

```bash
cd linux

make tinyconfig

make menuconfig
```

The `tinyconfig` target generates a very small kernel configuration suitable for minimal systems.

The `menuconfig` interface allows you to customize and enable additional kernel features required for booting the system.

---

## Step 4: Configure the Kernel

Inside `menuconfig`, enable the following options:

```text
Kernel Configuration (Minimal for Initramfs)

General Setup:
  [*] 64-bit kernel
  [*] Initial RAM filesystem and RAM disk (initramfs/initrd) support
  [*] Support initial ramdisk compressed using gzip
  [*] Configure standard kernel features
      [*] Enable support for printk

Device Drivers:
  Character devices:
      [*] Enable TTY

Executable File Formats:
  [*] Kernel support for ELF binaries
  [*] Kernel support for scripts starting with #!
```

These options enable:
- initramfs support
- terminal (TTY) support
- ELF executable loading
- shell script execution
- kernel logging support

---

## Step 5: Build the Linux Kernel

Compile the Linux kernel:

```bash
make -j4
```

The `-j4` option allows the build process to use 4 CPU threads simultaneously for faster compilation.

After the build completes successfully, you should see a message similar to:

```bash
Kernel: arch/x86/boot/bzImage is ready
```

The generated kernel image will be located at:

```bash
arch/x86/boot/bzImage
```

---

# User Space

## Step 6: Create the Initramfs

Now let’s create a simple `initramfs` file, which will serve as the temporary root filesystem loaded by the Linux kernel during the early boot process.

```bash
cd ../../
# You should now be inside the zyphor_minimal folder

cd rootfs_minimal

find . | cpio -o -H newc | gzip > ../initramfs
```

### Explanation

The `initramfs` (Initial RAM Filesystem) is a temporary root filesystem loaded directly into memory by the Linux kernel during the boot process.

Before the actual operating system filesystem is mounted, the kernel first executes the files and scripts inside the initramfs to initialize the system environment.

In this step, we are packaging the contents of the `rootfs_minimal` directory into a compressed `initramfs` archive that the kernel can load during boot.

### Command Breakdown

#### `cd ../../`
Moves two directories backward to return to the main `zyphor_minimal` project folder.

#### `cd rootfs_minimal`
Enters the minimal root filesystem directory that contains the essential Linux filesystem structure and initialization files.

#### `find .`
Recursively lists all files and directories inside the current root filesystem.

#### `cpio -o -H newc`
Creates a CPIO archive from the listed files.

- `-o` means “output archive”
- `-H newc` uses the modern `newc` archive format commonly used for Linux initramfs images

#### `gzip > ../initramfs`
Compresses the generated archive using gzip and saves the final compressed initramfs image as `initramfs` in the parent directory.

After this step, the generated `initramfs` file will be used together with the Linux kernel (`bzImage`) by the bootloader to start the operating system.

You should now have these two important files inside the `zyphor_minimal` folder:

1. `kernel/linux/arch/x86/boot/bzImage` — The compiled Linux kernel image.
2. `initramfs` — The compressed initial RAM filesystem.

---

# Bootloader

Now let’s configure the `/bootloader` directory inside the `zyphor_minimal` project.

## Step 7: Copy the Kernel Into the Bootloader Directory

```bash
cp kernel/linux/arch/x86/boot/bzImage bootloader/boot/vmlinuz
```

This command copies the Linux kernel into the `bootloader/boot` directory and renames it to `vmlinuz`.

---

## Step 8: Copy the Initramfs Into the Bootloader Directory

```bash
cp initramfs bootloader/boot
```

This command copies the generated `initramfs` image into the `bootloader/boot` directory.

---

## Step 9: Configure GRUB

Configure the `bootloader/boot/grub/grub.cfg` file.

This file contains the GRUB bootloader configuration that tells GRUB how to boot the Linux kernel and load the initramfs during system startup.

```bash
set timeout=0
set default=0

menuentry "MyOS" {
    linux /boot/vmlinuz
    initrd /boot/initramfs
}
```

You can replace `MyOS` with your preferred operating system name.

The `linux` line tells GRUB which Linux kernel to boot, while the `initrd` line specifies the initramfs image that will be loaded into memory during the early boot process.

---

## Step 10: Build the Bootable ISO Image

```bash
grub-mkrescue -o myos.iso bootloader
```

This command generates a bootable ISO image named `myos.iso` using the contents of the `bootloader` directory.

GRUB automatically creates the required boot structures for BIOS and UEFI boot compatibility.

---

# Final Result

After completing all steps, you should now have:

```text
myos.iso
```

This ISO image can be:
- tested using QEMU or VirtualBox
- written to a USB drive
- booted on real hardware
- used as the foundation for building your own Linux distribution
