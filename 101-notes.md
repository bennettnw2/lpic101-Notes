101.1 Determine and configure hardware settings

# Pseudo File Systems
What is a pseudo file system?  The best place to begin is to define what a regular file system is.  A regular file system is a method of laying out files and folders on a physical hard disk.  Something about information contained in the file systems headers.  What is that?

In Linux, every file and folder and even devices are really just files.  A folder is a file, that contains other files.  Devices are accessed, configured and utilized by manipulating the device file.  I think.

A pseudo file system does not actually exist on a hard disk; it only exists in RAM while the system is running.  These file systems are created by the kernel after the system has booted up.  Once you shut the computer off, a pseudo file system ceases to exist.  There are many different pseudo file systems in Linux; however, we are only going to focus on the /proc an /sys directories

`/proc` the /proc directory contains information about the processes running on a system.  Process are listed by a PID with hardware and process data both in in the same directory structure. If you look into the /proc folder you will see many folders with numbers for names, these numbers correspond to Process ID numbers or PIDs.  Therefore, all processes currently running on a file system can be seen from within this directory by matching the PID.

There are also other files within this directory like, cpuinfo, meminfo, partitions, uptime, version, etc.  You can read what is in these text files for more information regarding the system. `cat /proc/1/oom_score`  or `cat /proc/1/cmdline`

