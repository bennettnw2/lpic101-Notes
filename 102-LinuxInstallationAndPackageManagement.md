# 102.1 Design hard disk layout

## Main File System Locations
We are going to take a look at the main file system locations.

### Here are some primary locations you need to know
* / root - bottom of the directory tree, the root
* /var - variable location, log files and dynamic content(such as web sites)
* /home - this is the users home directory where their personal files are stored; each user gets their own home directory /home/bennettnw2  /home/user3
* /boot - this is where the kernel is stored and supporting files for the kernel; also the boot files to get the system to boot; often on a diff partition
* /opt - third party software vendors use this directory to install their programs; this is also a good candidate to be on it's own partition; enterprise uses this directory extensively 

### Swap Space
Swap is temporary storage that acts like RAM.  When a percentage of RAM is full, the kernel will move less used data to swap.

There are two ways to configure swap.  One is to use a swap partition and the other is to use a swap file.  The swap partition is faster than using a swap file. (similar to a page file in windows) The partition is faster because data is getting written directly to disk rather than data getting written to a file and then to disk as when you are using a swap file.

The general rule of sizing of the swap space used to be 1.5x to 2.0x the size of your available RAM.  With RAM being much cheaper nowadays, the rule of thumb is to use at least .50x of your available RAM.

### Partitions and Mount Points
/dev/sda is the first hard drive (ssusi, sata, etc.) the "a" means it is the first disk the kernel has found.
/dev/sda1 - the 1 means it is the first partition
Normally you would have different partitions and those would be represented by: /dev/sda2, /dev/sda3, etc.
Each partition has a specific function within the system.

Speaking of partitions and specific functions, each partition would need to be mounted to a specific directory.  For example
* /dev/sda1 can be the / (root) partition
* /dev/sda2 can be the /boot partition
* /dev/sda3 can be the /home partition
In this way they are isolated from each other

### Commands for mounting
* `mount` can be used to mount partitions and also to show all existing mounts without any options
* `lsblk` used to show all block devices on a system and their names
-- a block device is basically a hard disk or anything that takes a large amount of data and writes it in block sizes, to a location.
* `fdisk` helpful for partitioning hard drives, for now we will use it to list out partitions on a specific disk; ie: `fdisk -l /dev/xvda`
* `swapon --summary` will show a summary of the swap usage on a system; the same info can be found in /proc/swaps


## Introduction to LVM
Taking a quick look at LVM (Logical Volume Manager)
LVM allows the create of "groups" or disks or partitions that can be assembled into a single (or multiple) file systems
* GRUB cannot read LVM metadata, therefore LVM can not be used as a /boot partition.  All other partitions are fine like wine.
* LVM allows for easy and dynamic resizing of volumes
* snapshots can be taken of LVMs

### Example Layout of an LVM Group
On the bottom layer you have your PVs (Physical Volumes).
* this is made up of all your block devices on your system
Above the PV layer you will have the VG (Volume Group) layer.
* This layer encompasses the PV layer
* A volume group could encompass one disk to
At the top, we have LV (Logical Volumes).
* This is where we carve out individual sections of the VG layer for specific mount points
* These logical volumes are akin to partitions
* Then on top of the LV is the file systems mounted to directories
   _________________________________
LV |_lv_root__|__lv_var__|_lv_home_|
   _________________________________
VG |___________vg_base_____________|
   _________________________________
PV |/dev/sda_|_/dev/sdb_|_/dev/sdc_|

### LVM Commands to set up a layout
* `pvs` physical volume scan - this checks out all individual physical volumes in an LVM setup
* `vgs` virtual groups scan - this scans for the volume groups in the system
* `lvs` logical volume scan - this scans for the logical volume


# 102.2 Install a Boot Manager

## Legacy Grub
This is the older version of booting with GRUB (Grand Unified Boot Loader) on MBR
Boot: Bios checks hardware and then it looks for a boot loader (Boot.img) first 512 bytes, this is marked as the boot flag; also known as stage one.
The boot.img is looking for something else, it is looking for the core.img and this is known as stage 1.5.  The main job of the boot.img is to get the core.img loaded.
The core.img reason for living is to locate the boot partition of the system.  Stage two is where the boot partition actually gets read. Stage two is where GRUB comes into play /boot/grub
* grub.conf / menu.lst - GRUB is looking for either of these two files under /boot/grub; grub.conf is mostly for RHEL based systems and menu.lst is mostly for Debian based systems; even though they have different names the content is the same
* device.map - this points to drive that contains the kernel and the OS that needs to be loaded; once the file is read the kernel is loaded and the system boots up.

