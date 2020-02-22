# Topic 110: Security
## 110.1 Perform security administration tasks
##### Weight: 3

##### Description: Candidates should know how to review system configuration to ensure host security in accordance with local security policies.

#### Key Knowledge Areas:

Audit a system to find files with the suid/sgid bit set.
Set or change user passwords and password aging information.
Being able to use nmap and netstat to discover open ports on a system.
Set up limits on user logins, processes and memory usage.
Determine which users have logged in to the system or are currently logged in.
Basic sudo configuration and usage.
The following is a partial list of the used files, terms and utilities:

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

#### Key Knowledge Areas:

Awareness of shadow passwords and how they work.
Turn off network services not in use.
Understand the role of TCP wrappers.
The following is a partial list of the used files, terms and utilities:

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

#### Key Knowledge Areas:

Perform basic OpenSSH 2 client configuration and usage.
Understand the role of OpenSSH 2 server host keys.
Perform basic GnuPG configuration, usage and revocation.
Use GPG to encrypt, decrypt, sign and verify files.
Understand SSH port tunnels (including X11 tunnels).
The following is a partial list of the used files, terms and utilities:

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
