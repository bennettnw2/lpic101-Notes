# 107: Administrative Tasks

## 107.1 Manage User and Group Accounts and Related System Files

### Adding and Removing Users
We are going to learn how to create new users for our system.
* Linux is a multi user operating system
* Each user can be logged in at the same time using a slice of the system's resources
* We should learn the basics of user management which includes creating and removing user accounts
* You will always need to be root to manage these types of settings

* ##### `useradd`
  * this command is for the creation of new user account on a Linux system
  * `-m` will create a home directory
  * `-c` will add a comment to the username
    * eg: `-c "Gordon Freeman"
    * Q: will this be available via the /etc/passwd file?
  * `-s` will set the default shell for that user
    * eg: `-s /bin/zsh` <= need to specify shell name and location
  * eg: `useradd -m nbennett`
    * this will create a user named "nbennett"
    * and will create a home directory in `/home/nbennett`
  * this account will not be able to log in as they do not have a password just yet

* ##### `passwd`
  * this command will set a password for a specific user
  * this command can also be used by a user to change their password
  * `-e` will force expire a password so that it will be prompted to change on the next use
    * note this command/option combo must be run as root and it will look like:
    * eg: `passwd -e nbennett`
    * again, this will force nbennett to change their password once they login again

* ##### `userdel`
  * this command will remove the user but will keep their home folder
  * eg: `userdel nbennett`
    * note: if you do a long listing of their home dir, their user and groupname will be replaced by their UID and GID numbers
  * `-r` will remove their home directory too
  * eg: `userdel -r nbennett`

### Adding and Removing Groups
* Groups are collections of user accounts
* We use groups to dictate the set of permissions a group of users will have for a particular resource
  * Resources like what?
    * directories
    * files
    * or services

* ##### `groups`
  * this command will show a user what their primary and secondary groups that they are a member of
  * Different groups you can be in:
    * same as your username
    * users
    * wheel - sudo group in Centos
    * sudoers?

* ##### `groupadd`
  * this command will create a new group on the system
  * you will need to be root/have elevated privileges in order to add groups to the system
  * eg: `groupadd curators`
  * you can add a user to a group in two ways
    * when a user is created they will get added to a default, primary group
    * you can add a secondary group while creating a user with `-G`
    * eg: `useradd -G curators`
      * you will need to be sure that the group already exists 
    * Q: what is the other way?

* ##### `groupdel`
  * this will remove a group from the system
  * the users will still exist, they will just each have one less group
  * eg: `groupdel curators`

  

### User and Group Configuration Files
We will take a look at what happens at the system level when managing users and groups and also the configuration files that help with the managing of users and groups.
* Where is this data stored for users and groups?
  * ##### `/etc/passwd`
    * contains all of the account information on the system
    * root account; system service accounts; and user accounts
    * the only user that can modify `etc/passwd` is the root account
    * other users do need read access to the file to be able to log into the system 
      * NOTE: the permissions are 644 for `/etc/passwd`
  * what is contained in `etc/passwd`?
    * each line contains information about one system account
    * : (colons) are used to separate each field in a file
    * username : password : userid : primary groupid : comment `-c` goes here : full path to home dir : default shell environment
    * "nologin" shell means that it is a system account and does not need to login
    * root UID is always 0 and all system accounts have a UID below 1000
    * regular user accounts have a UID of 1000 and above
    * the password column will have an "x" in that spot which references /etc/shadow

  * ##### `/etc/shadow`
    * the layout of the etc shadow file is very similar to that of the `/etc/passwd`
    * the first field contains the username and the second field contains an encrypted password
    * the password will show the algorithm used, the salt used, and the hashed password
      * these values are all separated by a $ symbol
    * the next field shows how many days since the epoch that the password was changed
      * UNIX Epoch is January 1, 1970
      * this time is the standard used to calculate modification times
    * the next field indicates the minimum number of days between password changes
      * it shows how many days are remaining until you need to change your password
    * the next field shows the maximum number of days a password is valid before a user is required to change it
      * if you have 99999, that means about 273 years soooo, after the life of the server
    * the next field after that shows the number of days a user is warned that their password needs to be changed
    * the next is an indicator letting us know if a password is inactive
    * lastly is the date of when the account will expire
      * it is in the number of days since the epoch
    * system service accounts will typically have either an asterisk or single or double pops in the password field
    * double pops means that an account is locked
    * asterisks means the account is locked and never had a password

  * ##### `/etc/group`
    * This file contains group definitions along with what member belong to each group
    * This file only has four fields
      * the first is the name of the group
      * the second is the password field
        * groups do not have passwords as we rely on the user's passwords
      * the third field is the group ID number
      * there is a fourth field that will list the members of that group
        * if is is blank then the user listed in the only member of the group with the same name
        * if there are multiple users, they will be separated by a comma 

  * ##### `/etc/skel`
    * add files and folders to this directory so that when a new user gets created on a system, they get the same default set of folders and files each time
    * files like default `.bash_profile`, `.bashrc`, etc.
    * default shared folders they need access to

  * ##### `/etc/default/useradd`
    * this file is referenced when the useradd command is ran
    ```
    GROUP=100
    HOME=/home            -home folder location parent dir
    INACTIVE=-1           -account is not disabled when password expires
    EXPIRE=               -date when account will expire; for a temporary account
    SHELL=/bin/bash       -default shell a new user will be assigned
    SKEL=/etc/skel        -location of the skeleton directory
    CREATE_MAIL_SPOOL=yes -this will create a default mail directory
    ```
    * why is the GROUP value 100?
      * because that is the old school default
      * since more system accounts have been created we need more reserved numbers
      * this is why modern systems give UIDs starting in the 1000s
      * we can use `getent`
      * ##### `getent` is used to query a database for information about a user or a group
        * `getent passwd kenny`
        * `getent group 100` => `users:x:100:`
    * So how does `useradd` assign a UID of 1000+? 
      * can view the `/etc/login.defs` to view default configs for user creation with useradd
      * the settings in this file overrides the default behaviour of `/etc/default/useradd`
      * this is where you would make changes for the default configs for user creation

* What does this data do?

* How can I use this data to make my job easier?

### User and Group Modifications
In this section we will work with local user and group account by modifying their properties and what these modifications do to the config files we learned about previously.

#### Account Modification Commands:

* ##### `usermod`
  * this command is used to modify an existing user accounts settings
  * settings like default shell, UID, home directory
  * what is great about `usermod` is that is has mostly the same switches as `useradd`
  * eg: `usermod -s /bin/tcsh bcalhoun` will change the default shell of the user "bcalhoun"
  * you can then use `getent passwd bcalhoun` to view his line entry in `/etd/passwd`
    * this will confirm the change for you
  * eg: `usermod -aG engineering bcalhoun` 
    * this will -append the -Group "engineers" to bcalhoun's secondary groups
    * if you do not use the -a switch, the action would be an overwrite of the secondary groups
    * remember: with `useradd` and `usermod`, -g mean primary group and -G means secondary group
    * use the `groups bcalhoun` command to verify the addition of the group worked
      * => `bcalhoun : bcalhoun curators engineering`
    * can use the `getent group engineering` command to view everyone in a group
      * => `engineering:x:1004:bcalhoun,avance,nbennett`
  * `usermod -L bcalhoun` will lock the account while the person is away or something
    * can check the status with `getent shadow bcalhoun`
    * you will see that the encrypted password has a pop in front of it (!)
    * this indicated that the encrypted password is locked from usage
    * you can unlock an account/password with `-U`
  * Let's say you need to create a service account for a new application that will run on your system
    * `useradd -r projetx` the -r will ensure that a system account is created
    * however when you view the output of `getent passwd projectx`
      * => `projectx:x:990:984::/home/projectx:/bin/bash`
      * you will see that it has a default bash shell; it should be "nologon"
    * we can use `usermod -s /sbin/nologon projectx` to correct that
    * in addition, the system user's home directory is along side regular users
      * we can use `usermod -d /opt/projectx projectx` to change it to a different directory
      * however, this does not create the directory automatically
      * therefore we must create the directory and adjust the permissions
      * Q: if you specified the -d with the useradd command along with the -m, would that create the directory?
        * A: yes it would, however since it was created with the root user, sudo, you would have to change the permissions on the file so that the system user is able to access the file and execute properly

* ##### `chage`
  * this command can list and modify the aging parameters of a user's password
  * `chage -E 2021-01-01 bcalhoun`
    * `-E` means -Expires on this [date]
  * `chage -l bcalhoun`
    * `l` means to -list the parameters of the user's password options
    ```bash
    $ chage -l bcalhoun
    Last password change                    					: Aug 22, 2019
    Password expires				                         	: never
    Password inactive					                        : never
    Account expires					                          : never
    Minimum number of days between password change		: 0
    Maximum number of days between password change		: 99999
    Number of days of warning before password expires	: 7
    ```
  * `chage -E -1 bcalhoun` will remove any expiration date to a user's password `-1` 
  * `chage -W 14 bcalhoun` will change the days before expiration warning; WARN DAYS
  * In summary, any option you see in the command `chage -l` can be modified with the `chage` command

* ##### `groupmod`
  * this command can modify attributes of an existing group such as name, GID, etc
  * `groupmod -g 1100 engineering` will change the group ID; `-g`
  * `groupmod -n Engineering engineering` will change the group name `-n`
 
## 107.2 Automate System Administration Tasks by Scheduling Jobs

### Cron
We learn how to use a cron table so you can schedule repeatable tasks
* each user can use their own cron table (crontab)
* the system has its own crotabs that it uses
* So how do crontabs work?
  * Well, you can view them in Centos by invoking `cat /etc/crontab`
  ```
  SHELL=/bin/bash
  PATH=/sbin:/bin:/usr/sbin:/usr/bin
  MAILTO=root

  # For details see man 4 crontabs

  # Example of job definition:
  # .---------------- minute (0 - 59)
  # |  .------------- hour (0 - 23)
  # |  |  .---------- day of month (1 - 31)
  # |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
  # |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
  # |  |  |  |  |
  # *  *  *  *  * user-name  command to be executed
  ```
  * a crontab is broken up into seven fields
    * 1. the minute within the hour (0-59)
    * 2. the hour (0 - 23)
    * 3. day of month(1 - 31)
    * 4. month (1 - 12)
    * 5. day of the week (0 - 6) (Sunday is 0 or 7) OR (sun, mon, tue, wed, thu, fri, sat) 
    * 6. user-name for the task to run under
    * 7. command (or script?) to be run
  * you can begin to edit a crontab with `crontab -e`
    * this will typically use vi as the editor of choice
    * you can set your EDITOR environment variable to the editor of your choice
  * example of backing up a documents folder each week ever sat at 5 am
    * `0 5 * * sat bennettnw2 /usr/bin/tar -cfz documents-$(/bin/date +%F).tar.gz Documents`
      * it is best practice to list your username even though it is not needed
      * you will also want to have the full path to the command
      * cron runs in its own shell so it is best to be as explicit as possible
    * `crontab -l` will list the cron job(s) that you just created
    * there is no built-in error checking with crontab
      * the cron job will just fail to execute or execute improperly
      * you will want to be sure that it runs as expected
    * `sudo cat /var/spool/cron/bennettnw2`
      * this allows a root user to view crontabs
    * `crontab -r` will delete a cron table
      * this will delete the entire table
      * just delete the line in the table if you want to delete only one cron job

 * Let's take a look at the system crontab files located in the /etc dir
  * `/etc/cron.hourly`
  * `/etc/cron.daily`
  * `/etc/cron.weekly`
  * `/etc/cron.monthly`
    * they are ran at the intervals noted after the .
    * these are not really cron tables per se but just scripts in a file
    * this is handy to use if you want to drop a script into one of these and have them run accordingly
  * `/etc/cron.d` is a crontab table file for system jobs
    * these need to be in a cronjob format with the seconds / hours / date / month / dayofweek
  * `/etc/cron.deny` will deny users listed in this file from using crontabs
    * simply list their username, one per line
  * `/etc/cron.allow` will allow users listed to create crontabs
    * be careful, if this file exists and is empty, it will prevent all non-root users from creating cronjobs

### At
This utility will run a task at a later time and will only run once.
* `at` is typically not installed by default
* `atd` is the daemon that is run, it is the hook used for systemctl
  * sudo systemctl restart atd.service
  * sudo systemctl enable atd.service
* command syntax
  * `at now + 5 minutes`
    * this means 5 minutes from now
  * this will then drop you into an at prompt
  * this is where you can enter your command(s) to be run
  * you exit out with a CTRL+D
    * <EOT> will show up when you do CTRL+D
      * ASCII notation for End Of Transmission 
      * "We have finished transmitting our data to this `at` command"
      * __ASCII__ - American Standard Code for Information Interchange 
        * ASCII is a character encoding format
  * if you see an error about "garbled time"; that means you entered the time incorrectly
  * ##### `atq`
    * this command will let me view the job queue
  * ##### `atrm`
    * this will remove a scheduled job using the job number
    * eg: `atrm 3`
    * get the job number from the `atq` command
  * ##### `at.allow` and `at.deny`
    * if a user is listed in allow then they are the only ones who can schedule `at`jobs
      * located `/etc/at.allow`
    * if a user is listed in deny then they cannot schedule an `at` job
      * located `/etc/at.deny`
    * you can use vim to edit these files just like `/etc/cron.deny`
  * `at 4:00 AM tomorrow`
    * will run a job at 4AM the next day
  * `at -f /path/to/file.sh 10:15 PM Oct 8` 
    * `-f` <- this flag will let you specify a script to run

### Systemd Timer Unit Files
The timer unit is a timer that is controlled directly by systemd.
This is done so that there can be uniformity with the unit file configuration syntax and any time based activations would benefit from the feature set of systemd as a whole.
* each timer unit will end in a .timer extension
  * you *always* need a corresponding .service file to go along with the .timer file
  * you can think if the timer unit as a timer on an oven
  * and the service unit is the chef that is ready to pull the pizza out of the oven
  * you need the timer to alert and then a service to do the job completely
* There are two types of timer units in systemd
  * ##### `Montonic`
    * meant to be run after a certain amount of time has passed
    * this is based on some starting point of time
    * this timer is deleted is the computer is shut off or suspended
    * the timer can be repeated while the computer is on
    * the unit file will have a directive like either 
      * `OnBootSec=` or `OnActivateSec=`
  * ##### `Realtime`
    * These mostly resemble cron jobs and are based on calendar events
    * OnCalendar= is a directive type for a realtime timer

* Why work with systemd at all when you got cron and at?
  * simpler syntax  
  * apart of systemd as a whole
    * assoc with a timer with a c group?
    * run a timer within a different environment
 
* ##### Transient Timers
  * these are like `at` jobs
  * they do not require a service file
  * invoked with `systemd-run`

#### Timer Unit Files
* [Unit]
* [Timer]
* Montonic
* Realtime
* Unit= is the service unit
* [Install]
  * WantedBy=timers.target
  * Timer Unit Files will typically have the same name as their service unit file 
    * eg: web-backup.service and web-backup.timer

* ##### `systemctl list-timers --all`
  * list out all the timers in a system

* ##### `systemctl cat <foo.timer>`
  * use the systemctl cat command to review the contents of a timer unit file

```
[Unit]
Description=Fire up the backup every day

