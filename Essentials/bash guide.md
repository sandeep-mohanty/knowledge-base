# Bash Guide - Gamedev Guide

[](https://github.com/ikrima/gamedevguide/blob/master/docs/dev-notes/linux/bash-guide.md "Edit this page")[](https://github.com/ikrima/gamedevguide/raw/master/docs/dev-notes/linux/bash-guide.md "View source of this page")

originally from [Idnan/bash-guide](https://github.com/Idnan/bash-guide/)

* * *

[![bash logo](https://cloud.githubusercontent.com/assets/2059754/24601246/753a7f36-1858-11e7-9d6b-7a0e64fb27f7.png)](https://cloud.githubusercontent.com/assets/2059754/24601246/753a7f36-1858-11e7-9d6b-7a0e64fb27f7.png)

## Table of Contents[#](#table-of-contents "Permanent link")

1.  [Basic Operations](./#1-basic-operations)  
    1.1. [File Operations](./#11-file-operations)  
    1.2. [Text Operations](./#12-text-operations)  
    1.3. [Directory Operations](./#13-directory-operations)  
    1.4. [SSH, System Info & Network Operations](./#14-ssh-system-info--network-operations)  
    1.5. [Process Monitoring Operations](./#15-process-monitoring-operations)
2.  [Basic Shell Programming](./#2-basic-shell-programming)  
    2.1. [Variables](./#21-variables)  
    2.2. [Array](./#22-array)  
    2.3. [String Substitution](./#23-string-substitution)  
    2.4. [Other String Tricks](./#24-other-string-tricks)  
    2.5. [Functions](./#25-functions)  
    2.6. [Conditionals](./#26-conditionals)  
    2.7. [Loops](./#27-loops)  
    2.8. [Regex](./#28-regex)  
    2.9. [Pipes](./#29-pipes)
3.  [Tricks](./#3-tricks)
4.  [Debugging](./#4-debugging)
5.  [Multi-threading](./#5-multi-threading)

## 1\. Basic Operations[#](#1-basic-operations "Permanent link")

### a. `export`[#](#a-export "Permanent link")

Displays all environment variables. If you want to get details of a specific variable, use `echo $VARIABLE_NAME`.

Example:

Bash

`$ export AWS_HOME=/Users/adnanadnan/.aws LANG=en_US.UTF-8 LC_CTYPE=en_US.UTF-8 LESS=-R $ echo $AWS_HOME /Users/adnanadnan/.aws`

### b. `whatis`[#](#b-whatis "Permanent link")

whatis shows description for user commands, system calls, library functions, and others in manual pages

Example:

Bash

`$ whatis bash bash (1)             - GNU Bourne-Again SHell`

### c. `whereis`[#](#c-whereis "Permanent link")

whereis searches for executables, source files, and manual pages using a database built by system automatically.

Example:

Bash

`$ whereis php /usr/bin/php`

### d. `which`[#](#d-which "Permanent link")

which searches for executables in the directories specified by the environment variable PATH. This command will print the full path of the executable(s).

Example:

Bash

`$ which php /c/xampp/php/php`

### e. clear[#](#e-clear "Permanent link")

Clears content on window.

## 1.1. File Operations[#](#11-file-operations "Permanent link")

[cat](#a-cat)

[chmod](#b-chmod)

[chown](#c-chown)

[cp](#d-cp)

[diff](#e-diff)

[file](#f-file)

[find](#g-find)

[gunzip](#h-gunzip)

[gzcat](#i-gzcat)

[gzip](#j-gzip)

[head](#k-head)

[less](#l-less)

[lpq](#m-lpq)

[lpr](#n-lpr)

[lprm](#o-lprm)

[ls](#p-ls)

[more](#q-more)

[mv](#r-mv)

[rm](#s-rm)

[tail](#t-tail)

[touch](#u-touch)

### a. `cat`[#](#a-cat "Permanent link")

It can be used for the following purposes under UNIX or Linux.

-   Display text files on screen
-   Copy text files
-   Combine text files
-   Create new text files

Bash

`cat filename cat file1 file2  cat file1 file2 > newcombinedfile cat < file1 > file2 #copy file1 to file2`

### b. `chmod`[#](#b-chmod "Permanent link")

The chmod command stands for "change mode" and allows you to change the read, write, and execute permissions on your files and folders. For more information on this command check this [link](https://ss64.com/bash/chmod.html).

Bash

`chmod -options filename`

### c. `chown`[#](#c-chown "Permanent link")

The chown command stands for "change owner", and allows you to change the owner of a given file or folder, which can be a user and a group. Basic usage is simple forward first comes the user (owner), and then the group, delimited by a colon.

Bash

`chown -options user:group filename`

### d. `cp`[#](#d-cp "Permanent link")

Copies a file from one location to other.

Bash

`cp filename1 filename2`

Where `filename1` is the source path to the file and `filename2` is the destination path to the file.

### e. `diff`[#](#e-diff "Permanent link")

Compares files, and lists their differences.

Bash

`diff filename1 filename2`

### f. `file`[#](#f-file "Permanent link")

Determine file type.

Example:

Bash

`$ file index.html  index.html: HTML document, ASCII text`

### g. `find`[#](#g-find "Permanent link")

Find files in directory

Bash

`find directory options pattern`

Example:

Bash

`$ find . -name README.md $ find /home/user1 -name '*.png'`

### h. `gunzip`[#](#h-gunzip "Permanent link")

Un-compresses files compressed by gzip.

### i. `gzcat`[#](#i-gzcat "Permanent link")

Lets you look at gzipped file without actually having to gunzip it.

### j. `gzip`[#](#j-gzip "Permanent link")

Compresses files.

### k. `head`[#](#k-head "Permanent link")

Outputs the first 10 lines of file

### l. `less`[#](#l-less "Permanent link")

Shows the contents of a file or a command output, one page at a time. It is similar to [more](./#q-more), but has more advanced features and allows you to navigate both forward and backward through the file.

### m. `lpq`[#](#m-lpq "Permanent link")

Check out the printer queue.

Example:

Bash

`$ lpq Rank    Owner   Job     File(s)                         Total Size active  adnanad 59      demo                            399360 bytes 1st     adnanad 60      (stdin)                         0 bytes`

### n. `lpr`[#](#n-lpr "Permanent link")

Print the file.

### o. `lprm`[#](#o-lprm "Permanent link")

Remove something from the printer queue.

### p. `ls`[#](#p-ls "Permanent link")

Lists your files. `ls` has many options: `-l` lists files in 'long format', which contains the exact size of the file, who owns the file, who has the right to look at it, and when it was last modified. `-a` lists all files, including hidden files. For more information on this command check this [link](https://ss64.com/bash/ls.html).

Example:

$ ls -la
rwxr-xr-x   33 adnan  staff    1122 Mar 27 18:44 .
drwxrwxrwx  60 adnan  staff    2040 Mar 21 15:06 ..
-rw-r--r--@  1 adnan  staff   14340 Mar 23 15:05 .DS\_Store
-rw-r--r--   1 adnan  staff     157 Mar 25 18:08 .bumpversion.cfg
-rw-r--r--   1 adnan  staff    6515 Mar 25 18:08 .config.ini
-rw-r--r--   1 adnan  staff    5805 Mar 27 18:44 .config.override.ini
drwxr-xr-x  17 adnan  staff     578 Mar 27 23:36 .git
-rwxr-xr-x   1 adnan  staff    2702 Mar 25 18:08 .gitignore

### q. `more`[#](#q-more "Permanent link")

Shows the first part of a file (move with space and type q to quit).

### r. `mv`[#](#r-mv "Permanent link")

Moves a file from one location to other.

Bash

`mv filename1 filename2`

Where `filename1` is the source path to the file and `filename2` is the destination path to the file.

Also it can be used for rename a file.

### s. `rm`[#](#s-rm "Permanent link")

Removes a file. Using this command on a directory gives you an error.  
`rm: directory: is a directory`  
To remove a directory you have to pass `-r` which will remove the content of the directory recursively. Optionally you can use `-f` flag to force the deletion i.e. without any confirmations etc.

### t. `tail`[#](#t-tail "Permanent link")

Outputs the last 10 lines of file. Use `-f` to output appended data as the file grows.

### u. `touch`[#](#u-touch "Permanent link")

Updates access and modification time stamps of your file. If it doesn't exists, it'll be created.

Example:

## 1.2. Text Operations[#](#12-text-operations "Permanent link")

[awk](#a-awk)

[cut](#b-cut)

[echo](#c-echo)

[egrep](#d-egrep)

[fgrep](#e-fgrep)

[fmt](#f-fmt)

[grep](#g-grep)

[nl](#h-nl)

[sed](#i-sed)

[sort](#j-sort)

[tr](#k-tr)

[uniq](#l-uniq)

[wc](#m-wc)

### a. `awk`[#](#a-awk "Permanent link")

awk is the most useful command for handling text files. It operates on an entire file line by line. By default it uses whitespace to separate the fields. The most common syntax for awk command is

Bash

`awk '/search_pattern/ { action_to_take_if_pattern_matches; }' file_to_parse`

Lets take following file `/etc/passwd`. Here's the sample data that this file contains:

Text Only

`root:x:0:0:root:/root:/usr/bin/zsh daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin bin:x:2:2:bin:/bin:/usr/sbin/nologin sys:x:3:3:sys:/dev:/usr/sbin/nologin sync:x:4:65534:sync:/bin:/bin/sync`

So now lets get only username from this file. Where `-F` specifies that on which base we are going to separate the fields. In our case it's `:`. `{ print $1 }` means print out the first matching field.

Bash

`awk -F':' '{ print $1 }' /etc/passwd`

After running the above command you will get following output.

Text Only

`root daemon bin sys sync`

For more detail on how to use `awk`, check following [link](https://www.cyberciti.biz/faq/bash-scripting-using-awk).

### b. `cut`[#](#b-cut "Permanent link")

Remove sections from each line of files

_example.txt_

Bash

`red riding hood went to the park to play`

_show me columns 2 , 7 , and 9 with a space as a separator_

Bash

`cut -d " " -f2,7,9 example.txt`

### c. `echo`[#](#c-echo "Permanent link")

Display a line of text

_display "Hello World"_

_display "Hello World" with newlines between words_

Bash

`echo -ne "Hello\nWorld\n"`

### d. `egrep`[#](#d-egrep "Permanent link")

Print lines matching a pattern - Extended Expression (alias for: 'grep -E')

_example.txt_

Bash

`Lorem ipsum dolor sit amet,  consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet.`

_display lines that have either "Lorem" or "dolor" in them._

Bash

`egrep '(Lorem|dolor)' example.txt or grep -E '(Lorem|dolor)' example.txt`

Bash

`Lorem ipsum dolor sit amet, et dolore magna duo dolores et ea sanctus est Lorem ipsum dolor sit`

### e. `fgrep`[#](#e-fgrep "Permanent link")

Print lines matching a pattern - FIXED pattern matching (alias for: 'grep -F')

_example.txt_

Bash

`Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor foo (Lorem|dolor)  invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet.`

_Find the exact string '(Lorem|dolor)' in example.txt_

Bash

`fgrep '(Lorem|dolor)' example.txt or grep -F '(Lorem|dolor)' example.txt`

### f. `fmt`[#](#f-fmt "Permanent link")

Simple optimal text formatter

_example: example.txt (1 line)_

Bash

`Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet.`

_output the lines of example.txt to 20 character width_

Bash

`cat example.txt | fmt -w 20`

Bash

`Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet.`

### g. `grep`[#](#g-grep "Permanent link")

Looks for text inside files. You can use grep to search for lines of text that match one or many regular expressions, and outputs only the matching lines.

Bash

`grep pattern filename`

Example:

Bash

`$ grep admin /etc/passwd _kadmin_admin:*:218:-2:Kerberos Admin Service:/var/empty:/usr/bin/false _kadmin_changepw:*:219:-2:Kerberos Change Password Service:/var/empty:/usr/bin/false _krb_kadmin:*:231:-2:Open Directory Kerberos Admin Service:/var/empty:/usr/bin/false`

You can also force grep to ignore word case by using `-i` option. `-r` can be used to search all files under the specified directory, for example:

Bash

`$ grep -r admin /etc/`

And `-w` to search for words only. For more detail on `grep`, check following [link](https://www.cyberciti.biz/faq/grep-in-bash).

### h. `nl`[#](#h-nl "Permanent link")

Number lines of files

_example.txt_

Bash

`Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet.`

_show example.txt with line numbers_

Bash

`nl -s". " example.txt` 

Bash

     `1. Lorem ipsum     2. dolor sit amet,     3. consetetur     4. sadipscing elitr,     5. sed diam nonumy     6. eirmod tempor     7. invidunt ut labore     8. et dolore magna     9. aliquyam erat, sed    10. diam voluptua. At    11. vero eos et    12. accusam et justo    13. duo dolores et ea    14. rebum. Stet clita    15. kasd gubergren,    16. no sea takimata    17. sanctus est Lorem    18. ipsum dolor sit    19. amet.`

### i. `sed`[#](#i-sed "Permanent link")

Stream editor for filtering and transforming text

_example.txt_

Bash

`Hello This is a Test 1 2 3 4`

_replace all spaces with hyphens_

Bash

`sed 's/ /-/g' example.txt`

Bash

`Hello-This-is-a-Test-1-2-3-4`

_replace all digits with "d"_

Bash

`sed 's/[0-9]/d/g' example.txt`

Bash

`Hello This is a Test d d d d`

### j. `sort`[#](#j-sort "Permanent link")

Sort lines of text files

_example.txt_

_sort example.txt_

_randomize a sorted example.txt_

Bash

`sort example.txt | sort -R`

### k. `tr`[#](#k-tr "Permanent link")

Translate or delete characters

_example.txt_

Bash

`Hello World Foo Bar Baz!`

_take all lower case letters and make them upper case_

Bash

`cat example.txt | tr 'a-z' 'A-Z'` 

Bash

`HELLO WORLD FOO BAR BAZ!`

_take all spaces and make them into newlines_

Bash

`cat example.txt | tr ' ' '\n'`

Bash

`Hello World Foo Bar Baz!`

### l. `uniq`[#](#l-uniq "Permanent link")

Report or omit repeated lines

_example.txt_

_show only unique lines of example.txt (first you need to sort it, otherwise it won't see the overlap)_

Bash

`sort example.txt | uniq`

_show the unique items for each line, and tell me how many instances it found_

Bash

`sort example.txt | uniq -c`

### m. `wc`[#](#m-wc "Permanent link")

Tells you how many lines, words and characters there are in a file.

Example:

Bash

`$ wc demo.txt 7459   15915  398400 demo.txt`

Where `7459` is lines, `15915` is words and `398400` is characters.

## 1.3. Directory Operations[#](#13-directory-operations "Permanent link")

### a. `cd`[#](#a-cd "Permanent link")

Moves you from one directory to other. Running this

moves you to home directory. This command accepts an optional `dirname`, which moves you to that directory.

Switch to the previous working directory

### b. `mkdir`[#](#b-mkdir "Permanent link")

Makes a new directory.

You can use this to create multiple directories at once within your current directory.

Bash

`mkdir 1stDirectory 2ndDirectory 3rdDirectory`

You can also use this to create parent directories at the same time with the -p (or --parents) flag. For instance, if you wanted a directory named 'project1' in another subdirectory at '/samples/bash/projects/', you could run:

Bash

`mkdir -p /samples/bash/projects/project1 mkdir --parents /samples/bash/projects/project1`

Both commands above will do the same thing.  
If any of these directories did no already exist, they would be created as well.

### c. `pwd`[#](#c-pwd "Permanent link")

Tells you which directory you currently are in.

## 1.4. SSH, System Info & Network Operations[#](#14-ssh-system-info--network-operations "Permanent link")

[bg](#a-bg)

[cal](#b-cal)

[date](#c-date)

[df](#d-df)

[dig](#e-dig)

[du](#f-du)

[fg](#g-fg)

[finger](#h-finger)

[jobs](#i-jobs)

[last](#j-last)

[man](#k-man)

[passwd](#l-passwd)

[ping](#m-ping)

[ps](#n-ps)

[quota](#o-quota)

[scp](#p-scp)

[ssh](#q-ssh)

[top](#r-top)

[uname](#s-uname)

[uptime](#t-uptime)

[w](#u-w)

[wget](#v-wget)

[whoami](#w-whoami)

[whois](#x-whois)

[sync](#y-rsync)

[curl](#z-curl)

### a. `bg`[#](#a-bg "Permanent link")

Lists stopped or background jobs; resume a stopped job in the background.

### b. `cal`[#](#b-cal "Permanent link")

Shows the month's calendar.

### c. `date`[#](#c-date "Permanent link")

Shows the current date and time.

### d. `df`[#](#d-df "Permanent link")

Shows disk usage.

### e. `dig`[#](#e-dig "Permanent link")

Gets DNS information for domain.

### f. `du`[#](#f-du "Permanent link")

Shows the disk usage of files or directories. For more information on this command check this [link](http://www.linfo.org/du.html)

Bash

`du [option] [filename|directory]`

Options:

-   `-h` (human readable) Displays output it in kilobytes (K), megabytes (M) and gigabytes (G).
-   `-s` (supress or summarize) Outputs total disk space of a directory and supresses reports for subdirectories.

Example:

Bash

`du -sh pictures 1.4M pictures`

### g. `fg`[#](#g-fg "Permanent link")

Brings the most recent job in the foreground.

### h. `finger`[#](#h-finger "Permanent link")

Displays information about user.

### i. `jobs`[#](#i-jobs "Permanent link")

Lists the jobs running in the background, giving the job number.

### j. `last`[#](#j-last "Permanent link")

Lists your last logins of specified user.

### k. `man`[#](#k-man "Permanent link")

Shows the manual for specified command.

### l. `passwd`[#](#l-passwd "Permanent link")

Allows the current logged user to change their password.

### m. `ping`[#](#m-ping "Permanent link")

Pings host and outputs results.

### n. `ps`[#](#n-ps "Permanent link")

Lists your processes.

Use the flags ef. e for every process and f for full listing.

### o. `quota`[#](#o-quota "Permanent link")

Shows what your disk quota is.

### p. `scp`[#](#p-scp "Permanent link")

Transfer files between a local host and a remote host or between two remote hosts.

_copy from local host to remote host_

Bash

`scp source_file user@host:directory/target_file`

_copy from remote host to local host_

Bash

`scp user@host:directory/source_file target_file scp -r user@host:directory/source_folder target_folder`

This command also accepts an option `-P` that can be used to connect to specific port.

Bash

`scp -P port user@host:directory/source_file target_file`

### q. `ssh`[#](#q-ssh "Permanent link")

ssh (SSH client) is a program for logging into and executing commands on a remote machine.

This command also accepts an option `-p` that can be used to connect to specific port.

Bash

`ssh -p port user@host`

### r. `top`[#](#r-top "Permanent link")

Displays your currently active processes.

### s. `uname`[#](#s-uname "Permanent link")

Shows kernel information.

### t. `uptime`[#](#t-uptime "Permanent link")

Shows current uptime.

### u. `w`[#](#u-w "Permanent link")

Displays who is online.

### v. `wget`[#](#v-wget "Permanent link")

Downloads file.

### w. `whoami`[#](#w-whoami "Permanent link")

Return current logged in username.

### x. `whois`[#](#x-whois "Permanent link")

Gets whois information for domain.

### y. `rsync`[#](#y-rsync "Permanent link")

Does the same job as `scp` command, but transfers only changed files. Useful when transferring the same folder to/from server multiple times.

Bash

`rsync source_folder user@host:target_folder rsync user@host:target_folder target_folder`

### z. `curl`[#](#z-curl "Permanent link")

Curl is a command-line tool for requesting or sending data using URL syntax. Usefull on systems where you only have terminal available for making various requests.

Use `-X` or `--request` to specify which method you would like invoke (GET, POST, DELETE, ...).  
Use `-d <data>` or `--data <data>` to POST data on given URL.

## 1.5. Process Monitoring Operations[#](#15-process-monitoring-operations "Permanent link")

### a. `kill`[#](#a-kill "Permanent link")

Kills (ends) the processes with the ID you gave.

### b. `killall`[#](#b-killall "Permanent link")

Kill all processes with the name.

### c. &[#](#c- "Permanent link")

The `&` symbol instructs the command to run as a background process in a subshell.

### d. `nohup`[#](#d-nohup "Permanent link")

nohup stands for "No Hang Up". This allows to run command/process or shell script that can continue running in the background after you log out from a shell.

Combine it with `&` to create background processes

## 2\. Basic Shell Programming[#](#2-basic-shell-programming "Permanent link")

The first line that you will write in bash script files is called `shebang`. This line in any script determines the script's ability to be executed like a standalone executable without typing sh, bash, python, php etc beforehand in the terminal.

## 2.1. Variables[#](#21-variables "Permanent link")

Creating variables in bash is similar to other languages. There are no data types. A variable in bash can contain a number, a character, a string of characters, etc. You have no need to declare a variable, just assigning a value to its reference will create it.

Example:

The above line creates a variable `str` and assigns "hello world" to it. The value of variable is retrieved by putting the `$` in the beginning of variable name.

Example:

Bash

`echo $str   # hello world`

## 2.2. Array[#](#22-array "Permanent link")

Like other languages bash has also arrays. An array is a variable containing multiple values. There's no maximum limit on the size of array. Arrays in bash are zero based. The first element is indexed with element 0. There are several ways for creating arrays in bash which are given below.

Examples:

Bash

`array[0]=val array[1]=val array[2]=val array=([2]=val [0]=val [1]=val) array=(val val val)`

To display a value at specific index use following syntax:

Bash

`${array[i]}     # where i is the index`

If no index is supplied, array element 0 is assumed. To find out how many values there are in the array use the following syntax:

Bash has also support for the ternary conditions. Check some examples below.

Bash

`${varname:-word}    # if varname exists and isn't null, return its value; otherwise return word ${varname:=word}    # if varname exists and isn't null, return its value; otherwise set it word and then return its value ${varname:+word}    # if varname exists and isn't null, return word; otherwise return null ${varname:offset:length}    # performs substring expansion. It returns the substring of $varname starting at offset and up to length characters`

## 2.3 String Substitution[#](#23-string-substitution "Permanent link")

Check some of the syntax on how to manipulate strings

Bash

`${variable#pattern}         # if the pattern matches the beginning of the variable's value, delete the shortest part that matches and return the rest ${variable##pattern}        # if the pattern matches the beginning of the variable's value, delete the longest part that matches and return the rest ${variable%pattern}         # if the pattern matches the end of the variable's value, delete the shortest part that matches and return the rest ${variable%%pattern}        # if the pattern matches the end of the variable's value, delete the longest part that matches and return the rest ${variable/pattern/string}  # the longest match to pattern in variable is replaced by string. Only the first match is replaced ${variable//pattern/string} # the longest match to pattern in variable is replaced by string. All matches are replaced ${#varname}     # returns the length of the value of the variable as a character string`

## 2.4. Other String Tricks[#](#24-other-string-tricks "Permanent link")

Bash has multiple shorthand tricks for doing various things to strings.

Bash

`${variable,,}    #this converts every letter in the variable to lowercase ${variable^^}    #this converts every letter in the variable to uppercase ${variable:2:8}  #this returns a substring of a string, starting at the character at the 2 index(strings start at index 0, so this is the 3rd character),                  #the substring will be 8 characters long, so this would return a string made of the 3rd to the 11th characters.`

Here are some handy pattern matching tricks

Bash

`if [[ "$variable" == *subString* ]]  #this returns true if the provided substring is in the variable if [[ "$variable" != *subString* ]]  #this returns true if the provided substring is not in the variable if [[ "$variable" == subString* ]]   #this returns true if the variable starts with the given subString if [[ "$variable" == *subString ]]   #this returns true if the variable ends with the given subString`

The above can be shortened using a case statement and the IN keyword

Bash

`case "$var" in 	begin*) 		#variable begins with "begin" 	;; 	*subString*) 		#subString is in variable 	;; 	*otherSubString*) 		#otherSubString is in variable 	;; esac`

## 2.5. Functions[#](#25-functions "Permanent link")

As in almost any programming language, you can use functions to group pieces of code in a more logical way or practice the divine art of recursion. Declaring a function is just a matter of writing function my\_func { my\_code }. Calling a function is just like calling another program, you just write its name.

Bash

`function name() {     shell commands }`

Example:

Bash

`#!/bin/bash function hello {    echo world! } hello function say {     echo $1 } say "hello world!"`

When you run the above example the `hello` function will output "world!". The above two functions `hello` and `say` are identical. The main difference is function `say`. This function, prints the first argument it receives. Arguments, within functions, are treated in the same manner as arguments given to the script.

## 2.6. Conditionals[#](#26-conditionals "Permanent link")

The conditional statement in bash is similar to other programming languages. Conditions have many form like the most basic form is `if` expression `then` statement where statement is only executed if expression is true.

Bash

`if [ expression ]; then     will execute only if expression is true else     will execute if expression is false fi`

Sometime if conditions becoming confusing so you can write the same condition using the `case statements`.

Bash

`case expression in     pattern1 )        statements ;;    pattern2 )        statements ;;    ... esac`

Expression Examples:

Bash

`statement1 && statement2  # both statements are true statement1 || statement2  # at least one of the statements is true str1=str2       # str1 matches str2 str1!=str2      # str1 does not match str2 str1<str2       # str1 is less than str2 str1>str2       # str1 is greater than str2 -n str1         # str1 is not null (has length greater than 0) -z str1         # str1 is null (has length 0) -a file         # file exists -d file         # file exists and is a directory -e file         # file exists; same -a -f file         # file exists and is a regular file (i.e., not a directory or other special type of file) -r file         # you have read permission -s file         # file exists and is not empty -w file         # you have write permission -x file         # you have execute permission on file, or directory search permission if it is a directory -N file         # file was modified since it was last read -O file         # you own file -G file         # file's group ID matches yours (or one of yours, if you are in multiple groups) file1 -nt file2     # file1 is newer than file2 file1 -ot file2     # file1 is older than file2 -lt     # less than -le     # less than or equal -eq     # equal -ge     # greater than or equal -gt     # greater than -ne     # not equal`

## 2.7. Loops[#](#27-loops "Permanent link")

There are three types of loops in bash. `for`, `while` and `until`.

Different `for` Syntax:

Bash

`for name [in list] do   statements that can use $name done for (( initialisation ; ending condition ; update )) do   statements... done`

`while` Syntax:

Bash

`while condition; do   statements done`

`until` Syntax:

Bash

`until condition; do   statements done`

## 2.8. Regex[#](#28-regex "Permanent link")

They are a powerful tool for manipulating and searching text. Here are some examples of regular expressions that use each `metacharacter`:

[\`.\`(dot)](#a-dot)

[\`\*\`(asterisk)](#b-asterisk)

[\`+\`(plus)](#c-plus)

[\`?\`(question mark)](#d-question_mark)

[\`|\`(pipe)](#c-plus)

[\`\[\]\`(character class)](#c-plus)

[\`\[^\]\`(negated character class)](#c-plus)

[\`()\`(grouping)](#c-plus)

[\`{}\`(quantifiers)](#c-plus)

[\`\\\`(escape)](#c-plus)

### a. `.` (dot)[#](#a--dot "Permanent link")

Matches any single character except newline.

Output:

### b. `*` (asterisk)[#](#b--asterisk "Permanent link")

Matches zero or more occurrences of the preceding character or group.

Output:

### c. `+` (plus)[#](#c--plus "Permanent link")

Matches one or more occurrences of the preceding character or group.

Output:

Bash

`abc abbc abbbc abbbbc`

### d. `?` (question mark)[#](#d--question-mark "Permanent link")

Matches zero or one occurrence of the preceding character or group.

Output:

### e. `|` (pipe)[#](#e--pipe "Permanent link")

Matches either the pattern to the left or the pattern to the right.

Bash

`egrep "cat|dog" file.txt`

Output:

### f. `[]` (character class)[#](#f--character-class "Permanent link")

Matches any character inside the brackets.

Bash

`[aeiou] will match any vowel [a-z] will match any lowercase letter`

### g. `[]` (negated character class)[#](#g--negated-character-class "Permanent link")

Matches any character not inside the brackets.

Bash

`[^aeiou] will match any consonant [^a-z] will match any non-lowercase letter`

### h. `()` (grouping)[#](#h--grouping "Permanent link")

Groups multiple tokens together and creates a capture group.

Bash

`egrep "(ab)+" file.txt`

Output:

### i. `{}` (quantifiers)[#](#i--quantifiers "Permanent link")

Matches a specific number of occurrences of the preceding character or group.

Bash

`egrep "a{3}" file.txt`

Output:

### j. `\` (escape)[#](#j--escape "Permanent link")

Escapes the next character to match it literally.

Output:

\=======

## 2.9. Pipes[#](#29-pipes "Permanent link")

Multiple commands can be linked together with a pipe, `|`. A `|` will send the standard-output from command A to the standard-input of command B.  
Pipes can also be constructed with the `|&` symbols. This will send the standard-output **and** standard-error from command A to the standard-input of command B.

## 3\. Tricks[#](#3-tricks "Permanent link")

## Set an alias[#](#set-an-alias "Permanent link")

Run `nano ~/.bash_profile` and add the following line:

Bash

`alias dockerlogin='ssh [[email protected]](/cdn-cgi/l/email-protection) -p2222'  # add your alias in .bash_profile`

## To quickly go to a specific directory[#](#to-quickly-go-to-a-specific-directory "Permanent link")

Run `nano ~/.bashrc` and add the following line:

Bash

`export hotellogs="/workspace/hotel-api/storage/logs"`

Now you can use the saved path:

Bash

`source ~/.bashrc cd $hotellogs`

## Re-execute the previous command[#](#re-execute-the-previous-command "Permanent link")

This goes back to the days before you could rely on keyboards to have an "up" arrow key, but can still be useful.  
To run the last command in your history

A common error is to forget to use `sudo` to prefix a command requiring privileged execution. Instead of typing the whole command again, you can:

This would change a `mkdir somedir` into `sudo mkdir somedir`.

## Exit traps[#](#exit-traps "Permanent link")

Make your bash scripts more robust by reliably performing cleanup.

Bash

`function finish {   # your cleanup here. e.g. kill any forked processes  jobs -p | xargs kill } trap finish EXIT`

## Saving your environment variables[#](#saving-your-environment-variables "Permanent link")

When you do `export FOO = BAR`, your variable is only exported in this current shell and all its children, to persist in the future you can simply append in your `~/.bash_profile` file the command to export your variable

Bash

`echo export FOO=BAR >> ~/.bash_profile`

## Accessing your scripts[#](#accessing-your-scripts "Permanent link")

You can easily access your scripts by creating a bin folder in your home with `mkdir ~/bin`, now all the scripts you put in this folder you can access in any directory.

If you can not access, try append the code below in your `~/.bash_profile` file and after do `source ~/.bash_profile`.

Bash

`# set PATH so it includes user's private bin if it exists if [ -d "$HOME/bin" ] ; then     PATH="$HOME/bin:$PATH" fi`

## 4\. Debugging[#](#4-debugging "Permanent link")

You can easily debug the bash script by passing different options to `bash` command. For example `-n` will not run commands and check for syntax errors only. `-v` echo commands before running them. `-x` echo commands after command-line processing.

Bash

`bash -n scriptname bash -v scriptname bash -x scriptname`

## 5\. Multi-threading[#](#5-multi-threading "Permanent link")

You can easily multi-threading your jobs using `&`. All those jobs will then run in the background simultaneously and you can see the processes below are running using `jobs`.

The optional `wait` command will then wait for all the jobs to finish.

Bash

`sleep 10 & sleep 5 & wait`

## Contribution[#](#contribution "Permanent link")

-   Report issues [How to](https://help.github.com/articles/creating-an-issue/)
-   Open pull request with improvements [How to](https://help.github.com/articles/about-pull-requests/)
-   Spread the word

## Translation[#](#translation "Permanent link")

-   [Chinese | 简体中文](https://github.com/vuuihc/bash-guide)
-   [Turkish | Türkçe](https://github.com/omergulen/bash-guide)
-   [Japanese | 日本語](https://github.com/itooww/bash-guide)
-   [Russian | Русский](https://github.com/navinweb/bash-guide)
-   [Vietnamese | Tiếng Việt](https://github.com/nguyenvanhieuvn/hoc-bash)
-   [Spanish | Español](https://github.com/mariotristan/bash-guide)

## License[#](#license "Permanent link")

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)