# 108: Essential System Services

## 108.1 Maintain System Time
### Working with Remote Time Servers
* We use Network Time Protocol to manage the time between remote servers
  * we need to make sure that our systems are synced up across the globe
  * this is important for things like:
    * communication
    * bank transactions
    * stock trades
    * other important tasks 
  * NTP works by requesting the time from other computers connected to an "official clock"
    * these services send back the time to our local system
    * the local system then adjusts it's clock accordingly
  * The "official clocks" used have different levels
    * __Stratum 0__
      * extremely accurate clocks known as reference clocks
      * they use atomic clocks
      * astro GPS
      * radio clocks
      * Stratum 0 clocks are expensive to maintain; so they should not get queried directly
        * instead, they send their data to servers in Stratum 1
        * this is done so that Stratum 0 clocks just focus on keeping time
    * __Stratum 1__ 
      * These servers take the request from lower level systems
      * This level of servers communicate with each other to make sure they stay in-sync
      * Even though Stratum 1 servers are powerful, they are not the be all and end all for queries
      * to help even the load we have Stratum 2 servers
    * __Stratum 2__
      * Stratum 2 servers synchronize within a group to further keep the time accurate
      * below Stratum 2 are routers, regular computers, mobile devices, smart TV
      * the further down the Stratum you go, the less accurate the time; but not by much
 
 * So how is NTP configured on my local system?
  * ##### `ntpd`
    * this is the daemon that is traditionally used to contact upstream providers
    * `pgrep ntpd` with no output shows we do not have the ntp daemon running
    * can start it with `service ntpd start`; or? `systemctl start ntpd`
    * ntpd works over UDP port 123
    * then use the `date` command to get the time 
    * but if you want an up-to-date time, you use `ntpdate`
    * be sure to stop `ntpd` first though before you use `ntpdate`
      * this prevents any conflicts between the two date/time services 

  * ##### `ntpdate`
    * `ntpdate 1.pool.ntp.org`
      * `1.pool.ntp.org` is a pool of ntp servers that we are querying

  * ##### `/etc/ntp.conf`
    * this is how the `ntpdate` service knows what systems to look into to get date data
    * there are many directives in this file
    * these directive will help you to set up an ntp server
    * main takeaway is:
      * the lines that begin with the term "server"
      ```bash
      server 0.centos.pool.ntp.org iburst
      server 1.centos.pool.ntp.org iburst
      server 2.centos.pool.ntp.org iburst
      server 3.centos.pool.ntp.org iburst
      ```
      * these lines are the pointers to tell the system to look here
        * these can be a single server or a pool of servers
      * tell the ntpdaemon which servers to contact for the correct time
        * note that the `ntpdate` command does not use this file
          * this is why we hand to specify the pool of ntp servers to use 
        * but the ntpdaemon does indeedy do

  * ##### `ntpq`
    * you can use the this command with the ntpd daemon to make queries to NTP servers
      * be sure the ntpd service is running
    *`ntpq -pn`
      * know that ntpq will give you various information about your upstream NTP servers
      * `-p` to show the Peer servers; `-n` will show the ip address
      * also know that we can use it to get data regarding our upstream servers

#### Chronyd Commands
* use `timedatectl` to see if you are using ntp to sync
  * if you see `systemd-timesyncd.service active: no` then it is off
  * I had to install chronyd first on my centos machine be able to enable it
    * `sudo yum -y install chrony`
    * you then invoke the command with `chronyd`
    * start the daemon with `systemctl start chronyd`
* `timedatectl set-ntp on`
  * this will set the systemd service to on
  * if you see `systemd-timesyncd.service active: yes` then it is on
  * or in my case on centos `NTP synchronized: yes` and `NTP enabled: yes`
    * I think that the `NTP enabled means it is installed`
* Systemd systems use chronyd to manage the time syncing with NTP servers
* The configuration files are located here:
  * redhat  distribution => /etc/chrony.conf
  * ubunutu distribution => /etc/chrony/chrony.conf

* ##### `chronyc`
  * this command will drop you into a chrony shell
  * run `activity` to see what sources are online and what sources are offline
  * run `sources` to get more detail on the ntp servers
    * just like the `ntpq` command that checks on upstream communications
  * type `exit` to get out of the prompt
