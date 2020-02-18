# 107: Administrative Tasks
## 107.1 Manage User and Group Accounts and Related System Files

### Adding and Removing Users
We are going to learn how to create new users for our system.
* Linux is a multi user operating system
* Each user can be logged in at the same time using a slice of the system's resources
* We should learn the basics of user management which includes creating and removing user accounts
* You will always need to be root to manage these types of settings

##### `useradd`
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

##### `passwd`
* this command will set a password for a specific user
* this command can also be used by a user to change their password
* `-e` will force expire a password so that it will be prompted to change on the next use
  * note this command/option combo must be run as root and it will look like:
  * eg: `passwd -e nbennett`
  * again, this will force nbennett to change their password once they login again

##### `userdel`
* this command will remove the user but will keep their home folder
* eg: `userdel nbennett`
  * note: if you do a long listing of their home dir, their user and groupname will be replaced by their UID and GID numbers
* `-r` will remove their home directory too
* eg: `userdel -r nbennett`

### Adding and Removing Groups
Groups are collections of user accounts.  We use groups to dictate the set of permissions a group of users will have for a particular resource.
* Resources like what?
  * directories
  * files
  * or services

##### `groups`
This command will show the primary and secondary groups that a user is a member.
* A list of different groups you can be in:
  * same as your username
  * users
  * group names like, "marketing", "research", etc. 
  * wheel - sudo group in Centos
  * sudoers?

##### `groupadd`
* this command will create a new group on the system
* you will need to be root/have elevated privileges in order to add groups to the system
* eg: `groupadd curators`
* you can add a user to a group in two ways
  * when a user is created they will get added to a default, primary group
  * you can add a secondary group while creating a user with `-G`
  * eg: `useradd -G curators`
    * you will need to be sure that the group already exists 
  * Q: what is the other way?
    * A: maybe `usermod`?  eg: `usermod -aG curators nbennett `
    * this is used after a user is created
    * the `-a` means to append the group and not clobber it
    * the `-G` means a secondary group rather than `-g` which means a primary group

##### `groupdel`
* this will remove a group from the system
* the users will still exist, they will just each have one less group
* eg: `groupdel curators`


### User and Group Configuration Files
We will take a look at what happens at the system level when managing users and groups and also the configuration files that help with the managing of users and groups.
#### Where is this data stored for users and groups?
##### `/etc/passwd`
* contains all of the account information on the system
* root account; system service accounts; and user accounts
* the only user that can modify `etc/passwd` is the root account
* other users do need read access to the file to be able to log into the system 
  * NOTE: the permissions are 644 for `/etc/passwd`

#### What is contained in `etc/passwd`?
* each line contains information about one system account
* : (colons) are used to separate each field in a file
* username : password : userid : primary groupid : comment `-c` goes here : full path to home dir : default shell environment
* "nologin" shell means that it is a system account and does not need to login
* root UID is always 0 and all system accounts have a UID below 1000
* regular user accounts have a UID of 1000 and above
* the password column will have an "x" in that spot which references /etc/shadow

##### In summary and in numbered list form:
1. Username: It is used when user logs in. It should be between 1 and 32 characters in length.
2. Password: An x character indicates that encrypted password is stored in /etc/shadow file. Please note that you need to use the passwd command to computes the hash of a password typed at the CLI or to store/update the hash of the password in /etc/shadow file.
3. User ID (UID): Each user must be assigned a user ID (UID). UID 0 (zero) is reserved for root and UIDs 1-99 are reserved for other predefined accounts. Further UID 100-999 are reserved by system for administrative and system accounts/groups.
4. Group ID (GID): The primary group ID (stored in /etc/group file)
5. User ID Info: The comment field. It allow you to add extra information about the users such as user’s full name, phone number etc. This field is used by finger command.
6. Home directory: The absolute path to the directory the user will be in when they log in. If this directory does not exists then the users directory becomes `/`
7. Command/shell: The absolute path of a command or shell (/bin/bash). Typically, this is a shell. Please note that it does not have to be a shell.

##### `/etc/shadow`
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

Example of a regular user's entry in `/etc/passwd`
`bennett:$6$nHSeusmEfo414$FtoyBUyZzifgXMacps3GnhKz3zeCY/cGZBOQ1:18274:0:99999:7:::`

