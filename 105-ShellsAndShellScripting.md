# 105 Shells and Shell Scripting
## 105.1 Customize and Use the Shell Environment
This lesson takes a deeper dive into the interactive login shell and interactive non-login shell.  We do this by looking at the configuration files that make up these two environments and the order in which they are loaded.  This is very important to know about using automatic deployment tools.

### The Interactive Login Shell
  * Created when you log into a console
  * created when logged in remotely using ssh

### The Interactive Non-Login Shell
  * emulates a terminal environment
  * iterm2, gnome terminal, kde konsole

What is important to note here is to learn how these shells are created.  More specifially the process that go on behind the scenes with the files that are used when creating a shell environment.

#### What happens when you fire up a shell through ssh. (Login Shell)
  * `/etc/profile`
      * is the first program that is called right after you log in
      * gets read and it will call out to => /etc/profile.d/*
  * `/etc/profile.d/*`
    * more configurations are available through /etc/profile.d/
    * /etc/profile pulls in these configurations
  * /etc/profile will then make the configs taken from /profile.d/ apart of itself
  * then /profile/ will reach out to ~/.bash\_profile
  * `~/.bash_profile`
    * could be called `.profile`
    * resides in my home directory
    * `.bash_profile` then reaches out to `~/.bashrc` which is also in my home directory
      * .bashrc contains information about the system configuration
      * it does this by simply talking to `/etc/bashrc`
  * Then you have a running shell!

#### What happens when you fire up terminal from you Ubuntu dist?
  * you begin from a graphical desktop
  * `~/.bashrc` is the first file that is called
  * it still then reaches out to `/etc/bashrc`

#### How can you tell which one you are in?
  * `echo $0`
  * => `-bash` means you are using a login shell
    * "line shell" = login shell
  * => `bash` means you are using a non-login shell
    * "non-line shell" = non-login shell

### Bash Configuration Files
##### `/etc/profile`
  * This file is for system wide environment and startup programs, for login setup
  * You typically do not want to modify this file.
  * It is best to create a custom shell script in /etc/profile.d to make changes to a shell environment
  * this file will pull those changes in from /etc/profile.d
  * This file will set up your $PATH environment variable
  * This file will also set up your bash history size
  * It sets up the umask value
##### `~/.bash_profile` or `~/.profile` file
  * Remember: this file gets called after `etc/profile`
  * The first this this file does is to call the `~/.bashrc` file
  * This file will set up my environment variable for my path and then exports the path 
##### `~/.bashrc`
  * calls on the file `/etc/bashrc` and then pulls in global definitions that have been set up, server wide
    * what are global definitions?  custom functions, aliases and the like that have been set up for the server as a whole
    * individual user specific definitions are specified directly in `.bashrc`
##### `/etc/bashrc`
  * again you get a warning in the file to not modify the file but to modify /etc/profile.d
  * this is a spot for system wide functions and variables
  * environment stuff goes into /etc/profile
  * this will set up how my terminal looks and some shell options
    * like how your history is handled
##### `/etc/skel`
  * this contains the default .bash\_profile, .bashrc and other files that are added to a user's home directory when an account is created on the system
  * you need to use the -a flag from ls in order to see the template files
    ##### `~/.bash_logout`
      * this file get called on a user logout event
      * it can be used to shut down applications; display messages
      * you can think of it as general shell clean up on log out
      * you can add a message or any cleanup tasks
    ##### `~/.bash_login`
      * this file gets called on a user login
      * this has been mostly been replaced with .bash_profile/.profile file
      * used to start applications

Process and order that these files are called is very important.

First .bash\_login (if it exists)
Then /etc/profile which also pulls in /etc/profile.d
Then `.bash_profile` is tapped which in turn calls on `.bashrc`

### Customizing the Shell Environment
In addition to `env` `set` `unset` and `export` to tweak your shell environment, you can use `alias` `function` and `source`.  Also learn about how to use the $PATH variable to customize where bash will look for executables
  * ##### `env`
    * all environment variables that are set on the system
  * ##### `export`
    * you are able to have child shells access variables from the parent shell
    * `export FOO=BAR`
    * echo $FOO => BAR (this is in a child shell)
  * ##### `set`
    * by itself it will display all bash shell settings, variable and functions
    * it can be used to enable and disable Bash settings
    * `set | less`
    * Disable Globbing Functionality
      * `set -f`
      * `set +f` will turn file globbing back on
      * you will need to reference the man page for details of the set command
    ##### `unset`
      used to separate a variable from its' value
  * ##### `alias`
    * this is used to create a shortcut to a longer command
    * for instance, I use aliases to log into different Linodes; just type the alias like "nodedev" and it will run the command `ssh bennett@12.34.56.78`
    * aliases can be overridden by:
      * escaping the command eg: `\rm`
      * using the full path to the command eg: `/bin/rm`
  * ##### `unalias`
    * removes (for the current environmnent) the indicated alias
    * eg: `unalias rm`
    * `unalias -a` removes *ALL* aliases set in the current environment for the duration of the session
    * to permanently remove an alias you will need to remove it from the `.bashrc`
  * ##### `function`
    * this is a bash keyword used to indicate that a new bash function follows
    * a function is any custom command that a user can use in a bash shell
    * Q: can you use scripting?  Can you use control flow in a function?
    * example of a function
    ```
    function stuff() {
    ls ~
    ls /opt
    }
    ```
    * then you just enter "stuff" at the command line
    * you will loose this function upon reboot
    * put it in `.bashrc` along with aliases
  * ##### `.(dot)`
    * this command is used to source or apply functions from a file into the current bash session or a shell script
    * this is what you use to make your aliases and functions persistent after a reboot
    * eg: `. .bashrc`
  * ##### `source`
    * same as above but .(dot) is an alias for source
    * eg: `source .bashrc`

  * #### `$PATH`
    * the $PATH environment variable dictates where we have executable programs and other executable scripts
    * if you run `echo $PATH` you will get a listing of directories separated by a colon
    * these are the directories that bash will look through when a command is entered on the command line
    * bash will start at the first path in the list and look in each dir until it finds a match or it does not
    * use a command similar to `export PATH=$PATH:/usr/local/bin` to append a new path to the $PATH environment variable
    * to make any $PATH changes permanent, add the location to `.bash_profile`

## 105.2 Customize or Write Simple Scripts
### Basic Shell Scripts

What is the purpose of a shell scripts?
  * reduce repetitive work
  * useful for automated tasks
  * they are fun!

What are the basics of bash scripting?
  * use a text editor like vim, nano, emacs, etc
  * when naming your scripts, it does not matter what type of .filetype you give it
    * linux looks at the contents of a file to see the type
    * linux does not look at the .extension
    * typically named .sh or nothing at all
  * start with a blank page and add your shebang `#!`
    * and follow up with one of the below
      * `/bin/perl` for perl scripting
      * `/bin/python` for python scripting
      * `/bin/bash` for bash scripting
         * these next parts after the shebang tell bash what program to load up to run the script
    * `#!/bin/bash` is what it will look like
  * `#` will begin a comment line
  * enter any command after your shebang and comment line
    * echo "Hello World"
  * You will need to make your script executable
    * chmod 770
    * chmod o,u+x
  * you can use positional arguments
    * $1 means the first argument
    * $2 means the second argument etc
    * this is the easiest way to pass values into your script
    * eg: `ls $1 $2`

### Adding Logic to Your Shell Scripts
Why would I want to add logical operators to my shell scripts?
  * Logical operations allows us to test for certain types conditions and act on those conditions
  * Ultimately, it makes your scripts smarter.

#### Testing Mechanisms
  * ##### `if` statements
    * an `if` statement checks to see if something is true or not (evaluates a condition)
    * dependent on the outcome of the condition, the script will branch in one of two, a few or many different directions
    * __Basic Structure of an `if` statement__
      * `if [ ]; then`
      ```bash
      # this checks if the /opt directory exists
      if [ -d /opt ]; then
        echo "/opt exists"
      fi
      ```
      * Note that you will need to have a space after the opening bracket and before the closing bracket or you will get a "command not found" error
      * What if the directory we are looking for does not exist?
      * We need to branch with an `else` statement to handle that possibility
      * This is known as an `if else` statement
      ```bash
      if [ -d /nodir ]; then
        echo "/nodir exists"
      else
        echo "/nodir does not exist"
      fi
      ```
      * can use -f to test for a file existed and is a regular file

  * ##### `test`
    * with using `test` you do not need brackets for your conditions
    * the brackets actually represent this `test` command
      * so when you see brackets you can think, "this is a test"
    * the `-e` option below means to test for existence period
    ```bash
    if test -e /etc/hosts; then
      echo "This file exists"
    else
      echo "This file does not exist"
    fi
    ```
    ```
    `-d [ file ]`: `file` exists and is a directory
    `-f [ file ]`: `file` exists and is a regualr file
    `-h [ file ]`: `file` exists and is a symbolic link
    `-r/w/x [ file ]`: `file` exists and user has read, write or execute permissions (as indicated by letter
    `-s [ file ]`: `file` exists and is larger than 0 bytes in size
    ```
    * the `test` keyword can be omitted completely using the "square brackets" method []
      * eg: `if [ -f /home/user/testfile.txt ]`
        * this will test the existence of a file and if it is a regualr file (not a directory file)

  * ##### `||`
    * this is the logical OR operator
    ```bash
    if [[ -d /opt || -f ~/derp ]]; then
      echo "True"
    else
      echo "False"
    fi
    ```
    * this will return "True" because one of the two conditions are True
    * you will notice that we use double square brackets
      * we do this so we can contain both expressions in one set of brackets
      * the double brackets will allow us to use file globing and regEx
      * it is also easier to see the logic in the double brackets
      * otherwise the conditional statement would look like this:
      ```bash
      if [ -d /opt ] || [ -f ~/derp ]; then
      ```
    
  * ##### `&&`
    * this is the logical AND operator
    ```bash
    if [[ -d /opt && -f ~/derp ]]; then
      echo "True"
    else
      echo "False"
    fi
    ```
    * this will return "False" because one of the two conditions are false
    * if both the conditions were true, this would return "True"

  * ##### `=`
    * the equal sign is used to check the equality of a statement
    * the equal sign is used for comparing strings only
      * numbers to represent values
      * strings are just textual representation of numbers
    ```bash
    VALUE="things"
    if [[ "$VALUE"="things" ]]
    ```
    * this will evaluate to true
    * be sure to use soft quotes?  Why?

  * ##### `!=`
    * this character sequence indicates inequality
    ```bash
    VALUE="things"
    if [[ "$VALUE"!="things" ]]
    ```
    * this will evaluate to false
    * NOTE: be careful that you do not run into any logical errors
      * a logical error is an error in the logic of a script or program
      * it is not like a syntatical error in that a syntatical error will not allow a program to run
      * you will get some sort of error at runtime
      * a logical error is one where the script compiles just fine but the output is not what you expect it to be.
      * It is an error in the creation of the thinking of the program and not the program itself

  * ##### `elif` (else-if)
    ```bash
    if [[ $1 = "y" ]]; then
      echo "You said 'yes'."
    elif [[ $1 = "n" ]]; then
      echo "You said 'no'."
    else
      echo "You did not select yes or no."
    fi
    ```

  * ##### Testing Integers
    * `-eq` equal
    * `-ne` not equal to
    * `-gt` greater than
    * `-lt` less than
    * `-ge` greater than or equal
    * `-le` less than or equal
    ```bash
    if [ $NUMBER -gt 2 ]; then
      echo "NUMBER is higher"
    else
      echo "Number is not higher"
    fi
    ```
    
  * ##### `$?` return value
    * this variable you can use to test if an application is successful or not
    * 0 means it is successful and any other number is wrong
    * it is good to use when you need to check if a command ran successfully before you did your next step 
    * please see the example below:
    ```bash
    if [ $? -gt 0 ]; then
      echo "Something went wrong with the 'ls' command."
    else
      echo "Everything is hunky dory, proceed."
    fi
    ```

### Bash Loops and Sequences
This section is all about how to repeat commands using
* ##### `for`
  ```bash
  for i in 1 2 3 4 5
  do
    echo "$i"
  done
  ```
  * each time a loop iterates `i` changes its value from 1 through 5
  ```bash
  for i in $(ls /opt)  # this is command substitution
  do
    echo "$i"
  done
  ```
  * command substitution is when you run a command within another command, and the output of the first command is used as input for another command
  * in the above case we are using the output of the ls command on the /opt directory as input to be use for the for loop
  * command substitution is used/formatted in two different ways
    1. enclose the command you are going to run with `$( )`
      * this opens up a sub-shell 
      * the results of the sub-shell get passed back to the parent shell which is the for loop as it is executed
    2. enclose the command in back ticks 
      * this opens up in-place of the shell that has the for loop
      * this saves you some computing overhead
  * typically number one is used

  ```bash
  for i in $(seq 1 15)
  do
    echo $i
  done
  ```
  * the `seq` is the sequence command
    * will let you step through integer values
    * the default step is one at a time
    * instead of listing out each number
    * good for if you know how many time you want to loop through something
    * can use stepping values
      * $( 1 5 50)
      * the number between the first and last number is the stepping value

* ##### `while`
  ```bash
  COUNT=0
  while [ $COUNT -lt 6 ]
  do
    echo "Our current count is: $COUNT"
    let COUNT=COUNT+1
  done
  ```
  * while loops have a condition is already set
  * you want to run the loop until the condition is no longer set
  * while = do the thing while true
  * while this is true, do this

* ##### `until`
  ```bash
  COUNT=10
  until [ $COUNT -lt 1 ]
  do
    echo "Our count is: $COUNT"
    let COUNT=COUNT-1
  done
  ```
  * you have a condition that is set and is false
  * until = do the thing while false
  * until this is true, do this

* ##### `read`
  ```bash
  while read LINE
  do
    echo "$LINE"
  done < for.sh
  ```
  * the `read` command takes input from the user (or file) and applies that input to a variable for a command or script
  * use read to get input from a user in a script
  * this can also be used to read in lines from a file

* ##### `exit`
  * the `exit` command in a script will return a 0 to the operating system
  * you can specify a different exit code for troubleshooting purposes
  * `exit 45` will change the exit return value from 0 to 45
  * this is useful in trouble shooting and setting up different exit codes so you know where your script succeeds or fails.
  ```bash
  touch /etc/mytest 2> /dev/null

  if [ $? -eq 0 ]; then
    echo "mytest file created"
  else
    echo "mytest file not created"
    exit 75
  fi
  ```

* ##### `exec`
  * the `exec` command can but used to redirect all output from a shell into a file (or another process) without sending it to the current shell.
  * for instance you can use the exec command to redirect output of a script to a log file
