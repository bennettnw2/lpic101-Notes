# 103.1 Work on the Command Line
## Bash Shell Environment
* The shell is the CLI that you work with in on a Linux system
* The shell is text based and not GUI
  * bash - bourne again shell - the default
  * csh - C programing style syntax
  * ksh - KornShell, based on bourne with some feature of the c shell added
  * zsh - Z shell is a mix between Bash and Korn

### Bash Environment
* Environment Variables
  * settings that dictate common functionality and locations for various purposes
  * syntax: VARIABLE=path|command|alias
  * example: CWD=/home/user/Documents
* Bash Functions
  * Users can create their own custom function within Bash
  * Example
    ```
    function yo() {
      echo "yo"
    }
    ```
    * basically start with the function keyword, give it a name with parens and open curly brace

### Common Bash Environment Commands
  * ##### `env`
    * will display all the environment variables
    * can grep the `env` command to find a specific variable
  * ##### `echo`
    * this can do many things; can print the value of a variable to the screen
    * example: `echo $PATH`
    * enviro variable will always be uppercase and preceded by "$"
  * #### `set`
    * will display the shell settings _and_ shell variables for the session
    * can use `env` and `set` to see your env var
    * `set | less` since the output is vast, it is best to pipe through less
    * `set -x` will turn on debugging
      * this will show you what happens in the background as they are run
      * `set +x` will turn off debugging
  * ##### `unset`
    * will remove a variable or custom bash function
    * example: `unset yo` will turn the function yo() back into a non-command 
  * ##### `shopt`
    * "show options" will show all the options set for your current bash environment
      * histreedit
      * `shopt -s histreedit` -s means to set the option
  * ##### `export`
    * this command is used to export a variable to the current shell and any child shells started
    * NOTE: start a child shell by using bash, su, su -
  * ##### `pwd`
    * "print working directory"
    * shows your current working directory
    * gets its info from the $PWD variable
  * ##### `which`
    * shows which directory an application resides
    * also good for seeing if an application is installed on your system
    * `which pwd` for example
  * ##### `type`
    * used to findout if a command is an internal built-in command or external binary application
    * `type cd` is a function
    * `type ls` is an alias
    * `type type` is a built-in function

### Bash Quoting
  * Quoting is all about how the contents withing a particular type of quotes will get interpreted
    * "weak" quotes
      * seem to me that they are not even there
      * variables will be expanded in weak quotes
      * however, charaters for path substituion or pattern matching will not be expanded 
    * 'strong' quotes
      * will make it so that variables do not expand
      * characters in between strong quotes will make it a string literal

## Bash History and the Manual Pages
The bash histry records commands which is very helpful and useful.  In addition, everything in Linux has a user's manual which is easily accessibel and also very helpful and useful.  Learning how to utilize both is crucial for effectively Administering Linux systems.

### Bash History
##### `history`
  * this will show the most recently ran commands in numerical order
  * use up and down arrow keys to go through the history while on the cmd line
  * can use ![location number] to rerun a command
  * `~/.bash_history` is where the data is written to and stored for the `history` command
  * the env variable `HISTFILESIZE` dictates how many entry lines of history are stored
    * `echo $HISTFILESIZE`
    * can set `HISTAPPEND` to keep adding new entries once you reach

### Manual Pages
* Built-in documentation for commands, configuration files and system admisitration tasks
##### `man [command name]`
  * this is how you launch the man page for any given command
  * Man pages are broken into sections:
    * Section 1: Executable programs or shell commands
    * Section 2: System calls - functions provided by the kernel
    * Section 3: Library calls - functions within in program libraries
    * Section 4: Special files - typically those found in /dev
    * Section 5: File formats and conventions - for example /etc/passwd and other config files
    * Section 6: Games
    * Section 7: Miscellaneous items and conftntions - eg: man(7), regex(7)
    * Section 8: System administration commands - usually only for root
    * Section 9: Kernel rounties (non standard)
  * Examples of `man` usage
    * `man ls`
      * gives us a page with LS(1) at the top left corner this means we are in section 1
        * section 1 pertains to user commands
      * within each page we have a heading
        * `NAME`
          * the name of the command with a short description
          * eg: `ls - list directory contents`
        * `SYNOPSIS`
          * short hand on how to use the command
          * eg: `ls [OPTION]... [FILE]...`
        * `DESCRIPTION`
          * here we will have a longer description along with all the options and flags you can set
        * `AUTHOR`
        * `COPYRIGHT`
        * `SEE ALSO`
          * this section is very important as it lets you know were you can find more details regarding a command.
  * What if you have a command you want to look up with man but you don't quite know what that command is?  Or you don't know which man page you want to use?
  * You can do a man page search by a keyword
    * ##### `man -k [keyword]`
    * this search would return the manpage if the keyword matched one
    * this search would also return the other manpages where ever this keyword appears in
    * the key world could appear in the title or in the description
    * ##### `appropos [keyword]`
      * will do the same thing as `man -k [keyword]`
  * How do we open up a specific section of a man page for a keyword?
  * ##### `man [n] [keyword]`
    * n is the section number you want to look at


# 103.2 Process Text Streams using Filters
## Text File Statistics
Learn how to glean statistics from files that will help you to test files
### Common Commands

##### `nl`
  * "number line" will show us how many, non - blank lines there are in a file
  * to include the blank lines we use the `-b` switch followed by an `a`
    * `nl -b a [filename]`

##### `wc`
  * "word count" will show us how many lines, words, and bytes are contained in a file
  * uses white space between charaters to determine an individual word
  * `wc text.sh` => `3 4 30 test.sh`
    * the numbers are lines, words, bytes
    * by default, wc will count blank lines
    * can use flags to get individual info
      * `-l` will give us all lines
      * `-w` will give us a word count
      * `-c` will give us the byte count

##### `od`
  * "octal dump` will print out a file in octal or any other format the user specifies
  * this will not show you statistics per say but will give you deeper information regarding the structure of a file
  * by default, the output of `od` will give you the contents of a file in octal format
  * __INSERT GRAPHIC__
    * the first coloumn is the byte offset
    * the octal repesentation of the data that is within the byte offset
  * you can use other switches to make the data human readable
    * `od -c test.sh` will show us the data in character format
    __INSERT GRAPHIC__
  * so it looks like each ofset is 16 characters
    * `od -a test.sh` will show data in ascii format
  * `od` is helpful to locate rouge data that should not be in a file

### Message Digests
These are the fingerprints of a file.  It is a hashed value of the contents in a file.  Like when you download a file and you get an extra text file.  You compare the current hased value with that of the text file.  It it a great way to make sure that you do not get a corrupt file or one that has been tampered with.

##### `md5sum`
  * this is the weakest of the cryptographic algorithms but it is still useful
  * to check the hash value you use the 
  * `md5sum test.sh` or `mdfsum test.sh > test.md5`
  * if we add another command to our test.sh script, it will change the md5 hash
  * run the command `md5sum -c test.md5` to test if the md5 are the same

#### `sha256`
  * this is much more stronger than `md5sum`
  * `sha256sum -c test.sha256`

#### `sha512`
  * this is much more stronger than `sha512`
  * `sha512sum -c test.sha512`



## Text Manipulation

## More Text Manipulation




# 103.3 Perform Basic File Management

# 103.4 Use Streams, Pipes and Redirects

# 103.5 Create, Monitor, and Kill Processes

# 103.6 Modify Process Execution Priorities

# 103.7 Search Text Files using Regular Expressions

# 103.8 Perform Basic File Editing Operatins in vi