[Timer]
OnCalendar=*-*-* 21:06:00
Persistent=true
Unit=web-backp.service

[Install]
WantedBy=multi-user.target
```
* Then we run:
  * `systemctl enable web-backup.timer`
  * `systemctl start web-backup.timer`

* NOTE: the `*-*-*` in the OnCalendar directive means the day-month-year of the job
  * it will be beneficial to check out the man pages of systemd.time for more deets

* ##### `systemd-run --on-active=`
  * can be used to create a transient timer
  * this does not need a service file as you specify the action in the command
  * eg `systemd-run --on-active=lm /bin/touch /root/hello`

## 107.3

### Working with the System's Locale
This is important if you are working on Linux systems around the world
By modifying a system's locale you get get it to use different character encodings and keyboards
* ##### `locale`
  * this will show you the system's `locale` settings
  * things like encoding type and language
  ```
  LANG=en US.UTF-8
  LC_NUMERIC="en_US.UTF-8"
  ...
  LC_ALL
  ```
* ##### `UTF-8`
  * is a character encoding type
  * it is an encoding standard that includes
    * all the characters I type
    * in addition special key sequences
      * eg: `CTRL-D`
  * ascii is a subset of utf-8 unicode formatting

* ##### `localectl`
  * this will set the default system language and character encoding
  * example of the output from running `localectl`
  ```
  System Locale: LANG=en_US.USF-8
      VC Keymap: us
     X11 Layout: us
  ```
  * ##### `localectl list-locale`
    * this will list out all the different available locales
    * `locale -a` will also provide the same output
      * however, `localectl list-locale` provides a pager by default
    * with this output you will notice that there are two options for encoding
    * example output:
    ```
    ca_FR
    ca_FR.iso885915
    ca_FR.utf8
    ```
  * by changing the `LANG` environment variable, we are able to change the language on the system
    * eg: `LANG=pl_PL.utf8` <= changes language to polish
    * even though we used `...utf8` we only changed the language and not the encoding
    * now when you run the `locale` command, you will see "pl_PL.utf8" on all the options
    * now all system messages are in polish
    * but the man pages are not in polish unless you install the polish man pages
      * NOTE: you may need to install language packs for certain applications to match the setting for the system.

* #### Character Encoding
  * ##### `ISO-8859`
    * This is an 8-bit, international organization for standard character encoding format
    * it is made up of 15 parts
    * iso88592 is one of them
      * this is commonly used in central European countries

  * ##### `UTF-8`
    * remember that this is the 8-bit character encoding standard of Unicode

* `LANG=C` will use the c language environment
* you can ensure consistent results using this in scripts

What if you had a file that you did not know the encoding to?
* `file -i enc-filename.txt` is the command to find out
```
=> filename.txt: text/plain; charset=iso-8859-1
```
* ##### `iconv`
  * this is a utility that can be used to convert files from once char encoding to another
  * eg: `iconv -f ISO-8859-1 -t UTF-8 -o newfile.txt original file 