! Remember as you move down the Stratum layer, the time gets less accurate
! Remember that NTP uses the UDP protocol on port 123


## 108.2 System Logging
One of thee most important skills an administrator can have is system logging skills.  Knowing what is going in in your log files will unlock many mysteries of what is happening with your system.  Most every service in your system will create a logfile.

* ### Legacy Logging Systems
  * Common log locations
    * ##### `/var/log/dmesg`
      * Linux kernel boot messages
      * gets recreated every time the system boots
      * contains messages from the kernel regarding hardware and startup services
    * ##### `/var/log/messages`
      * Standard system log messages
      * if a service does not have a logfile, this is where logs get written
      * if a daemon or service is acting out, take a look here to see if you can glean deets
      * this is the core logfile for any Linux system
    * ##### `/var/log/secure`
      * Security log messages; login attempts information
      * if a user cannot log in, monitor this location and see what errors are getting logged
      * you can also monitor login attempts from folks that should not be there
    * ##### `/var/log/maillog`
      * local email log messages to and from the server

  * ##### `rsyslog`
    * on systems that do not use systemd, `rsyslog` is daemon that manages the writting of logs to logfiles

  * The cool thing about log files in Linux is they are formatted in plain text
    * you can use any text viewer to read these files
    * tail is very useful as you will view the latest entries
    * except for the `dmesg` output; you simply invoke `dmesg` to view it

  * #### Logging Priority Levels (MEMORIZE!)
  ```
  Val___Keyword_____Condition_______________________
  0     emerg       system is unusable and need to investigate immedeatly
  1     alert       action must be taken immediately before system goes to emergency and unusable
  2     crit        critical conditions attention is needed with hardware?
  3     err         error conditions that come from a running service or daemon
  4     warn        warning conditions should be investigated but system should function ok
  5     notice      normal but significant enough condition to look at before it gets to warning
  6     info        informational is just they system giving you an FYI
  7     debug       debug-level messages
  ```
  * `dmesg`
    * `dmesg --level=err,warn | less` will give us just the err and warn level messages
    * `dmesg -x --level=err,warn | less` will give us the facility and priority

  * #### Linux Logging Facilities
  ```
  Facility___Description_____________________________
  kern       Kernel Messages
  user       Random user-level messages
  mail       Mail system
  daemon     System daemon 
  auth       Security/authorization messages
  syslog     Messages generated internally by syslogd
  lpr        Line printer subsystem
  news       Network news subsystem
  ```

  * A facility is something that is generating the particular log message
    * the main facility is the kernel
    * the user is a system user or a user user by the use of an application
    * email deals with email services installed on the system
    * daemon are messages generated by system services
    * the auth facility deals with anything security and authentication mechanisms
    * syslog is anything that is generated by the logging daemon
    * lpr logs printing data
    * news service using the etntp protocol?
  

  * `/var/log/messages`
    * this can contain messages from other host systems
    * you can configure logs to send their data to a different host
    * Output of: `tail /var/log/messages
    ```bash
    Nov 22 14:57:03 pydev dbus[597]: [system] Successfully activated service 'org.freedesktop.nm_dispatcher'
    Nov 22 14:57:03 pydev systemd: Started Network Manager Script Dispatcher Service.
    Nov 22 14:57:03 pydev nm-dispatcher: req:1 'dhcp4-change' [eth0]: new request (4 scripts)
    Nov 22 14:57:03 pydev nm-dispatcher: req:1 'dhcp4-change' [eth0]: start running ordered scripts...
    Nov 22 15:00:01 pydev systemd: Created slice User Slice of root.
    Nov 22 15:00:01 pydev systemd: Started Session 2633 of user root.
    Nov 22 15:00:01 pydev systemd: Removed slice User Slice of root.
    Nov 22 15:01:01 pydev systemd: Created slice User Slice of root.
    Nov 22 15:01:01 pydev systemd: Started Session 2634 of user root.
    Nov 22 15:01:01 pydev systemd: Removed slice User Slice of root.
    ```
      * the fields are as follows
        * Date&TimeStamp 
        * Hostname for system that generated the log message (logs can come from different hosts)
        * Item that generated the log
          * if you see brackets with a number, that is the PID
        * last is the message of the event

  * `logger`
    * you can use this utility to send messages to `/var/log/messages`
    * logger "hello world" will send the string with a time stamp, hostname and username
    * the most practical use for `logger` is through scripts and automation and the like
    * you can specify tags with `logger` to sort through your data easier
      * `logger -t "from-script" "Just a test"
      * then if you check out the `/var/log/messages` you will see
        * `Nov 11 10:10:38 centos70 from-script: Just a test`
        * so this will send the string "Just a test" to `/var/log/messages`
        * it will also add a tag so that if you do grep search with your tag...

  * `/var/log/secure`
    * this is also an important log file as it logs good and bad signins

