# 104.1 - Create Partitions and Filesystems
## Legacy MBR Partitions
We will create older / outdated MBR partition tables.  We will create these MBR partition tables using fdisk and parted.

### MBR Partitioning Tools
#### `lsblk`
* command used to list out block devices like hard drives
* When beginning to partition drives, it is a good idea to run lsblk to see the available disks on the system
* What are block devices?
  * Any hardware that has data written to it in block sized chunks.
  * Hard drives, solid state disk drives, etc
* the output will give us the name of all our connected disks
  * sda
    * s means scsi/sata drive
    * d means it is a disk drive 
    * a means it is the first disk of its bus type (scusi/sata) that the kernel has found
  * vda1
    * v means this is a virtual drive
    * 1 means this is the first partition of the vda drive
  * vdb
    * b means it is the second disk of its bus type (virtual) that the kernel has found

#### `fdisk`
* legacy command used to create partitions of the MBR(DOS) type
* the usage is as follows: `fdisk [device such as /dev/sda]`
* this will drop us into an fdisk shell / interactive mode
* Use m to show the help menu
  * But it should be pretty much the same set up that you use time and time again.
* Press the p key to see what the disk currently looks like
  * it is a good idea to look at this before you get started with configuring of the drive
* Press the n key to create a partition
  * you will be prompted to chose a partition type: p for __primary__ or e for __extended__
  * p for primary will then prompt for a partiton number
    * NOTE: an MBR disk can only have 4 partitions
    * one of those partitions can be an extended partition that can hold 23? more partitions?
    * Typically just select the default
  * Next, fdisk will ask you about the geometry of the disk (sectors and such)
    * it is good to just select the defaults
  * Next you will select the size of the disk in K, M, G; kilo, mega, giga
    * again, you can just select the default if you like
  * press the p key again to to see the changes you made to the disk
* Press the w key to write new filesystem tables/partitions to the drive
  * this will also drop you out of the `fdisk` shell

* We will still need to add a filesystem and mount it to a mount point to be able to use the newly created drive system

* You can use `fdisk -l /dev/sda` to list the partition(s) created on a disk

#### `parted`
* Modern command used to create partitions of MBR or GPT types
* You can also use `gparted`; it uses a menu driven prompt system just like fdisk
* Press p key to see what the disk looks like.
  * "p"rint partition tables on a device
* If you need to create a MS-Dos partition, you will also need to create a label
  * eg: `mklabel msdos`
* You use `mkpart` to start the partition creation process using parted.
  * then you get prompted for the partion type primary/extended
  * then you get prompted for the file system type; default ext2
  * then it will ask you about partitioning blocks
  * press "p" to git er done

#### Partiton ID's:
* 82 -  Linux swap partitions
* 83 -  Standard Linux Filesystems
* 8e -  Linux LVM Volumes

## GPT Partitions
Learning how to create new partitions using GPT.  I need to know the differenced between MBR and GPT and how to crate partitons in either format.
* How do you know if you are in a GPT disk or an MBR disk?
  * you can use `lsblk` and look for efi which means that you are on a GPT disk
    * efi acts as BIOS on a GPT system
  * you can also use fdisk and if you get a warning about GPT, chances are, you are running GPT
### `gdisk`
* You can use parted or gdisk to create a GPT based disk
  * `gdisk` is pretty much the same thing as `fdisk` except it has support for GPT systems
  * `?` will take you to the menu
  * you use the `n` option to create a new partition, just like `fdisk`
    * GPT disks can contian upto 128 partitions which is a far cry from 4 in MBR (23 logical)
  * select the default starting sector
  * next specify a size `+500M`
  * then it will ask about partitions IDs
  * `p` key will show you your partitions before you commit them
  * use the w key to write; it will ask to verify
  * confirm and then enter
#### `parted`
  * does most the same thing as gdisk and fdisk
  * `parted /dev/sdb`
  * `mklabel gpt` from within the parted program
  * `mkpart` will start the creation of the partition
  * can create a name for the partion
  * next you select the partition type
  * start and end of the partition
  * `p` to review
  * `q` to quit 

