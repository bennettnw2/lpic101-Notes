# Topic 107: Administrative Tasks
## 107.1 Manage user and group accounts and related system files
#####Weight: 5

#### Description: Candidates should be able to add, remove, suspend and change user accounts.

### Key Knowledge Areas:

* Add, modify and remove users and groups.
* Manage user/group info in password/group databases.
* Create and manage special purpose and limited accounts.
* The following is a partial list of the used files, terms and utilities:

* /etc/passwd
* /etc/shadow
* /etc/group
* /etc/skel/
* chage
* getent
* groupadd
* groupdel
* groupmod
* passwd
* useradd
* userdel
* usermod
 

## 107.2 Automate system administration tasks by scheduling jobs
##### Weight: 4

####Description: Candidates should be able to use cron and systemd timers to run jobs at regular intervals and to use at to run jobs at a specific time.

Key Knowledge Areas:

Manage cron and at jobs.
Configure user access to cron and at services.
Understand systemd timer units.
The following is a partial list of the used files, terms and utilities:

/etc/cron.{d,daily,hourly,monthly,weekly}/
/etc/at.deny
/etc/at.allow
/etc/crontab
/etc/cron.allow
/etc/cron.deny
/var/spool/cron/
crontab
at
atq
atrm
systemctl
systemd-run

## 107.3 Localisation and internationalisation
* Configure timezone settings and environment variables.
* Configure locale settings and environment variables.

#### Configure timezone settings and environment variables.
**Setting the System's Time Zone**
Linux looks to the /etc/localtime file for information about its local time zone.  You can see your current timezone by running either, `date` or more specifically, `ls -l /etc/localtime`
```bash
$ date
Sat Feb 15 10:37:13 EST 2020
$ ls -l /etc/localtime
lrwxrwxrwx. 1 root root 38 Jan 13 15:15 /etc/localtime -> ../usr/share/zoneinfo/America/New_York
```

To set the system's time zone you will basically take a file from `/usr/share/zoneinfo/....` and either copy it to or link it to `/etc/localtime`

#### Configure locale settings and environment variables.
Need to understand locale in Linux talk.  A locale is a way of specifying the computer's or user's language, country, and related information for purposes of customizing displays.  Like how some countries use commas instead of decimal points.  Or the format of the date and time.  These are things that make up a locale.

A single locale takes the following form:
`[language[_territory][.codeset][@modifier]]`
eg: `en_US.UTF-8` `jp_JP.UTF-8`
* language - is a two or three letter code
  * en English | fr French | jp Japanese
* territory are codes for specific regions; generally nations
  * US United States | FR France | JP Japan
* the codeset can be ASCII, UTF-8 or other encoding names
  * ASCII - American Standard Code for Information Interchange is the oldest encoding.  It only supports 7 bit encodings and is not able to support many characters used in many non-English languages.
  * ISO-8859 tried to extend ASCII by adding another bit.  This works well enough but there are sub-standards that handles one language or a small group of languages.  ISO-8859-1 covers Western Europe; ISO-8859-5 supports Cyrillic characters.
  * UTF-8 is the latest and the best because it handles all of the writing systems automatically without the need for sub-standards.
* modifier is a locale-specific code that modifies now the codeset works.  For instance, it may affect the sort order on a language-specific manner.

To find out what your locale settings are, just run, `locale` or if you really want to nerd out `/usr/bin/locale`
```bash
$ /usr/bin/locale
LANG=en_US.UTF-8
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
LC_ALL=
```

To view your current locale, you can run `localectl`.  Note that X11 has its own setting so you will need to change that too if you access your shell with X11.
```bash
$ localectl
   System Locale: LANG=en_US.UTF-8
       VC Keymap: us
      X11 Layout: usf
```

To view all possible locale options, you can run either, `locale -a` or `localectl list-locals`

To change the locale setting there are a few ways to do this.
1. Temporarily 
  * `export LANG=en_GB.UTF-8` and also `export LC_ALL=en_GB.UTF-8`
2. Permanently
  * add the above lines to either `~/.bashrc` or `~/.bash_profile`
  * or `localctl set-locale el_GR.iso88597`  sets it for the Greek language

If you have a file that happens to be encoded into a different format, you can use the `iconv` command to convert from one character encoding to another.
eg:`iconv -f ISO-8859-1 -t UTF-8 -o newtext.txt oldtext.txt`
* This says to take the file old file (oldtext.txt) and convert it from (-f) ISO-8859-1 to (-t) UTF-8 and then output (-o) the results to newtext.txt.
