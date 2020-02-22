# Security

## 110.1 Perform Security Administration Tasks
### Determine the Current State of a System
##### `who`
* this will list the users currently logged into a system
* it also displays what terminal they are using and what time they logged in
```bash
root tty1     2019-10-01 08:30
```

##### `w` 
* this will also list the currently logged users
* this also contains more details
  * how long idle
  * how much CPU they are using
  * what command they are running

It is always a good idea to know who logs into the system and what their typical "Descriptor" looks like
This way, you will have an idea of what a normal day looks like so that you can detect any anomalies

##### `last`
* this command shows a listing of users who were logged into the system but are now logged out
* you can find what users failed their login attempts with: `last -f /var/log/btmp`

##### `lsof`
* list of open files command
* you can use this command to see what files are open right now and whom is opening them
* best to pipe through less
* Command | PID | USER | File Descriptor | Type (Dir or File) | Node Name (process file)
* since network ports and sockets are considered files, you can see what sockets or ports are open
* `lsof -u henry | less` this will show us what user has files open
* `lsof /home/henry | less` this will show us what items are open in the directory listed

It is important to know what files have the suid and sgid bit set on them;
* you can find these files with a `find` command
* eg: `find / -perm -u+s` <= set user id bit 
* it is good to keep an eye on these types of files to make sure no one has tampered with them
* can also find by looking for the set octal
  * eg: `find / -perm -04000` <= set octal for user, group would be -02000
  * `find / -perm -g+s` <= set group id bit

It is also good to keep track of resources
##### `ulimit`
* is the command to use to set limits on resources a user can utilize
* `ulimit -a` will show you system limits
  * this also give you the flags to change the limits
  * however, this change is lost with a reboot
* set the limits thusly to make the permanent:
* /etc/security/limits.conf file
  * the soft limit cannot exceed the hard limit
  * this a very well documented document
  * under domain is either a user or a group or everyone
    * if you specify a group, put @ in front like `@engineering`
    * a user is just specified as the username `bennettnw2`
* just need to know how to set limits on:
  * CPU time
    * `cpu -t`
    * set in minutes but not concurrent minutes but cumulative on a process
    * can set hard or soft limits
  * concurrent logins
    * `maxlogins -u`
  * amount of memory a user process can use.
    * `memlock   -l`

* `passwd` 107.1

#### Granting Elevated Privileges
##### `/etc/sudoers`
* this is the default config file for configuring user and or groups that get elevated privileges
* use visudo to edit this file
##### `sudo`
* this command allow a user to run a command as root
* basically they turn into the root user for that command
##### `su`
* this is the "substitute user" command
* you can use a - and then the username
* if you don't use a dash or the field after the dash is blank; the root user is assumed


### Checking Local Network Security
Since Linux is a network operating system any Linux admin needs to be proactive with network security
##### `lsof` (revisited)
* if you remember, we can use `lsof` to view currently open files
* since ports and sockets are files, we can see which ones are currently open
* it is important to know what ports are open because each port is a door into our system
  * if you leave a door unsecured, it is only a matter of time before someone breaks and enters
