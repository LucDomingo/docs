# RHEL
## Table of Contents
[Managing Storage](#Managing_Storage)
[Managing Advanced Storage](#Managing_Advanced_Storage)
[Basic Kernel Management](#Basic_Kernel_Management)
[Understanding Systemd](#Understanding_Systemd)
[Managing and Understanding the Boot Procedure](#Managing_and_Understanding_the_Boot_Procedure)
[Essential Troubleshooting Skills](#Essential_Troubleshooting_Skills)

## Managing Storage
### Understanding MBR and GPT Partitions
Using more than one partition on a system makes sense for multiple reasons:
- It's easier to distinguish between different types of data.
- Specific mount options can be used to enhance security or performance
- It's easier to create a backup strategy where only relevant portions of
the OS are backed up.
- If one partition accidentally fills up completely, the other partitions still
are usable and your system might not crash immediately.

### Understanding the MBR Partitioning Scheme
While booting a computer, the Basic Input Output System (BIOS) was loaded to access hardware devices.From the BIOS, the bootable disk device was read, and on this bootable
device, the MBR was allocated. The MBR contains all that is needed to start a computer, including a boot loader and a partition table. The MBR was defined as the first 512 bytes on a computer hard drive, and in the MBR an operating system boot loader was present, as well as a partition table. The size that was used for the partition table was relatively small, just 64 bytes, with the result that in the MBR no more than four partitions could be created. Since partition size data was stored in 32-bit values, and a default sector size of 512 bytes was used, the maximum size that could be used by a partition was limited to 2 TiB.
In the MBR, just four partitions could be created. Because many PC
operating systems needed more than four partitions, a solution was found to
go beyond the number of four. In the MBR, one partition could be created
as an extended partition, as opposed to the other partitions that were created
as primary partitions. Within the extended partition, multiple logical
partitions could be created to reach a total number of 15 partitions that
could be addressed by the Linux kernel.
### Understanding the Need for GPT Partitioning
Current computer hard drives have become too big to be addressed by MBR
partitions. That is one of the main reasons why a new partitioning scheme
was needed. This partitioning scheme is the GUID Partition Table (GPT).
On computers that are using the new Unified Extensible Firmware Interface
(UEFI) as a replacement for the old BIOS system, GPT partitions are the
only way to address disks. Also, older computer systems that are using
BIOS instead of UEFI can be configured with GUID partitions, which is
necessary if a disk with a size bigger than 2 TiB needs to be addressed.
Using GUID offers many benefits:
- The maximum partition size is 8 zebibyte (ZiB), which is 1024 × 1024
× 1024 × 1024 gibibytes.
- In GPT, up to a maximum number of 128 partitions can be created.
- The 2-TiB limit no longer exists.
- Because space that is available to store partitions is much bigger than
64 bytes, which was used in MBR, there is no longer a need to distinguish between primary, extended, and logical partitions.
- GPT uses a 128-bit global unique ID (GUID) to identify partitions.
- A backup copy of the GUID partition table is created by default at the
end of the disk, which eliminates the single point of failure that exists
on MBR partition tables.

### Creating MBR Partitions with fdisk
``` 
1. Type dd if=/dev/sda of=/root/diskfile bs=1M count=1. (If your disk
is /dev/vda and not /dev/sda, change the disk name accordingly.)
Using this command allows you to create a backup of the first
megabyte of raw blocks and write that to the file /root/diskfile. This
file allows you to easily revert to the situation that existed at the start
of this exercise, as the command makes a backup of the MBR and all
relevant metadata on disk.
2. Type cp /etc/fstab /root/fstab to make a backup of the /etc/fstab file
as well.
3. Open a root shell and run the fdisk command. This command needs
as its argument the name of the disk device where you want to create
the partition. This exercise uses /dev/sda. Change that, if needed,
according to your hardware.
Click here to view code image
[root@localhost ~]# fdisk /dev/sda
Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to
write them.
Be careful before using the write command.
Command (m for help):
4. Before you do anything, it is a good idea to check how much disk
space you have available. Press p to see an overview of current disk
allocation:
Click here to view code image
Command (m for help): p
Disk /dev/sda: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x7ad1a34b
Device Boot Start End Sectors Size Id Type
/dev/sda1 * 2048 1026047 1024000 500M 83 Linux
/dev/sda2 1026048 24111103 23085056 11G 8e Linux LVM
In the output of this command, in particular look for the total number
of sectors and the last sector that is currently used. If the last partition
does not end on the last sector, you have available space to create a
new partition.
5. Type n to add a new partition:
Click here to view code image
Command (m for help): n
Partition type
p primary (2 primary, 0 extended, 2 free)
e extended (container for logical partitions)
6. Assuming you have a /dev/sda1 and a /dev/sda2 partition and nothing
else, press p to create a primary partition. Accept the partition number
that is now suggested, which should be /dev/sda3.
7. Specify the first sector on disk that the new partition will start on. The
first available sector is suggested by default, so press Enter to accept.
8. Specify the last sector that the partition will end on. By default, the
last sector available on disk is suggested. If you use that, after this
exercise you will not have any disk space left to create additional
partitions or logical volumes, so you should use another last sector.
To use another last sector, you can do one of the following:
Enter the number of the last sector you want to use.
Enter +number to create a partition that sizes a specific number
of sectors.
Enter +number(K,M,G) to specify the size you want to assign to
the partition in KiB, MiB, or GiB.
Type +1G to make this a 1-GiB partition.
Click here to view code image
Command (m for help): n
Partition type
p primary (2 primary, 0 extended, 2 free)
e extended (container for logical partitions)
Select (default p): p
Partition number (3,4, default 3):
First sector (24111104-41943039, default 24111104):
Last sector, +sectors or +size{K,M,G,T,P} (24111104-
41943039,
default 41943039): +1G
Created a new partition 3 of type 'Linux' and of size 1
GiB
After you enter the partitions ending boundary, fdisk will show a
confirmation.
9. At this point, you can define the partition type. By default, a Linux
partition type is used. If you want the partition to be of any other
partition type, use t to change it. For this exercise there is no need to
change the partition type. Common partition types include the
following:
82: Linux swap
83: Linux
8e: Linux LVM
10. If you are happy with the modifications, press w to write them to disk
and exit fdisk. If you have created a partition on a disk that is already
in use, you may now see the following message:
Click here to view code image
Command (m for help): w
The partition table has been altered!
Calling ioctl() to re-read partition table.
WARNING: Re-reading the partition table failed with error 16:
Device or resource busy.
The kernel still uses the old table. The new table will be
used at the next reboot or after you run partprobe(8) or
kpartx(8) Syncing disks.
[root@localhost ~]#
11. This message indicates that the partition has successfully been added
to the partition table, but the in-memory kernel partition table could
not be updated. You can see that by comparing the output of fdisk -l
/dev/sda with the output of cat /proc/partitions, which shows the
kernel partition table.
12. Type partprobe /dev/sda to write the changes to the kernel partition
table. The partition has now been added, and you can create a file
system on it as described in the section Creating File Systems.
``` 
### Using Extended and Logical Partitions on MBR
If you want to go beyond four partitions on an MBR disk, you have to create an extended partition. Following that, you can create logical partitions within the extended partition.
See : parted, gdisk

### Creating File Systems
**XFS**
The default file system in RHEL 8.
**Ext4**
The default file system in previous versions of RHEL; still available
and supported in RHEL 8.
**Ext3**
The previous version of Ext4. On RHEL 8, there is no need to useExt3 anymore.
**Ext2**
A very basic file system that was developed in the early 1990s.
There is no need to use this file system on RHEL 8 anymore.
**BtrFS**
A relatively new file system that is not supported in RHEL 8.
**NTFS**
A Windows-compatible file system that is not supported on RHEL8.
**VFAT**
A file system that offers compatibility with Windows and Mac and
is the functional equivalent of the FAT32 file system. Useful on USB
thumb drives that exchange data with other computers but not on a
server's hard disks.

### Mounting File Systems
Just creating a partition and putting a file system on it is not enough to start using it. To use a partition, you have to mount it as well. By mounting a partition (or better, the file system on it), you make its contents accessible through a specific directory. To mount a file system, some information is needed:
- What to mount: This information is mandatory and specifies the name
of the device that needs to be mounted.
- Where to mount it: This is also mandatory information that specifies
the directory on which the device should be mounted.
- What file system to mount: Optionally, you can specify the file system
type. In most cases, this is not necessary. The mount command will
detect which file system is used on the device and make sure the correct
driver is used.
- Mount options: Many mount options can be used when mounting a
device. Using options is optional
``` 
[root@stpv1 ~]# mount /dev/sda5 /mnt
[root@stpv1 ~]# blkid
/dev/sdb: PTUUID="e8db1d30" PTTYPE="dos"
/dev/sda1: SEC_TYPE="msdos" UUID="706C-E83F" BLOCK_SIZE="512" TYPE="vfat" PARTLABEL="EFI System Partition" PARTUUID="66168081-9598-4a7c-8424-a18b72639023"
/dev/sda2: UUID="ba86bb7b-3a5b-4829-9f7a-5b4a008db3f3" BLOCK_SIZE="1024" TYPE="ext4" PARTUUID="3f1e5a33-8f9c-4a1f-85a5-ed728e3cb137"
/dev/sda3: UUID="NQLZws-0sMe-BJUe-jC2F-e4Xk-ZZ43-lwxsOo" TYPE="LVM2_member" PARTUUID="023a6d49-8a78-4f70-be60-5bf811c19f8f"
/dev/mapper/vg_systeme-lv_root: UUID="5c91d3e4-acd9-492a-aead-c419fa757071" BLOCK_SIZE="512" TYPE="xfs"
/dev/mapper/vg_systeme-lv_usr: UUID="2ad1d28f-4401-4fbf-87cc-124747f7b6da" BLOCK_SIZE="512" TYPE="xfs"
/dev/mapper/vg_systeme-lv_opt: UUID="d7cf8751-a07c-4cc4-8834-4b04d2eb859c" BLOCK_SIZE="512" TYPE="xfs"
/dev/mapper/vg_systeme-lv_home: UUID="3330c862-c078-475c-ba52-2671745131ac" BLOCK_SIZE="512" TYPE="xfs"
/dev/mapper/vg_systeme-lv_log: UUID="4b1a36a7-ef34-4fb0-b751-a936d719ee0e" BLOCK_SIZE="512" TYPE="xfs"
/dev/mapper/vg_systeme-lv_var: UUID="51f78974-f7aa-484c-aa1f-d836fbd4199a" BLOCK_SIZE="512" TYPE="xfs"
/dev/mapper/vg_systeme-lv_tmp: UUID="d889006d-8e58-4759-9cc5-26d707a4fa7c" BLOCK_SIZE="512" TYPE="xfs"
``` 
Automatic mounting via /etc/fstab

## Managing Advanced Storage
You first have to convert physical devices, such as disks or partitions, into
physical volumes (PVs); then you need to create the volume group (VG) and assign PVs
to it. As the last step, you need to create the logical volume (LV) itself
``` 
[root@stpv2 ~]# lsblk
NAME                     MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
loop0                      7:0    0   4.3G  0 loop /home/emergency/stpv
sda                        8:0    0 931.5G  0 disk
+-sda1                     8:1    0   500M  0 part /boot/efi
+-sda2                     8:2    0   250M  0 part /boot
+-sda3                     8:3    0 930.8G  0 part
  +-vg_systeme-lv_root   253:0    0  14.7G  0 lvm  /
  +-vg_systeme-lv_usr    253:1    0  14.7G  0 lvm  /usr
  +-vg_systeme-lv_home   253:2    0 842.8G  0 lvm  /home
  +-vg_systeme-lv_vartmp 253:3    0  14.7G  0 lvm  /var/log/audit
  +-vg_systeme-lv_log    253:4    0  14.7G  0 lvm  /var/log
  +-vg_systeme-lv_var    253:5    0  14.7G  0 lvm  /var
  +-vg_systeme-lv_tmp    253:6    0  14.7G  0 lvm  /tmp
``` 
### Creating a Physical Volume
1. Open a root shell and type parted /dev/sdc.
2. Type print. This will show the current partition table layout. There should be none
at this point.
3. Type mklabel msdos to set the MBR-compatible partition type.
4. Type mkpart to start the procedure to create a partition, and enter primary when
asked for the partition type.
5. Type xfs to specify that the XFS file system should be used. When asked for the
Start position of the partition, type 1MiB, and when asked for the end, enter 1GiB.
6. Type set 1 lvm on to enable the LVM partition type on the partition.
7. Now that the partition with the correct partition type has been created, type quit to
close the parted interface.
8. Now that the partition has been created, you need to flag it as an LVM physical
volume. To do this, type pvcreate /dev/sdc1. You should now get this output:
Physical volume /dev/sdc1 successfully created.
9. Type pvs to verify that the physical volume has been created successfully. The
output may look like Example 15-3. Notice that in this listing another physical
volume already exists; that is because RHEL uses LVM by default to organize
storage.
#### Utils
Command
``` 
pvs, pvdisplay
``` 
### Creating the Volume Groups
Just type vgcreatefollowed by the name of the volume group you want to create and the name of the physical device you want to add to it. So, if the physical volume name is /dev/sdc1, the complete command is vgcreate vgdata /dev/sdc1.
#### Utils
Command
``` 
vgdisplay, vgs
``` 
### Creating the Logical Volumes and File Systems
The volume size can be specified as an absolute value using the -L option. Use, for
instance, -L 5G to create an LVM volume with a 5-GiB size. Alternatively, you can use
relative sizes with the -l option. For instance, use -l 50%FREE to use half of all
available disk space. Youll further need to specify the name of the volume group that the
logical volume is assigned to, and optionally (but highly recommended), you can use -n
to specify the name of the logical volume. For instance, use lvcreate -n lvvol1 -L 100M
vgdata to create a logical volume with the name lvvol1 and add that to the vgdata
volume group. Once the logical volume has been created, you can use the mkfs utility to
create a file system on top of it.

##### Appendix A. The Device Mapper
https://access.redhat.com/documentation/en/red_hat_enterprise_linux/5/html/logical_volume_manager_administration/device_mapper

Device mapper devices are generated on detection and use meaningless names like
/dev/dm-0 and /dev/dm-1. To make these devices easier to access, device mapper creates
symbolic links in the /dev/mapper directory that point to these meaningless device
names. The symbolic links follow the naming structure /dev/mapper/vgname-lvname.
So, the device /dev/vgdata/lvdata would also be known as /dev/mapper/vgdata-lvdata.
When working with LVM logical volumes, you can use either of these device names.
```
[root@stpv1 ~]# ls -l /dev/vg_systeme/
total 0
lrwxrwxrwx. 1 root root 7 Sep 20 08:46 lv_home -> ../dm-3
lrwxrwxrwx. 1 root root 7 Sep 20 08:46 lv_log -> ../dm-4
lrwxrwxrwx. 1 root root 7 Sep 20 08:46 lv_opt -> ../dm-2
lrwxrwxrwx. 1 root root 7 Sep 20 08:46 lv_root -> ../dm-0
lrwxrwxrwx. 1 root root 7 Sep 20 08:46 lv_tmp -> ../dm-6
lrwxrwxrwx. 1 root root 7 Sep 20 08:46 lv_usr -> ../dm-1
lrwxrwxrwx. 1 root root 7 Sep 20 08:46 lv_var -> ../dm-5
[root@stpv1 ~]# ls -l /dev/mapper/
total 0
crw-------. 1 root root 10, 236 Sep 20 08:46 control
lrwxrwxrwx. 1 root root       7 Sep 20 08:46 vg_systeme-lv_home -> ../dm-3
lrwxrwxrwx. 1 root root       7 Sep 20 08:46 vg_systeme-lv_log -> ../dm-4
lrwxrwxrwx. 1 root root       7 Sep 20 08:46 vg_systeme-lv_opt -> ../dm-2
lrwxrwxrwx. 1 root root       7 Sep 20 08:46 vg_systeme-lv_root -> ../dm-0
lrwxrwxrwx. 1 root root       7 Sep 20 08:46 vg_systeme-lv_tmp -> ../dm-6
lrwxrwxrwx. 1 root root       7 Sep 20 08:46 vg_systeme-lv_usr -> ../dm-1
lrwxrwxrwx. 1 root root       7 Sep 20 08:46 vg_systeme-lv_var -> ../dm-5
```

##### LVM Management Essential Commands
**pvcreate** => Creates physical volumes
**pvs** => Shows a summary of available physical volumes
**pvdisplay** => Shows a list of physical volumes and their properties
**pvremove** => Removes the physical volume signature from a block device
**vgcreate** => Creates volume groups
**vgs** => Shows a summary of available volume groups
**vgdisplay** => Shows a detailed list of volume groups and their properties
**vgremove** => Removes a volume group
**lvcreate** => Creates logical volumes
**lvs** => Shows a summary of all available logical volumes
**lvdisplay** => Shows a detailed list of available logical volumes and their properties
**lvremove** => Removes a logical volume

### Resizing LVM Logical Volumes
One of the major benefits of using LVM is that LVM volumes are easy to resize, which is
very useful if your file system is running out of available disk space. If the XFS file
system is used, a volume can be increased, but not decreased, in size. Other file systems
such as Ext4 support decreasing of the file system size also. You can decrease an Ext4
file system offline only, which means that you need to unmount it before you can resize
it.
### Resizing Volume Groups
The vgextend command is used to add storage to a volume group, and the
vgreduce command is used to take physical volumes out of a volume group.
This procedure is relatively easy:
1. Make sure that a physical volume or device is available to be added to the volume
group.
2. Use vgextend to extend the volume group. The new disk space will show
immediately in the volume group.
### Resizing Logical Volumes and File Systems
To grow the logical volume size, use lvextend or lvresize, followed by the -r option to
resize the file system used on it. Then specify the size you want the resized volume to be.
The easiest and most intuitive way to do that is by using -L followed by a + sign and the
amount of disk space you want to add, as in lvresize -L +1G -r /dev/vgdata/lvdata

```
1. Type pvs and vgs to show the current physical volume and volume group
configuration.
2. Use parted to add another partition with a size of 1 GiB. Do not forget to flag this
partition with the LVM partition type using set lvm on. Ill assume this new
partition is /dev/sdb2 for the rest of this exercise. Replace this name with the name
used on your configuration if it is different.
3. Type vgextend vgdata /dev/sdb2 to extend vgdata with the total size of the
/dev/sdb2 device.
4. Type vgs to verify that the available volume group size has increased.
5. Type lvs to verify the current size of the logical volume lvdata.
6. Type df -h to verify the current size of the file system on lvdata.
7. Type lvextend -r -l +50%FREE /dev/vgdata/lvdata to extend lvdata with 50% of
all available disk space in the volume group.
8. Type lvs and df -h again to verify that the added disk space has become available.
9. Type lvreduce -r -L -50M /dev/vgdata/lvdata. This shrinks the lvdata volume by
50 MB. Notice that while doing this the volume is temporarily unmounted, which
happens automatically. Also note that this step works only if youre using an Ext4
file system. (XFS cannot be shrunk.)
```
##### See also
Stratis and VGO

## Basic Kernel Management
The kernel manages the I/O instructions it receives from the software and translates them into the processing instructions that are executed by the central processing unit and other hardware in the computer. The kernel also takes care of handling essential operating system tasks. One example of such a task is the scheduler that makes sure any processes that are started on the operating system are handled by the CPU.

The operating system tasks that are performed by the kernel are implemented by different kernel threads. Kernel threads are easily recognized with a command like ps aux. The kernel thread names are listed between square brackets.

The Linux kernel is modular, and drivers are loaded as kernel modules.

### Analyzing What the Kernel Is Doing
The first utility to consider if you require detailed information about the
kernel activity is dmesg. This utility shows the contents of the kernel ring
buffer, an area of memory where the Linux kernel keeps its recent log
messages.
Another valuable source of information is the /proc file system. The /proc
file system is an interface to the Linux kernel, and it contains files with
detailed status information about what is happening on your server. Many
of the performance-related tools mine the /proc file system for more
information.
As an administrator, you will find that some of the files in /proc are very
readable and contain status information about the CPU, memory, mounts,
and more. Take a look, for instance, at /proc/meminfo, which gives detailed
information about each memory segment and what exactly is happening in
these memory segments.
### Working with Kernel Modules
A modular kernel consists of a relatively small core
kernel and provides driver support through modules that are loaded when
required. Modular kernels are very efficient, as they include only those
modules that really are needed.
### Understanding Hardware Initialization
The loading of drivers is an automated process that roughly goes like this:
1. During boot, the kernel probes available hardware.
2. Upon detection of a hardware component, the systemd-udevd process
takes care of loading the appropriate driver and making the hardware
device available.
3. To decide how the devices are initialized, systemd-udevd reads rules
files in /usr/lib/udev/rules.d. These are system-provided rules files that
should not be modified.
4. After processing the system-provided udev rules files, systemd-udevd
goes to the /etc/udev/rules.d directory to read any custom rules if these
are available.
5. As a result, required kernel modules are loaded automatically, and
status about the kernel modules and associated hardware is written to
the sysfs file system, which is mounted on the /sys directory. The
Linux kernel uses this pseudo file system to track hardware-related
settings.
### Managing Kernel Modules
An alternative method of loading kernel modules is through the
/etc/modules-load.d directory. In this directory, you can create files to load
modules automatically that are not loaded by the systemd-udevd method
already. For default modules that should always be loaded, this directory
has a counterpart in /usr/lib/modules-load.d.
The lsmod command lists all kernel modules that currently are used, including the
modules by which this specific module is used.
If you want to have more information about a specific kernel module, you
can use the modinfo command.

### Checking Driver Availability for Hardware
You might find that some devices are not supported properly because their modules are not currently loaded. The best way to find out whether this is the case for your hardware is by using the lspci command. If used without arguments, it shows all hardware devices that have been detected on the PCI bus. A very useful argument is -k, which lists all kernel modules that are used for the PCI devices that were detected. Example 16-7 shows sample output of the lspci -k command.

### Managing Kernel Module Parameters
To make this an automated procedure, you can create a file in the /etc/modprobe.d directory, where the module is loaded including the parameter you want to be loaded.

## Understanding Systemd
Systemd system and service manager is used to start stuff. The stuff is referred to as units. Units can be many things. One of the most important unit types is the service.
Typically, services are processes that provide specific functionality. Apart from services, other unit types exist, such as socket, mount, and target.
```
[root@server1 ~]# systemctl -t help
Available unit types:
service
socket
target
device
mount
automount
swap
timer
path
slice
scope
```
Unit filescan occur in three locations: 
- /usr/lib/systemd/system contains default unit files that have been
from RPM packages. You should never edit these files directly.
- /etc/systemd/system contains custom unit files. It may also contain
include files that have been written by an administrator or generated by
the systemctl edit command.
- /run/systemd/system contains unit files that have automatically been
generated.

If a unit file exists in more than one of these locations, units in the /run directory have highest precedence and will overwrite any settings that were defined elsewhere. Units in /etc/systemd/system have second highest
precedence, and units in /usr/lib/systemd/system come last.
### Understanding Systemd Service Units
**[Unit]** Describes the unit and defines dependencies. This section also important After statement and optionally the Before statement. These statements define dependencies between different units, and they relate to the perspective of this unit. The Before statement indicates that this unit should be started before the unit that is specified. The After statement indicates that this unit should be started after the unit that is specified.
**[Service]** Describes how to start and stop the service and request status installation. Normally, you can expect an ExecStart line, which indicates how to start the unit, or an ExecStop line, which indicates how to stop the unit. Note the Type option, which is used to specify how the process should start. The forking type is commonly used by daemon processes, but you can also use other types, such as oneshot, which will start any command from a Systemd unit. See man 5 systemd.service for more details.
**[Install]** Indicates in which target this unit has to be started.
### Understanding Systemd Mount Units
A mount unit specifies how a file system can be mounted on a specific directory.
```
[Unit]
Description=Temporary Directory (/tmp)
Documentation=man:hier(7)
Documentation=https://www.freedesktop.org/wiki/Software/systemd/
APIFileSystems
ConditionPathIsSymbolicLink=!/tmp
DefaultDependencies=no
Conflicts=umount.target
Before=local-fs.target umount.target
After=swap.target
[Mount]
What=tmpfs
Where=/tmp
Type=tmpfs
Options=mode=1777,strictatime,nosuid,nodev
```
### Understanding Systemd Socket Units
A socket may be defined as a file but also as a port on which Systemd will be listening for incoming connections. That way, a service doesnt have to run continuously but instead will start only if a connection is coming in on the socket that is specified. Every socket needs a corresponding service file.
```
[Unit]
Description=Cockpit Web Service Socket
Documentation=man:cockpit-ws(8)
Wants=cockpit-motd.service
[Socket]
ListenStream=9090
ExecStartPost=-/usr/share/cockpit/motd/update-motd ''
localhost
ExecStartPost=-/bin/ln -snf active.motd /run/cockpit/motd
ExecStopPost=-/bin/ln -snf
/usr/share/cockpit/motd/inactive.motd
/run/cockpit/motd
[Install]
WantedBy=sockets.target
```
The important option in Example is ListenStream. This option defines the TCP port that Systemd should be listening to for incoming connections. Sockets can also be created for UDP ports, in which case you would use ListenDatagram instead of ListenStream.
### Understanding Systemd Target Units
To make it possible to load them in the right order and at the right moment, you use a specific type of unit: the target unit. A simple definition of a target unit is **a group of units**.
```
[Unit]
Description=Multi-User System
Documentation=man:systemd.special(7)
Requires=basic.target
Conflicts=rescue.service rescue.target
After=basic.target rescue.service rescue.target
AllowIsolate=yes
[Install]
Alias=default.target
```
You can see that by itself the target unit does not contain much. It just defines what it requires and which services and targets it cannot coexist with. It also defines load ordering, by using the After statement in the [Unit] section. The target file does not contain any information about the units that should be included; that is defined in the [Install] section of the different unit files. When you add a unit to a target, under the hood a symbolic link is created in the target directory in /etc/systemd/system. If, for instance, you enabled the vsftpd service to be automatically started, youll find that a symbolic link /etc/systemd/system/multi-user.target/wants/vsftpd.service has been added, pointing to the unit file in /usr/lib/systemd/system/vsftpd.service and thus ensuring that the unit will automatically be started. In Systemd terminology, this symbolic link is known as a want, as it defines what the target wants to start when it is processed.
### Managing Dependencies
In general, there are two ways to manage Systemd dependencies: Unit types such as socket and path are directly related to a service unit. Accessing either of these unit types will automatically trigger the service type. Dependencies can be defined within the unit, using keywords like Requires, Requisite, After, and Before. As an administrator, you can request a list of unit dependencies. Type systemctl list-dependencies followed by a unit name to find out which dependencies it has; add the --reverse option to find out which units are
dependents of this unit.
To ensure accurate dependency management, you can use different
keywords in the [Unit] section of a unit:
- Requires: If this unit loads, units listed here will load also. If one of the
other units is deactivated, this unit will also be deactivated.
- Requisite: If the unit listed here is not already loaded, this unit will fail.
- Wants: This unit wants to load the units that are listed here, but it will
not fail if any of the listed units fail.
- Before: This unit will start before the unit specified with Before.
- After: This unit will start after the unit specified with After.

See also : Excellent blog about systemd (https://www.freedesktop.org/wiki/Software/systemd/)

## Managing and Understanding the Boot Procedure

- emergency.target: In this target only a minimal number of units are started, just
enough to fix your system if something is seriously wrong. Youll find that it is
quite minimal, as some important units are not started.
- rescue.target: This target starts all units that are required to get a fully operational
Linux system. It doesnt start nonessential services though.
- multi-user.target: This target is often used as the default target a system starts in.
It starts everything that is needed for full system functionality and is commonly
used on servers.
- graphical.target: This target also is commonly used. It starts all units that are
needed for full functionality, as well as a graphical interface.

### Understanding GRUB 2
The GRUB 2 boot loader makes sure that you can boot Linux. GRUB 2 is installed in
the boot sector of your servers hard drive and is configured to load a Linux kernel and
the initramfs:
- The initramfs contains drivers that are needed to start your server. It contains a
mini file system that is mounted during boot. In it are kernel modules that are
needed during the rest of the boot process (for example, the LVM modules and
SCSI modules for accessing disks that are not supported by default).

Normally, GRUB 2 works just fine and does not need much maintenance. In some
cases, though, you might have to change its configuration. To apply changes to the
GRUB 2 configuration, the starting point is the /etc/default/grub file, which has
options that tell GRUB what to do and how to do it.
Apart from the configuration in /etc/default/grub, there are a few configuration files in
/etc/grub.d. In these files, youll find rather complicated shell code that tells GRUB
what to load and how to load it. You typically do not have to modify these files. You
also do not need to modify anything if you want the capability to select from different
kernels while booting. GRUB 2 picks up new kernels automatically and adds them to
the boot menu automatically, so nothing has to be added manually.
Based on the configuration files mentioned previously, the main configuration file is
created. If your system is a BIOS system, the name of the file is /boot/grub2/grub.cfg.
On a UEFI system the file is written to /boot/efi/EFI/redhat on RHEL and
/boot/efi/EFI/centos on CentOS. After making modifications to the GRUB 2
configuration, you'll need to regenerate the relevant configuration file, which is why
you should know the name of the file that applies to your system architecture. Do not
edit it, as this file is automatically generated.
While working with GRUB 2, you need to know a bit about kernel boot arguments.
There are many of them, and most of them youll never use, but it is good to know
where you can find them. Type man 7 bootparam for a man page that contains an
excellent description of all boot parameters that you may use while starting the kernel.
```
# regenerate grub2 config from /etc/default/grub
grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg
```

## Essential Troubleshooting Skills
### Understanding the RHEL 8 Boot Procedure
The following steps summarize how the boot procedure happens on Linux:
1. Performing POST: The machine is powered on. From the system
firmware, which can be the modern Universal Extended Firmware
Interface (UEFI) or the classical Basic Input Output System (BIOS),
the Power-On Self-Test (POST) is executed, and the hardware that is
required to start the system is initialized.
2. Selecting the bootable device: Either from the UEFI boot firmware
or from the BIOS, a bootable device is located.
3. Loading the boot loader: From the bootable device, a boot loader is
located. On RHEL, this is usually GRUB 2.
4. Loading the kernel: The boot loader may present a boot menu to the
user or can be configured to automatically start a default operating
system. To load Linux, the kernel is loaded together with the
initramfs. The initramfs contains kernel modules for all hardware that
is required to boot, as well as the initial scripts required to proceed to
the next stage of booting. On RHEL 8, the initramfs contains a
complete operational system (which may be used for troubleshooting
purposes).
5. Starting /sbin/init: Once the kernel is loaded into memory, the first of
all processes is loaded, but still from the initramfs. This is the
/sbin/init process, which on RHEL is linked to Systemd. The udev
daemon is loaded as well to take care of further hardware
initialization. All this is still happening from the initramfs image.
6. Processing initrd.target: The Systemd process executes all units
from the initrd.target, which prepares a minimal operating environment, where the root file system on disk is mounted on the
/sysroot directory. At this point, enough is loaded to pass to the system
installation that was written to the hard drive.
7. Switching to the root file system: The system switches to the root
file system that is on disk and at this point can load the Systemd
process from disk as well.
8. Running the default target: Systemd looks for the default target to
execute and runs all of its units. In this process, a login screen is
presented, and the user can authenticate. Note that the login prompt
can be prompted before all Systemd unit files have been loaded
successfully. So, seeing a login prompt does not necessarily mean that
your server is fully operational yet.

| Boot Phase  | Configuring It          | Fixing It |
| :---------------: |:---------------:| :-----:|
| Selecting the bootable device  |   BIOS/UEFI configuration or hardware boot menu.| Replace hardware or use rescue system. |
| Loading the boot loader | grub2-install and edits to /etc/defaults/grub. | Use the GRUB boot prompt and edits to /etc/defaults/grub, followed by grub2-mkconfig. |
| Loading the kernel | Edits to the GRUB configuration and /etc/dracut.conf.|Use the GRUB boot prompt and edits to /etc/defaults/grub, followed by grub2-mkconfig.|
Starting /sbin/init | Compiled into initramfs.| Use the init = kernel boot argument, rd.break kernel boot argument.|
Processing initrd.target |Compiled into initramfs.| Use the dracut command.|
Switch to the root file system | Edits to the /etc/fstab file.| Apply edits to the /etc/fstab file. |
Running the default target | Using systemctl set-default to create the /etc/systemd/system/default.target symbolic link | Start the rescue.target as a kernel boot argument.|
**Example**
1. (Re)start your computer. When the GRUB menu shows, select the
first line in the menu and press e.
2. Scroll down to the line that starts with linux $(root)/vmlinuz. At the
end of this line, type systemd.unit=rescue.target. Also remove the
options rhgb quit from this line. Press Ctrl-X to boot with these
modifications.
3. Enter the root password when you are prompted for it.
4. Type systemctl list-units. This shows all unit files that are currently
loaded. You can see that a basic system environment has been loaded.
5. Type systemctl show-environment. This shows current shell
environment variables.
6. Type systemctl reboot to reboot your machine.
7. When the GRUB menu appears, press e again to enter the editor
mode. At the end of the line that loads the kernel, type
systemd.unit=emergency.target. Press Ctrl-X to boot with this
option.
8. When prompted for it, enter the root password to log in.
9. After successful login, type systemctl list-units. Notice that the
number of unit files loaded is reduced to a bare minimum.
