# Topic 110: Security
## 110.1 Perform security administration tasks
##### Weight: 3

##### Description: Candidates should know how to review system configuration to ensure host security in accordance with local security policies.

### Key Knowledge Areas:

##### Audit a system to find files with the suid/sgid bit set.
`find / -perm -u+s`  or `find / -perm -04000`
* this will find the set user id bit across your system

`find / -perm -g+s` or `find / -perm -02000`
* this will find the set group id bit across your system

It is a great idea to have a cron job run regularly to see if anything has been added or removed from this list as bad actors will sometimes mess with permissions to gain a foot hold in a system.

##### Set or change user passwords and password aging information.
`passwd` is the command you would use to change your own password.  Only the root user is able to change other's passwords.  Root would invoke, `passwd -e nbennett` to change the user, nbennnett's password.

Regarding password aging, the `chage` command is the command to use.

`chage -E 2021-01-01 avance` 
* will change the expiration date to Jan 1, 2021 for the user, avance.

`chage -E $(date -d +180days +%Y-%m-%d) avance`
* will set an account to expire in 180 days from today

`chage -E -1 avance`
* this will remove any expiration date to a user's password effectively locking the customer/employee out

`change -l bcalhoun`
* to (l)ist the settings of the user, bcalhoun's password aging settings.

`chage -W 14 bcalhoun` will change how many days, before the expiration date, that the password will expire.

##### Being able to use nmap and netstat to discover open ports on a system.
`nmap $IPAddress` will give you a default list of ports that it discovers as open, closed or filtered on a system.  You can use the flag -p to specify a port like `nmap 108.23.232.99 -p 22,23,25,80,443,465,587,3306,27017`  You would typically use this from outside of system and probe the ports from the outside looking in for the status.  Remember that by default, nmap sends ICMP echo requests.

`sudo lsof -i` will show you all the service and ports that are being used.

##### Set up limits on user logins, processes and memory usage.
##### `ulimit`
* This is the command used to set limits on resources a user can utilize.  "-a" will show you system limits and also give you the flags to change the limits.  This change is lost with a reboot.  To thusly set the limits to make them permanent, you will want to modify /etc/security/limits.conf file.

```bash
$ ulimit -a
core file size          (blocks, -c) unlimited
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 15649
max locked memory       (kbytes, -l) 16384        # know how to set this
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited    # know how to set this
max user processes              (-u) 15649        # know how to set this
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```
To set the above limits (temporarily) you would run `ulimit` along with the appropriate switch and value:
* `ulimit -m 2048`  # this will set the max memory a user process can use; very useful if a program is hogging to much memory
* `ulimit -u 25`    # this sets the maximum number of processes available to a single user
* `ulimit -t 360`  # this sets the maximum cpu time in seconds
##### `/etc/security/limits.conf`
```bash
# /etc/security/limits.conf
#
#This file sets the resource limits for the users logged in via PAM.
#It does not affect resource limits of the system services.
#
#Also note that configuration files in /etc/security/limits.d directory,
#which are read in alphabetical order, override the settings in this
#file in case the domain is the same or more specific.
#That means for example that setting a limit for wildcard domain here
#can be overriden with a wildcard setting in a config file in the
#subdirectory, but a user specific setting here can be overriden only
#with a user specific setting in the subdirectory.
#
#Each line describes a limit for a user in the form:
#
#<domain>        <type>  <item>  <value>
#
#Where:
#<domain> can be:
#        - a user name
#        - a group name, with @group syntax
#        - the wildcard *, for default entry
#        - the wildcard %, can be also used with %group syntax,
#                 for maxlogin limit
#
#<type> can have the two values:
#        - "soft" for enforcing the soft limits
#        - "hard" for enforcing hard limits
#
#<item> can be one of the following:
#        - core - limits the core file size (KB)
#        - data - max data size (KB)
#        - fsize - maximum filesize (KB)
#        - memlock - max locked-in-memory address space (KB)
#        - nofile - max number of open file descriptors
#        - rss - max resident set size (KB)
#        - stack - max stack size (KB)
#        - cpu - max CPU time (MIN)
#        - nproc - max number of processes
#        - as - address space limit (KB)
#        - maxlogins - max number of logins for this user
#        - maxsyslogins - max number of logins on the system
#        - priority - the priority to run user process with
#        - locks - max number of file locks the user can hold
#        - sigpending - max number of pending signals
#        - msgqueue - max memory used by POSIX message queues (bytes)
#        - nice - max nice priority allowed to raise to values: [-20, 19]
#        - rtprio - max realtime priority
#
#<domain>      <type>  <item>         <value>
#

#*               soft    core            0
#*               hard    rss             10000
#@student        hard    nproc           20
#@faculty        soft    nproc           20
#@faculty        hard    nproc           50
#ftp             hard    nproc           0
#@student        -       maxlogins       4
userBenn         hard    memlock         2048 # limites only 2GB of memory to be used
@dataEng         -       maxlogins       4   # limits the number of logins of the dataEng group
                                             # the `-` here means that it is a hard & soft limit
userBenn         soft    cpu             150 # not contiguous minutes 
userBenn         hard    cpu             200 

# End of file
```


