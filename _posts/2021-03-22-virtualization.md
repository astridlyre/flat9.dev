---
layout: post
title: 'Linux as a Virtualization Guest'
date: 2021-03-22
categories: linux virtualization
---

Linux is a versatile operating system and is able to host other operating systems or
individual applications in an isolated and secure environment using _virtualization_. This
environment allows a software platform called a _hypervisor_ to run processes that contain
a fully emulated computer system. The hypervisor manages the physical hardware resources
that can be used by individual _virtual machines_ (VM). These VMs are also called
_guests_.

## Common Hypervisors for Linux

There are several popular hypervisors for Linux, including the following:

- [Xen](https://xenproject.org/) is an open source Type-1 hypervisor, which means it does
  not rely on an underlying operating system. This is known as a _bare-metal_ hypervisor, as
  the computer boot directly into the hypervisor platform.
- [KVM](https://en.wikipedia.org/wiki/Kernel-based_Virtual_Machine) is the _Kernel
  Virtual Machine_: A Linux kernel module for virtualization. It is both Type-1 and Type-2,
  since it requires a generic Linux operating system to run. It uses the `libvirt` daemon.
- [VirtualBox](https://www.virtualbox.org/) is a Type-2 hypervisor.

Note: Some hypervisors allow for the dynamic relocation or _migration_ of a VM. If this is
performed while the machine is running, it is a _live migration_.

## Types of Virtual Machines

There are three main categories of virtual machines:

- **Fully Virtualized** is a guest OS that runs within a fully-virtualized environment.
  All instructions are simulated (or are real hardware). It does not appear to the guest OS
  that it is a virtual machine. This requires [Intel-VT-x](https://software.intel.com/content/www/us/en/develop/articles/intel-virtualization-technology-for-directed-io-vt-d-enhancing-intel-platforms-for-efficient-virtualization-of-io-devices.html) or AMD-V CPU extensions to be
  enabled in BIOS or UEFI settings.
- **Paravirtualized** is when the guest OS is "aware" it is a virtual machine. This uses a
  modified kernel and special _guest drivers_ that make use of the software and hardware
  resources of the hypervisor. Performance is better than a fully virtualized environment.
- **Hybrid** is a combination of both above approaches, and offers near-native I/O
  performance, using paravirtualized drivers on a fully virtualized operating system.

## Example `libvirt` Virtual Machine

Virtual machines consist of a collection of files: an XML file that _defines_ the virtual
machine (such as hardware configuration, network connectivity, display capabilities, and
more) and an associated hard disk image file that contains the installed of the guest OS
and included software.

The XML file defines hardware settings such as:

- Amount of RAM assigned to the VM.
- Number of CPU cores the VM has access to.
- The hard disk image that is associated with it.
- Display capabilities (via [SPICE](https://www.spice-space.org/) protocol).
- Guest access to USB devices, emulated keyboard and mouse input.

The associated hard disk image file has two primary types of _disk provisioning_:

- **Copy on write** (COW), also known as _thin-provisioning_ or _sparse images_, is a
  method where the disk is created with a pre-defined upper space limit. The disk size
  increases as new data is written. The guest OS sees the predefined limit, while the host
  sees the real data size.
- **RAW** or _full disk_ is when the image has all the space pre-allocated. There is a
  performance benefit to this, since it is not necessary to adjust the size of the image as
  data is written to it.

Other virtualization platforms, such as _Red Had Enterprise Virtualization_ and _oVirt_
can use physical disks to act as backup storage locations for a virtual machine's
operating system. These platforms are also able to use _storage area network_ (SAN) or
_network attached storage_ (NAS) to storage data, and the hypervisor keeps track of what
storage locations belong to which VM. Such storage schemes can also use _Logical Volume
Management_ (LVM) to grow/shrink volume sizes as required.

Note: Another benefit of virtual machines is the availability of _templates_ with a basic
operating system install some pre-configured settings to ease future system launches.

### The D-Bus Machine ID

One important identifying data point is a number generated at install time called the
_D-Bus Machine ID_. This number must be unique, so if a VM is duplicated a new ID must be
created. In order to validate that a D-Bus machine ID exists for a running system, use the
command `dbus-uuidgen --ensure` and view the current ID with `dbus-uuidgen --get`.

The D-Bus machine ID is located at `/var/lib/dbus/machine-id` and is a symbolic link to
`/etc/machine-id`. Changing this while the machine is running is not advised. To generate
a new one, first remove the old one with `sudo rm -f /etc/machine-id` then use the command
`sudo dbus-uuidgen --ensure=/etc/machine-id`. If `/var/lib/dbus/machine-id` is _not_ a
symbolic link, it must also be removed.

## Deploying VMs to the Cloud

Many companies now exist to provide _Infrastructure as a Service_ ([IaaS](https://www.oracle.com/cloud/what-is-iaas/)). These companies
provide hypervisor systems and can deploy virtual guest images for clients. When assessing
a deployment of a Linux system in an IaaS environment, it is important to consider the
following elements:

- **Computing Instances**: Many cloud providers charge usage rates based on "computing
  instances", or how much CPU time the infrastructure will use. Careful planning of how much
  processing time is required will aid in keeping costs manageable. This also can refer to
  the number of VMs provisioned.
- **Block Storage**: Cloud providers offer various levels of storage, and the cost varies
  depending on the amount of storage required and the speed of the storage media.
- **Networking**: How will the routes, subnetting and firewall configurations be
  implemented? Some cloud providers offer DNS solutions so that _fully qualified domain
  names_ ([FQDN](https://en.wikipedia.org/wiki/Fully_qualified_domain_name)) can be publicly assigned to the systems. Hybrid systems can tie together
  both types of systems via a [VPN](https://en.wikipedia.org/wiki/Virtual_private_network).

### Securely Accessing Guests in the Cloud

The prevalent method for accessing a remote virtual guest is with [_OpenSSH_](https://www.openssh.com/).
To generate a public and private pair of SSH keys, use the `ssh-keygen` command. Then, to
copy the public key to a remote server, use `ssh-copy-id -i /path/to/publickey.pub user@server`.
The permissions for SSH keys are important, they **must** be `0600` for a private key and
`0644` for a public key, otherwise the `ssh-agent` will throw an error.

## Preconfiguring Cloud Systems

Modern cloud deployments use pre-defined virtual machine images, along with associated
configuration files. Utilities such as `cloud-init` are _vendor neutral_ methods for
deploying a Linux guest system. The `cloud-init` utility uses [YAML](https://en.wikipedia.org/wiki/YAML)
plain-text files to pre-configure network settings, software package selections, SSH key
configuration, user account creations, locale settings, along with many other options to
streamline the deployment of a system.

An example configuration file might look a bit like this:

```yaml
#cloud-config
timezone: Africa/Dar_es_Salaam
hostname: my-server

# Update the system when it first boots up
apt_update: true
apt_upgrade: true

# Install the Nginx web server
package:
  - nginx
```

The `cloud-init` tool can be used to pre-configure containers, such as
[LXD containers](https://linuxcontainers.org/lxd/introduction/), prior to deployment.

## Containers

Similar to virtual machines, containers aim to provide an isolated environment, with just
enough software to run the desired application. This means a container has less _overhead_
when compared to a virtual machine, and a container is also more flexible. For example, a
container does not need to be powered off to be migrated.

A few of the most popular _container orchestration software_ include:

- [Docker](https://www.docker.com/)
- [Kubernetes](https://kubernetes.io/)
- [LXD/LXC](https://linuxcontainers.org/lxd/)
- [systemd-nspawn](https://wiki.archlinux.org/index.php/Systemd-nspawn)
- [OpenShift](https://www.openshift.com/)

Containers use the _cgroups_ (control groups) mechanism provided by the Linux kernel. This
is a way to partition system resources such as memory, processor time, disk and network
bandwidth, for individual applications. While an administrator can be cgroups to set
system resource limits on an application (or group of applications), container
orchestration software provides tools that ease the management and deployment of such
configurations.

## Summary

Linux is an excellent choice for both a host OS and a guest OS. Virtualization now an easy
and efficient way to deploy Linux machines. Whether it is as a fully emulated OS, or as a
container to run a specific application, Linux can do it all.