* `lsof -i` will give you a list of ports that are open and listening =>
```
COMMAND    PID     USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
loginwind   95 nbennett    8u  IPv4 0xf70436706f318ec1      0t0  UDP *:*
UserEvent  297 nbennett    3u  IPv4 0xf70436706f3cc421      0t0  UDP *:*
SystemUIS  306 nbennett   10u  IPv4 0xf704367075108731      0t0  UDP *:*
SystemUIS  306 nbennett   11u  IPv4 0xf7043670751089e1      0t0  UDP *:*
SystemUIS  306 nbennett   12u  IPv4 0xf7043670751023c1      0t0  UDP *:54043
SystemUIS  306 nbennett   13u  IPv4 0xf704367075108c91      0t0  UDP *:*
SystemUIS  306 nbennett   20u  IPv4 0xf70436706c2a7ec1      0t0  UDP *:*
identitys  328 nbennett   19u  IPv4 0xf70436706f319981      0t0  UDP *:*
sharingd   334 nbennett    4u  IPv4 0xf70436706e1a1461      0t0  UDP *:*
sharingd   334 nbennett    8u  IPv4 0xf70436706e19f981      0t0  UDP *:*
sharingd   334 nbennett    9u  IPv4 0xf70436706e1a1c71      0t0  UDP *:*
sharingd   334 nbennett   10u  IPv4 0xf70436706e1a1f21      0t0  UDP *:*
sharingd   334 nbennett   11u  IPv4 0xf70436706e0089a1      0t0  UDP *:*
assistant  337 nbennett    4u  IPv4 0xf70436706f3cf481      0t0  UDP *:*
rapportd   352 nbennett    3u  IPv4 0xf70436707187d389      0t0  TCP *:55506 (LISTEN)
rapportd   352 nbennett    4u  IPv6 0xf70436706fa3a001      0t0  TCP *:55506 (LISTEN)
WiFiAgent  359 nbennett    5u  IPv4 0xf70436706e00a731      0t0  UDP *:*
LogiMgrDa  377 nbennett    6u  IPv4 0xf704367078300389      0t0  TCP *:59866 (LISTEN)
LogiMgrDa  377 nbennett   11u  IPv4 0xf704367075103941      0t0  UDP *:59867
```
* statuses
  * LISTEN - port is open and listening out for the arrival of data packets to establish a communication
  * ESTABLEHED - is a current connection that is running on a port
  * CLOSEWAIT - the client has closed their connection and we are waiting on our side to close down

##### `fuser`
* this command can be used to list all the PIDs that are assigned to a particular file or network port that is in use
* said another way, this is what you look at if you want the PID of a connection
* `fuser` in combination with `lsof` you can find errant and malicious connections to your system

##### `netstat`
* this allows you to view active connections
* `netstat -tunalp`

##### `nmap`
* this allows you to view ports from the outside in

## 110.2 Set Up Host Security
### Securing Local Logins
* Making sure that local users are protected as well
* We will look at a command to use when an account is disabled
* also review info about our user config files

Assume a user went away on vacation; we want to lock the account
* you can use the `usermod` command to lock the user's password
  * eg: `usermod -L avance`
* we can also lock the account with the usermod command by setting the expired date to the number 1
  * eg: `usermod -e 1`
* use the `getent` command to check out the shadow database file for the changes we just made
  * eg: `getent shadow avance`
* you should see an exclamation point at the beginning of the entry showing that the password has been locked
  * near the end you will see an expiration date of 1 which effectively disables the account
  * eg: `avance:!$6$8xhLKYmd$17i4cRfYyioY0KspqutzbdvMyTiTEGO06soAgyfec:18217:0:99999:7::1:`
* The account still has access to a login shell some how
  * we need to change the login shell to a no-login shell
  * eg: `usermod -s /sbin/nologin avance`
  * running the above command will give us a passwd entry with `nologin` as the shell switch
* If we try to login with this user, we will see that we will not get in with the account
* To re-enable the account:
  * example needed except the login shell
  * example needed for the result
  * with the shell set to nologin you will not be able to get into the system
* When someone is assigned a no-login shell we can create a `/etc/nologin` file
  * they will get a funny thing to happen to them if they try to login 
  * save and close the file and try to login again
  * `usermod -s /bin/bash avance`
    * this will return the users account back to normal
#### `/etc/passwd`
* anyone can view this file but only root can modify
* look at the permissions on the /etc/passwd vs /etc/shadow 
```
$ ls -l /etc/passwd
-rw-r--r--. 1 root root 1160 Sep 25 06:37 /etc/passwd
$ ls -l /etc/shadow
----------. 1 root root 800 Sep 25 06:37 /etc/shadow
```
* the /etc/shadow file looks like there are no permissions at all
  * how does anything get done and how do we change our password if there are no permissions on the /etc/shadow file
  * lets take a look at the permissions on the passwd command with:
    ```
    ls -l /bin/passwd => 
    -rwsr-xr-x. 1 root root 27832 Jun 10  2014 /bin/passwd
    ```
    * we can see that the owner has write access and the set user id bit is set
    * as the set user id is set, when you run the command, you magically run it as root
  * the root user can change anyone's password
  * the permissions on these two files should never be changed
  * you should be aware of these default permissions for the exam
    * /etc/passwd = 0644
    * /etc/shadow = 0600
  * sometimes the default permissions can be different from distro to distro
  * it is good to know that the default permissions 