#### Determine which users have logged in to the system or are currently logged in.

##### `who` 
```bash
$ who
userBenn pts/0        2020-02-22 08:58 (123.245.85.22)
```
This gives us the username, the terminal they are logged in on, the time they logged in, and the IP address they logged in from

##### `w`
```bash
$ w
 08:58:56 up 6 days, 16:25,  1 user,  load average: 0.07, 0.03, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
userBenn pts/0    123.245.85.22    08:58    0.00s  0.00s  0.00s w
```
This output is similar to the `who` output but we get some extra info.  We get, how long they've been idle, how much CPU they are using and what command they are running.

##### `last`
```bash
$ last | head
userBenn pts/0        123.245.85.22    Sat Feb 22 08:58   still logged in
userBenn pts/0        111.222.333.444      Thu Feb 20 13:12 - 16:40  (03:28)
userBenn pts/0        111.222.333.444      Wed Feb 19 04:41 - 05:11  (00:30)
userBenn pts/0        123.245.85.22    Tue Feb 18 17:34 - 17:38  (00:03)
userBenn pts/0        123.245.85.22    Mon Feb 17 15:20 - 21:04  (05:44)
userBenn pts/0        123.245.85.22    Mon Feb 17 11:16 - 14:27  (03:11)
userBenn pts/0        123.245.85.22    Sun Feb 16 15:13 - 17:41  (02:28)
userBenn pts/0        123.245.85.22    Sun Feb 16 13:11 - 13:14  (00:02)
userBenn pts/0        123.245.85.22    Sun Feb 16 12:20 - 12:26  (00:05)
userBenn pts/0        123.245.85.22    Sun Feb 16 10:22 - 10:50  (00:27)
```
This shows us who has logged in in the past.  For a list of failed logins, you can run:
##### `last -f /var/log/btmp`
```bash
$ sudo last -f /var/log/btmp | head
admin    ssh:notty    106.215.93.146   Sat Feb 22 09:59    gone - no logout
admin    ssh:notty    146.196.45.185   Sat Feb 22 09:46 - 09:59  (00:13)
admin    ssh:notty    106.215.93.146   Sat Feb 22 09:36 - 09:46  (00:10)
nmrsu    ssh:notty    202.191.200.227  Sat Feb 22 09:18 - 09:36  (00:17)
admin    ssh:notty    117.242.221.44   Sat Feb 22 09:12 - 09:18  (00:06)
dolphin  ssh:notty    180.76.114.218   Sat Feb 22 09:09 - 09:12  (00:03)
admin    ssh:notty    106.66.161.199   Sat Feb 22 08:54 - 09:09  (00:14)
aono     ssh:notty    41.63.0.133      Sat Feb 22 08:53 - 08:54  (00:01)
info     ssh:notty    112.253.11.105   Sat Feb 22 08:49 - 08:53  (00:04)
info     ssh:notty    112.253.11.105   Sat Feb 22 08:44 - 08:49  (00:05)
```

#### Basic sudo configuration and usage.
##### `/etc/sudoers`
This is the default config file for configuring users and/or groups that get eleveated privileges.  Said another way, if a user or a group is listed in the `/etc/sudoers` file, they will be allowed to run some or all commands as the root user.  The commands they are allowed to run depends on the configuration of this file.

