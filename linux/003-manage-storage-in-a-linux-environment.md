# Manage Storage in Linux Environment

## Understanding GPT & MBR

- __GPT__ and __MBR__ are two different ways of taking a harddrive and shpening it up into pieces so that those pieces can be recognized and mounted as different drives on your system
- __GPT__ is newer and much more feature-rich than the old-school __MBR__.
- There is also __Protective MBR__.

- harddrive /dev/sda

| GPT | MBR |
| --- | --- |
| GUID Partition Table | Master Boot Record |
| newer and feature-rich | old-school |
| GPT supports PETABYTES | MBR is limited to 2 TB |

- GPT is part of UEFI which replaces BIOS, BIOS can still see GPT drives.

## Filesystem Hierarchy

- Linux filesystem is really cool as all of the network mounts and different harddrives and usb drives are all on one giant filesystem hierarchy

- We have a harddrive `/dev/sda1` which has root partiion on it `root (/)`
- `root (/)` partiion is where everything starts and is the basis of the linux filesystem, inside there's tons of files and folders.
- some files are directly on `/dev/sda` drive, some of the folders and files in there are a virtual file system `proc`folder, `sys` folder, these are dynamically created file systems that are just a way to interact with the kernel itself. if you want to make changes in running kernel you can make changes into the files in these folders - virtual file system .
- we can also have a remote NFS server which is mounted on the folder inside of root system
- USB drive `/media/usb1/` its just mounted somewhere on a folder in monolith filesystem
- even a second harddrive `/dev/sdb2` will mount inisde `root (/)` drive for eg its going to appear as a folder `/mnt/data/` inside root filesystem
- everything is inside root filesystem
- virtual filesystem is not actually a filesystem but its like an interface desinged as a filesystem so that we can interact with kernel itself

- tree representation of all the files and folders

```bash
tree <foldername>
```