##### In summary and in numbered list form:
1. Username : It is your login name.
2. Password : It is your encrypted password. The password should be minimum 8-12 characters long including special characters, digits, lower case alphabetic and more. Usually password format is set to $id$salt$hashed, The $id is the algorithm used On GNU/Linux as follows:
* $1$ is MD5
* $2a$ is Blowfish
* $2y$ is Blowfish
* $5$ is SHA-256
* $6$ is SHA-512
Last password change (lastchanged) : Days since Jan 1, 1970 that password was last changed
3. Minimum : The minimum number of days required between password changes i.e. the number of days left before the user is allowed to change his/her password
4. Maximum : The maximum number of days the password is valid (after that user is forced to change his/her password)
5. Warn : The number of days before password is to expire that user is warned that his/her password must be changed
6. Inactive : The number of days after password expires that account is disabled
7. Expire : days since Jan 1, 1970 that account is disabled i.e. an absolute date specifying when the login may no longer be used.

##### `/etc/group`
* This file contains group definitions along with what member belong to each group
* This file only has four fields
  * the first is the name of the group
  * the second is the password field
    * groups do not have passwords as we rely on the user's passwords
  * the third field is the group ID number
  * there is a fourth field that will list the members of that group
    * if is is blank then the user listed in the only member of the group with the same name
    * if there are multiple users, they will be separated by a comma 

1. group_name: It is the name of group. If you run ls -l command, you will see this name printed in the group field.
2. Password: Generally password is not used, hence it is empty/blank. It can store encrypted password. This is useful to implement privileged groups.
3. Group ID (GID): Each user must be assigned a group ID. You can see this number in your /etc/passwd file.
4. Group List: It is a list of user names of users who are members of the group. The user names, must be separated by commas.

##### `/etc/skel`
* add files and folders to this directory so that when a new user gets created on a system, they get the same default set of folders and files each time
* files like default `.bash_profile`, `.bashrc`, etc.
* default shared folders they need access to

##### `/etc/default/useradd`
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

##### `usermod`
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

##### `chage`
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

##### `groupmod`
* this command can modify attributes of an existing group such as name, GID, etc
* `groupmod -g 1100 engineering` will change the group ID; `-g`
* `groupmod -n Engineering engineering` will change the group name `-n`

## 107.2 Automate System Administration Tasks by Scheduling Jobs

