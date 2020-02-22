# Topic 105: Shells and Shell Scripting
## 105.1 Customize and use the shell environment
##### Weight: 4

##### Description: Candidates should be able to customize shell environments to meet users' needs. Candidates should be able to modify global and user profiles.

### Key Knowledge Areas:

##### Set environment variables (e.g. PATH) at login or when spawning a new shell.
* You can configure either your `.bash_profile` or your `.bashrc`

##### Write Bash functions for frequently used sequences of commands.
```bash
function_name () {
  read input
  echo $input
}
```
Or you could write it this way too
```bash
function fname () {
  read input
  echo $input
}
```
##### Maintain skeleton directories for new user accounts.
This is located in `/etc/skel` and is invoked upon user creation with -s?
##### Set command search path with the proper directory.
As the $PATH is an environment variable, this would be set within the `~/bash_profile` file.  You would set it by invoking the commands:
```bash
PATH=$PATH:/.localdir
. ~/.bash_profile
```


### The following is a partial list of the used files, terms and utilities:

##### . (dot)
This is a shorthand for the `source` command below.  It is used to source a file into the current shell.  I like to think we are letting the shell know that we have made a change to the configs and the shell needs to be aware of this fact so, here you go, shell!
##### source
Please see above for an explanation and please see below for a demonstration:
```bash
source ~/.bash_profile  # this is the same as the next command
. ~/.bash_profile
```
##### /etc/bash.bashrc
This is a Debian config file which forces a non-login shell to read this file first before it reads `~/.bashrc`.  Which, typically with a non-login shell, `~/.bashrc` is read first and then `/etc/bashrc`.  But, Debian did something a bit different with its own compilation and now here we are.
##### /etc/profile
This file is read first by the bash shell when launching a login shell.  You do not want to edit this file directly but you can incorporate your own environment variables and startup programs by placing scripts into `/etc/profile.d`
##### `env`
This command will list out all the shell environment variables.
```bash
LANG=en_US.UTF-8
HISTCONTROL=ignorespace:erasedups
HOSTNAME=JavaDev
EDITOR=vim
S_COLORS=auto
XDG_SESSION_ID=143
USER=user-v245
PWD=/home/user-v245
HOME=/home/user-v245
CLICOLOR=Yes
SSH_CLIENT=71.52.08.70 49964 22
SSH_TTY=/dev/pts/0
MAIL=/var/spool/mail/user-v245
TERM=xterm-256color
SHELL=/bin/bash
LS_OPTIONS=--color=auto
```

##### export
##### set
##### unset
##### ~/.bash_profile
##### ~/.bash_login
##### ~/.profile
##### ~/.bashrc
##### ~/.bash_logout
##### function
##### alias


## 105.2 Customize or write simple scripts
##### Weight: 4

##### Description: Candidates should be able to customize existing scripts, or write simple new Bash scripts.

#### Key Knowledge Areas:

##### Use standard sh syntax (loops, tests).
##### Use command substitution.
##### Test return values for success or failure or other information provided by a command.
##### Execute chained commands.
##### Perform conditional mailing to the superuser.
##### Correctly select the script interpreter through the shebang (#!) line.
##### Manage the location, ownership, execution and suid-rights of scripts.
##### The following is a partial list of the used files, terms and utilities:

##### for
##### while
##### test
##### if
##### read
##### seq
##### exec
##### ||
##### &&


Also, lets settle it once and for all the order in which these files are accessed

I started with `/etc/profile` as I was reading that environment variables are set, system-wide in `/etc/profile`.  Seems to be a good start!  I catted out the file and looked it over.  The notes at the top of the file indicate that this file is for system wide environment variable and startup programs.  It also says that login setup functions and aliases go in `/etc/bashrc`.  Then it goes through a bunch of initializations and then it finishes with sourcing `/etc/bashrc`.

Let's have a look at `/etc/bashrc`.  At the top of this file, it indicates this is for system-wide funtions and aliases and that "Environment stuff goes in /etc/profile".  Cools!  But it does not source anything but it does talk alot about being either in a log in shell or a non-login shell.  Both of these files reccomend puting in custom scripts into `/etc/profile.d` to make modifictions to these files as it is not a good idea to edit them directly.

Let's have a quick look at `/etc/profile.d`.  So this directory is a bunch of scripts that I imagine get initialized when `/etc/profile` is run and `/etc/bashrc`.

This is a snippet of `/etc/profile` that will go through the `/etc/profile.d` directory and source each file in the directory.
```bash
for i in /etc/profile.d/*.sh /etc/profile.d/sh.local ; do
    if [ -r "$i" ]; then
        if [ "${-#*i}" != "$-" ]; then  # I am curious to know what is going on here!
            . "$i"
        else
            . "$i" >/dev/null
        fi
    fi
done
```

This is all well and good!  But how does `.bashrc` and `.bash_profile` get roped into all this?

In reading through this book, they give a great overview of a matrix of the type of shell (login shell vs non-login) and the config files as global or personal.

I've had trouble understanding a login shell and a non-login shell.  I used to think it was the difference between logging in through an SSH connection (login shell) vs opening up a terminal from a GUI (non-login).  In reading this book, LPIC-1 Certification Study Guide, I think a deeper and easier distiction would be that of, "Do you need authentication to use this shell?".  A non-login shell can also be created by a script that needs to run shell commands.  Or a non login shell is one that you run the command `bash` while already in a shell.  Great!

Found this gem in `/etc/profile`:
```bash
\# Bash login shells run only /etc/profile
\# Bash non-login shells run only /etc/bashrc
```

I found a nice tidbit on Quora.com on this and will stop here on my journey about login shells and stuffs:
> * /etc/profile, ~/.bash_rc, and ~/.bash_profile, are all files and are called configuration scripts. They can contain variable declarations, export variables, commands to be executed on login like mail or news checking, setting umask, among others. Typical things users do are: adding some dir to $PATH, exporting some variable, changing $PS1, setting display colors, adding a greeting text message, etc.
> * All those files, except for /etc/profile, are by default hidden, as denoted by the leading dot (so its not bash_profile but actually .bash_profile).
> * When you login (either locally or remotely), this is called a login shell, and is treated a little bit different from normal shell invocation. In that case, the /etc/profile file, if present, is executed, after which either a ~/.bash_profile or a ~/.bashrc file is executed, in that order.
> * For an interactive shell, i.e. the one you can interact with, because the shell’s stdin and stderr are both TTYs, the ~/.bash_profile file is not executed, but the ~/.bashrc is.
> * For a non-interactive shell, i.e. the one with either one or both stdin/stderr is not a TTY, no configuration script is executed.
> * In a login shell, at logout, a ~/.bash_logout file, if present, is executed.
> * As per POSIX, stderr (and not stdout) is the stream that determines if a shell is interactive. If stderr is redirected, it is not an interactive shell, unless -i is specified in the shell invocation. stdout doesn’t have the same effect.

