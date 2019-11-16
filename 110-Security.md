# Security

## 110.1 Perform Security Administration Tasks
  * ### Determine the Current State of a System
    * ##### `who`
      * this will list the users currently logged into a system
      * it also displays what terminal they are using and what time they logged in
      ```
      root tty1     2019-10-01 08:30
      ```

    * ##### `w` 
      * this will also list the currently logged users
      * this also contains more details
        * how long idle
        * how much CPU they are using
        * what command they are running

    * It is always a good idea to know who logs into the system and what their typical "Descriptor" looks like
    * This way, you will have an idea of what a normal day looks like so that you can detect any anomalies

    * ##### `last`
      * this command shows a listing of users who were logged into the system but are now logged out
      * you can find what users failed their login attempts with: `last -f /var/log/btmp`

    * ##### `lsof`
      * list of open files command
      * you can use this command to see what files are open right now and whom is opening them
      * best to pipe through less
      * Command | PID | USER | File Descriptor | Type (Dir or File) | Node Name (process file)
      * since network ports and sockets are considered files, you can see what sockets or ports are open
      * `lsof -u henry | less` this will show us what user has files open
      * `lsof /home/henry | less` this will show us what items are open in the directory listed

    * It is important to know what files have the suid and sgid bit set on them;
      * you can find these files with a `find` command
      * eg: `find / -perm -u+s` <= set user id bit 
      * it is good to keep an eye on these types of files to make sure no one has tampered with them
      * can also find by looking for the set octal
        * eg: `find / -perm -04000` <= set octal?
        * `find / -perm -g+s` <= set group id bit

    * It is also good to keep track of resources
    * ##### `ulimit` is the command to use to set limits on resources a user can utilize
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
          * `cpu`
          * set in minutes but not concurrent minutes but cumulative on a process
          * can set hard or soft limits
        * concurrent logins
          * `maxlogins`
        * amount of memory a user process can use.
          * `memlock`

      * `passwd` 107.1

    * Granting Elevated Privileges
      * #### `/etc/sudoers`
        * this is the default config file for configuring user and or groups that get elevated privileges
        * use visudo to edit this file
      * #### `sudo`
        * this command allow a user to run a command as root
        * basically they turn into the root user for that command
      * #### `su`
        * this is the "substitute user" command
        * you can use a - and then the username
        * if you don't use a dash or the field after the dash is blank; the root user is assumed


  * ### Checking Local Network Security
    * Since Linux is a network operating system any Linux admin needs to be proactive with network security
    * ##### `lsof` (revisited)
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

    * ##### `fuser`
      * this command can be used to list all the PIDs that are assigned to a particular file or network port that is in use
      * said another way, this is what you look at if you want the PID of a connection
      * `fuser` in combination with `lsof` you can find errant and malicious connections to your system

    * ##### `netstat`
      * this allows you to view active connections
      * `netstat -tunalp`

    * ##### `nmap`
      * this allows you to view ports from the outside in
   
## 110.2 Set Up Host Security
  * ### Securing Local Logins
    * Making sure that local users are protected as well
    * We will look at a command to use when an account is disabled
    * also review info about our user config files

  * assume a user went away on vacation; we want to lock the account
    * you can use the `usermod` command to lock the user's password
      * example needed
    * we can also lock the account with the usermod command by setting the expired date to the number 1
    * use the `getent` command to check out the shadow database file for the changes we just made
    * you should see an exclamation point at the beginning of the entry showing that the password has been locked
    * near the end you will see an expiration date of 1 which effectively disables the account
    * The account still has access to a login shell some how
      * we need to change the login shell to a no-login shell
      * example needed
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
    * #### `/etc/passwd`
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
          `ls -l /bin/passwd` => 
          `-rwsr-xr-x. 1 root root 27832 Jun 10  2014 /bin/passwd`
          ```
          * we can see that the owner has write access and the set user id bit is set
          * as the set user id is set, when you run the command, you magically run it as root
        * the root user can change anyone's password
        * the permissions on these two files should never be changed
        * you should be aware of these default permissions for the exam
        * sometimes the default permissions can be different from distro to distro
        * it is good to know that the default permissions 

  * ### Securing Network Services
    * all about securing our network ports
    * firstly turn off and disable any network services you are not going to be using on your system  
    * use `lsof -i` to show all services and ports that are being used
      * we want to concern ourselves with the ones that have a '*' next to a ':'
      * this means they are listing for connections coming in from anywhere
      * ones that say "localhost" are listening internally for local services like smtp email
    * then any that you are not going to use, just shut the service off and disable starting on boot
      * `service cups stop`
      * `chkconfig cups off`

    * ##### `xinetd`
      * a super-server sounds like it is a catch all listening service;
      * said another way, it would provide access control to network services
      * a request would come in for a service that is shut off
        * the super-server would do some security checks
        * if the checks came back clear, the super-server would start the service to respond to the request
      * `xinetd` was the best implementation of a super-server.
      * `xinetd` is found on older systems; Centos 6 for example
      * the reason is because of systemd
        * systemd uses socket unit files to perform the same tasks as `xinetd`

      * ##### `/etc/xinetd.conf`
        * the config file for xinetd is `/etc/xinetd.conf
        * there are logging settings for `xinetd`
          * each service that `xinetd` covers will get its own log file
        * we can restrict access to certain services
          * `cps` means connections per second
            * eg: `cps      = 50 10`
            * means, if more than 50 connections per second are requested of the service, disable the service for 10 seconds
          * `instances = 50` means we can have up to 50 instances of a particular service
          * `per_source = 10` means no more than 10 incoming source IP addresses
          * we also see that the file `/etc/xinetd.d` is referenced and included into this config file

      * ##### `/etc/xinetd.d/`
        * this contains the configuration files for the network services that `xinetd` controls
        * `telnet` for example 
          * you can check this config file with `vim /etc/xinetd.d/telnet`
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

      * ##### TCP Wrappers
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

     * ##### `systemd.socket`
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
  * ### `GPG`
    * GNU Privacy Guard
    * GPG is a mechanism that we can use to exchange encrypted files with other users
    * Each user needs to have a private and a public key
    * These are the steps needed to exchange encrypted files:
    1. Generate a pair of keys:
      * `gpg --gen-key`
        * this will start the process and generate a new gpg public key
        * you need to be logged in at that particular user
          * you should not be sudo or su
          * when generating a key, it takes account of your user environment
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


  * ### SSH