### Cron
We learn how to use a cron table so you can schedule repeatable tasks
* each user can use their own cron table (crontab)
* the system has its own crotabs that it uses
#### So how do crontabs work?
You can view them in Centos by invoking `cat /etc/crontab`
```bash
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
A crontab is broken up into seven fields
* 1. the minute within the hour (0-59)
* 2. the hour (0 - 23)
* 3. day of month(1 - 31)
* 4. month (1 - 12)
* 5. day of the week (0 - 6) (Sunday is 0 or 7) OR (sun, mon, tue, wed, thu, fri, sat) 
* 6. user-name for the task to run under
* 7. command (or script?) to be run

You can begin to edit a crontab with `crontab -e`
* this will typically use vi as the editor of choice
* you can set your EDITOR environment variable to the editor of your choice

Example of backing up a documents folder each week ever sat at 5 am
* `0 5 * * sat bennettnw2 /usr/bin/tar -cfz documents-$(/bin/date +%F).tar.gz Documents`
  * it is best practice to list your username even though it is not needed
  * you will also want to have the full path to the command
  * cron runs in its own shell so it is best to be as explicit as possible
##### `crontab -l`
  * will list the cron job(s) that you just created
* there is no built-in error checking with crontab
  * the cron job will just fail to execute or execute improperly
  * you will want to be sure that it runs as expected
##### `sudo cat /var/spool/cron/bennettnw2`
  * this allows a root user to view crontabs
##### `crontab -r`
  * will delete a cron table
  * this will delete the entire table
  * just delete the line in the table if you want to delete only one cron job

#### Let's take a look at the system crontab files located in the /etc dir
  * `/etc/cron.hourly`
  * `/etc/cron.daily`
  * `/etc/cron.weekly`
  * `/etc/cron.monthly`
    * they are ran at the intervals noted after the . (dot)
    * these are not really cron tables per se but just scripts in a file
    * this is handy to use if you want to drop a script into one of these and have them run accordingly
##### `/etc/cron.d`
  * is a crontab table file for system jobs
    * these need to be in a cronjob format with the seconds / hours / date / month / dayofweek
##### `/etc/cron.deny`
  * will deny users listed in this file from using crontabs
    * simply list their username, one per line
##### `/etc/cron.allow` 
  * will allow users listed to create crontabs
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
    * \<EOT> will show up when you do CTRL+D
      * The above is ASCII notation for End Of Transmission which means, "We have finished transmitting our data to this `at` command"
      * __ASCII__ - American Standard Code for Information Interchange 
        * ASCII is a character encoding format
  * if you see an error about "garbled time"; that means you entered the time incorrectly
##### `atq`
* this command will let me view the job queue

##### `atrm`
* this will remove a scheduled job using the job number
* eg: `atrm 3`
* get the job number from the `atq` command

##### `at.allow` and `at.deny`
* if a user is listed in allow then they are the only ones who can schedule `at`jobs
  * located `/etc/at.allow`
* if a user is listed in deny then they cannot schedule an `at` job
  * located `/etc/at.deny`
* you can use vim to edit these files just like `/etc/cron.deny`
`at 4:00 AM tomorrow`
* will run a job at 4AM the next day
`at -f /path/to/file.sh 10:15 PM Oct 8` 
* `-f` <- this flag will let you specify a script to run

### Systemd Timer Units
The timer unit is a timer that is controlled directly by systemd.
This is done so that there can be uniformity with the unit file configuration syntax and any time based activations would benefit from the feature set of systemd as a whole.  Instead of the timer and the action together as in cron jobs, these are configured separately.

The purpose of a timer unit file is that it is a timer controlled by systemd and allows for uniformity with other systemd processes and it can benefit from that.  Each `.timer` unit file must have a matching `.service` unit file.  You can think of it the timer is the timer and the service is the thing to be done once the timer goes off.  

Each timer unit will end in a .timer extension
* you *always* need a corresponding .service file to go along with the .timer file
* you can think if the timer unit as a timer on an oven
* and the service unit is the chef that is ready to pull the pizza out of the oven
* you need the timer to alert and then a service to do the job completely

#### There are two types of timer units in systemd:
##### Montonic Timer
* Is meant to be run after a certain amount of time has passed.  Meaning, these run independent of wall-clocks or timezones.  If the computer is shut off or suspended, the timer is also shutoff or suspended.
* The timer can be repeated while the computer is on
* The unit file will have a directive like, `OnBootSec=` or `OnActivateSec=`, where an event and a time (defaulted in seconds) are present

##### Realtime Timer
* These are configured to run dependent of wall-clocks and timezones meaning, this will check for the time rather than an event.
* These mostly resemble cron jobs and are based on calendar events
* `OnCalendar=` is a directive type for a realtime timer

#### Why work with systemd at all when you got cron and at?
* Simpler syntax  
* A part of systemd as a whole
  * assoc with a timer with a c group?
  * run a timer within a different environment
 
#### Transient Timers
* these are like `at` jobs
* they do not require a service file
* invoked with `systemd-run`
  * eg: `systemd-run --on-active=1m /bin/touch /root/hello`
  * this will create a file called 'hello' in the /root directory 1 minute from when the command executes

### Timer Unit Files
Here is a sample timer unit file named `web-backup.timer`:
```bash
[Unit]
Description=Fire up the backup every day

[Timer]
OnCalendar=*-*-* 21:06:00
Persistent=true
Unit=web-backup.service