### Example Legacy grub.conf
* `default=0` this means to use the title directive which matches the number to the titles in order, 0 means the first one, 1 means the second one etc.
* `timeout=5` this means the timeout before the system boots; defaults to 5 seconds
* splashimage is just that

#### title directive
* refers to the kernel you are going to boot
* shows also the location of on the hard drive it will look for root
* shoes more detail regarding the kernel and the boot options that go along with the kernel
* also shows the location of the ramdisk image to load
* each title entry is like an item in an array; the first is 0, the second is 1, etc

You can have multiple kernels so that you can boot into other kernels just in case a kernel update breaks something on your system

### Installing Grub
`grub-install [device]` is the command used to install GRUB to the specified device.
You have to look for the boot mount point with which to install GRUB.
`findmnt /boot` will tell you the exact location as a device; it will tell you exact disk and partition like, /dev/vda1.
You can even make it easier and use /dev/hd0 or '(hd0)' to refer to the first drive of the system.

If you run `grub` by itself, it will look for the correct partition to install itself.  You can even use a grub "find" command where it will let you know where to find stage1 = /grub/stage1.

This search is relative to root being /boot.  So when the answer you receive is /grub/stage1, it really means /boot/grub/stage1

If your system is up and running fine.  Running `grub-install` is potentially dangerous.  This command is usually run from a liveCD/USB where GRUB is getting installed to a new disk as part of a new system installation.  In short, I should not ever really have to run `grub-install`.

### Here are some GRUB shell commands:
`grub` by itself from the bash shell as root will place you into the GRUB shell
`help` will give you more information regarding the list of commands you can use in the grub shell
`find` `find /grub/stage1` will tell you (hd0,0)
`quit` will quit the shell for us

Be sure to back up your grub config before tooling around with grub

## GRUB2
What are the differences between MBR and GPT?
* MBR only supported 4 partitions, with 1 being extended to 23 partitions a: - z: for a total of 26 partitions
* MBR only supported partitions sizes up to 2 TB
* GPT (GUID Partition Table) supported 128 partitions
* GPT could support partition sizes up to 1 ZB (zetabyte) or 909.5 million TB

For GUID to work we needed a replacement for the traditional BIOS.  This is where UEFI comes in (Unified Extensible Firmware Interface).
* UEFI was still compatible with legacy BIOS.
* Requires a 64 bit operating system
* Prevents unauthorized operating systems from booting on the system

### Booting with GRUB2 on GPT with UEFI
* UEFI was a modern drop in replacement for BIOS; once UEFI gets started it will also look for a MBR in the first 512 bytes of a drive to find a boot loader (boot.img) Stage 1
  * Something to understand about GPT disks:
    * after the MBR, we are going to have a GPT header which lets the system know that this is a GPT style disk.
    * After the header, we will have a partition entry array. A large listing of all the partitions, thier ID, and their locations (similar to fstab?)
    * We will then have the core.img file which is Stage 1.5 just like MBR
* This next step is a big departure from MBR; core.img is looking for /boot/efi partition which can only be either a vfat or FAT32 partition. This partition is known as the ESP (EFI System Partition) UEFI can only read from those types
* IMPORTANT!  the /boot/efi partition is were the boot images are located and booting begins in a UEFI environment
  * this is the hand-off to grub2 /boot/grub2 where Stage 2 begins
  * within /boot/grub2 are two main files
  * grubenv
  * themes

### Commands to run to modify our grub2 setup and check current config
IMPORTANT! RHEL based = grub2-<command> and Debian based = grub-<command>
* `grub2-editenv` you can use this to edit the environment file located /boot/grub/grubenv
  * typically you do not want to edit anything under /boot/grub/ like you would with legacy grub
  * so the best use of `grub2-editenv` is to extract information from that file with `grub2-editenv list`
  * `grub2-editev list` will show us the saved_entry of the default OS and kernel to be booted when grub2 boots the system; view the default boot entry for the grub configuration file
  * So what file can we modify by hand to edit the boot environment?  Glad you asked!