## Swap Partitions
Creating swap partitions
* What is swap?
  * it is used when ram is nearly full, the kernel will use the swap space to store old ram
  * if a system ran out of memory, it would freeze and no work would get done
  * you can have a swap file and a swap disk
    * a swap file has more steps involved, the will write to a file that is already on a file system
    * a swap partition is much more quicker that a swap file as data only has to be written once
    * but using swap is slower either way that just using regular RAM
* In creating a swap partition, you start off with `lsblk` as always to see what your current set up is like
* depending on if your disk is MBR(BIOS) or GPT(EFI) you can use fdisk, gdisk or parted
* we are using GPT so well use gdisk
  * `gdisk /dev/sda`
  * press p to view partitions
  * n to create new
  * default partition number
  * default partition start sector
  * next enter either 82 / 8200 / 0x82 to select a linux swap partition
  * p to check your partitions again
  * w to write the changes to the disk
  * `lsblk` to confirm the changes have been made
  * but we are not finished, we just created the partition; there are a few more steps
* Next we need to put a swap filesystem onto the swap partition
  * `mkswp -L SWAP /dev/sda2`
  * `free` will show us the swap is available
  * but we need to turn on the swap disk
    * you can either run `swapon -a` which will look for swap that is turned off in /etc/fstab
    * or you can run `swapon -L SWAP` or `swapon /dev/sda2` or using the UUID
  * if you run `free` after running `swapon` you will see that it has increased by the size we just created
  * However, this will not be persistent.  Do make it persistent, we need to add it to /etc/fstab
    * \<physical device location> \<mount-point> \<filesystem type> \<options> \<dump>\<fsck>
    * \<LABEL=SWAP>               \<swap>        \<swap>            \<defaults>  \<0> \<0>
  * you can use the `swapoff -L SWAP` command to turn off swap space 


## Creating Linux File Systems
We will learn about some of the various file systems available to Linux and also creating said file systems.
### Linux File Systems
  * Non-Journaling
    * ext2 - second extended filesystem
      * legacy system released in 1993
  * Journaling
    * a journaling file systems keeps track of changes not yet written to the file system
    * think of it as note taking / journaling
    * depending on the file system, the journal can exist in ram with a DB index, or on a hard disk
    * Example Journaling file systems are:
      * ext3 released in 2001, introduced journaling to ext2
      * ext4 released in 2006, added extra features
      * XFS created in 1993 for IRIX OS; great for MongoDB and other dbs
  * Copy on Write (CoW)
    * B Tree file system
    * uses subvolumes
    * have main and you can use subvolumes
    * can take snap shots
      * take a snap shot of a 500GB in a few seconds
  * FAT File Systems
    * File Allocation Table file systems have been around since the 1970s
    * Linux can use vFAT (virtual file allocation table) which allows for long filenames
    * EFI boot partitons on GPT disks, need to use FAT or on Linux, vFAT
  * exFAT
    * allows for files larger than 2GB in size
    * primarily used ofr external disk drives, USB thumbdrives etc

### Creating file systems
  * as always, start with `lsblk` to get a lay of the disk landscape
  * find the partition location you would like to create the filesytem on
    * eg: sdb2, vda1, etc
  * ##### `mkfs`
    * "make file system" command
    * There are two syntaxes that are correct for using `mkfs`
      1. `mkfs -t ext4 /dev/sdb2` - using the `-t` type flag and the current device location
      2. `mkfs.ext4 -L SRV /dev/sdb2` - using the dot notation and assinging a label
  * You can use `lsblk -f` to show the file system type along with the UUID
    * when a file system is created on a partition, that partition will get a UUID
  * ##### `blkid`
    * "block id" command will display information about a block device
    * `blkid` will tell you: device location, label, UUID, file system type

# 104.2 - Maintain the Integrity of Filesystems
## Disk Space Usage
Every linux admin needs to keep track of disk space usage; using the ls command and various options, we can get an idea of the size, therefore, usage of the individual files.  However, we want to look at the system as a whole.