You use the `visudo` command to edit this file becuase `visudo` will check the configuration to make sure it is all good in the hood.  By all good in the hood, I mean that it will check that you did not make a configuration change that could break your system.  It will also lock the file so that only one (root) user at a time can make changes to it.  Snippets from the man page:
> visudo parses the sudoers file after editing and will not save the changes if there is a syntax error.

> visudo locks the sudoers file against multiple simultaneous edits, provides basic sanity checks, and checks for parse errors

##### Sudoer File Snippet
This is a snippet of the default sudoers file that provides the various configurations for the various users.
```bash
## Next comes the main part: which users can run what software on
## which machines (the sudoers file can be shared between multiple
## systems).
## Syntax:
##
##      user    MACHINE=COMMANDS
##
## The COMMANDS section may have other options added to it.
##
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL

## Allows members of the 'sys' group to run networking, software,
## service management apps and more.
# %sys ALL = NETWORKING, SOFTWARE, SERVICES, STORAGE, DELEGATING, PROCESSES, LOCATE, DRIVERS

## Allows people in group wheel to run all commands
%wheel  ALL=(ALL)       ALL

## Same thing without a password
# %wheel        ALL=(ALL)       NOPASSWD: ALL

## Allows members of the users group to mount and unmount the
## cdrom as root
# %users  ALL=/sbin/mount /mnt/cdrom, /sbin/umount /mnt/cdrom

## Allows members of the users group to shutdown this system
# %users  localhost=/sbin/shutdown -h now

## Read drop-in files from /etc/sudoers.d (the # here does not mean a comment)
#includedir /etc/sudoers.d
```

The Syntax of these entries are **user or group ______ MACHINE=COMMANDS ______  asOtherUser**


##### `root    ALL=(ALL)       ALL`
This line means that the root user can run all commands on all hosts as any user

##### `%wheel  ALL=(ALL)       ALL`
This line means that anyone in the user group "wheel" will have the same permissions as the root user defined above

##### `%users  ALL=/sbin/mount /mnt/cdrom, /sbin/umount /mnt/cdrom`
This line means that anyone in the user group "users" can run the listed commands on any system only as themselves.


### The following is a partial list of the used files, terms and utilities:

##### `find`
##### `passwd`
##### `fuser`
##### `lsof`
##### `nmap`
##### `chage`
##### `netstat`
##### `sudo`
##### `/etc/sudoers`
##### `su`
This command means to *switch user*. It looks like `su - username` or `su username`.  The difference is the \`-\` which means that when the dash is there, use a login shell and when the dash is not there, use a non-login shell.  You can also run just `su -` or `su` in this case the root user is assumed.
##### `usermod`
##### `ulimit`
##### `who, w, last`


## 110.2 Setup host security
##### Weight: 3

##### Description: Candidates should know how to set up a basic level of host security.

### Key Knowledge Areas:

##### Awareness of shadow passwords and how they work.
A shadow password file is a system file in which encrypted user passwords are stored so that they aren't available to people who try to break into the system. Ordinarily, user information, including passwords, is kept in a system file called /etc/passwd. A typical entry would look like this:
`avance:!$6$8xhLKYmd$17i4cRfYyioY0KspqutzbdvMyTiTEGO06soAgyfec:18217:0:99999:7::1:`
The crazy string of numbers is the hashed and salted password

##### Turn off network services not in use.
An example of this would be to run `service cups stop` or `systemctl stop cups.service`.  Check to see which network services are up and running with `sudo lsof -i`.  Linux treats everything as a file so when a network port is open, it will show up with the aforementioned command.  The switch `-i`, to me, means internet.

You can also use xinetd (sysvinit) or systemd.socket (systemd) to act as a gatekeeper to network services.

##### Understand the role of TCP wrappers.
This functionality uses a host.allow and/or a hosts.deny file to determine access to network services.  Said another way, tcp wrappers uses the two configuration files to determine if an incoming request is allowed to access a network service or not.  .allow is checked first and then .deny.  If an incoming request does not match either file, the request is by default, allowed to do its thing.

### The following is a partial list of the used files, terms and utilities:

##### `/etc/nologin`
##### `/etc/passwd`
##### `/etc/shadow`
##### `/etc/xinetd.d/`
##### `/etc/xinetd.conf`
##### `systemd.socket`
##### `/etc/inittab`
##### `/etc/init.d/`
##### `/etc/hosts.allow`
##### `/etc/hosts.deny`


## 110.3 Securing data with encryption
##### Weight: 4

##### Description: The candidate should be able to use public key techniques to secure data and communication.

### Key Knowledge Areas:

##### Perform basic OpenSSH 2 client configuration and usage.
##### `ssh-keygen -b 1024 -t DSA`
This will initiate the creation of a 1024 bit key encrypted with DSA.  Typically, for minium security you will want a 2048 bit key encrypted with RSA.  I believe DSA provides minimal protection and should be avoided.

##### Understand the role of OpenSSH 2 server host keys.
The new OpenSSH format has increased resistance to brute-force password cracking.





Perform basic GnuPG configuration, usage and revocation.

##### Use GPG to encrypt, decrypt, sign and verify files.

###### Create Key
`gpg --gen-key`
* You will be prompted for options

`gpg --list-keys`
* Will show you the key you just created and write down the key name eg: "15DEFSS2"

`gpg -o pubkeyName --export 15DEFSS2`
* Create a publicKey to give to other person to decrypt; send to other person via email or ssh

`gpg --import pubkeyName`

* Other person imports `pubkeyName` into their keyring

###### Encrypt your file
`gpg -r "EncryptedFile" -e passwd
* Will now have file called passwd.gpg to send to other person

  