### Securing Network Services
* all about securing our network ports
* firstly turn off and disable any network services you are not going to be using on your system  
* use `lsof -i` to show all services and ports that are being used
  * we want to concern ourselves with the ones that have a '\*' next to a ':'
  * this means they are listing for connections coming in from anywhere
  * ones that say "localhost" are listening internally for local services like smtp email
* then any that you are not going to use, just shut the service off and disable starting on boot
  * `service cups stop`
  * `chkconfig cups off`

##### `xinetd`
* a super-server sounds like it is a catch all listening service;
* said another way, it would provide access control to network services
* a request would come in for a service that is shut off
  * the super-server would do some security checks
  * if the checks came back clear, the super-server would start the service to respond to the request
* `xinetd` was the best implementation of a super-server.
* `xinetd` is found on older systems; Centos 6 for example
* the reason is because of systemd
  * systemd uses socket unit files to perform the same tasks as `xinetd`

##### `/etc/xinetd.conf`
* the config file for xinetd is `/etc/xinetd.conf
* there are logging settings for `xinetd`
  * each service that `xinetd` covers will get its own log file in `/etc/xinetd.d/`
* we can restrict access to certain services
```bash
# Define access restriction defaults
#
#       no_access       =
#       only_from       =
#       max_load        = 0
        cps             = 50 10
        instances       = 50
        per_source      = 10
```
  * `cps` means connections per second
    * eg: `cps      = 50 10`
    * meaning, if more than 50 connections per second are requested of the service, disable the service for 10 seconds
  * `instances = 50` means we can have up to 50 instances of a particular service
  * `per_source = 10` means no more than 10 incoming source IP addresses
  * we also see that the file `/etc/xinetd.d` is referenced and included into this config file at the very end of the file
  ```bash
  includedir /etc/xinetd.d
  ```

##### `/etc/xinetd.d/`
* this contains the configuration files for the network services that `xinetd` controls
```bash
# ls /etc/xinetd.d/
chargen-dgram   daytime-stream  echo-dgram     time-dgram
chargen-stream  discard-dgram   echo-stream    time-stream
daytime-dgram   discard-stream  telnet
```
`telnet` for example 
* you can check this config file with `vim /etc/xinetd.d/telnet`
```bash
# default: on
# description: The telnet server serves telnet sessions; it uses \
#       unencrypted username/password pairs for authentication.
service telnet
{
        flags           = REUSE
        socket_type     = stream
        wait            = no
        user            = root
        server          = /usr/sbin/in.telnetd
        log_on_failure  += USERID
        disable         = yes
}
```
* fyi, telnet is deprecated as it does not encrypt transmissions
* this file will tell us:
  * the name of the service
  * the user this service will run as
  * the binary location for the service
  * if there is an error the user making the connection will be added to the log file
  * the disable option is basically the switch to make the connection to xinetd
    * by default the disble option is set to yes thereby not connection through `xinetd`
    * switch disable to no, to have the service go through `xinetd`
* let's start `xinetd` with the init-script way of doing it:
  * `/etc/init.d/xinetd start`
* and then check which services are being managed by `xinetd`:
  * `chkconfig --list --type xinetd`
  * we will see `telnet:      on` indicating `telnet` is being managed by `xinetd`
* if we check out our ports again with `lsof -i` we will see that `telnet` is listening
  * however, we also now see that the `command` is `xinetd`
* if we try to login, `xinetd` is now standing guard rather than `telnet` directly

##### TCP Wrappers
* this functionality uses a host.allow an/or a hosts.deny file to determine access to network services
* said another way, this uses two configuration files to determine if an incoming request is allowed to access a network service or not
* the first file to get checked is the `/etc/hosts.allow` file
  * this includes a listing of services and what networks or hostnames are allowed access to those services
  * if a request matches an entry in the hosts.allow file, then the client is allowed to connect to the server
  * if not, then the hosts.deny file is checked and if there is a match, it is denied
  * if the incoming request does not match either hosts.allow or hosts.deny, by default, it is granted access 
  * the rules in each of these files are read from top to bottom and as soon as it hits a rule, that is it.  The allow or deny is executed.  It is important to make sure you list your rules properly so as not to short circuit any requests
  * Here is an example of what an entry looks like:
    * on the left you will include the name of a service, then a space followed by a colon and then a network(IP address) or hostname that you are allowing or denying depending on which file you put the entry into
    * eg: `in.telnetd : 192.168.122.1`
  * NOTE: we will need to restart `xinetd` because telnet is now managed by the `xinetd` daemon
    * eg: `service xinetd restart` or `/etc/init.d/xinetd restart`

 ##### `systemd.socket`
* in this config, a systemd socket unit file is used in place of `xinetd` on modern distros
* systemd uses socket based activation for everything?
  * this means that a socket for a network service is not going to be running until a service requests a connection
  * this makes systemd.socket very much like `xinetd` and having both on a system would be redundant
* instead of using `xinetd` config files, under systemd, we would use socket unit files
* for example:
```
$ cat /usr/lib/systemd/system/sshd.socket
[Unit]
Description=OpenSSH Server Socket
Documentation=man:sshd(8) man:sshd_config(5)
Conflicts=sshd.service