#### `df`
  * disk free will show us the individual file systems on a system
  * it will show us the total space available on each system
  * how much space is being used by each filesystem as MB or GB and also in a percentage
  * how much disk space is left over or how much is free
  * and it also shows the mount point
  * can also use the `-h` option to see these details in human readable form
  * think of this command as the starting point for researching diskspace usage
    * it gives a broad overview of all the disk space usage on a system
    * disk freeeeeeeeeeeeeeeeeeeeeeeeee! (big, gregarious)
  * You may see a `tmpfs` disk in the output
    * these are just pseudo/temporary filesystems that get deleted once the computer restarts
    * they will only exist when your computer is up and running
#####  `df -h /`  or `df -h /dev/sda`
  * You can get just specific filesystem information too
  * These two commands will give you the same output
    * you can either specify the mountpoint or the filesystem / device location
##### `df --total -h`
  * will give us a total at the bottom of our output
  * the grand total of the entire system size
##### `df -h --max-depth=2`
  * can just see max depth down into the directory
  * if you start at root, you can drill down by using a max depth of 1 to find where large files are

#### `du`
  * "disk usage" is another command we can use to take a look at disk space usage
  * `du` will list out the disk space in use, within the current folder I am in
  * We are also given a total amount of space used in bytes at the bottom
    * can use `-h` option to show output in human readable format
##### `du -s`
  * `-s` means "summary"
  * using this option basically gives us how much space the folder we are in is using
  * it will be the same number as the total at the bottom of running the `du` command by itself 
  * this command does not list all the individual files, just the total at the bottom
  * you can also specify a directory you would like to see the "disk space makeup of"
    * disk space makeup means how much space, does each file in this directory utilize?
    * the command would look like this: `du -sh /tmp`
    * you can only read into files and folders that we have permissions to (r and x?)
    * the output you get is only for what your user has permissions to
    * simply use sudo to get access to everything: `sudo du -sh /tmp`
##### `du -h --max-depth=2 etc/`
  * can just see max depth down into the directory
  * if the depth is two, you will see the deeper directories first and then the one above that
    * then it goes inside the next diretory lists all the files and folders then then back to the one
    * then finally at the end you will get the top level directory you started at
  __INSERT GRAPHIC__
  * in the etc/ folder, you will see all the folders and their contents' size; along with the total at the bottom
  * you will only see folders and their sizes, actual files are not listed when using `du`

#### How would you compare and constrast `df` and `du`?
  * So while `df` shows disk usage from the outside looking in, `du` will show you disk usage from the inside looking out
  * Maybe it is more like `df` is a birds eye view of a department store layout, and du is like an individual listing of each department
    *(I need to come up with a better analogy than a department store)
  * `df` is macro view and `du` is micro view
  * `df` is how much space is free and `du` is how much is being used by a particular folder and sub folders if you wanted

Now we know how to see how much total space is free and we also know how to tell how much disk space each individual file uses.  So how does the file system keep track of all these files and corresponding data?  I am glad you asked!  The answer is inodes!
#### `inode`
  * means "index node"
  * you can think of is as a reference number the kernel uses to reference or point to a file
  * It also contains information about the file or folder it points to 
  * this info includes permissions, ownership and file type, etc
  * most file systems contain a maximum number of inodes that it can contain
    * The number of inodes is dependent on the size of a disk and the type of file system on it
  * inodes help to keep track of data of files
  __IMPORTNT DISTINCTION:__ A service can crash if you run out of inodes; eg: logging goes haywire and writes millions of files that take up all the Inodes; you will have plenty of memory and disk space but no more inodes.
* How do I get a total of the inodes?  See the commands below:
##### `ls -i`
  * shows the individual inode number of a file
##### `df -i`
  * this shows the overview of inodes on a system
  * will show you the inodes available and how many are in use
##### `du --inode`
  *shows how many inodes are used in a directory
  * will show the individual folders and how many inodes are in use in that folder