What if I wanted to set this and have it be permanent?
* `localectl set-locale el_GR.iso88597`

### Time and Date on the Linux System
We need to make sure that the time and date on a system stay correct.

* #### Time and Date Settings
* Time and date on a system are important because:
  * system logs need proper time stamps for events
  * system user authentication relies to time/date
  * databases need good time to keep data storage consistent

* ##### `date`
  * this command will display the current date and time in multiple formats
  * you can use `date` to set the date and time as well
  * `date` => `Sat Sep 14 14:10:40 EDT 2019`
  * ##### `-u`
    * this gives you time in UTC
    * this is useful to make sure that all your systems are in sync
  * ##### `+`
    * this is a modifier for the date command
    * eg: `date +%F` => `2019-09-14`
    * or: `date +%D` => `09/14/19`
    * or: `date +%m%d%y` => `09/14/19`
      * you can use a capital letter and that will show the long version
      * y = 19; Y = 2019 / a = Sat; A = Saturday

* ##### `timedatectl`
  * this command will display the current date and time settings
  * this will also allow the updating of the system time and RTC clock
  * run by itself this gives you a ton of info:
```
      Local time: Sat 2019-09-14 16:35:12 EDT
  Universal time: Sat 2019-09-14 20:35:12 UTC
        RTC time: Sat 2019-09-14 20:35:11
       Time zone: America/New_York (EDT, -0400)
     NTP enabled: n/a
NTP synchronized: no
 RTC in local TZ: no
      DST active: yes
 Last DST change: DST began at
                  Sun 2019-03-10 01:59:59 EST
                  Sun 2019-03-10 03:00:00 EDT
 Next DST change: DST ends (the clock jumps one hour backwards) at
                  Sun 2019-11-03 01:59:59 EDT
                  Sun 2019-11-03 01:00:00 EST
```

* What do we do if we discover the date is incorrect on our system?
  * we can set it using the `date` command
  * `date -s "12/1/2019 12:00:00"`
  * this only temporary though until after a restart
  * after a restart it will revert back to the RTC (hardware) clock or the time supplied by the NTP server
  
* How do set a timezone?
  * `timedatectl set-timezone`
  * `timedatectl list-timezone`
  * `timedatectl set-timezone "Antartica/Davis"`
  * `timedatectl set-timezone "America/New_York"`
  * ##### `tzselect`
    * this will give us a menu driven command that will assist in finding a region's time zone
    * this command does not set the timezone for you
    * is is just a wizard to give you the variable to use in a script or to actually make the setting
  * ##### `TZ`
    * this is the environment variable you would set in order to change your timezone

* #### Time and Date Files
  * so how does a Linux system keep track of all this time and date data?
  * in RedHat based distros the file, `/etc/localtime` will hold this information
    * this file is really just a pointer file to /usr/share/zoneinfo/<time zone>
  * in Debian systems, the file is located, `/etc/timezone` 
  * tzselect and timedatectl list-timezones pulls its information from `/usr/share/zoneinfo`