[Socket]
ListenStream=22
Accept=yes

[Install]
WantedBy=sockets.target
```
* note that sshd.service confilcts with sshd.socket
* instead of continually listening with sshd.service we can have an on demand listener with sshd.socket
* to achieve this setup you will want to disable sshd.service and enable sshd.socket
  * `systemctl stop sshd.service`
  * `systemctl start sshd.socket` seems that if I run this it will automatically stop the conflicting service
* in addition to this set up, you are able to use host.allow and host.deny file too with socket unit files just like `xinetd`
* Q: how do you make sure that sshd.socket starts on boot?


## 110.3 Securing Data with Encryption
### `GPG`
* GNU Privacy Guard
* GPG is a mechanism that we can use to exchange encrypted files with other users
* Each user needs to have a private and a public key
* These are the steps needed to exchange encrypted files:
1. Generate a pair of keys:
  * For this example, we will create a pair of keys for two users and then we will exchange a file between these two users
  * Be sure that you are fully logged in as your user.  You can not be su or sudoed in.
    * the reason is that the gpg agent will use your full environment to create your keys
    * this will help so that no one tries to fake a key with your user
    * the gpg agent will also use entropy to create your key so the more entropy the better
      * entropy is not on the exam but is good to know about for real world applications.
        * with cryptography relies on entropy
        * entoropy is a random pool of data
        * it is used as a starting point for encryption algorithm
        * the more entropy the better
        * entropy comes from many different things
        * intall rng-tools to give you more entropy on a system
        * `yum -y install rng-tools`
        * `sudo systemctl start rngd.service`
  
`gpg --gen-key`
* this will start the process and generate a new gpg public key
* it will prompt you for:
  * type of key to be created
  * length of the key (longer key is better but takes more time to generate)
  * expiration of they key (a highly secure environment will be much more frequent)
  * Your Real name which will be your pub key on the system
  * your email address
  * a comment
* Press "o" for ok
* enter a passphrase for 2fa
* the gpg public key will be created and you want to save the key name
  * gpg: key 16FECFF5 marked as ultimately trusted
  * eg: "16FECFF5"

2. View the keys that have been created:
  * `gpg --list-keys`
    * we will be able to see the key we just created
  * next move into the .gnupg file to create your public key you will give to another person
  * `gpg -o avancepubkey --export 16FECFF5`
    * the -o means to output a name of the public key
    * the --export will use the private key you just made to make a public key that will connect the `avancepubkey` together
      * it is like makeing a special public key to your private lock?

3. Exchange keys

4. Import Keys into your keyrings
  * `gpg --import avancepubkey`

5. Encrypt files with:
  * `gpg -r "Alyx Vance" -e passwd`
  * you will then see a file `passwd.gpg`
  * this is the encrpyted file and now you can email the encrypted file to avance
Just as a summary for myself:
* first I create my private key?  and it also creates a keyring?
* then I create an individual key that I will give to someone in order for them to decrypt my files individually

#### Revoking a key
#####`gpg -a -o revoce.asc --gen-revoke F60C5E97`
* you will then go through a few prompts and you can provide a reason with an optional comment
* press y and your passphrase for your key.
* then you will need to import the revoked key into your keyring
  * `gpg --import revoke.asc`
* will also need to send anyone my revoke key as well
  * the better way to do this is to upload your revoked key to the MIT keyserver
  * `gpg --keyserver pgp.mit.edu --send-keys E34C2R58`

#### SSH
* this is how to connect via encrypted connections
* it is the sshd daemon that does everything
* located `/etc/ssh/sshd\_config`
  * these are the config files for system wide
* example file and only some important parts:
```
HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_dsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key