* ### Rsyslog
  * Here is where we explore how the rsyslog service is configured.  We will also find out how to send logs from one server and to be sent to another.  We also look at how to modify log retention.
  * Remember `rsyslog` is only for older linux systems that do not use `systemd`
  * ##### `/etc/rsyslog.conf`
    * the config file for `rsyslog` is located here
    * this file is very self explanatory
    * there are even html files you can view for more documentation
    * you can also view `man rsyslog.conf` for more info
      * review the BASIC STRUCTURE of the man file
    * the `#### RULES ####` section of `rsyslog.conf` will dictate which log messages go where
      * `facility.priority` is the naming convention
        * (.) dot is actually the selector
        * so it will look like `kern.warn` or `kern.*`
          * `kern.*` means to log all log priorities
    * there are also config files under `/etc/rsyslog.d`
      * this is a great space to have custom logging configs
    * NOTE: `rsyslog` daemon needs to restart for changes made to `rsyslog.d` and `rsyslog.conf`
  * You can also configure rsyslog to receive log messages from one system to another
    * from within the default config files located at `/etc/rsyslog.conf` you will want to comment out the proper rules
      * on the receiving machine you want to allow connections
      * on the sending machine you will want to send all messages

    * ##### `/etc/logrotate.conf`
      * storage management of your logs is very important
      * lets start by taking a look at the /var/log directory
      ```bash
      $ ls /var/log
      anaconda           dmesg                  maillog-20191110   secure-20191117
      audit              dmesg.old              maillog-20191117   spooler
      boot.log           fail2ban.log           messages           spooler-20191027
      boot.log-20190823  fail2ban.log-20191027  messages-20191027  spooler-20191103
      boot.log-20190912  fail2ban.log-20191103  messages-20191103  spooler-20191110
      boot.log-20191025  fail2ban.log-20191110  messages-20191110  spooler-20191117
      boot.log-20191107  fail2ban.log-20191117  messages-20191117  tallylog
      btmp               firewalld              ntpstats           tuned
      btmp-20191101      grubby                 qemu-ga            wtmp
      chrony             grubby_prune_debug     rhsm               Xorg.0.log
      cron               httpd                  sa                 Xorg.0.log.old
      cron-20191027      lastlog                secure             yum.log
      cron-20191103      maillog                secure-20191027    yum.log-20191027
      cron-20191110      maillog-20191027       secure-20191103
      cron-20191117      maillog-20191103       secure-20191110
      ```
      * do you notice those folders with what looks to be a date behind them?
      * well, they are dates and those are the dates with which the log was rotated out and the new one was started
      * this helps you to find information from an event that has happened in the past
      * the logrotate daemon handles this funtionality and it is configured at `/etc/logrotate.conf` 
      ##### Example of `logrotate.conf` file
      ```bash
      # see "man logrotate" for details
      # rotate log files weekly
      weekly

      # keep 4 weeks worth of backlogs
      rotate 4

      # create new (empty) log files after rotating old ones
      create

      # use date as a suffix of the rotated file
      dateext

      # uncomment this if you want your log files compressed
      #compress

      # RPM packages drop log rotation information into this directory
      include /etc/logrotate.d

      # no packages own wtmp and btmp -- we'll rotate them here
      /var/log/wtmp {
          monthly
          create 0664 root utmp
              minsize 1M
          rotate 1
      }

      /var/log/btmp {
          missingok
          monthly
          create 0600 root utmp
          rotate 1
      }

      # system-specific logs may be also be configured here.
      ```
      * it is prety self explanitory
      * made these changes to the file:
      ```bash
      # use date as a suffix of the rotated file
      dateext

      # modify the way the date looks
      dateformat _%Y-%m-%d

      # uncomment this if you want your log files compressed
      compress
      ```
      * now if i run `logrotate /etc/logrotate.conf` this will trigger the logrotate daemon to run
      * however, since we did not have a minium of 1M it did not run
      * we can force is to run with the *force* flag `-f`
      * `logrotate -f /etc/logrotate.conf` which will give me the output below:
      ```bash
      [root@li1244-43 ~]# service rsyslog restart
      Shutting down system logger:                               [  OK  ]
      Starting system logger:                                    [  OK  ]
      [root@li1244-43 ~]# logrotate /etc/logrotate.conf
      [root@li1244-43 ~]# ls /var/log/
      anaconda.ifcfg.log    anaconda.storage.log  anaconda.yum.log  btmp   dmesg.old   maillog   secure   tallylog
      anaconda.log          anaconda.syslog       audit             cron   dracut.log  messages  spooler  wtmp
      anaconda.program.log  anaconda.xlog         boot.log          dmesg  lastlog     sa        syslog   yum.log
      [root@li1244-43 ~]# logrotate /etc/logrotate.conf  -f
      [root@li1244-43 ~]# ls /var/log/
      anaconda.ifcfg.log    audit               dmesg.old               sa                     wtmp
      anaconda.log          boot.log            dracut.log              secure                 wtmp_2019-11-23.gz
      anaconda.program.log  btmp                lastlog                 secure_2019-11-23.gz   yum.log
      anaconda.storage.log  btmp_2019-11-23.gz  maillog                 spooler
      anaconda.syslog       cron                maillog_2019-11-23.gz   spooler_2019-11-23.gz
      anaconda.xlog         cron_2019-11-23.gz  messages                syslog
      anaconda.yum.log      dmesg               messages_2019-11-23.gz  tallylog
      ```
      * note the date format and the gzip compression

    * ##### `/etc/logrotate.d`
      * each file in this directory is a logrotate configuration package created by that package's installer