[Install]
WantedBy=multi-user.target
```
##### [Timer]
Timer unit files have the `[Timer]` section which, other types of unit files will never have.  If you see `[Timer]` it is a dead give away that you have a timer unit file.  Depending on the timer type (`OnBootSec`, `OnCalendar`, etc.), you will have different directives.

Notice the `Unit=web-backup.service` line.  This where where the service unit file is specified.  Also notice that the service unit specified has the same name as the timer unit file of `web-backup.timer`  This is not necessary to do as it is assumed they are the same, but it is a good practice to be explicit about this.  

**==Options to the [Timer] section:==**
##### `OnActiveSec=`
  * defines a timer relative to the moment the timer itself is activated.  
##### `OnBootSec=` 
  * defines a timer relative to when the machine was booted up.  
##### `OnStartupSec=` 
  * defines a timer relative to when systemd was first started.
##### `OnUnitActiveSec=` 
  * defines a timer relative to when the unit the timer is activating was last activated.
##### `OnUnitInactiveSec=` 
  * defines a timer relative to when the unit the timer is activating was last deactivated.
##### `OnCalendar=`
  * defines realtime (i.e. wallclock) timers with calendar event expressions.

The arguments to the directives are time spans configured in seconds. Example: `OnBootSec=50` means 50s after boot-up. The argument may also include time units. Example: `OnBootSec=5h 30min` means 5 hours and 30 minutes after boot-up.

### Calendar Events
Calendar events may be used to refer to one or more points in time in a single expression.  Calendar events are used by timer units.
#### `*-*-* *:*:*`  
  * year-month-day hour:minute:second
  * star means "any"
##### `Thu,Fri 2012-*-1,5 11:12:13`

The above refers to 11:12:13 of the first or fifth day of any month of the year 2012, but only if that day is a Thursday or Friday.

* The weekday specification is optional
* Specifying two weekdays separated by ".." refers to a range of continuous weekdays.
* "," and ".." may be combined freely.

In the date and time specifications, any component may be specified as "\*" in which case any value will match. Alternatively, each component can be specified as a list of values separated by commas. Values may be suffixed with "/" and a repetition value, which indicates that the value itself and the value plus all multiples of the repetition value are matched. Two values separated by ".." may be used to indicate a range of values; ranges may also be followed with "/" and a repetition value.

Either time or date specification may be omitted, in which case the current day and 00:00:00 is implied, respectively. If the second component is not specified, ":00" is assumed.

Timezone can be specified as the literal string "UTC", or the local timezone, similar to the supported syntax of timestamps (see above), or the timezone in the IANA timezone database format (also see above).

The following special expressions may be used as shorthands for longer normalized forms:
```bash
    minutely → *-*-* *:*:00
      hourly → *-*-* *:00:00
       daily → *-*-* 00:00:00
     monthly → *-*-01 00:00:00
      weekly → Mon *-*-* 00:00:00
      yearly → *-01-01 00:00:00
   quarterly → *-01,04,07,10-01 00:00:00
semiannually → *-01,07-01 00:00:00
```   
Examples for valid timestamps and their normalized form:
```bash
  Sat,Thu,Mon..Wed,Sat..Sun → Mon..Thu,Sat,Sun *-*-* 00:00:00
      Mon,Sun 12-*-* 2,1:23 → Mon,Sun 2012-*-* 01,02:23:00
                    Wed *-1 → Wed *-*-01 00:00:00
           Wed..Wed,Wed *-1 → Wed *-*-01 00:00:00
                 Wed, 17:48 → Wed *-*-* 17:48:00
Wed..Sat,Tue 12-10-15 1:2:3 → Tue..Sat 2012-10-15 01:02:03
                *-*-7 0:0:0 → *-*-07 00:00:00
                      10-15 → *-10-15 00:00:00
        monday *-12-* 17:00 → Mon *-12-* 17:00:00
  Mon,Fri *-*-3,1,2 *:30:45 → Mon,Fri *-*-01,02,03 *:30:45
       12,14,13,12:20,10,30 → *-*-* 12,13,14:10,20,30:00
            12..14:10,20,30 → *-*-* 12..14:10,20,30:00
  mon,fri *-1/2-1,3 *:30:45 → Mon,Fri *-01/2-01,03 *:30:45
             03-05 08:05:40 → *-03-05 08:05:40
                   08:05:40 → *-*-* 08:05:40
                      05:40 → *-*-* 05:40:00
     Sat,Sun 12-05 08:05:40 → Sat,Sun *-12-05 08:05:40
           Sat,Sun 08:05:40 → Sat,Sun *-*-* 08:05:40
           2003-03-05 05:40 → 2003-03-05 05:40:00
 05:40:23.4200004/3.1700005 → *-*-* 05:40:23.420000/3.170001
             2003-02..04-05 → 2003-02..04-05 00:00:00
       2003-03-05 05:40 UTC → 2003-03-05 05:40:00 UTC
                 2003-03-05 → 2003-03-05 00:00:00
                      03-05 → *-03-05 00:00:00
                     hourly → *-*-* *:00:00
                      daily → *-*-* 00:00:00
                  daily UTC → *-*-* 00:00:00 UTC
                    monthly → *-*-01 00:00:00
                     weekly → Mon *-*-* 00:00:00
    weekly Pacific/Auckland → Mon *-*-* 00:00:00 Pacific/Auckland
                     yearly → *-01-01 00:00:00
                   annually → *-01-01 00:00:00
                      *:2/3 → *-*-* *:02/3:00
