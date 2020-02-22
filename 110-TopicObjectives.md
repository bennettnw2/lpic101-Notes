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
`ulimit`
* This is the command used to set limits on resources a user can utilize.  "-a" will show you system limits and also give you the flags to change the limits.  This change is lost with a reboot.  To thusly set the limits to make them permanent, you will want to modify /etc/security/limits.conf file.

```bash
ulimit -a
core file size          (blocks, -c) unlimited
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 15649
max locked memory       (kbytes, -l) 16384
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 15649
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

`etc/security/limits.conf .file
```bash
[sudo] password for bennettnw2:
COMMAND     PID       USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
NetworkMa   756       root   18u  IPv4 538693      0t0  UDP JavaDev:bootpc
sshd        769       root    6u  IPv4  24040      0t0  TCP *:ssh (LISTEN)
sshd        769       root    8u  IPv6  24051      0t0  TCP *:ssh (LISTEN)
chronyd   15942     chrony    7u  IPv4 320157      0t0  UDP localhost:323
chronyd   15942     chrony    8u  IPv6 320158      0t0  UDP localhost:323
sshd      28104       root    5u  IPv4 561475      0t0  TCP JavaDev:ssh->172.104.2.4:57179 (ESTABLISHED)
sshd      28118 bennettnw2    5u  IPv4 561475      0t0  TCP JavaDev:ssh->172.104.2.4:57179 (ESTABLISHED)
___________________ JavaDev | bennettnw2 ~
$ ulimit -a
core file size          (blocks, -c) unlimited
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 15649
max locked memory       (kbytes, -l) 16384
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
 18 #        - priority - the priority to run user process with
 17 #        - locks - max number of file locks the user can hold
 16 #        - sigpending - max number of pending signals
 15 #        - msgqueue - max memory used by POSIX message queues (bytes)
 14 #        - nice - max nice priority allowed to raise to values: [-20, 19]
 13 #        - rtprio - max realtime priority
 12 #
 11 #<domain>      <type>  <item>         <value>
 10 #
  9
  8 #*               soft    core            0
  7 #*               hard    rss             10000
  6 #@student        hard    nproc           20
  5 #@faculty        soft    nproc           20
  4 #@faculty        hard    nproc           50
  3 #ftp             hard    nproc           0
  2 #@student        -       maxlogins       4
  1
61  # End of file
```



##### Determine which users have logged in to the system or are currently logged in.
who w and last
##### Basic sudo configuration and usage.

### The following is a partial list of the used files, terms and utilities:

find
passwd
fuser
lsof
nmap
chage
netstat
sudo
/etc/sudoers
su
usermod
ulimit
who, w, last


## 110.2 Setup host security
##### Weight: 3

##### Description: Candidates should know how to set up a basic level of host security.

### Key Knowledge Areas:

Awareness of shadow passwords and how they work.
Turn off network services not in use.
Understand the role of TCP wrappers.
### The following is a partial list of the used files, terms and utilities:

/etc/nologin
/etc/passwd
/etc/shadow
/etc/xinetd.d/
/etc/xinetd.conf
systemd.socket
/etc/inittab
/etc/init.d/
/etc/hosts.allow
/etc/hosts.deny


## 110.3 Securing data with encryption
##### Weight: 4

##### Description: The candidate should be able to use public key techniques to secure data and communication.

### Key Knowledge Areas:

Perform basic OpenSSH 2 client configuration and usage.
Understand the role of OpenSSH 2 server host keys.
Perform basic GnuPG configuration, usage and revocation.
Use GPG to encrypt, decrypt, sign and verify files.
Understand SSH port tunnels (including X11 tunnels).
### The following is a partial list of the used files, terms and utilities:

ssh
ssh-keygen
ssh-agent
ssh-add
~/.ssh/id_rsa and id_rsa.pub
~/.ssh/id_dsa and id_dsa.pub
~/.ssh/id_ecdsa and id_ecdsa.pub
~/.ssh/id_ed25519 and id_ed25519.pub
/etc/ssh/ssh_host_rsa_key and ssh_host_rsa_key.pub
/etc/ssh/ssh_host_dsa_key and ssh_host_dsa_key.pub
/etc/ssh/ssh_host_ecdsa_key and ssh_host_ecdsa_key.pub
/etc/ssh/ssh_host_ed25519_key and ssh_host_ed25519_key.pub
~/.ssh/authorized_keys
ssh_known_hosts
gpg
gpg-agent
~/.gnupg/