## Maintaining a Filesystem
Learning how to maintain a filesystem with various utilities
#### `fsck`
  * means "file system check utility"
  * checking a file system for errors
  * will get you a report of a file system
  * Start by looking at block devices `lsblk -f`
  __INSERT GRAPHIC__
    * then we find a device we would like to look at
      * a devise labled SRV in this example
    * so it woud look like: `fsck -r LABEL=SRV`
      * `-r` mean report
  __INSERT GRAPHIC__
    * however, this will not work as the device labled SRV is currently mounted
    * you can only run `fsck` on drives that are not mounted
    * you run the `umount` command to unmount drives attached to a system
      * you can specify the device name, label name or mount point
      * for example: `umount /srv` (we've selected the mount point)
    * then you can rerun `fsck -r LABEL=SRV` and it will be all good as it is now, not mounted
  * in /etc/fstab there is a spot for order of checking of disks   
__Be sure to review the man pages__
  
#### `e2fsck`
  * this is the same utility as above but it specificly for ext file systems
  * ext2, ext3 and ext4
So what does this do differently?
  * ##### `e2fsck -f /dev/sda1`
    * this will "force" a check of a file system whether or not the initial check passes
  * ##### `e2fsck -p /dev/sda1`
    * this will "preen" or repair a disk automatically without prompting you
__Be sure to review the man pages__
  
#### `mke2fs`
  * this one is related to `mkfs` or "make filesystem"
  * you can think of this command as formatting a disk to an ext2,3,4 type file system
  * when we run `mkfs` to make a filesystem of ext type, it calls on `mke2fs` 
  * `mke2fs` will build the filesystem based on the options given to the `mkfs` command 
  * so this basically makes/formats file systems for you
  * Let's have a look at a `mke2fs` configuration file for a hot minute
    * this is located at /etc/mke2fs.conf
    __INSERT GRAPHIC__
    * you will see some base features that get applied to a file system
    * you will also see some default mount options
      * NOTE: these default mount options are the ones executed, as indicated, in the /etc/fstab file when an item is set to mount with default settings
      __INSERT GRAPHIC__
      * this is only related to ext type file systems; other file systems use different defaults
    * you will also see the file system types listed that it can create
How do I create a new ext4 file system?
  * you want to start with a new partition or a partition that you want to change the file system type of
  * it will look like: `mke2fs -t ext4 -L EXTRA /dev/sdb1`
    * `-t` means file system type and `-L` means the label you want to give the new file system
  * mke2fs pulls from the configuration file, /etc/mke2fs.conf, the parameters for the file system type selected

#### `tune2fs`
  * this is a utility to adust parameters on any ext type file system, after the file system has been created
  * to list the current parameters of an ext type file system, you use the `-l` option with the `tune2fs` command
    * `tune2fs -l /dev/sdb1 | less` ( pipe it to less as it contains a lot of data )
    * most of the data is the same data you see after creating a file system with `mke2fs`
    * however, there are some extras that you would not see and only exist here
      * for expample: "Errors Behavior:       Continue"
      * this instructs the kernel to keep booting the system and rather than stop the boot, just take note of the error      * "Force Read-Only Mode"
      * "Cause kernel panic" to get information from the logs?
  * well this is all well and good but how to I modify these settings
    * everything in the output of `tune2fs -l` is modifyable
    * simply use the `tune2fs` utility like so to change the setting "Check insterval:":
      * `tune2fs -i 3w /dev/sdb1`
        * `-i` means interval and `3w` means three weeks
      * if you check the parameters again, you wil notice the "Next check after:" will have a date three weeks from today
   * there is usually a "lost+found" directory in linux file systems
     * this is the result of a file system check being done (e2fsck or fsck) and there was a damaged file
     * the file or damaged file will be moved here for further exploration or restoration

### xfs file system utilites (mongodb's favorite file system)
#### `xfs_repair`
  * this is similar to the e2fsck in that it checks the file system for errors
  * `xfs_repair` is the xfs file system's rendition of fsck
  * it will look like this: `xfs_repair /dev/vdb1`
  * it goes through 7 phases
#### `xfs_fsr`
  * less contigues data bring them together again
  * kinda like a defrag
  * needs to be mounted to work
#### `xfs_db`
  * allows you to debug a file system
  * interative program where you enter options and it executes those for you
    * the command `frag` will defrag the file system     __IMPORTANT FOR EXAM__
    * `freesp` will show you the free space on a device  __IMPORTANT FOR EXAM__

# 104.3 Control Mounting and Unmounting of Filesystems
## Understanding Mount Points
Let's have a quick look at how mounting works
  * you can think of mounting a disk to a mount point like a ship docking at a port
  * cargo(data) is loaded and unloaded to that ship(disk) and when the ship leaves, it takes the cargo with it
  * if there is any underlying file system or files, the mounted disk simply "overtakes" that space
    * the underlying file system is no longer available until the disk is unmounted
    * the underlying file system is still there, it just needs the mounted file system to be unmounted for it to reappear
  * `mount /dev/sdb1 /opt` is what a typical mount command looks like
  * if you want the device to consistently be mounted on boot, automatically, you would add an entry to /etc/fstab


## Mount and Unmount Filesystems
Let's have a look at the `mount` command and the dynamics that go along with it
#### `mount`
  * simply running the `mount` command by itself will list every single mount point on the system
  * some of these are created by different system services, like systemd 
  * others are mount points described in /etc/fstab
  * you can narrow your output by specifying a system type
    * `mount -t ext4`
    * use the `-t` switch for type and then enter the type
  * So where does `mount` get its information from?
    * from `/etc/mtab`
    * pro-tip: if you cat `/etc/mtab` you will see the same output as running `mount`
    * but lo and behold, `/etc/mtab` is a symbolic link to `/proc/mounts`
    * which has the same output of `cat /etc/mtab` and `mount`
  * Cool!  So how do I use `mount`?
    * with the `mount` command you will specify the device and then the mount point
    * that is the bare bones of the command; you have many options available to you
      * check out line 566 of `man mount`
      * to use these options you will need to use a `-o` and then list your options separated by commas 
      * eg: `mount /dev/sdc  -t ext4 __-o rw,noexec,async__ /opt`
        * these options are independent of the file system; meaning the file system has no control of these
      * `-a` or `auto` - any file system that in located in fstab will get mounted
        * if it is already mounted, `auto` will skip and look at the next entry
      * `-t` or `type` - will try to mount the file system as the type described
      * defaults
        * rw - mount read/write
        * suid - mount with set user id
        * dev - treat like a device to write data to
        * exec - make binaries and applications able to executed
        * nouser - only a root user can mount this file system
        * async - contents destined for the file system will be written asyncronsly
      * `remount` will attempt to mount a file system that is already mounted
      * `no exec` will not allow any apps or binaries to be run from that file system
      * `-ro` will mount as read-only and not allow any writes to the file system
#### `umount`
This is how you would un-mount an already mounted file system
* `umount /dev/vdb1`
* remember how mounting and unmounting works

#### `/etc/fstab`
* this is where you set your permanent mount configs
* if you create an entry, you can mount it with `mount -a`
  * remember what the -a option does?  Yes. Good.

* /media is a good mount point for USB/CD/DVD media
* `mount /dev/sr0 /media`
* you can mount .iso files as well
  * `mount /root/install.iso -t iso9960 -o ro,loop /media`
    * need the loop option here so that the system treats it like a cd rom by contninuosly reading the file

# 104.5 - Manage File Permissions and Ownership
## Basic File and Folder Permissions
Permissions are an important aspect of local security on a Linux system.  If permissions are improperly maintained, then others who should not have access to files may end up with the ability to modify them.  You as an administrator could have a bad time if that happens.
* can start with a long listing, `ls -l`, so we can see the permissions of files and folders
* you will see output like: `drwxrwxr-x 2 nygel nygel 62 Feb 1 23:22 Documents`
* the `d` means it is a directory and a `-` in the same spot means a file
  * a `b` in this spot means a block device like a hard drive or disk drive
  * a `c` in this spot means a charater device like, terminals, mice, keyboards, monitors, etc
* the next part, `rwxrwxr-x` indicates the mode; you can think of it like a method of access
  * the mode part dictates who has what types of permissions on the file/directory
  * there are four __symbolic__ permissions:
    * r = read _(able to read the name of the file or directory)_
    * w = write _(able to modify, write to a file or change the directory name)_
    * x = execute _(able to execute the action of opening the file or directory)_
      * IMPORTANT DISTINCTION: you can list a directory's contents with the read mode avaialble, but you cannot `cd` into it unless you have the execute mode available to you
    * \- = no permission in the spot where it is
  * there are also four __octal__ permissions:
    * 4 = read
    * 2 = write
    * 1 = execute
    * 0 = no permission
      * by adding these numbers, you will receive one number that will give you the rwx modes for a user, group or other/world
      * eg: 6 = rw-; 5 = r-x; 7 = rwx
      * therefore: rwxrwxr-x = 775 and rw-rw-r-- = 664
  * The who part is separated into three groups
    * user - first three modes `rwx------`
    * group - next three modes `---rwx---`
    * other - last three modes `------rwx`
* Permissions do have an order of presidence
  * first user, then group, then others
  * wherever you fall between user and group will give you the one that occurs first
  * if you are in neither user or group, then you are in others

## Modify Basic Access Modes/Permissions

#### `chmod`
* change the mode of a file or directory, which will effect the item's permissions
* can use either symbolic or octal method with `chmod` to change a mode
  * __symbolic method__ will consist of the who to change along with adding or removing the permission
    * eg: `chmod o-r file.txt` this will remove the read permission from the others
    * `chmod w+g file.txt` will add the write permission to group
    * the symbolic method allows you to change just one or more at a time
    * you can even use `chmod` to recusively modify permissions of files in a directory
      *eg: `chmod -R g-r Documents/*`
  * __octal method__ will look like `chmod 664 file.txt`
    * note that you will change all the permissions at once for user, group and others

#### `chown`
* use `chown` to change the ownership of a file or directory
* you will specify a username and then a group
* it will look like `chown nygel:research file.txt`
  * NOTE: you can still use the legacy notation of a period instead of a colon `nygel.research`
  * if you just wanted to change the group you can do it thusly; `chown :research file.txt`
  * if you just wanted to change the user you can do it thusly; `sudo chown nygel file.txt`
    * NOTE: you must be a root user in order to change the user owner

#### `chgrp`
* while `chown` is the long way to change the user, you can use `chgrp` for the short way to change group ownership
* eg: `chgrp research file.txt`

#### Changing the user owner with chown
* Only root can change the user owner of a file due to security reasons
  * `sudo chown eva.kenny file.txt`

## Modifying Advanced Permissions
#### `suid`
* this is known as the "set user id" bit
* you will know this is present when you see the letter "s" in the place of the "x" permission in the user owner's column
* so what does this do?  It will allow a regular user to access or execute a file as if they were a root user.
* said another way, when you execute a file with the set user id in place, you are effectively taking on the userID of the root user so you can access or execute whatever
* you will not use this very often as its main application is for the /etc/passwd file
  * /etc/passwd is where passwords are stored (encrypted of course)
  * you have the ablility to change your own password
  * if you take a look at the `passwd` command you will see the suid bit
    * ls -l /usr/bin/passwd => -rwsr-xr-x.
    * notice the "s" in the "x" spot of the user owner
* NOTE: most file systems are mounted with the `nosuid` option to prevent accidental or purposely abuse of this powerful tool
  * therefore you could not write a script, set the suid on it and have it be run by another user
  * it is just way to dicey, security wise, to allow something like that
* With that being said here is how you do set and unset the suid bit:
  * There are two ways to change the mode of a file to have the suid bit, symbolically and octally
  * __octally__ this is the most straightforward method
    * basically you put a 4 in front of the octal notation
    * eg: `chmod 4760 script.sh`
    * NOTE: octal notation without a bit set is really 4 digits with the first being 0
    * eg: `chmod 0700 script.sh`
    * Why is the number four used to set the suid bit?  Because it is in the user owner column!
    * to remove the suid bit, you just put a 0 in front of the octal notation to remove it
    * REMEMBER: this functionallity has been removed in modern systems and you can easily use more secure methods to run programs as root such as `sudo` or `su` etc.
  * __symbolically__ this is fairly straightforward as well
    * instead of using r,w, or x you would use "s"
    * eg: `chmod u+s script.sh` and to remove: `chmod u-s script.sh`
    * u means user and s means set user id

#### `sgid` 
* This is known as the "set group id" bit
* You will know this is present when you see the letter "s" in place of the "x" permission in the group owner's column
* So what does this do?  This will assign group ownership to files so that when an application is run, it is effectively run as the group ID instead of any other ID 
* Why is it useful? This is useful for managing shared group directories and modes of access within these shared directories.
* How's that? Let's look at an example:
  * well start with a long listing of the `write` command
  * `ls -l /usr/bin/write` => -rwxr-sr-x. root tty ...
  * note the "s" in the "x" spot of the group owner column
  * when `write` is run, it is always run with the "tty" group ID
* Well that is all well and good but how is this an effective use of this feature?
* Glad you asked!  It is most useful when the sgid bit is applied to a shared directory
  * Users would be able to create and share content within the dir so that everyone has access to the files within it
  * The group will retain ownership of the content created within the directory
  * For example, lets create a directory called "teamDocs"
  * Its default permissions will be drwxrwx---. root team
    * NOTE: the user for this folder is root and the group is team
  * It is important to know at this point that a single user can belong to multiple groups
    * to find out what groups that you as a user belong to, just type the command `groups`
    * you can also use the `id` command for more through information regarding you and your groups
  * We have two users "kristi" and "nygel" and each are part of the team group
  * When they create files/directories within this shared folder without the sgid bit set, the user and the group will be thier respective, default user and group which would be kristi:kristi and nygel:nygel.
  * So if nygel wanted to add something to kristi's file, he would be unable to as the file she created the user and group are her and there are no others permissions so nygel would be meet with a permission denied error
  * This is where the magic of the sgid bit happens 
  * once you set the sgid with `chmod -R 2770 /srv/team`, all the subsequent files and directories created by any user, as long as they are a part of said group, within that directory, will inherit the group ownership of the group ownership of the parent directory
  * Why is the sgid bit "2"?  Becasue is correlates to the group owner column in octal notation

#### sticky bit
* What is the "sticky bit"? The sticky bit, when set, allows only the creator of a file to remove the file.
* You will know this is present when you see the letter "t" in place of the "x" permission in the other owners column
* This comes in handy in a shared directory as well.  Without the sticky bit set, any one could come along and delete your files in the shared directory.
* Once it is set however, only the user that created the file can delete it.
* You set the sticky bit by prepending a 1 to the octal value
* eg: `chmod 1777 /srv/sticky`
* Yes, you've guessed it.  It is a 1 because it correleates to the others owner column.



## Default File and Folder Permissions
We are going to look at where the default permissions of directories and files come from and how to change their values.  We will also look at the difference between a regualr user's file and folder permissions, to that of the root account.

#### umask
* running umask by itself will give you the current default umask settings
* `umask` => 0002
* Uh, so what does that mean?
* umask will take away whatever permissions from the default permissions
* For example: whenever you create a new directory on a system the main default permissions are 777
  * This means that everyone has full rwx access
  * So you will take the umask value and subtract from the default permissions
  * 0777 - 0002 = 0775
  * so with a umask setting of 0002 the 775 is now the default permission for folders
  * since the default permissions for files are 666, with a umask setting of 002; they become 664
* the root user has a more restrictive umask value of 0022
  * a direcotry a root user creates will have a value of 755
  * a file a root user creates will have a umask value of 644
  * So what does a regular user have 002 and the root user have 022?
  * There are two configuration location for the umask value
    * one is in `/etc/bashrc` and that is the umask for the whole system
    * the other is in /home/[user]/.bashrc which is set for the individual user
* How do I set default umasks?  By using the `umask` command of course!
  * Below is how we give the user owner full read, write and execute permissions and nothing to the group or others ownership
  * eg: `umask u=rwx,g=,o=,` => 0077 (after you run the `umask` command)
  * so this means that the 0 will not take anything away and the two "7" will take everything away
  * setting the umask value on the cmd line like this is only temporary
  * in order to make it permanent you need to add it to your .bashrc file in your home directory as described above
    * `echo "umask 0077" >> ~/.bashrc`
    * this command will redirect the output that is in quotes and append it to the .bashrc file
    * you can run `source .bashrc` or log out and log back in for the new configuration to take effect

# 104.6 Create and Change Hard and Soft Links
## Understanding Links
Symbolic links are essentially shortcusts to other files and folders.  In this section we create symbolic links and discuss how it realtes to the orginal file.  They are kind of like shortcuts in Windows.

### Syntax
#### `ln`
* `ln` will create a hard link to a file or directory
* this type of link only works on the file system of the originiating file

#### `ln -s`
* `ln -s the/original/file where/the/link/file.lnk`
* The -s means to make a symbolic link
* You know you have a link when you do a long listing `-l` and you see the file name with a -> which means: "this file is linked to -> this file" 
* eg: test.txt.lnk -> Documents/test.txt
* You will also see the permissision of a symbolic link listed thusly
* eg: `lrwxrw-rw-` the l at the beginning means it is a symbolic link too
* if you edit the link file, you will have changed the original file
* if you delete the link file the original file will be just fine
* however if you delete the original file, the link will have nothing to point to and will be an orphan link
* this goes to show that a symbolic link(shortcut) is its own file
  * said another way, when a symbolic link is created, another inode number is used as we have a new and slightly different file
 
#### `unlink`
* You can use the `unlink`  command to remove a symbolic link
* eg: `unlink file.lnk`  to unlick you just use unlink and then the name of the link/shortcut; not the name of the file

# 104.7 Find System Files and Place Files in the Correct Location
## File System Hierarchy Standard
We explore the layout of the Linux file system.
What is the purpose of a file system?
  *The purpose of the file system is to store data within a certain manner so that:
    * The data is organized and easlily located
    * The data can be saved in a persistent manner
    * Data integrity is preserved
    * The data can be quickly retreived for a user in a later point in time

#### Standard Layout fo the Linux File System
* Filesysem Hierarchy Standard (FHS)
  * this standard was created so that any Linux user would know where to find certain types of files
  * it is more of a guideline than a standard and many Linux distros do adhere to the structure
* Directory Structure:
  * inverted tree with a single root
  * case sensitive
  * paths demlimted by a forward slash (/)
  * folder or file that begins with a . will be hidden from normal view
  * a single dot . means the current directory
  * a double dot .. means the parent directory; one directory up in the file heirarchy

#### Common Folders and their typical use:
  * `/` this is the root file system
  * bin contains executable programs that system users can run
    * ls, pwd, echo, etc
  * boot contains file needed to get computer booted up
    * the kernel also resides here
  * dev location where all devices are referenced from
    * all the hardware connected to the computer
  * etc contains config files for most of the system services and information
  * home this is where regular users have their own folders
  * lib for library files that contain code that is shared among applications on the system
    * this is for the 32 bit files
  * lib64 for library files that contain code that is shared among applications on the system
    * this is for the 64 bit versions
  * media is used a good mount point for CD/DVD or usb media
  * mnt is used for a good mount point for other harddrives within the system
  * opt is used to install third party applications
  * proc provides info about a linux system; it reads info from the kernel and relays it all over
  * root this is the root user's home directory
  * sbin this is used for system binaries; sys admin tools and programs
  * srv is used for server application like web servers
  * sys contains info about hardware that is on the system
  * tmp used by applications to store temporary data; data inside here does not survive a reboot
  * usr contains its own set of a directory tree that closely resembles this directory structure
    * there is a plethora of data and information within this directory
  * var contains files that tend to change alot, log files or local system email files

## Finding Commands on the Linux System
We take a look at a few commands that will help me to find files that are on my system
* There are other commands that we can use besides `find` and here are those commands now:
#### `locate`
  * `locate` searches a local database rather than the live system
    * eg: `locate passwd`
  * it is much quicker than find
  * however, the search can be inaccurate as the local database could be outof date
  * use `updatedb` to update the index for locate
  * you can configure by changing `/etc/updatedb.conf`
    * updatedb.conf has PRUNE sections or sections that the `updatedb` command will not index
    * PRUNEFS - filesystem
    * PRUNENAMES - extensions of files like .git .gh .svn
    * PRUNEPATHS - paths like /tmp /var/cache /mnt /media (basically anything that is not really persistant)
#### `whereis`
  * this will find binarys and \/ or manual pages for a command
  * eg: `whereis cd`
