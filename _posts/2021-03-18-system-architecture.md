---
layout: post
title: 'Learning The Basics of System Architecture'
date: 2021-03-18
categories: linux systems computers boot
---

Let's talk a bit about basic _system architecture_. The process of [booting](https://en.wikipedia.org/wiki/Booting) a system takes
it from a powered-off state to running an _operating system_ or _kernel_. First, the
pre-installed firmware such as [BIOS](https://en.wikipedia.org/wiki/BIOS) or
[UEFI](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface) loads the _bootloader_ (in Linux this is
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
protection, and change hardware settings such as _interrupt request_ ([IRQ](https://en.wikipedia.org/wiki/Interrupt_request_%28PC_architecture%29)) and _direct
memory access_ ([DMA](https://en.wikipedia.org/wiki/Direct_memory_access)). In many cases, disabling features can reduce power consumption and
can increase system protection as CPU features can contain known vulnerabilities. One of
the most common reasons to enter the BIOS is to reorder the boot devices, to boot from a
USB drive for example.

Checking the BIOS can also aid when troubleshooting a hardware device that is not working.
If the BIOS does not detect it, chances are good the physical device is defective.

## BIOS vs UEFI

The BIOS is stored in a non-volatile memory chip attached to the motherboard. It assumes
the first 440 bytes in the first storage device (following the order defined in the BIOS
configuration tool) are the first stage of the bootloader. The first 512 bytes of a
storage device are called the _Master Boot Record_ (MBR). The [MBR](https://en.wikipedia.org/wiki/Master_boot_record) uses the standard DOS
partition schema and contains the partition table.

Similar to the BIOS, UEFI is also firmware. It can identify partitions and ready many of
the filesystems found in them. Unlike BIOS, UEFI does not rely on the MBR, instead it
takes into account only the settings stored in its non-volatile memory ([NVRAM](https://en.wikipedia.org/wiki/Non-volatile_random-access_memory)) attached to
the motherboard. These settings include definitions of the location of UEFI-compatible
programs called _EFI applications_ that will be executed from a boot menu. EFI
applications include bootloaders, operating system selectors and tools for system diagnostics
or repair. These applications must be in a conventional storage device partition,
typically formatted with a [FAT](https://en.wikipedia.org/wiki/File_Allocation_Table) filesystem. This _EFI System Partitino_ (ESP) cannot be
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

1. The [POST](https://en.wikipedia.org/wiki/Power-on_self-test), like above.
2. The UEFI activates basic components to load the system.
3. The UEFI's firmware read the definitions stored in NVRAM to execute the pre-defined EFI
   application stored in the ESP partition's filesystem (usually this is a bootloader).
4. The bootloader then loads the kernel to start the operating system.

### Secure Boot

Users installing Linux may come across [_Secure Boot_](https://wiki.debian.org/SecureBoot). UEFI supports a safety mechanism
designed to allow only the execution of signed EFI applications. This is called Secure
Boot. These signed applications must be authorized by the hardware manufacturer. The aim
is to increase protection against malicious software, but it makes it difficult to install
other operating systems not covered by the manufacturer's warranty. It is sometimes
necessary to disable this feature to install Linux on computers which had a different operating
systems installed by default.

## The Bootloader

The [_Grand Unified Bootloader_ (GRUB)](https://www.gnu.org/software/grub/) is the most popular bootloader for Linux on x86
architecture. GRUB displays a list of operating systems available to boot (or can be
invoked by pressing `Shift` for BIOS or `Esc` for UEFI). The GRUB menu allows a choice of
which kernel to load and enables the user to pass parameters. These parameters generally
follow the `option=value` syntax.

### Useful Kernel Parameters

Some of the common [kernel](<https://en.wikipedia.org/wiki/Kernel_(operating_system)>) parameter are:

- `acpi`, which enables or disables [ACPI](https://en.wikipedia.org/wiki/Advanced_Configuration_and_Power_Interface) support (ie. `acpi=off`).
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
configuration and memory addressing. The kernel then loads the _initramfs_ ([Intial RAM Filesystem](https://en.wikipedia.org/wiki/Initial_ramdisk)),
which is a temporary root filesystem used to provide the required modules to
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
modules. To view currently loaded modules, use the command `lsmod`. The `modprobe` command
intelligently adds or removes a module from the Linux kernel. To unload a module, use
`modprobe -r <module>`.

To view information about a module, use the `modinfo` command with the module name as an
argument. This will display a description, the file, the author, the license, the
identification, the dependencies and the available parameters for the given module. To
display all available parameters, for example, use the `-p` option `modinfo -p <module>`.

### Customizing Parameters

Sometimes it is necessary to include custom parameters. Such modifications can be done in
the `/etc/modprobe.conf` configuration file, or included as distinct files in the
`/etc/modprobe.d/` directory. This can be useful when **blacklisting** a module, for
example when using the nVidia drivers instead of the nouveau drivers. A blacklist can be
added as a file under `/etc/modprobe.d/blacklight.conf`, with the line `blacklight nouveau` in this case.

### Reading Hardware Information

While the commands `lspic`, `lsusb` and `lsmod` act as a "front-end" to read hardware
information from the operating system, the same information is also available in the
`/proc` and `/sys` directories (which are virtual filesystems).

The `/proc` directory contains files with information about running processes and hardware
resources. Some of the notable files include:

- `/proc/cpuinfo` which lists detailed information about the CPU(s).
- `/proc/interrupts` which provides a list of numbers of the interrupts per IO device for
  each CPU.
- `/proc/ioports` lists the currently registered input / output port regions in use.
- `/proc/dma` lists the registered _direct memory access_ (DMA) channels in use.

While `/proc` contains information about kernel data structures, including running
processes and configuration, the `/sys` directory stores device information and kernel
data related to hardware. In addition, the `/dev` directory contains a file for each
system device (ie. storage devices like `/dev/sda`). These devices are handled by the
_udev_ subsystem.

### Hardware Devices

When the Linux kernel captures the hardware detection event, it passes it to the
[_udev_](https://en.wikipedia.org/wiki/Udev)
process, which then identifies the device and dynamically creates a file in `/dev`
according to predefined rules. This can include _coldplug detection_, for identification
and configuration of devices already present during machine power-up, or _hotplug
detection_ for devices identified while the system is running. When a new device is
detected, _udev_ searches the `/etc/udev/rules.d/` directory for a matching rule.

Block devices, like a hard drive, where data is read to and from in blocks, appear in the
`/dev` directory like `/dev/sda1` or `/dev/nvme0n1p1` depending on the type of device.

## Run Levels / Boot Targets

Once the kernel starts the initialization program, the `init` program (be it _systemd_,
_SysV_ or _Upstart_) launches various services, also known as _daemons_. [Daemons](<https://en.wikipedia.org/wiki/Daemon_(computing)>) are
processes that control distinct functions of the system, like network application services
(HTTP server, file sharing, email, etc), databases or on-demand configuration. Some
low-level aspects of the operating system are affected by daemons, such as load balancing
and firewall configuration. As the first process the kernel starts, the `init` program has
a _process ID_ (PID) of 1.

### SysV (init)

The traditional initialization program for Linux has been _SysV_. It provides a set of
system states or _runlevels_. Run levels are numbered from 0-6:

- (0): System shutdown.
- (1): Single user mode (s), without network or other non-essential capabilities.
- (2, 3, 4): Multi-user mode, usually only level 3 is used. Users can log in by console or
  network.
- (5): Multi-user mode, plus a graphical login (GUI).
- (6): System restart.

The program responsible for managing runlevels and the associated daemons is `/sbin/init`.
The `init` program identifies the requested runlevel defined by a kernel parameter or in
the `/etc/inittab` file and then loads the associated scripts listed therein. Each
runlevel has scripts in the `/etc/init.d/` directory.

#### Syntax of `/etc/inittab`

The `/etc/inittab` configuration file consists of colon-delimited lines:
"ID:RUNLEVELS:ACTION:PROCESS", where:

- ID is a generic name up to four characters in length to identify the entry.
- RUNLEVELS is a list of numbers for which an action should be done.
- ACTION defines how `init` will execute the process indicated.
- PROCESS is the process to perform the action on.

Various actions are available for processes in `/etc/inittab`. Some of the most important
ones are:

- `boot`, which means the process will be executed during system initialization. The runlevels
  field is ignored.
- `bootwait`, means the process is executed during boot and will block until it finishes
  running.
- `sysinit`, means the process will be executed after system initialization, regardless of
  runlevel.
- `wait`, means the process will be executed for the given runlevels, and will block until
  it has finished.
- `respawn`, requests the process be restarted if it terminates.
- `ctrlaltdel`, means the process is started when `init` receives the `SIGINT` signal,
  triggered by the key sequence "Ctrl + Alt + Delete".

The `/etc/inittab` file is also responsible for defining the default runlevel, like
`id:x:initdefault`, where "x" is the default runlevel. This should never be "0" or "6"
otherwise the system will shutdown as soon as it finishes booting. After the configuration
file is modified, the command `telinit q` should be used to reload the configuration.

Scripts used by `init` to set up each runlevel are stored in the `/etc/init.d/` directory.
Each runlevel has a directory in `/etc/` as well, such as `/etc/rc0.d/`. The first letter
of the filename in the runlevel's directory indicates if the service should be started (S)
or killed (K) when entering the runlevel. To view the current runlevel, use the command
`runlevel`, and to change the runlevel use `telinit N` where "N" is the desired numerical
target.

### systemd

[systemd](https://systemd.io/) is a modern replacement for SysV, and is the predominant system initialization
program on Linux. It manages system resources and services, which it refers to as _units_.
A unit consists of a name, a type and a configuration file (ie. `httpd.service`). The most
important types of systemd units are:

- `service`, for active system resources that can be initiated, interrupted and reloaded.
- `socket`, which is a filesystem socket or a network socket, and has a corresponding
  service unit loaded when the socket receives a request.
- `device`, is associated with a hardware device identified by the kernel, but only if a
  _udev_ rule for it exists. This can be used to resolve configuration dependencies when
  certain hardware is detected, given that properties from the udev rule can be used as
  parameters.
- `mount`, is a point point definition, similar to an entry in `/etc/fstab`.
- `automount`, is a mount point that is mounted automatically. It has a corresponding mount
  unit that is initiated when the automount point is accessed.
- `target`, is a group of other units managed as a single unit, like a runlevel.
- `snapshot`, is a saved state of the systemd manager.

The `systemctl` command is used to execute all tasks regarding unit activation,
deactivation, execution, interruption and monitoring. Here are some examples:

```bash
# Start unit x
systemctl start x

# Stop unit x
systemctl stop x

# Restart unit x
systemctl restart x

# Show status of x
systemctl status x

# Show is unit x is running
systemctl is-active x

# Enable unit x (so it loads during system initialization)
systemctl enable x

# Disable unit x
systemctl disable x

# Verify if unit x starts with the system
systemctl is-enabled x
```

#### Controlling System Targets

In addition to controlling system services, systemd can control targets (what `init` calls
runlevels). To switch to target "x" use `systemctl isolate x`. Unlike `init`, systemd does
not use the `/etc/inittab` configuration file. To change the default target, the option
`systemd.unit=x` is added to the kernel parameters, where "x" is the default target. In
addition, the target can be changed by modifying the symbolic link
`/etc/systemd/system/default.target` so that it points to the desired target. The command
`systemctl set-default x` can also be used. View the system's default target with
`systemctl get-default`.

#### Configuration Files for systemd

Configuration files for systemd are stored in `/lib/systemd/system/`. To list available
units, use the command `systemd list-unit-files`. It is possible to filter by type with
the `--type=x` option, like `systemctl list-unit-files --type=target`. Active units (or
units that have been active during the current session) can be listed with `systemctl list-units`.

One of the contentious aspects of systemd is that it tries to offer a large scope of
functionality. In addition to managing daemons and targets, it offers power management
capabilities. For example, to suspend the system, use `systemctl suspend` or to hibernate
use `systemctl hibernate`. Configuration files for power management are located in
`/etc/systemd/logind.conf` or in separate files in the
`/etc/systemd/system/logind.conf.d/` directory. These features cannot be used if another
power manager, such as the `acpid` daemon, is running. The `acpid` daemon is the main
power manager for Linux as it allows finer adjustments.

## Shutting Down

To shut down a Linux machine, the command `shutdown` is used. It takes additional options,
such as a time and a message to send to logged in users. To shutdown immediately:
`shutdown now`. On a systemd machine, the commands `shutdown` and `reboot` are linked to
`systemctl`.

## Conclusion

Well that's a rather broad and technical overview of the Linux boot process, along with
the various programs that control the general functionality of the operating system
environment.