* ### Introduction to the systemd Journal
* systemd Journal
  * this is a binary file that records everything that goes on in a computer
  * what do we mean by everything?
    * kernel log messages / dmesg
    * system log messages the same as what syslog would collect
    * system services that send output to STDOUT and STDERR
    * audit records for SELinux messages (RHEL / Centos)
  * the default location is /run/log/journal/
    * this info is lost on reboot as the journal collects so much data
    * you can keep the journal permanent by running the below commands
      ```bash
      mkdir -p /var/log/journal
      systemd-tmpfiles --create --prefix /var/log/journal
      ```
    * you can control the size of the journal in the journal config file
      * `/etc/systemd/journald.conf` is the location of the journal config file
      * `man 5journald.conf`
    * [Journal]
      * Storage=
        * this setting will determine whether to store the log on the disk or in memory
        * storing on disk will be persistent; storing in ram is temporary/non-persistent
        * auto - the default setting
          * this will store data to `/var/log/journal` (if it exists)
          * will also store to `/run/log/journal`
        * persistent - data is stored to `/var/log/journal` hierarchy
        * volatile - data is only sent to `/run/log/journal` and resides in memory
        * none - no data is kept; all logs are dropped
      * Compress=
        * takes a boolean value
        * yes - is the default; data is compressed before written to disk
        * no - no data objects are compressed
      * SystemMaxUse=, RuntimeMaxUse=
        * both of these relate to the amount of space that the journal will occupy
        * SystemMaxUse pertains to the physical disk space the journal can use; the default is 10%
        * RuntimeMaxUse pertains to the amount of RAM the journal can use; the default is 10%

      * SystemMaxFileSize=, RuntimeMaxFileSize=
        * these act like a quota setting for how large individual journal files can get
        * NOTE: this does not apply to the journal directory
          * this applies to each file on the system?
          * can specify the size from kilobytes to exabytes
      * MaxRetentionSec=
        * the maximum amount of time to store journal entries
        * typically you will only really need to set the file size parameters (eg: SystemMaxUse)
        * the default is 0 (off)
        * it is set by a number and an unit like years months weeks days h or m
        * the default is s seconds

