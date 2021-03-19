---
layout: post
title: 'Learning The Basics of System Architecture'
date: 2021-03-18
categories: linux systems computers boot
---

Let's talk a bit about basic _system architecture_. The process of booting a system takes
it from a powered-off state to running an _operating system_ or _kernel_. First, the
pre-installed firmware such as BIOS or UEFI loads the _bootloader_ (in Linux this is
usually GRUB, the _Grand Unified Bootloader_). Then, the bootloader loads the kernel,
which receives parameters, such as which partition contains the root filesystem or in
which mode the operating system should execute.

Once the kernel is loaded, it identifies and configures the computer's hardware
components. Finally, the kernel calls the utility responsible for starting and managing
the system's services, such as _systemd_ or _init_.

## Hardware Configuration

Modern computers typically use UEFI, _Unified Extensible Firmware Interface_, which offers
a more modern implementation of a BIOS, _Basic Input/Output system_. The BIOS is firmware
containing basic configuration routines. UEFI offers more sophisticated features for
identification, testing, and configuration of hardware, as well as making firmware
upgrades easier.

There are various keys to get into BIOS/UEFI, the common ones being `Del`, `F2` or `F12`.
Often, the correct key to press will be displayed on the power-on screen. Once in the
BIOS, the user can enable or disable integrated peripherals, activate basic error
protection, and change hardware settings such as _interrupt request_ (IRQ) and _direct
memory access_ (DMA). In many cases, disabling features can reduce power consumption and
can increase system protection as CPU features can contain known vulnerabilities. One of
the most common reasons to enter the BIOS is to reorder the boot devices, to boot from a
USB drive for example.

Checking the BIOS can also aid when troubleshooting a hardware device that is not working.
If the BIOS does not detect it, chances are good the physical device is defective.

## BIOS vs UEFI

The BIOS is stored in a non-volatile memory chip attached to the motherboard. It assumes
the first 440 bytes in the first storage device (following the order defined in the BIOS
configuration tool) are the first stage of the bootloader. The first 512 bytes of a
storage device are called the _Master Boot Record_ (MBR). The MBR uses the standrad DOS
partition schema and contains the partition table.

Similar to the BIOS, UEFI is also firmware. It can identify partitions and ready many of
the filesystems found in them. Unlike BIOS, UEFI does not rely on the MBR, instead it
takes into account only the settings stored in its non-volatile memory (NVRAM) attached to
the motherboard. These settings include definitions of the location of UEFI-compatible
programs called _EFI applications_ that will be executed from a boot menu. EFI
applications include bootloaders, operating system selectors and tools for system diagnostics
or repair. These applications must be in a conventional storage device partition,
typically formatted with a FAT32 filesystem. This _EFI System Partitino_ (ESP) cannot be
shared with other filesystems.

### Pre-OS Steps to Boot

With BIOS, the steps are:

1. The _Power-On Self Test_ (POST) process checks for hardware failures
2. The BIOS activates the basic components to load the system, like video output, keyboard
   and storage media.
3. The BIOS loads the first stage of the bootloader from the MBR.
4. The first stage of the bootloader calls the second stage of the bootloader, which is
   responsible for presenting boot options and for loading the kernel.

With UEFI, the steps are:

1. The POST, like above.
2. The UEFI activates basic components to load the system.
3. The UEFI's firmware read the definitions stored in NVRAM to execute the pre-defined EFI
   application stored in the ESP partition's filesystem (usually this is a bootloader).
4. The bootloader then loads the kernel to start the operating system.

### Secure Boot

Users installing Linux may come across _Secure Boot_. UEFI supports a safety mechanism
designed to allow only the execution of signed EFI applications. This is called Secure
Boot. These signed applications must be authorized by the hardware manufacturer. The aim
is to increase protection against malicious software, but it makes it difficult to install
other operating systems not covered by the manufacturer's warranty. It is sometimes
necessary to disable this feature to install Linux on computers which had a different operating
systems installed by default.

## The Bootloader

The _Grand Unified Bootloader_ (GRUB) is the most popular bootloader for Linux on x86
architecture. GRUB displays a list of operating systems available to boot (or can be
invoked by pressing `Shift` for BIOS or `Esc` for UEFI). The GRUB menu allows a choice of
which kernel to load and enables the user to pass parameters. These parameters generally
follow the `option=value` syntax.