- absolute vs relative, `.` and `..` are special folder entries, `.` means current folder. `..` means directory above our current direcotory
- we can use `..` to talk about relative path of where we are going
- absolute path starts at root level `/home/bob/Pictures/Trips/Grocery\ Store/`
- we can use relative path using `..`
- tilda `~` is shortcut for home directory (`/home/username)

- linux filesystem is one big monolith filesystem, we can mount network shares, usb drives, second harddrives or even a virtual filesystem to interact with kernel its all under same filesystem.

## Creating Paritions

- paritions are organizational unit that are on a hard drive.
- we can divide a harddrive in different partiions and those partitions are used for different things
- depending on system we are using MBR (master boot record) or GPT (GUID partition table) you can have multitude of partiions or just a few also depening on which scheme you use dpends how big your hard drive can be.
- we will look at tools like
  - `parted` - partition editor and `gparted` - graphical partiion editor
  - `fdisk` which is command line tool
  - info tools - that lets you see what block devices are available to partition on your system

- identify blockdevices on your system

```bash
lsblk 
```

```bash
cat /proc/partitions 
```

```bash
ls /dev | grep sd 
```

- we can use `gparted` to partition our drives if system has GUI
- we can use `parted` or partition editor CLI program to partiion our drives
- `fdisk` - almost every linux system has

```bash
fdisk /dev/sdb
```

- `/dev/sdb` is the device we want to use
- type `m` for help
- we can use `g` for creating GPT partition table or `o` to create DOS partition table
- `g` - press enter to use GPT, this will print the GUID and build the new GPT disklabel
- `p` - to create a partition first type `p` to see what exists
- `n` - to add a new partition, give partion number (1-128), default will be the first one availalbe, Last sector (size) choose default to fill entire drive with this partiion
- `p` - we can check the partition using `p` and it will show our partiion
- `q`- press q to exit wihout making changes
- `w` - use `w` to make changes and exit

- next we can use `lsblk` it will show up the partition we have created

## Formatting Various Filesystems

Bunch of different options available in linux for filesystems

Various Filesystems in linux

- ext
  - most common family of file systems
  - mature
  - later versions support journals
  - ext4 is the newest and has the most features

- xfs
  - older
  - still used by CentOS
  - has its own set of xfs tools

- btrfs
  - new
  - many features like snapshot
  - abandoned

- dos (windows world)
  - ntfs
  - vfat
  - fat32

choose ext4 as default filesystem as its widely used and has big ecosystem around it.

- to Create a filesystem you need a partitioned harddrive so that you can have a partition to put filesystem on.
- use `lsblk` to check block devices and partiions on your system
- the file formatting or hardrive formatting programs all start with `mkfs` make filesystem and then press tab to see all the various tools for creating different types of filesystems.
- `mkfs.ext4 /dev/sdc1`
- `lsblk -f` -f to show us the filesystems that are on the particualr block devices
- you need to have a partition before you can create a filesystem on it.

## Mounting Partitions (manually, and at boot)

- mount / unmount
- /etc/fstab
- blkid

- to be able to access the data on the drives or partitions of the drives that we put into our system we have to mount them into our local filesystem.
- we can do this mounting manually using `mount` / `umount` or
- we can use `/etc/fstab` to mount automatically on boot

- conceptually we have a new hardrive and we want to mount it into our filesystem
- when we mount a partition or a harddrive it goes into a folder and that folder becomes what is in that drive. it has to be mounted on a empty folder

- usually we have regualr root (/) mounted harddrive.
- using `lsblk` we can see our partion which is setup and ready to mount which is not currently mounted on our system
- in order to mount we look inside /mnt `ls /mnt/` we need to look for empty folder
- `blkid` will show little bit more information about the device
- `mount -t ext4 /dev/sdb1 /mnt/10gig/` usually mount can figureout the filesystem type but if we know the filesystem type its good to specify that
- `ls /mnt/10gig` will now show the root level of our harddrive which now lives in our file system
- we can type `mount` alone - it will show us where the drive is mounted
- we can unmount our drive by typing `umount /mnt/10gig`

- to automatically mount on boot we need to edit the file `/etc/fstab`

```bash
<filesystem> <mount point>  <type>  <options>   <dump>  <pass>
/dev/sdb1    /mnt/10gig     ext4    defaults    0       0
```

- options - defaults
- dump - deprecatd we can put 0
- pass - 0 - never run a filesystem check, 1 - run the filesystem check first you put 1 on the root partition any other partion where you wanna have a check when system boots up you will put a 2, for eg 5 different partions can all have a pass of 2

- save the /etc/fstab file
- run `mount -a` to mount everything that is specified in fstab file.
- we can look by running `mount` and it will show our partion mounted on linux filesystem.
- now if we reboot our computer it will automatically mount our partiion.

## Scanning Filesystems

- tune2fs
- /etc/fstab
- fsck

- to scan a linux filesystem generally we use the tool `fsck`
- real key is to have it scan automatically periodically on boot so that you donot have to manually scan
- in order to run `fsck` filesytem has to be unmounted. its not a problem for secondary or tertiary drive mounts like home directory but root directory its difficult to unmount the root directory and scan it unless you're in the bootup process or you've booted from a cd or something.
- we can setup the system to scan the filesystem on boot including the root filesystem so that you can have it automatically maintain itself.

- for scanning automatically there are few different things that go on -
  - the very first thing kernel looks for is inside your `/etc/fstab` file if the PASS setting is setup.
  - if PASS is set to 0 then it won't scan
  - if PASS is set to 1 or 2
    - kernel will check has max number of allowed mounts been reached ?
      - if yes - scan drive
      - if not - donot scan even if PASS is setup to scan if it haven't met the MAX. MAX by default is -1 means it will never scan. by default you will never get auto scan.

- on ubuntu type `mount` to check the mounted partitions.
- in `/etc/fstab` file its setup with PASS of 2 kernel is going to check and scan if its time.
- to check the max number of allowable mounts before it will scan we can use

```bash
tune2fs -l /dev/sdb1
```

- look for `Maximum Mount count` in the results of above command by default it never scan as this value set to -1.
- to change the value of `Maximum Mount count`

```bash
tune2fs -c 10 /dev/sdb1
```

- now `Maximum Mount count` is 10,  this means everytime system boots it mounts the partition and once the Mount Count reaches more than 10 it will scan on boot.

- we can increase `Mount Count` manually

```bash
umount /dv/sdb1; mount /dev/sdb1
```

- Now `Mount Count` will increment by 1
- kernel only scans on boot. when mount count is higher than max count
- after scan the Mount Count will be reset to 1

- sometimes it makes sense not to automatically scan and manually scan using fsck by unmounnging and booting from cd

## LVM - Logical Volume Manager

- logical volume manager is basically like the software version of SAN storage area network.
- it allows you to take a whole bunch of physical devices and lump them into one big group that allows you to create storage for use in your local system
- it consists of a bunch of parts
  - physical volumes
  - volume groups
  - logical volumes
- its a way of taking  raw storage combining it to divide, expand, contract, add more things to it without disrupting the existing services

- We start with PV - Physical Volumes, lets say 4 10gb drive
  - 10gb
  - 10gb
  - 10gb
  - 10gb

  - these drives could be
    - hard drives
    - RAID device
    - partition on a harddrive
  - we create these physical volumes out of these drives
  
- we combine these physical volumes into VG - Volume Groups
  - all above 4 10gb drives are combined into a Volume Group of 40 GB
    - big bucket of storage (40 GB)
    - no protection
- we can use a slice of this 40 GB storage say 7 GB or 30 GB or 32 GB this slice of storage is callled a LV - Logical Volume
- This logical volume is what we foramt with filesystem and mount on our local filesystem

- CentOS use LVM even if we have 1 drive
- we can check `/etc/fstab` file

```bash
cat /etc/fstab
```

- output will show our device

```bash
/dev/mapper/centos-root  /      xfs     defaults    0   0
/dev/mapper/centos-swap  swap   swap    defaults    0   0
```

- logical volume manger creates logical volumes for us to use
- if we look inside `/dev/mapper`

```bash
ls /dev/mapper
```

- we can see we have

```bash
centos-root    centos-swap    control
```

- `pvdisplay` will show us behined the scenes

```bash
--- Physical volume ---

PV name         /dev/sda2
VG name         centos
PV Size         <19.00 GiB / not usable 3.00 Mib>
Allocatable     yes (but full)
PE Size         4.00 MiB
Total PE        4863
Free PE         0
Allocated PE    4863
PV UUID         5Yy9MU-xt8M-pZru-qunw-v4ws-YGyR-dMJ6B0
```

- we have 1 physical volume and its a partition `/dev/sda2`. its in a volume group named `centos` and gives us 19.00 GiB of storage. PV physical volume is inside of VG Volume Group called `centos`

- `lvdisplay` to check our sub slices of `centos` VG Volume Group
- we see two Logical Volumes

```bash
--- Logical volume ---
LV Path             /dev/centos/swap
LV Name             swap
VG Name             centos
LV UUID             XXXXXX-XXXX-XXXX-XXXX-XXXX-XXXX-XXXXXX
LV Write Access     read/write
...

--- Logical volume ---
LV Path             /dev/centos/root
LV Name             root
VG Name             centos
LV UUID             XXXXXX-XXXX-XXXX-XXXX-XXXX-XXXX-XXXXXX
LV Write Access     read/write
...

```

- to check our two different logical volumes

```bash
ls /dev/centos
root  swap
```

## Building an LVM System

- pvcreate
- vgcreate
- lvcreate

- Bulding and LVM is very stratightforward thing you can do when it comes to block storage devices on linux
- Once LVM system is built we can expand it by adding more drives in the system

- `lsblk` to list blockstorage
- we can see the drive on which system is installed
- we can also see addtional drives/devices sdb, sdc, sdd, sde
- Notice we donot have partitions created on these drives
- we can create partitions, some prefer to use partitions for their PV physical volumes in an LVM, some prefer to use the raw devices either one works fine, they work the same.
- advantage of setting up partition is that if someone else comes to the system they are going to see that there's partitions on the system and they're going to know that something is already done there whereas if we leave them as raw devices other users might think of raw devices as empty drives

- to turn the raw devices in to physical volumes we use `pvcreate`

```bash
pvcreate /dev/sdb /dev/sdc /dev/sdd /dev/sde
```

- now we do `pvdisplay` - it will show all the devices we have

- Next Step is to create a volume group using `vgcreate <nameofVG> <lsitofPVtoaddtoVG>`

```bash
vgcreate bucket /dev/sdb /dev/sdc /dev/sdd /dev/sde
```

- we can check our volume group by using `vgdisplay`

- We can use `lvcreate` to use some part of the Volume Group we created and create a filesystem on it

```bash
lvcreate -L 32G -n BIG_SLICE bucket
```

- use `lvdisplay` to check our newly create logical volume

- now we want to use our logical volume as a block device, create a filesystem on our logical volume

```bash
mkfs.ext4 /dev/bucket/BIG_SLICE
```

- we can mount this filesystem somwhere and everytime system starts this filesystem will be avaialble on `/dev/bucket/BIG_SLICE` and if we put it in `/etc/fstab` its going to be mounted on boot

- we can use `lvextend` to increase the size of our logical volume

```bash
`lvextend -L+5G /dev/bucket/BIG_SLICE`
```

- we increased the size without doing anything to the system just by using the tools to change the size of our logical volumes.

## Things to remember about LVM

- LVM provides flexibility in your system
- LVM doesn't provides any redundancy
- if we have 1 physical volume fail, its going to crash the entire volume group and logical volumes are going to get messed up
- You want your PV physical volumes to be like a raid device. If you are worried about something going wrong underneath and losing data.
- Setting up LVM is very simple

## RAID

- RAID is a Redundant Array of Independent Disks or Drive
- It means you take bunch of drives and put them togeather and you end up with a larger pool of storage
- It doesn't just pull things together like LVM, it allows you to do some pretty neat things with performance or redundancy

## RAID Levels

- RAID Ø
- RAID 1
- RAID 5
- Others

- There are different RAID levels that we can offer using RAID, specifically linux RIAD - linux has software version of RAID which is very powerful very robust and surprisigly efficient

- RAID Ø : Drives are setup in a stripe, this means they work togeather, reads and write occours across two drives very very quickly. With RAID Ø or striped array its very fast, we can get dual writes and dual reads at the same time.
- If you loose a single drive in a striped array or RAID Ø, your data is gone as half of your data is written in the losen drive.

```bash
[ ] [ ]
```

- RAID 1 : Has multiple drives which are setup in a mirror which means, if we write someting to the first drive, you also write it to the second drive, so you have a complete copy of both.
- if one of the drive dies, we still have full set of data, so we are not getting the speed increase.  we are not spreading writes across drives - we're actuall writing all of our data two times once to each drive. We donot get any advantage over single drive when it comes to speed but we do get an advantage that either drive an die and we still have full data.

```bash
[ ]
[ ] 
```

- RAID 5 : RAID 5 uses a parity disk. You have multiple drives (3 minimum) and any one of these drives can die and all of our data is still in place. We get the advantage of being able to loose any drive in your array. Disadvantage is you loose one drive's worth of storage. for eg we have three 5 GiB drives and togeather we have 15 GiB storage and if we loose one drive's we will only have 10 GB of usable space. Advantage is its writing to multiple drives as its going along and if one of the drives dies you still have all of your data represented in the remaining drives. If you loose two out of three drives then your data is corrupt and you loose your data. but if you loose any one of the three drives your data is still there

```bash
  [ ]
[ ] [ ] 
```

- there are some Hybrid levels as well for eg RAIDØ1 where you have 4 drives and here you have a stripe of mirrors, dirves are mirrored and striped across the mirror or RAID1Ø which is a mirror of stripes.

- with RAID5 there is acutally a RAID6 which requires another 4rth drive and then we can loose upto 2 drives and still have our data, downside is we loose two drives worth of storage on our full array

## Configuring RAID with mdadm

- we can buy a RAID card like a hardware raid device and then we can use it on our system and we will be able to have a hardware RAID, but linux has a really awesome and powerfull software RAID program that will use kernel level tools to allow you to create your own raid devices without needing any speciality hardware at all.

- partitions - vs - raw
- mdadm.conf
- /proc/mdstat

- lets say you have setup RAID with two hard drives both with different make one with size 1028 MB another with 1022 MB lets say if 1028 MB drive fails and we wnat to replace it with 1022 MB drive then we cannot replace it that's why we first partition both drives to lets say 999 MB size before configuring RAID.

- We generally use partitions even though we use raw device, which would work untill we need to replace it wil a smaller drive that is your replacement.

- use `lsblk`  to check your drives along with partitions
- lets say we have 4 drives with 10 G size we have 4 drives with partition of 9.9 G size

- partition a drive

```bash
fdisk /dev/sde
```

- `o` - to create a new partition type with DOS (can be any type)
- `n` - for new partition (default primary), parition number (default 1), First Sector (default 2048), Last Sectory -- no here we need slighly smaller partition so instead of choosing default we specify +9.9G now we have a partition of 9.9 G and any 10 G drive an replace this if reqiuired.
- if we do `t` for type it will ask what partition type do we want, here it says it created a new partition type of linux but if we type caps `L` we will see all of the availalbe code, this is not a format but just a hint to kernel what type of partition it would be for our case we want to put `fd` `Linux raid auto` this will change partition type to `Linux raid autodetect`.
- `w`  - to write the changes to disk.

- now if we do `lsblk` we can see all our 4 drives with 9.9 G partition

- Now we will create RAID5 device with four 10 GB devices, we will end up with 30 GB usable space for our RAID5 array.

- tool we use is `mdadm`

```bash
mdadm --create --verbose /dev/md0 --level=5 --raid-devices=4 /dev/sdb1 /dev/sdc1 /dev/sdd1 /dev/sde1 
```

- check `cat /proc/mdstat` this virtual filesystem `proc` and its gonna show us the `mdstat` which is current raid arrays on our system

- look into `ls /dev | grep md` we can see `md0` and we can use it as a harddrive in our system.

- we can save this configuration of RAID5 array into our system so that on boot it know exactly what sort of array to build.

```bash
mdadm --detail --scan 
```

- this will show us the configuration of current array. we can save that to `/etc/mdadm/mdadm.conf`

```bash
mdadm --detail --scan > /etc/mdadm/mdadm.conf
```

- now everytime we boot the system `md0` is going to be created.
- and now we can treat this as any other harddrive on our system. we can now create filesystem

```bash
mkfs.ext4 /dev/md0
```

- this is now a part of our filesystem
- `lsblk` will show us the new filesystem `md0` with 29.6G RAID5 storage on our system.