```

##### `systemctl list-timers --all`
* list out all the timers in a system
```bash
$ systemctl list-timers --all
NEXT                         LEFT          LAST                         PASSED       UNIT                         ACTIVATES
Mon 2020-02-17 14:10:00 EST  5min left     Mon 2020-02-17 14:00:42 EST  3min 19s ago sysstat-collect.timer        sysstat-collect.service
Mon 2020-02-17 14:51:45 EST  47min left    Mon 2020-02-17 13:51:44 EST  12min ago    dnf-makecache.timer          dnf-makecache.service
Mon 2020-02-17 16:50:16 EST  2h 46min left Sun 2020-02-16 16:50:16 EST  21h ago      systemd-tmpfiles-clean.timer systemd-tmpfiles-clean.service
Tue 2020-02-18 00:00:00 EST  9h left       Mon 2020-02-17 00:00:08 EST  14h ago      unbound-anchor.timer         unbound-anchor.service
Tue 2020-02-18 00:07:00 EST  10h left      Mon 2020-02-17 00:07:02 EST  13h ago      sysstat-summary.timer        sysstat-summary.service

5 timers listed.
```

##### `systemctl cat <foo.timer>`
* use the `systemctl cat` command to review the contents of a timer or service unit file
```bash
$ systemctl cat systemd-tmpfiles-clean.service
# /usr/lib/systemd/system/systemd-tmpfiles-clean.service
#  SPDX-License-Identifier: LGPL-2.1+
#
#  This file is part of systemd.
#

[Unit]
Description=Cleanup of Temporary Directories
Documentation=man:tmpfiles.d(5) man:systemd-tmpfiles(8)
DefaultDependencies=no
Conflicts=shutdown.target
After=local-fs.target time-sync.target
Before=shutdown.target

[Service]
Type=oneshot
ExecStart=/usr/bin/systemd-tmpfiles --clean
SuccessExitStatus=65
IOSchedulingClass=idle
```
```bash
# /usr/lib/systemd/system/systemd-tmpfiles-clean.timer
#  SPDX-License-Identifier: LGPL-2.1+
#
#  This file is part of systemd.
#

[Unit]
Description=Daily Cleanup of Temporary Directories
Documentation=man:tmpfiles.d(5) man:systemd-tmpfiles(8)

[Timer]
OnBootSec=15min
OnUnitActiveSec=1d
```

Then we run the below commands to start the timer and to enable it to be active after a restart:
* `systemctl enable web-backup.timer`
* `systemctl start web-backup.timer`

NOTE: the `*-*-*` in the OnCalendar directive means the day-month-year of the job
* it will be beneficial to check out the man pages of systemd.time for more deets

##### `systemd-run --on-active=`
* can be used to create a transient timer
* this does not need a service file as you specify the action in the command
* eg `systemd-run --on-active=lm /bin/touch /root/hello`

## 107.3 Localisation and internationalisation
### Working with the System's Locale
This is important if you are working on Linux systems around the world
By modifying a system's locale you can get it to use *different character encodings and default language*
##### `locale`
* this will show you the system's `locale` settings
* things like encoding type and language
```bash
LANG=en_US.UTF-8  determines default lcoale in absense of other locale related env variables
LANGUAGE          this sets how system messages are translated; no formatting
LC_NUMERIC        how numbers are formatted
LC_TIME           how time is displayed
LC_MONETARY       how money is diplayed
LC_PAPER          how paper sizes are displayed
LC_ADDRESS        how addresses are formatted
LC_TELEPHONE      how telephone numbers are formatted
LC_MEASUREMENT    default measurement system used in the region
LC_IDENTIFICATION
...
LC_ALL
```
Normally how `locale` works is you set `LANG` to your preferred locale.  If there are some aspects of your primary locale that you don't like (date formats) then you can set specific variable to override those features (above) only.  You do not want to set the LC_ALL variable, at least not permanently.  It is reserved for situation where you need to enforce a specific locale (usually "C") on a temporary basis.
  * one example of this would be reporting error messages on an English-speaking mailing list; you can use `LC_ALL=C your command` to ensure that the errors are in English and follow all the POSIX norms.


##### `UTF-8`
* is a character encoding type
* it is an encoding standard that includes
  * all the characters I type
  * in addition special key sequences
    * eg: `CTRL-D`
* ascii is a subset of utf-8 unicode formatting

##### `localectl`
* this will set the default system language and character encoding
* example of the output from running `localectl`
```
System Locale: LANG=en_US.UTF-8
    VC Keymap: us
   X11 Layout: us