* /etc/default/grub
  * this is like modifying grub.conf in legacy GRUB with very similar options like:
    * timeout
    * submenu
    * and also adding options to the command line to modify how GRUB boots
  * So the process is to edit /etc/default/grub and then run `grub2-mkconfig` to programmatically modify GRUB2
* `grub2-mkconfig` will create or update a /boot/grub2/grub.cfg file based on entries from the /etc/default/grub file
  * NOTE: On Debain systems, the "2" is omitted from the command name
  * it is just grub-mkconfig
* /etc/grub.d is another location that you can make changes to so that you can edit the boot environment
  * These are individual configuration files that grub2-mkconfig will read from in order to generate the /boot/grub2/grub.cfg file
  * NOTE: on Debian systems, you would use update-grub to update a GRUB configuration after changes to /etc/default/grub have been made.  So said another way, with RHEL you use the same command but with Debian, you would use a different command to update /boot/grub2/grub.cfg if you make changes within /etc/grub.d/
  * These options are what you would edit to make changes to a multiple OS computer


## Interacting with the Boot Loader
Learning how to work with the GRUB boot menu in both GRUB and GRUB2.  Practice booting the system using different kernel boot options and learn hot to reinstall GRUB to our disk.  Also a walk through of boot using one command at a time.

### GRUB Legacy menu
You will have to press any key to get to the menu before the kernel takes over.  Once there you will be able to:
* Select which kernel to boot into using your arrow keys
* Press the "a" key to add new kernel arguments to the kernel you wish to boot
  * you can remove the quiet option
  * you can remove the graphical 
  * you can add 2 to boot into runlevel 2
* Press the "c" key to get to the grub command line
  * you can read out some of the files in the boot directory
  * you can set new options for your configuration file
  * you can even reinstall GRUB in the event GRUB breaks
    * you could use the `install` command but that is for GRUB Pros
    * the best way is to use the `setup` command `setup (hd0)`

### GRUB2 Menu
This will take you directly to the initall GRUB2 menu
  * There is no option for "a"
  * The "e" key to edit; it looks way different as it gives us all the options rather than just the command line
  * These are systemd options rather than sysvinit
    * so instead of just appending 2 for runlevel 2; we would need to set up the unit as systemd.unit and the target as rescue.target
    * it would look like `systemd.unit=rescue.target`
    * use <ctrl+x> to boot with the options we just changed

### Booting GRUB Manually
  * ls
  * ls (hd0,1)/ will show us the boot directory contents
  * set root=(hd0,1)
  * linux /boot/vmlinz-4.24.9-43-generic root=/dev/vda1
  * intrd /boot/initrd.img-4.24.9-43-generic
  * ram disks and kernel names are very similar
  * use the boot command to boot with all the options we just set


# 102.3 Manage Shared Libraries

## What is a shared library?

A shared library is a group of files that contains functionality that other applications can use.  This is good for developers who only have to focus on their application and can rely on the functionality of shared libraries to interact with the operating system.

* Shared libraries end in a .so file extension which means shared object
* Library files can be found in the following locations on a Linux system:
  * /lib
  * /usr/lib(for 32bit systems) /usr/lib64(for 64bit systems)
    * some 64bit programs can still use the 32bit libraries
  * /usr/local/lib
  * /usr/share

* There are two types of library files: Dynamic(ends in .so) and Static(ends in .a)
  * Statically linked library files are compiled into an application when the application is installed
  * Dynamically linked libraries are called when needed by an application