`/sys' the /sys directory contains information about the system's hardware and kernel modules.  No process information is listed here.  We have access to these plain text files to let us know what is going on with the hardware and the kernel exposes information here on what it is working on; other applications can use the information supplied by the kernel to, in effect, communicate with the kernel?

For example, there is an /fs directory within /sys.  This /fs folder contains information regarding all the file systems attached to the computer system.  So you can cat these plain text files and if you know what you are looking for and how to read the output, you can glean a lot of great information regarding your system.

The biggest thing to takeaway from all this is that Linux is run on one large, plain text, file system.  I think this is what provides the elegance and beauty of a Linux system.  Everything is in plain view and is mostly in human readable form?

`man proc` shows local documentation on the /proc pseudo file system

# Working with Kernel Modules
The Linux kernel is the core framework of the operating system.  What do we mean by framework?  GNU is utilities and programs; Linux is the rest of the system to operate with hardware, memory, networking and even managing itself.
Linux kernel is monolithic meaning it is big and over encompassing.  It handles all memory management and hardware device interactions.  Extra functionality can be loaded and unloaded dynamically through kernel modules.  The modules can be loaded dynamically and with out rebooting.  Many third party modules are device drivers.

`uname` displays information about the currently running kernel
`-m` shows architecture
`-r` shows release version
`-a` shows all the information regarding a system; OS, hostname, kernel release version, build time, machine architecture, cpu type, and the OS Label
eg: Linux goldenserver 3.10.0-862.14.4.el7.x86_64 #1 SMP Wed Sep 26 15:12:11 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux

`lsmod` list modules will display a listing of all currently loaded kernel modules
`modinfo [module name]` displays information abut a specified kernel module

`modprobe` you can use this to dynamically load and unload kernel modules at runtime
`modprobe -r [module name]` will remove a module dynamically at runtime
`modprobe [module name]` will add a module back dynamically at runtime

 # Investigating Hardware 
Linux Device Management
`udev` is a Linux service that monitors for devices
How Linux handles hardware and how we can view hardware info from the command line;
There is a service that runs called udev which is the actual Linux device manager.  `udev` monitors for devices being attached to the system.

For example, when a hard-drive is attached to your computer, the udev service detects and passes info about the hard-drive via the dbus or data-bus service (to me is seems like a data highway for the whole system) to the pseudo file system, /dev

`D-Bus` sends data messages between applications.  It is a conduit of information about what is going on in the system, udev utilizes dbus to notify users and the system when new hardware is attached.

`/dev` The hard-drive is attached to the /dev pseudo file system. The /dev pseudo file system is different from /proc and /sys pseudo file systems. (How are the different?  I seems the are different in that /sys contains internal hardware and internal processes while the /dev is external hardware and software?)  /dev contains all the handles for all the devices.  Any hardware picked up by the udev service get sent to /dev.  Using the lsblk command will grab the handles of the hardware and allow a user to interact with the hardware

So these are not text files in the /dev directory, these are character files that pass information back and forth to the system throughout he /dev directory.  We cannot open an individual file as we will on see binary information.  There are tools that will read the information for us though.  Also in /dev is info about the hard drives.  Even individual partitions of hard-drives are listed individually in /dev 

## Short List of commands to interact with /dev
`lspci` displays information on PCI devices attached
`lsusb` displays information on USB devices attached
`lscpu` displays information on processors on a system
`lsblk` displays information on all block devices (hard drives) on a system

`lspci -k` will show kernel drivers?
`lspci -v` will give you much more verbose information about?

The output of lsusb will show you ~route hubs~.
eg: ~Bus 002 Device 002:~ ID 1045:23a4 Western Digital USB Hard Drive
`lsusb -v` for more verbose output
`lsusb -t` will show, in tree view, which device is attached to which host controller

`lsblk -f` will show what the file system type is on each partition

Remember all this info comes from the /dev directory!!!  Pertains to all hardware configured on our system.


101.2 Boot the System

# The Linux Boot Sequence
Also know as the initialization
1. Power 2. Bios 3. Boot Sector/Grub 4. Kernel 5. Initial Ram Disk 6. Initialization of system (systemd)

Boot logs are generated from the kernel ring buffer
`dmesg` details about kernel hardware and memory management; use dmesg
`journalctl -k` is the dmesg of systemd; the -k means show everything

# init
Goal: be aware of and understand the init sequence using sysvinit
`init` short for initialization and it is based off the System V init used in UNIX systems.  It is now known as sysvinit.  Services are started one after the other in a serial fashion.  If one service or function was not ready, it could hang the system. :(

After the kernel loads up and loads the initial ram disk, it then looks for an initialization system to hand over control of the computer.  The first place the kernel looks for is `/sbin/init`.  If it is found, the kernel gives it control.

Then sysvinit will firstly look for instructions in `/etc/inittab`.  This is the sysvinit configuration file. So what does sysvinit want to get out of /etc/inittab? One thing is to determine the runlevel to run the system on.

#### * `Runlevel` is a predefined configuration to run the system as.  Each runlevel will start or stop scripts depending on what the runlevel asks for.  Only one runlevel can be active at a time.  The runlevel applies to the system as a whole.  These are the based on the Linux standards base.  Slackware and Gentoo have a slightly different.

```
0   Halt
1   Single user mode
2   Multi-user mode (no networking)
3   Multi-user mode (with networking)
4   unused
5   Multi-user, with networking and a graphical desktop
6   Reboot
```
```
0 stop services and power off
1 single user mode, the root user is the only user allowed to log into the system; used for repair and maintenance
2 multiple users but no networking or remote file systems
3 same as above except networking is available; most Linux servers were configured to run at this level.
4 for a custom environment
5 same as 3 but with a graphical desktop
6 stops services and restarts system
```

#### Example inittab file:
for example:  `id:3:initdefault:`
<identifier>:<runlevel>:<action>:<process>
`action` there are predefined actions that init understands.  There are no processes to act on from the above example
`process` si::sysinit:/etc/rc.d/rc/sysinit  <- what is in the process section is what will be ran; this is a startup script as there is no run level defined as it always needs to start?

`wait` l0:0:wait:/etc/rc.d/rc 0; wait means that the process specified will wait be started once the runlevel is entered.  In addition, init will wait for the termination of the process specified, in this case /etc/rc.d/rc 0

1. Boot partition is found
2. Kernel and initial ram disk are loaded
3. Kernel pulls initial drivers and set up tools from the ram disk
4. Kernel then hands over the control of the system to /sbin/init
5. `init` then reads /etc/inittab and then performs some maintenance tasks from /etc/rc.d/re.sysinit 
6. `init` then reads the initdefault line in /etc/inittab and then enters the specified runlevel 6.
7. Then the system is ready for use and you log in!

What scripts is init working with to get the system ready?  These are the locations?
Red hat `/etc/rc.d`
Debian `/etc/init.d`

`rc` run commands

/etc/rc.d or /etc/init.d contains: rc0.d - rc5.d are all different run levels

`rc.sysinit` runs some scripts before entering a run level as defined by /etc/inittab file
`rc.local` is run after the runlevel is completely loaded; used by admins to open their own things...

In looking at what is contained in /etc/rc.d/rc3.d or any run command level:  All the scripts are symbolic links under etcinit.d; the symlinks beginning with a k need to be killed and the ones with an s indicates which to be started.  They are also numbered so that they can start in a specified order

`/etc/init.d` contains the scripts for the services on they system

`/etc/init.d/rc` is the script that orchestrates how the runlevel scripts run and what occurs when a runlevel changes

## sysvinit boot sequence
/sbin/init --> /etc/inittab --> /etc/rc.d/rc.sysinit --> /etc/rc.d/rcX.d(in numerical order) --> Login

This is how systems were traditionally started.  This will help us to understand why systemd is soooo cool.

# upstart
`startup(7)`
`mountall(8)`
`telinit(8)`
`runlevel(7)`
Upstart was first developed for Ubuntu 6.10 in 2006
Eventually REHL6 Debain and Fedora included upstart
The big deal at the time is that upstart offered synchronous starting of services; this is the opposite of sysvinit which started jobs one at a time and waited for the job to finish
Since upstart to start jobs in parallel, it decreased boot times.
Another cool thing upstart could do is monitor for system events

## Upstart boot sequence
To maintain compatibility with older sysvinit kernels, upstart was installed in /sbin/init and also named "init"
The first task of upstart is to start the `startup(7)` event.  The startup event will check and mount the disks and files systems with `mountall(8)`
In an asynchronous manner, the startup event will also check out /etc/init/rc-sysinit.conf to see if there is an /etc/inittab file to see if there are any config options set there
the /etc/init/rc-sysinit.conf ultimately calls the telinit(8) which will then set the runlevel(7)
and then runlevel(7) would be passed into /etc/init/rc.conf
after start up, there are multiple jobs running at the same time

## Upstart Service Monitoring
Upstart was able to monitory for changes on the system; for instance a monitor being plugged in.
When anything changes on a Linux system, that is known as an event.
An event will then trigger a job, which can be one of two categories: Tasks or Services
A task is a one time quick in and out job while a service is a continuous and on going task.
A task will do what is requested and then return to a waiting state
A service will typically not stop by itself, only if an event calls for the stoppage or an admin would command the stoppage.

## What is a Waiting State?
Wait is the initial position
starting - the job is about to start
running - the job is running
stopping - it has processed the pre-stop part
killed - this is where the is actually stopping
post-stop - the job is completely stopped
re-spawning - job quits unexpectedly; upstart will try to restart 10 times @ 5 second intervals

# Systemd
systemd unit files are important as they tell us how the system will boot up
What did systemd do to depreciate upstart and sysvinit
- init and upstart relied on bash shell scripts; this slows down the system as the shell was slow
- systemd replaced many of the bash shell scripts with compiled C code which was much more performant
- systemd is still compatible with sysvinit, like 99% compatible.

## unit file locations
do not ever edit unit files; if you want to change a unit file is in /etc/systemd/system; this location has the highest priority over the other two locations.  But /usr/lib/systemd/system, has priority over /run/systemd/system

/etc/systemd/system
/usr/lib/systemd/system <- do not edit the files here as they may get changed
/run/systemd/system

View all unit files by running `systemctl list-unit-files`

## Components of the Unit Files
Very similar to INI files from MS-DOS

[Unit]
Description=Multi-User System  <- do not need quotes even though there is a space
Documentation=man:systemd.special(7)
-- `systemctl help unit` will access the documentation location
-- can use multiple documentation location
Requires=basic.target
-- this is important
Wants=basic.target
Conflicts=rescue.service rescue.target
After=basic.target 
-- the targets listed here will start first and then this one will start after them
Before= same as above but this will start before others listed here
AllowIsolate=yes
-- this allows us to isolate (whatever that means) and then to switch the active and loaded target
-- for example: switching from multi-user.target to rescue.target without powering down

[Service]

[Install]

Check out systemd.unit(5)

`systemctl cat something.unit` will print out the contents of the unit file specified

## Systemd is the New Init
How does systemd boot the system?
Remember how the kernel will look for an init file in /sbin/init?
Well, a symbolic link was created! /sbin/init -> ../lib/systemd/systemd
So while the kernel thinks it is starting /sbin/init, it is really starting off systemd, the service manager.
Then systemd reads its unit files and starts and stops process as dictated by the target units


# 101.3 Change runlevels / boot targets and shutdown or reboot system

## Change Your Working Environment: runlevels
So this is really only for sysvinit and upstart systems
0   Halt
1   Single user mode
2   Multi-user mode (no networking)
3   Multi-user mode (with networking)
4   unused (but you can use it to create a custom runlevel)
5   X11 / Multi-user, with networking and a graphical desktop
6   Reboot

`runlevel` use this command to see what runlevel you are currently in
The output would look like `N 5` N means there was no previous runlevel and 5 is the current

You can use either `init 1` or `telinit 1` to change the runlevel; note you will have to be the root user in order to change runlevels.  This is what it looks like to change from 5 to 3
`telinit 3`
So if you run `runlevel` again the output will be `5 3` which mean

/etc/inittab file is where the runlevels are stored; it will also tell you what the default runlevel is which is typically 5

### Changing the runlevel at boot
While the computer is booting, just press any key to get to the grub menu
Next you press <a> to modify the arguments being passed to the kernel during boot
We will then see the kernel command line which looks like this

`<TYPE=pc KEYTABLE=us rd_NO_DM rhgb quiet`

Then we just append the runlevel we want at the end of this line like so:

`<TYPE=pc KEYTABLE=us rd_NO_DM rhgb quiet 3`

You could even shutdown(halt) your system by changing the runlevel to 0


## Change Your Working Environment: targets

### Purpose of a Systemd Target
So this section is for systemd (service manager)
Remember a systemd unit file is a file that will tell the system how to boot up
systemd unit files are important as they tell us how the system will boot up
So moving from unit files overall, we will now focus on more granular, target units
A target unit's purpose is to sync with other units on a system when it boots up or when it is instructed to change states

The most common use for a target is to get the system into a new state. Like when you want a multi user command line that you can invoke with `multi-user.target` or if you want a graphical desktop with `graphical.target`

When you run a target unit there are other units (scope, services, slices, etc ) that associate with the target unit to "make that target unit happen" or more succinctly, define the operating environment.
Each unit has a role in the overall setup of a target unit; a target unit is made up of other smaller units?

### Types of Targets
`multi-user.target` similar to runlevel 3, multi-user system command line with networking 
`graphical.target` similar to runlevel 5, multi-user system with desktop environment 
`rescue.target` similar to runlevel 1, pulls in a basic system and file system mounts and provides a rescue shell
`basic.target` basic system that is used during the boot process before another target takes over
`sysinit.target` is what takes over the system from the kernel during boot; ie: system initialization / sysinit

Check out for more information:
man 5 systemd.target - defines the target unit configuration
man 7 systemd.special - listing of all target units and definitions

### Sample Target Unit File
`systemctl cat graphical.target`
We will see that we are looking at the unit file for the graphical.target
display-manager.service provides the graphical part

`systemctl list-unit-file -t target`
this will show all the target files on the system with their current states; the states will vary depending on the systems configurations

`systemctl list-units -t target`
will list out all the current active units specified by the type of target

## Changing default targets
When systemd starts up it checks to see what the default target should be for the computer.  The default target is just a symbolic link to whichever target environment the administrator has deemed to be the one the system uses by default.  Therefore, when you change the default target, all you are doing is changing the pointer from /etc/systemd/system/default.target to lib/systemd/system/yourdefault.target 

`systemctl get-default` will show us what the default target of the system is currently set to

`systemctl set-default yourdefault.target` will change the default target to a different target
You will see an output showing that the symlink has been created from the `default.target` file in /etc/systemd/system/ is now point to `whichever.target` in /lib/systemd/system/

### Switching from one target to another
To change from one running target to another, we will need to isolate the target.  Remember the AllowIsolate target directive in the unit file? This is where it comes into play.
You can think of AllowIsolate as a gatekeeper as to which targets you can switch to and from.
This is like using `runlevel` or `init` to change the runlevel in sysvinit

`systemctl isolate your.targethere` will change your current running state of the system from the current target to a different target; ie: `systemctl isolate multi-user.target` this will put you into the multi-user target

You do not even have to use the isolate command.  You can simply do `systemctl your.targethere` ie: systemctl rescue | systemctl reboot | systemctl poweroff


# Reboot and Shutdown Your System

## Reboot commands
`reboot`
`telinit 6` runlevel 6 is the reboot runlevel
`shutdown -r now` you can use numbers instead of "now"
`systemctl isolate reboot.target`

`wall` will broadcast a message to all login in users (after message is typed, press <CTRL+d> to send the message

## Shutdown commands
`telinit 0` will shutdown the system
`shutdown -h 1 minute` means halt in 1 minute; wall will automatically send a message for you
You can cancel a pending shutdown with <CTRL+c>
`systemctl isolate poweroff.target`
`systemctl isolate shutdown.target`

## acpid - Advanced Configuration and Power Interface
This will register system events such as pressing the power button or closing laptop lid 
This is located in /etc/acpid and there are two folders in there called events and actions
If you look in events you will see configuration files

### Things to remember:
* `telinit n` is the command to change a sysvinit system's runlevel to n
* after using `telinit`, the change is only temporary, it will not survive a reboot
* `runlevel` is the command to simply show the runlevel on a sysvinit system