###### Decrypt
`gpg EncryptedFile`
* This will decrypt a file provided I have their public key

###### Sign and Verify



##### Understand SSH port tunnels (including X11 tunnels).
Outgoing tunnels/local port forwarding accepts traffic on a local port (localhost:port) and then pushes the traffic to a specified remote port (the internet at large) 
`$ ssh2 -L 1234:localhost:23 username@host`
* all traffic coming to port 1234 on the client (localhost) will be forwarded to port 23 on the server (host). 

Incoming tunnels/remote port forwarding will forward traffic coming into a remote port and push it to a local port.  This takes internet traffic from the outside and serve it through a localhost:port.
`$ ssh2 -R 1234:localhost:23 username@host`
* all traffic which comes to port 1234 on the server (host) will be forwarded to port 23 on the client (localhost).

NOTE: I think they mean local network like "localhost"  To me, it seems like mail forwarding.  For Outgoing port forwarding, I send mail from my local address to a relay proxy (remote port) and then my mail gets sent as if it came from that remote proxy I sent my mail to.  For an incoming port forwarding, incoming mail gets sent to my remote proxy and then it gets sent to me.  But the sender thinks, I am the remote proxy in both situations.

##### What is application tunneling?
This is the process of securing data transmission through otherwise unsecure protocols, via port forwarding?  The definition reads: ...is a way to tunnel otherwise unsecured TCP traffic through Secure Shell.  The SSHv2 connection protocol provides channels that can be used for a wide range of purposes.  All these channels can be used for forwarding any TCP/IP ports and X11 connections.  

### The following is a partial list of the used files, terms and utilities:

##### `ssh`
##### `ssh-keygen`
##### `ssh-agent`
##### `ssh-add`
##### `~/.ssh/id_rsa and id_rsa.pub`
##### `~/.ssh/id_dsa and id_dsa.pub`
##### `~/.ssh/id_ecdsa and id_ecdsa.pub`
##### `~/.ssh/id_ed25519 and id_ed25519.pub`
##### `/etc/ssh/ssh_host_rsa_key and ssh_host_rsa_key.pub`
##### `/etc/ssh/ssh_host_dsa_key and ssh_host_dsa_key.pub`
##### `/etc/ssh/ssh_host_ecdsa_key and ssh_host_ecdsa_key.pub`
##### `/etc/ssh/ssh_host_ed25519_key and ssh_host_ed25519_key.pub`
##### `~/.ssh/authorized_keys`
##### `ssh_known_hosts`
##### `gpg`
##### `gpg-agent`
##### `~/.gnupg/`