# Authentication:

#LoginGraceTime 2m
PermitRootLogin no
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10

X11Forwarding yes
```

* if we take a look at what is in the /etc/ssh/ folder we will see the key files that are used for encryption algorithims
  * they also determine what types of encryption we will accept on our server
* private keys here will only have rw for the root user and read for the root group
* public keys in this dir will be rw for root and read for everyone else 

* individual settings for ssh will be located in the home dir of the user under .ssh
  * individual ssh keys are stored here
  * only the user has full access to this directory or 700 permissions
* what happens when you make a connection to another server?
  * to initate a session you use `ssh $USER@IPADDRESS/HOSTNAME`
  * the user you are attempting to log in with will need to be sure that they have an account on the remote system
  * the first time you attempt to connect you will be presented with the fingerprint of the keys of the ssh server of the computer
    * these fingerprints are taken from those keys in /et/ssh/ssh_config
  * then you enter 'yes' and also enter your password
  * if you log out of the system and then check your local .ssh folder you will now see a file called `known_hosts`
    * each remote system will have it's own line in this file
    * each line will start with the IP address or hostname of the remote system
      * then you will have the ecryption method
      * then you will have the fingerprint from the remote system
    * this is an important file as if someone tampers with the keys on the host and we try to login, we will get a warning
      * this is to protect one from loggin into a compormised host
      * this is similar to those errors of `man in the middle attack` but the IP has just changed
      * which I will run `ssh-keygen -r $IPADDRESS`
  * If we attempt to login again, there will be no prompt regarding the fingerprint because we have already done that stuff and the finger print is saved in our `known_hosts` file
  * We will forever have to enter a password which is a bit tedious
    * we can share keys between my local machine and the remote machine so I do not have to do that
    * the ssh service will use the key files between the two machines for authentication
    * in order to set it up we will need to create a new private and public key pair from the local account?
      * NOTE:  you will need to set up a key pair for each separate account that you want to access the remote system
      * you need individual pairs because the keys are stored in the home directory of a individual user
    * to get the process started, you will want to run `ssh-keygen`
      * by default, this creates a 2048 bit RSA encrypted key pair
      * if you want to customize you can run something like:
        * eg: `ssh-keygen -b 1024 -t DSA`
    * you will then be asked where you want to save your keys
      * you will just want to save them to the default location as there is no real good reason to move the location; unless your company has some cool and different security policy
    * next you will be presented with the option to enter a passphrase for your key
      * while not necessary, it will provide an extra layer of security
      * heck, you could even have a passphrase keyfile, it think, somewheres
    * this completes the creation of our key
    * if we check our ~/.ssh folder, we will now see two new files: id_rsa and id_rsa.pub
      * id_rsa is our private key and we keep that key secret and hidden and do not give it out anywheres
      * id_rsa.pub is our public key and we give this out to the remote servers so that we are able to access them with our private key!
        * if either of keys are tampered with, they will not match up and I will not be able to access the remote servers
      * the next step is to uplad the key to the remote server
        * a great way to do this would be to use the `ssh-copy-id` command
          * this method will create a secure tunnel and copy our pub key and set proper permissions for us
          * eg: `ssh-copy-id kenny@192.168.0.1`
            * you will then log in like normal, ssh-copy-id will copy the key over and then drop you back into your local system
          * once completed you will login and instead of entering in your password for your user, you will enter your passphrase that you configured earlier

##### `ssh-agent`
* you can use `ssh-agent` to make multiple secure connections without entering in your passphrase
* eg: `ssh-agent bash`
* then `ssh-add` to add your passphrase to the `ssh-agent`
* this will only work for the bash shell that is wrapped up in the `ssh-agent`

##### `.ssh/authorized_keys`
* this file indicates to the remote system which shared public keys it will accept connections from
* permissions of `authorized_keys` are 600

##### ssh tunneling with X11
* allows us to use ssh to connect to a remote system using a X11 desktop environment
* this allow us to open up a graphical window from our remote computer on our local computer
* `ssh -X` will allow x-forwarding
* `ssh -x` will not allow x-forwarding
* `ssh -Y` is the most secure way to allow x-forwarding