```
##### `localectl list-locales`
* this will list out all the different available locales
* `locale -a` will also provide the same output
  * however, `localectl list-locales` provides a pager by default
* with this output you will notice that there are two options for encoding
* example output:
```
ca_FR
ca_FR.iso885915
ca_FR.utf8
```

By changing the `LANG` environment variable, we are able to change the language on the system
* eg: `LANG=pl_PL.utf8` <= changes language to polish
* even though we used `...utf8` we only changed the language and not the encoding
* now when you run the `locale` command, you will see "pl_PL.utf8" on all the options
* now all system messages are in polish
* but the man pages are not in polish unless you install the polish man pages
  * eg: `yum -y install man-pages-pl`
  * NOTE: you may need to install language packs for certain applications to match the setting for the system.

#### Character Encoding
##### `ISO-8859`
* This is an 8-bit, international organization for standard character encoding format
* it is made up of 15 parts
* iso88592 is one of them
  * this is commonly used in central European countries
* *Remember* that you can use any of these iso8859[1-15] to set the encoding for a user's specific country

##### `UTF-8`
* remember that this is the 8-bit character encoding standard of Unicode

`LANG=C` will use the c language environment
you can ensure consistent results using this in scripts

#### What if you had a file that you did not know the encoding to?
`file -i enc-filename.txt` is the command to find out
* will yield the result:
```
=> filename.txt: text/plain; charset=iso-8859-1
```

#### What if you wanted to convert a file from one encoding format to another?
##### `iconv`
* this is a utility that can be used to convert files from once char encoding to another
* eg: `iconv -f ISO-8859-1 -t UTF-8 -o newfile.txt original file`
  * -f means encoding from; -t means encoding to; -o means output of file; then you have the file to operate on

#### What if I wanted to set this and have it be permanent?
* `localectl set-locale el_GR.iso88597`

Summary for myself: you set the locale setting and the character encoding which can be UTF-8 or ISO-8859[1-15

### Time and Date on the Linux System
We need to make sure that the time and date on a system stay correct.

#### Time and Date Settings
Time and date on a system are important because:
* system logs need proper time stamps for events
* system user authentication relies to time/date
* databases need good time to keep data storage consistent

##### `date`
* this command will display the current date and time in multiple formats
* you can use `date` to set the date and time as well
* `date` => `Sat Sep 14 14:10:40 EDT 2019`
##### `-u`
* this gives you time in UTC
* this is useful to make sure that all your systems are in sync
##### `+`
* this is a modifier for the date command
* eg: `date +%F` => `2019-09-14`
* or: `date +%D` => `09/14/19`
* or: `date +%m%d%y` => `09/14/19`
  * you can use a capital letter and that will show the long version
  * %y = 19; %Y = 2019 / %a = Sat; %A = Saturday
    * this convention not all that consistent though
      * %d = day of month in number format; %D = a full date output %m/%d/%y
      * %h is the same as %b (abbreviated month!); %H is the hour in 24 hour notation (00..23)

##### `timedatectl`
* This command will display the current date and time settings
* This will also allow the updating of the system time and RTC clock
* Run by itself, this command gives you a ton of info:
```
$ timedatectl
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

#### What do we do if we discover the date is incorrect on our system?
* we can set it using the `date` command
* `date -s "12/1/2019 12:00:00"`
* this only temporary though until after a restart
* after a restart it will revert back to the RTC (hardware) clock or the time supplied by the NTP server

You can set the date and change the RTC(Real Time Clock) by running:
* `sudo timedatectl set-time "2020-01-01 00:00:00"`
* this will not work though if NTP is enabled.  NTP will block the change
  * `Failed to set time: Automatic time synchronization is enabled`

#### How do you set a timezone?
There is a directory at `usr/share/zoneinfo` and you just create a symbolic link from the desired timezone file from `/usr/share/zoninfo/` and link it to `/etc/localtime`
* `timedatectl set-timezone`
* `timedatectl list-timezones`
* `timedatectl set-timezone "Antartica/Davis"`
* `timedatectl set-timezone "America/New_York"`
* ##### `tzselect`
  * this will give us a menu driven command that will assist in finding a region's time zone
  * this command does not set the timezone for you
  * is is just a wizard to give you the variable to use in a script or to actually make the setting
* ##### `TZ`
  * this is the environment variable you would set in order to change your timezone
  * this is good for in a profile or a script

#### Time and Date Files Locations
* in RedHat based distros the file, `/etc/localtime` will hold this information
  * this file is really just a pointer file to /usr/share/zoneinfo/\<time zone>
* in Debian systems, the file is located, `/etc/timezone` 
* tzselect and timedatectl list-timezones pulls its information from `/usr/share/zoneinfo`
