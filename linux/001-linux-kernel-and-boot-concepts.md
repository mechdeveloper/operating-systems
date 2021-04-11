# Explain Linux Kernel and Boot Concepts

## BIOS or UEFI

BIOS and UEFI are interfaces b/w hardware and operating system

- BIOS - Basic Input Output System
- UEFI - Unified Extensible Firmware Interface

### UEFI

- UEFI supports secure boot (BIOS doesn't support secure boot).
- UEFI is replacement of BIOS, UEFI replaces the functionality of connecting the hardware to the software of the Operating System.
- Most computers are now coming with UEFI, Computers booted using UEFI creats UEFI partition and puts all the boot code in that partition.

## Boot Loaders

> GRUB stands for Grand Unified Boot Loader

- GRUB
  - GRUB "Legacy"
  - menu.lst, grub.conf
  - difficult to modify
  - boot menu usually displays on boot

- GRUB2
  - grub.cfg
  - customizable in /etc/default/grub
  - can boot ISO, USB, UUID, device
  - hidden boot menu (press shift!)

linux system can boot in multiple ways

- PXE
  - Pre Boot Execution Environment
- iPXE
- USB
- CD
- ISO (ISO images)

### Hardware

BIOS or the UEFI - the part that takes place before linux is introduced.

#### PXE - Pre Boot Execution Environment

- when hardware is not setup to use a harddrive, it queries a `dhcp` server and dhcp server responds with an ipaddress and also a boot file and a `tftp` location (`tftp` server is just a place where we can store files on a network). Computer then downloads that boot image from `tftp` server, that bootfile is the linux kernel downladed from local network and then hardware puts it in memory and boots itself from there.
- PXE starts as a hardware thing and turns into a software

#### iPXE

- iPXE is very similar but instead of using `tftp` to download bootfile it allows you to use `http` which is faster and usually more reliable than `tftp`

#### USB

- Hardware determines/knows it can boot from USB
- On USB itself is where linux code is
- Similar to USB CD or HD (HardDrive)

### Software

- ISO booting with GRUB2 that's after linux starts or after GRUB2 starts
- Select Kernel
- Memtest
- Most things have to start in hardware before software takes control
- PXE, iPXE, USB, CD are not linux specific boot methods, these are boot methods that computer supports that linux also supports

## Boot Process

1. BIOS or UEFI
    - Hardware on Computer
    - looks for GRUB or GRUB2

2. GRUB or GRUB2
    - Boot code of GRUB - Points us to the Kernel which is actual linux kernel one file called `vmlinux` or `vmlinuz` only diff is vmlinuz is compresssed.

3. vmlinux or vmlinux
    - vmlinux or vmlinux are linux kernel itself with no modules, this is just the base kernel
    - once the kernel boots up it will mount the file system
    - then kernel will have access to all the modules that it needs to insert to make things work like USB mouse, keyboard, monitor, video card all these are modules that are loaded into the kernel as these are not part of static kernel. kernel becomes full kernel

    - `initrd` - init ram disk - this is just enough information like module and driver information to be able to have the linux kernel access the filesystem so that it can get to its modules.
    - `initramFS` is similar to `initrd` except that initramFS is acutally part of actual kernel vmlinux. `initrd` is mounted alongside kernel. `initrd` is not used after system is booted up `initrd` just a staging grount to get to the full kernel

## Kernel Panic

- Faulty Hardware
  - Overclocked CPU
  - Bad RAM stick
  - Add-on cards like video cards
  - Any hardware which fails can cause kernel pacnic
- System Updates
  - Go to Grub menu (restart and press shift) and pick and older kernel
  - System update on linux keeps the older kernel
- You can also boot from USB or CD to get into the harddrive

## Loading/Blacklisting Kernel Modules on Boot

Linux kernel is modular and doesn't loads all the drivers automatically by default. It had modules that kernel dynamically loads when it discovers that it needs like a driver for a sound card. We can configure kernel to automatically load modules by setting it up in a config file, behined the scenes there is some incredible dependency checking going on to check what modules a particular module depends on. __Blacklisting__ is to make sure that wrong module is not loaded on boot automatically by the system detecting a dependency that you donot want it to use.

```bash
cat etc/modules
```

- normally system automatically detects the hardware and knows what to load.
- we can manually load a kernel module and have it done automatically on boot even if the system doesn't detect it on boot.
- If there are dependencies on a module system will also load those dependencies

- Blacklist module from being loaded

    ```bash
    ls etc/modprobe.d
    ```

    you can add blacklisted modules on any of the `.conf` files inside `etc/modprobe.d`.

    anything which we manually add we put it in `blacklist.conf`

## Manupulating Kernel Modules

- Linux kernel is modular
- this makes linux kernel very efficient becuase of modular design
- to take advantage of this we have to make sure that we are putting the right modules in the right place with all the dependencies
- there are plenty of tools we can use to properly manipulate the kernel modules

  - insmod
  - modprobe
  - depmod
  - lsmod
  - rmmod

- `insmod` and `modprobe` appear to do the same thing insert modules into the running kernel, this is after we are booted it'll insert modules into the running kernel

- insmod
  - pathonly
  - no deps
  - fail with no explaination

  - insmode is very basic program, you have to give it full path of the kernel module that you want to install it doesn't do any dependency checking and fails with no explaination

- modprobe
  - name only
  - deps
  - needs map

  - advanced program, efficient model
  - give just the name of module, modprobe will look into all the dependencies that module will need and loads the dependent modules first
  - needs a map of all the needs and dependencies on a system, there's a program to do this - run `depmod` when we are done installing a new kernel module and it will recrete dependency map so `modprobe` knows where to find dependencies. `modprobe` behind the scenes uses `insmod` to do the acutal inserting.

  - `lsmod` will show all the installed modules
  - `rmmod` remove modules