* ### `journalctl`
  * ##### `journalctl`
    * this command will show us the systemd journal
    * the default `journalctl` will show us the oldest data at the top of the output
      * it looks like it uses the less pager
    * `-r` will show us the newest entries first
    * `-e` will jump to the end of the page upon load
    * `-n 30` will show the most recent entries
    * `-f` will allow us to follow the entry
    * `-u` will allow us to see a service
      * eg: `journalctl -u http.service`
      * you can combine -f and -u to follow along with a particular unit or service
        * eg `journalctl -fu sshd`
        * this will follow ssh logs as they are written
    * `-o verbose` will show us the full structure of the log files
      * technically the log file is not a flat text file but a database
      * this shows you all the database stuff
      * `-o json-pretty` will put this all into a json format to use for database analysis
    * `man 7 systemd.journal-fields` to find out what all the fields mean in the output
    * `systemd-cat` will allow you to add your own text/notes to the journal
      * eg: `echo "Hello, you!" | systemd-cat`
      * then when you run `journalctl -r` you will see the echo string right there!
    * `-x` means to give some extra explanation about stuff
      * this uses catalog service entries to match up events with documentation
    * `-k` means to just show us the kernel ring buffer
    * `-b` sow all of the journal entries that have been collected since the most recent system boot up
    * `--list-boots` will show a listing of recorded boot sessions
      * the journal must be persistent to the disk for this to work
      * this is useful for security audits
    * `--since` and `--until` (must be able to be written to disk)
    * `--disk-usage`
    * `--rotate`

## 108.3 Mail Transfer Agent (MTA) Basics
* ### Basics of a Message Transfer Agent
  * What is an MTA?  It is a program(service) that routes email to its intended destination
  * the MTA will send the message to a Message Delivery Agent (MDA) via port 25
  * then the MDA will send the mail to the Message User Agent (Email Client), when it makes a request for new mail
  * MTA Software Offerings
    * sendmail - is one of the oldest MTA systems around
      * this used to be the default on many Linux distros
      * however it is notoriously difficult to configure
    * postfix - a more modern offering which is simpler to configure
      * this also has more robust security than sendmail
    * exim - used to be the default for debian systems
      * good security and easier to configure than sendmail
    * sendmail emulation layer - a command wrapper
      * allows you to use sendmail commands but you have postfix or exim on the back end

* ### Email Forwarding and Aliases
  * learn how to setup local email forward to send email
  * `/etc/aliases` is the file you will want to work with in order to get email forwarding to work
    * the aliases on the left are users on the system (both system users and user users)
    * then you have a colon
    * then you have the user that will receive that email
      * eg: `bin: root`  or `root:  nygel` <= this will send roots email to the user nygel 
    * so the account on the right will received messages from the accounts on the left
  * `newaliases`
    * be sure to run this command after modifications to the /etc/aliases file is made
    * this command will then regenerate the `/etc/aliases.db` file
      * `/etc/aliases.db` is the file that the MTA uses for mail delivery
  * `mail`
    * use this command so that you can see if you have any mail
    * `-s` will be what you want to use to set the subject
      * `mail -s "Subject: Testing" root@localhost`
        * this will setup an email to the root user with the subject of "Subject: Testing"
      * after you run this command it will drop you into a prompt where you can type the body of the message
      * press <ctrl-d> when you are finished with crafting your email
    * if you 
    * enter the number of the email to view it
    * enter the 'd' key to delete the email
    * enter the 'q' key to quit the `mail` program 

  * if you send an email to a user that does not exist you will get an error message
    * you can see this error message in the `/var/log/maillog` file
    * each email message has a message ID
    * you will see that the status is bounced and unknown user
    * the next status is that the message ID, therefore the message, has been removed

  * ##### `mailq`
    * sometimes and email can get stuck and you will want to view the queue
    * you can simulate this by shutting down the MTA with:
      * `systemctl stop postfix`
    * send an email with `mail -s "Stopped Service" nygel@localhost`
      * enter the body of the email 
      * press <ctrl-d> to send
    * enter `mailq` and that will show us the queue for emails to be sent
      * we will see the queue id, msg size, arrival time in queue, and the sender and recipient

   * ##### `~/.forward` file
    * this is a user directory file that a user can use to forward emails to their personal email accounts
    * simply add your external email address to the file
    * any system emails that get sent to the user will be forwarded to the email listed in the `~/.forward` file



