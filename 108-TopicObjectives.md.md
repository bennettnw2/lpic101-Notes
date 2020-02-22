# Topic 108: Essential System Services
## 108.1 Maintain system time
##### Weight: 3

##### Description: Candidates should be able to properly maintain the system time and synchronize the clock via NTP.

#### Key Knowledge Areas:

Set the system date and time.
Set the hardware clock to the correct time in UTC.
Configure the correct timezone.
Basic NTP configuration using ntpd and chrony.
Knowledge of using the pool.ntp.org service.
Awareness of the ntpq command.
The following is a partial list of the used files, terms and utilities:

/usr/share/zoneinfo/
/etc/timezone
/etc/localtime
/etc/ntp.conf
/etc/chrony.conf
date
hwclock
timedatectl
ntpd
ntpdate
chronyc
pool.ntp.org


## 108.2 System logging
##### Weight: 4

##### Description: Candidates should be able to configure rsyslog. This objective also includes configuring the logging daemon to send log output to a central log server or accept log output as a central log server. Use of the systemd journal subsystem is covered. Also, awareness of syslog and syslog-ng as alternative logging systems is included.

#### Key Knowledge Areas:

Basic configuration of rsyslog.
Understanding of standard facilities, priorities and actions.
Query the systemd journal.
Filter systemd journal data by criteria such as date, service or priority.
Configure persistent systemd journal storage and journal size.
Delete old systemd journal data.
Retrieve systemd journal data from a rescue system or file system copy.
Understand interaction of rsyslog with systemd-journald.
Configuration of logrotate.
Awareness of syslog and syslog-ng.
Terms and Utilities:

/etc/rsyslog.conf
/var/log/
logger
logrotate
/etc/logrotate.conf
/etc/logrotate.d/
journalctl
systemd-cat
/etc/systemd/journald.conf
/var/log/journal/


## 108.3 Mail Transfer Agent (MTA) basics
##### Weight: 3

##### Description: Candidates should be aware of the commonly available MTA programs and be able to perform basic forward and alias configuration on a client host. Other configuration files are not covered.

#### Key Knowledge Areas:

Create e-mail aliases.
Configure e-mail forwarding.
Knowledge of commonly available MTA programs (postfix, sendmail, exim) (no configuration).
Terms and Utilities:

~/.forward
sendmail emulation layer commands
newaliases
mail
mailq
postfix
sendmail
exim

## 108.4 Manage Printers and Printing
* Basic CUPS configuration for (local and remote printers)
* Manage user print queues
* Troubleshoot general printing problems
* Add and remove jobs from configured printer queues

#### Basic CUPS configuration for local and remote printers
The CUPS config files are located in `/etc/cups`  You can also configure the the sevice from the web interface @ `http://localhost:631`.  The most important config files to know are cupsd.conf and printers.conf.

##### `cupsd.conf`
* this contains configs for the printer server

##### `printers.conf`
* this contains configs for the individual printers

#### Manage User print queues
* You can use either `lpstat` or `lpq` to view print queues
* `lprm` will remove a print job by the print number
* `cupsreject` will reject jobs that are sent to the printer and not be able to wait in the queue
* `cupsdiable` will disable the printer and jobs will sit and wait in the queue

#### Troubleshoot general printing problems
* Error logs are located in /var/log/cups
  * `access_log` and `error_log` are the important ones for figuring out troubleshooting
* Config files are located in /etc/cups
  * `cupsd.conf` - service config
  * `printers.conf` - printers config

#### Add and remove jobs from configured printer queues
**===CUPS (Common Unix Printing Service)===**

**===LPD (Line Print Daemon) Legacy Commands===**
You will use the `lpstat` command to view the print queues with the CUPS interface running.
* `-a` will display the acceptance status of printers
* `-p` will display the print status as either `enabled` or `disabled`
* `-s` will display a summary of printers and their devices(config files)

`lpadmin` is what you will use to add, modify and delete printers
* eg: `lpadmin -p EPSON-610 -L "Kitchen" -v socket://192.168.0.3:9100 -m everywhere`
  * `-p` gives the printer a name
  * `-L` gives the printer a location
  * `-v` is the network socket is going to connect to
  * `-m` is the driver you want to use; we specified "everwhere" as a default until we can find a better fit
  * `-E` means to enable the `-p` printer
    * `lpadmin -p EPSON-610 -E

You use `lpinfo -v` to see all the different types of connections you can make
* `lpinfo --make-and-model "$MAKEnMODEL" -m` will give us the various ppd files we can use to make our printers more functional.


To see all the printers configured on your system, you will want to run `lpc status` 

To start a print job you run, `lpr` (line print run).  By default, if no printer is specified, it will print to the default printer.  In order to print to a specific printer, you simply add the `-P` flag and then specify the printer name.
eg: `lpr -P EPSON-WF610 /etc/passwd`

You can use the command `lpq` (line print queue) to view the print queue of the default printer.  If you want to see all of the system's printers you can just pass the `-a` flag to see them *A*ll.

To remove a print job you use, `lprm` (line print remove) and you pass along the job ID number