## Commands and options when dealing with shared libraries
  * `ldd` prints out shared object (library) dependencies
    * `ldd /bin/cp` shows what library files are linked to the `cp` application
  * `ldconfig` configures dynamic linker run-time binding, create a cache based on tlibrary directories and can show you what is currently cached
    * will create a cached listing of the most used libraries based off of a directory I've provided
    * this command needs to be run as root/sudo
  * `/etc/ld.so.conf` configuration file that points to directories and other configuration files that hodl reference to library directory locations
    * so this is the configuration file that links applications and libraries together?
  * `LD_LIBRARY_PATH` legacy environment variable that points to a path where library files can be read from
    * you can use this instead of a configuration file to link libraries and applications
    * however, it is best to use a config file as this is old school and outdated but comes in handy if you are just testing something out
    * example of usage `$ export LD_LIBRARY_PATH=/opt/java/jre/lib
    * similar to the PATH variable but this is for libraries

# 102.4 Use Debian Package Management

## APT
Advance Package Tool used as the default for Debian based Linux distros
* APT is used to install applications and their dependencies
* it is configured to use a network for retrieving data and dependencies
* removes applications and also updates and upgrades packages

How does APT work?
* reads the file at /etc/apt/sources.list
  * which contains url listing of software repositories
* once it reads the file and make the calculations for dependencies, it will download all the needed files
* then directs commands to dpkg to get it all done

Commands and options used for apt
* `/etc/apt/sources.list`
  * configuration file that lists out repository locations for packages
* `apt-get upgrade` or `apt upgrade`
* `apt-get update` or `apt update`
  * updates the local apt cache so that the upgrade command does not use excessive bandwidth?
* `apt-get install` or `apt install`
* `apt-get remove` will only remove the application but not the dependencies
* `apt-get purge` will remove the application and the dependencies and config files
* `apt-get dist-upgrade` will install new packages to get the system up-to-date while the `apt upgrade` will not install new packages
* `apt-get download` will download a package file but not install it
* `apt-cache search [package]` searches through your local apt cache for a package that can be installed
* `apt-cache show` lists out basic information about a package
* `apt-cache showpkg` displays more technical information about a package


## Using Debian Package (dpkg)

### The Debian Package Utility
* The .deb package contains:
  * the application or utility you want to install
  * default configuration files
  * how and where to install the files that come with the package
  * listing of dependencies the package requires
* Dependencies should already be installed or be installed with the package
  * `apt` will automatically handle dependencies for you while `dpkg` will not

### Some common commands and flags for use with dpkg
First you will want to download a .deb file using `apt-get download [pkg name]`
* `dpkg --info` taking a look at basic information about a package (apt-cache show is similar)
* `dpkg --status` on a package that has already been installed
* `dpkg -l htop` list packages that match the string on the search on a system `dpkg -l htop`
* `dpkg -i htop` need to be sudo to -i install a package; run --status to verify the package has been installed
* `dpkg -L htop` will list the files that were installed for htop
* `dpkg -c htop` will show the contents of the htop .deb package
* `dpkg -r htop` will remove a package
* `dpkg -P htop` will purge a package
* `dpkg -S htop` will search to see if there is anything in the dpkg database that refers to htop
* `dpkg-reconfigure` IMPORTANT TO KNOW FOR TEST - rerun configuration utility
  * sudo dpkg-reconfigure console-setup

# 102.5 Use RPM and YUM package management

## YUM
### The Yellowdog Updater Modified
* Originally used for the Yellowdog Linux Distribution
* Handles RPM package dependencies (keeps you out of dependency hell like apt)
* Installs, upgrades and removes packages

* The yum setup
  * global yum configuration options are set in /etc/yum.conf
  * reads repository information from /etc/yum.repos.d
  * caches latest repository information in /var/cache/yum

### Other RPMs (RedHat Package Managers)
* Zypper is used on SUSE Linux Distros
* DNF (Dandified yum) is used on Fedora distros
  * future replacement for yum in RHEL
  * uses same command syntax as yum

### Common yum commands:
* `yum update` searches online repositories for updated package compared to what is currently installed on the system; also upgrades packages
  * it will also calculation and process dependencies
* `yum search [string]` searches the yum repositories for the string specified in the search
  * ie: `yum search https`
  * it will match any package with a reference to the search string
  * `yum search` uses /etc/yum.repos.d to do it's search
  * It is important to understand the basic contents of a repo file
    * baseurl is the web address that indicates where packages are loaded from
* `yum info [string]` lists information about a specified package
  * `yum info httpd`
    * this gives us information abut the httpd package like name, architecture, version, repo it comes from and a short description
* `yum list installed` displays list of all installed packages on the system
  * yum list installed | less`