### Useful Kernel Parameters

Some of the common kernel parameter are:

- `acpi`, which enables or disables ACPI support (ie. `acpi=off`).
- `init`, which allows setting an alternative system initiator program (ie. `init=/bin/bash`
  to start `bash` right after kernel boot).
- `systemd.unit` sets the _systemd_ target (ie. `systemd.unit=graphical.target`, or also
  accepts the numeric value instead `systemd.unit=3`).
- `mem`, sets the amount of available RAM which can be useful for virtual machines in
  order to limit guest resource usage.
- `maxcpus`, sets a limit on the number of processors (or cores) visible to the system.
  This is also useful for virtual machines. Setting this to `0` or `nosmp` disables support
  for multi-processor machines.
- `quiet` is used to hide most boot messages.
- `vga` is used to select a video mode (ie. `vga=ask` lists available modes and allows a
  choice).
- `root` sets the root partition (ie. `root=/dev/sda3`).
- `rootflags` sets mount options for the root filesystem.
- `ro` makes the initial mount read-only.
- `rw` allows writing in the root filesystem during the initial mount.

### Changing Kernel Parameters

In order to change the parameters passed to the kernel, it is possible to specify them at
boot time in GRUB's menu by pressing `e` to "edit" the kernel options. Then, press
"Ctrl+X" to boot with the new parameters. To make changes permanent, kernel parameters
must be added to the file `/etc/default/grub` on the line `GRUB_CMDLINE_LINUX`. Then, the
configuration must be regenerated:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

Note: To verify the kernel parameters of the current boot, use `cat /proc/cmdline`.

## System Initialization

After the bootloader has loaded the kernel into RAM, the kernel then takes control of the
CPU and sets up the fundamental aspects of the operating system, like basic hardware
configuration and memory addressing. The kernel then loads the _initramfs_ (Initial RAM
filesystem), which is a temporary root filesystem used to provide the required modules to
access the "real" root filesystem. The kernel then mounts all filesystems configured in
`/etc/fstab` and executes the first program: `init`.

This first program the kernel runs is the system initialization program. This has
traditionally been _System V_ or _SysV_ (init), but now most distributions of Linux use
_systemd_ to start and manage system services.

### Initialization Inspection

If any errors occur during the boot process, they are logged to the _kernel ring buffer_.
This log can be accessed with `dmesg`, and provides information in the following format:
Seconds since boot, process and message. If the system is using _systemd_, the following
commands all show initialization messages: `journalctl -b`, `journalctl --boot`,
`journalctl -k` or `journalctl --dmesg`.

It is possible to view prior boot messages by passing the option `-b N` or `--boot=N`
where "N" is a count backwards through previous boots. For example, `journalctl --boot=-1`
would show the previous boot before the current one. In addition to the `journalctl`
command, other messages are stored in various files within the `/var/log` directory. If
the system is not booting, another machine can be used to access the disk and these logs
may provide troubleshooting answers.

## Inspecting Hardware Devices

Linux offers numerous commands for inspecting the devices currently detected by the
operating system:

- `lspci` shows all devices connected to the _Peripheral Component Interconnect_ (PCI)
  bus. These devices can be attached to the motherboard, like a disk controller, or may be
  in an expansion slot, like a graphics card.
- `lsusb` lists all _Universal Serial Bus_ (USB) devices that are connected, such as
  keyboards, pointing devices and removable storage media.

For every hardware component, an operating system requires software to control it. This is
called a _kernel module_ or a _driver_. The above two commands list the hexadecimal
addresses of each device, which the device's unique identifier. In addition to the
overview these commands provide, more detailed information can be found by querying the
specific device address. For example:

- `lspic -s <device_address> -v`
- `lsusb -v -d <hex_address>`

Not all devices have kernel modules associated with them, but when a matching module
exists, it appears as "Driver=MODULE" and the device class identifies the general
category of the module. To search for a device with its bus and dev numbers use `lsusb -s <bus>:<dev>`.

## Kernel Modules

It is common to have many loaded kernel modules at any given time. The `kmod` package can
be used to insert, remove, list, check properties, resolve dependencies and aliases of
modules. To view currently loaded modules, use the command `lsmod`.