## 108.4 Manage Printers and Printing
### The Common Unix Printing System (CUPS)

What does the interface consist of?
* set up a virtual printer to send stuff to a pdf document
* install pdf driver along with cups service
  * `sudo apt-get install cupst printer-driver-cups.pdf`

How do we add and remove printers?
  * you can use a web interface on a desktop computer to set up cups
  * visit `http://localhost:631`
  * remember that localhost is a default network name for my computer
    * we are checking in to port 631 on our local network to configure CUPS
    * CUPS default network port is 631
  * most commonly used printer protocols:
    * IPP and IPPS
      * internet printing protocol (unencrypted)
      * internet printing protocol secure (encrypted)
    * HP jetdirect


How do we use the log files?
  * just like most any other log files

How can we configure CUPS?
  * `/etc/cups`
  * through the web interface you can access the above file
  * make changes and save those changes
  * Two important files to know are:
    * `cupsd.conf`
      * contains configs for the printer server
      * this is the same file we saw in the web interface
      * if you modify this file you will also need to restart the service
      * the web interface will handle the restarting for you
    * `printers.conf`
      * contains configs for individual printers that are installed
      * do not make changes to this file while the service is running
      * it is better to use the web interface here as well
      * can view `man printers.conf` for more info

### The Line Print Daemon (`lpd`)
* The line print daemon is a legacy tool that would manage print jobs and print documents from the command line.
* printing outputs from shell commands and also printing reports is useful
* cups provides backwards capabilities to use `lpd`

  * ##### `lpstat`
    * this will show us a (-s)ummary of our printers configured on the system
    * this will also show us a (-l)ong listing of the status of individual printers
      * this will give you the print job number of a print job
      * you can use this number to remove a print job with the use of the `lprm` command

  * ##### `lpadmin`
    * you can use this to install printers to your system
    * `-p` will allow you to name the printer to install
    * `-L` will allow you to name the location of the printer
    * `-v` this is where we describe the connection information
      * we will need to provide the uniform resource identifier or (URI)
      * it is like a url but it is for devices on a network?
        * it is much more complicated
    * `-m` will specify the device driver to use
      * if you do not have one, you can use "everywhere"
      * this will search for a device driver that you can use for the printer
    * eg: `lpadmin -p ENVY 4510 -L "downstairs office" -v socket://192.168.0.8:9100
      * hp printers use port 9100; commit this to memory
    * to go back and add the proper driver for the printer do a search for the drivers:
      * this will search the local cups database for a postscript printer description (ppd)
      * eg: `lpinfo --make-and-model "HP Envy 4510" -m`
      * the command says "for the make and model of <printer> show me all the drivers (-m)"
    * once we locate a suitable driver (look for cups specific ones); you can assign it
      * you use the `lpadmin -p <specify printer> -m <ppd location>` to assign the ppd file
      * eg: `lpdadmin -p ENVY-4512 -m "drv:///hpcups.drv/hp-envy_4510_series.ppd" -E`
        * the `-E` option allows us to enable the printer
    * `-x` will remove a printer from your system
      * eg: `lpadmin -x Deskjet-2040`
      then run `lpstat -s` and you will see that the Deskjet-2040 is no longer listed


  * ##### `lpinfo`
    * will show us the available printer devices and drivers that can be used
    * `-v` will show us the available types of printer connections we can make 

  * ##### `lpc` 
    * `lpc status` will show us all our printers on the system

  * ##### `lpr`
    * this will print documents from the command line
    * `lpr /etc/passwd` <= this will print to the default printer
    * `lpr -p ENVY-4512 /etc/passwd` <= this specifies the printer to print to
    
  * ##### `lpq`
    * this is to view the print queue of your printer
    * you need to specify the `-a` to see all of the system's printers

  * ##### `lprm`
    * using this along with the print job ID number; you can remove a print job
    * `lprm 22`
    * `lpq -a` will no longer have any entries since the print queue is clear

  * ##### `cupsreject`
    * no new jobs will be able to be sent to a printer
    * eg: `cupsreject ENVY-4512`

  * ##### `cupsdiable`
    * disable printing on a printer from a server
    * eg: `cupsdiable ENVY-4512`