* `yum clean all` cleans up all your cache information and ist local database file
  * use this if your database is old or corrupted
  * the next time I run the yum command it will download fresh meta data from the repositories yum.repos.d directory
* `yum install [package] -y` will assume yes to the install of a package as yum will always want to confirm every package you want to install
* `yum remove [package] -y` will assume yes and remove a package that we have installed
* `yum autoremove [package] will remove a package and also its dependencies
* `yum whatprovides */httpd` will show you what package contains a particular file you are looking for 
* `yum reinstall [package]` this will download the package and reinstall for you
  * will need yum-utils installed on your system
  * yumdownloader mc (midnight commander)

## The Red Hat Package Manager (rpm)
### RPM
* An .rpm package contains
  * the application or utility
  * default config files
  * how and were to install files
  * list of dependencies that the package requires

* The rpm database:
  * located in /var/lib/rpm
  * use the `rpm --rebuilddb` to repair a corrupted rpm database

* Dependencies need to already be on the system or installed with the package
  * yum handles dependencies automatically but rpm does not

### Common rpm Commands
* `rpm -qpi [package]` Queries Package Information
  * the output will be the same as the `yum --info` command
* `rpm -qpl [package]` lists all the files in a package (query package, list of files)
* `rpm -qa` list all installed packages on the system
* `rpm -i [package]` installs a specified package; often combined with other options to proved more verbose output with progress hashmarks ie: -ivh
* `rpm -U [path-to-package-file]` upgrades an installed package with a newer version
* `rpm -e [package]` this will erase or uninstall the selected package
* `rpm -Va` will verify all installed packages
* `rpm2cpio` converts an .rpm file into a cpio archive file and then you can extract the contents of the rpm package
  * often combined with the cpio command
  * ie: `rpm2cpio [some.rpm] | cpio -idmv`
    * -i means extract
    * -d means extract with the same directory structure
    * -m modifed time the same
    * -v verbose output

NOTE: you can use `which` to see if a package is installed on your system or not


# 102.6 Linux as a Virtualisation Guest

## Virtualization and Containers
### What is a Virtual Machine?
* It is the emulation of a specific computer system type
* They operate based on the architecture and functions of a real computer and ist implementation
  * can involve specialized hardware, software or both
* Virt software will let you set up one operating system within another
  * they will both share the same physical hardware
  * the VM is isolated from the hardware but communicates with the hardware via a hypervisor
* Examples of hypervisors are:
  * KVM - kernel virtual machine; built into the linux kernel
  * QEMU - which KVM is built from
  * VMWare - proprietary
  * Xen - 
  * VirtualBox - free system

### Virtual Machine Basics:
* Two types of virtualization
  * full virtualization - does not know it is a virtual machine
  * para-virtualization - knows it is using guest drivers
    * para-virtualization typically performs better with guest drivers
* Virtual machine can be cloned or templated to rapidly deeply new systems
  * you will likely need to change the Linux system's Dbus ID
  * `dbus-uuidgen` will ensure that each running kernel interacts with a system that has a unique ID

### Virtual Machines in the Cloud
* Virtual servers can be provisioned from cloud providers
* `cloud-init` is typically used to insure that user data is new and unique
  * creates new SSH keys
  * sets systems default location
  * sets the system's host name
  * sets up mount points

### What is a Container?
* A container is an entirely isolated set of packages, libraries, and/or applications that are completely independent from their surroundings
* For getting an applications up and running quickly
* Machine Container will share a kernel and file system with the host computer
* Application Container: shares everything but the application files and library files that the application needs
  * able to rapidly deploy web servers when traffic is high

* Examples of Container Tech:
  * Docker
  * nspawn (from systemd)
  * LXD
  * OpenShift

### What is the difference between the two and why is it important
* Virtualization
  * was invented to allow the sharing yet segregation of server instances from each other
  * protects one operating system from another
  * prevention of spare CPU cycles, memory, or disk space go to waste
    * get the best bang for your buck to make use of all the resource
  * emulating virtual hardware through a hyper visor
    * heavy in terms of system requirements

* Containers
  * use shared operating systems
  * more efficient in system resource terms
  * more granular management of system resources
