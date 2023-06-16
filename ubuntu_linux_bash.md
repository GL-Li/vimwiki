## Outline

- Major reference
- Workflow and SOP
- Minimal examples
- QA - quick answers to quick questions
- Raw notes

## Major reference
- [Udemy: Linux Mastery: Master the Linux Command Line in 11.5 Hours](https://www.udemy.com/course/linux-mastery/learn/lecture/8526918#overview)
- [ Udemy: Bash Mastery: The Complete Guide to Bash Shell Scripting](https://www.udemy.com/course/bash-mastery/learn/lecture/25412436#overview)

## Workflow and SOP

### SOP: run bash script as command from anywhere

**Summary**: place all well-written bash script under `~/bin` and add `~/bin` to PATH so the bash scripts can be run just like any terminal command.

- Make a new directory `$HOME/bin`
    ```sh
    $ cd
    $ mkdir bin
    ```

- Add the directory to `.bashrc` as shown below. Restart terminal to add the new path, and check with `$ echo $PATH` to make sure it is added to PATH.
    ```text
    # add to the end of .bashrc
    PATH="$PATH:$HOME/bin"
    ```

- Create a bash script, for example, `aaa.sh` 
    ```bash
    #! /usr/bin/bash
    
    echo "collecting all file names in current directory"
    echo "save them in file all_files.txt"
    ls -ltr | tee all_files.txt
    ```

- Convert to executable file, which can be run as `$ ./path/to/aaa` 
    ```sh
    $ chmod 777 aaa.sh          # change to executable file for owner, group, and other
    ```

- To run it from anywhere as a terminal command, create a soft link in `$PATH/bin`
    ```sh
    $  cd ~/bin
    $ ln -s path/to/aaa.sh aaa
    ```

- Run command `aaa` from anywhere.

## QA =========================================================================

### QA: where to store user-compiled executable files or symlinks?

They should be copied to `/usr/local/bin/`, which is not a part of the OS so they will not be overwritten in system update. And by default, this directory is in $PATH by default.

### QA: how to back up ubuntu system for rerstore

Using `timeshift` app. To install, simply run `sudo apt install timeshift`. To use it

- `$ sudo timeshit --create` to create a snapshot
- `$ sudo timeshift --list` to list all snapshot
- `$ sudo timeshift --restore --snapshot "2023-06-13_07-37-08"` to restore to a snapshot named by date created.
- `$ sudo timeshift --delete --snapshot "2023-06-13_07-37-08"` to delete a snapshot.

### QA: how to back up files 

Using Unison.

## Raw notes ==================================================================

### Linux brace expansion using {}

**Note**: no space between elements in `{}` in brace expansion.

```shell
$ touch {a,b,c}_{1,2,3}.{txt,pdf} # create files a1.txt, a2.txt, ..., c3.pdf
$ rm {*.txt,*.pdf}                # delete all txt and pdf files
$ echo a{11,22,33}b               # generate a11b, a22b, a33b
$ echo {10..1}                    # 10, 9, 8, ..., 1
$ echo {a..h}                     # a, b, c, d, e, f, g, h
$ echo {1..10..3}                 # 1, 4, 7, 10
$ echo {a..h..2}                  # a, c, e, g
$ echo month_{01..12}             # month_01, month_02, ..., month_12
```

### Linux command wildcards *, ?, and [ in file / directory names

**Note**: these wildcards only work in pathnames like file and directory names. `/` in pathname cannot be matched. They are similar to, but not, regular expressions.

- `*` any number of strings
- `?` any single character
- `[abcd]` any one character in string "abcd"
- `[!abcd]` any one character not in string "abcd"

```shell
$ man 7 glob          # manual page of wildcard matching
$ grep abc *.R        # find lines containing "abc" in all .R files
$ grep abc file?.R    # find lines containing "abc" in file1, fileA, ...
$ grep abc file[0-3]  # find lines containing "abc" in file0, ..., file5
```

### Linux locate to search path names, update database before search

`locate` search all path names in a database, which is updated one time a day. To search for new files, update the database.

- `-i` ignore case
- `-e` check the file actually exists

```shell
# install mlocate if updatedb not found
$ sudo updatedb         # update database before search by locate
$ locate abc            # search path name containing "abc"

# to use wildcard in search path, the path must represent full path
$ locate abc*.md        # the path [start](start) with abc and end with .md
$ locate *abc*.md       # the path contains abc and end with .md
```

### Linux find command to search files by name, type, size, ...

`find` has many options

- `-maxdepth 2` search up to second level, only one `-`, not `--`
- `-type f` show only files, `-type d` only directories
- `-name "file.txt"` search by name, must be exact file or directory name or using wildcards, not partial string
- `-size +100k`  file size > 100k, `-size +100k -size -1M`  size in 100k - 1M, `-size -100k -o -size +1M` size < 100k or > 1M.

```shell
$ find                        # show all diretories and files in current diretory
$ find eee/                   # list everything in eee/
$ find -name "*01"            # find path end with 01, quote to
                              # avoid shell expansion.
```

### Linux: grep recursively in files whose names match a pattern

```shell
$ grep -r "Linux" --include=*.R --exclude=*model*  # all .R files that do not have "model" in path names under current directory
```

### Linux: find files and execute on the found

- `{}`: placeholder for files found
- `-exec` or `-ok`,  perform command on each file found.
- `\;`, end the execution of one file and then start execution of another one
- `+`: run execution on all files at once if possible.

```shell
$ find -name "file1*" -exec cp {} copy_to_folder \;
$ find -name "file1*" -ok cp {} copy_to_folder \;    # to confirm for each file
$ find -name "file1*" -ok cat {} \;                  # print files on terminal, y or n
$ find -name "file1*" -exec cat {} +

# append a line to all found files. sh -c for shell command
$ find -name "file1*" -exec sh -c `echo "abc 123" >> {}` \;

# insert "abcd efg hijk" before the first line of all found files
find -name "file?" -exec sh -c 'sed -i "1 i\abcd efg hijk" {}' \;

```

### Linux: sort command to sort lines

- `-r` reverse
- `-n` by numerical value
- `-u` show only unique values
- `-h` by human readable data
- `-M` by month Jan, Feb, ...
- `-k n` sort by nth column of table data

```shell
$ ls -l | sort -k 5 -n   # sort by the 5th column
$ ls -lh | sort -k 5 -h  # sort by the 5th column by using human-readable value
```

### Linux: tar command, tarball, archive, compression

Create a tar ball

- `-c` create a new archive
-  `-v` verbose
- `-f`, `--file=ARCHIVE` use archive file
- `-z` compress with `gzip`
- `-j` compress with `bzip2`

```shell
$ tar -cvf xxx.tar file1 file2 ...
$ tar -cvzf xxx.tar.gz file1 file2 ...
$ tar -cvjf xxx.tar.bz2 file1 file2 ...
```

To preserve the directories, for example, if we want to only back up `OneDrive` in home directory, we can create a bash script:

```shell
# first go to parent directory of OneDrive
$ cd ~/
# compress the all in the directory. When decompressed, we get a folder OneDrive.
tar -cvzf /mnt/work/backup/onedrive/onedrive.tar.gz OneDrive/*

```

View files in a tarball

- `-t`, `--list` list names in a tarball

```shell
$ tar -tf xxx.tar
```

Extract from a tarball

- `-x` extract files

```shell
$ tar -xvf xxx.tar
$ tar -xvzf xxx.tar.gz   # extract file from gzip compressed tar ball
$ tar -xvjf xxx.tar.bz2  # extract file from bzip2 compressed tar ball
$ tar -xf xxx.tar.gz -C path/to/folder  # extract to specific folder
```

### Linux: gzip, bzip2 to compress tar balls, not for Windows and Mac

`gzip` is faster but less compression

```shell
$ gzip xxx.tar      # will create xxx.tar.gz and delete xxx.tar
$ gunzip xxx.tar.gz # get xxx.tar back and delete xxx.tar.gz
```

`bzip2` usually gets more compression but slower

```shell
$ bzip2 xxx.tar         # change to xxx.tar.bz2
$ bunzip2 xxx.tar.bz2   # get back xxx.tar
```

### Linux: zip files for share to Windows and Mac users

```shell
$ zip xxx.zip file1 file2 ...
$ unzip xxx.zip
```

### Linux: bash script run as command

**Summary**: place all well-written bash script under `~/bin` and add `~/bin` to PATH so the bash scripts can be run just like any terminal command.

Create a minimal example and name it `aaa.sh`

```bash
#! /usr/bin/bash

echo "collecting all file names in current directory"
echo "save them in file all_files.txt"
ls -ltr | tee all_files.txt
```

Run it from terminal locally

```shell
$ bash aaa.sh
```

Convert to executable file, which can be run as `$ path/to/aaa` without `bash`

```shell
$ cd                    # back to home diretory
$ mkdir bin             # create a bin to hold all bash script
$ mv aaa.sh aaa         # rename to just xxx
$ chmod +x aaa          # change to executable file
```

To run it from anywhere as a terminal command, add the path to `.bashrc`. Restart terminal to add the new path.

```text
# add to the end of .bashrc
PATH="$PATH:$HOME/bin"
```

### Linux: crontab to schedule tasks, https://crontab.guru/ for schedule, full path to executable script as cron restrict $PATH to /bin and /usr/bin

A cron task include six elements

- m: minute, 0-60 or * for any. Can be multiple values separated by ","
- h: hour, 0-24 or *
- dom: day of month, allowed day in the month or *
- mon: month, 1-12 or `*`, or JAN, FEB, ...
- dow: day of week, 0-6 or `*`, or SUN, MON, ..., SAT
- command: command to run

Create a crontab task that

- save "Hello World!" to hello.txt every Friday at 23:15.
- at 0 min and 30 min, set m to 0,15
- every three days, set dom to */3
- every two hours, set h to */2

```shell
$ crontab -e                 # open crontab to edit tasks, use full path to bash scripts
```

```
#  m    h    dom    mon    dow    command
  15   23      *      *    FRI    echo "Hello World!" >> ~/hello.txt
0,30  */4      *      *    FRI    ~/bin/backup_onedrive
```

### Linux: install packages with apt (advanced package tool)

- Understand  `/etc/apt/sources.list` file

  This file lists repositories of packages from which Ubuntu can use `apt` to install packages. There are four types of repositories: main, restricted, universe, and multiverse for each Linux distribution (`$ lsb_release -a`)

  ```
  deb http://us.archive.ubuntu.com/ubuntu/ jammy main restricted
  ```

  A new Linux installation has  official repositories in the file. Users can add other respositories to this file, but are recommended to add them under `/etc/apt/sources.list.d`.

- Add repositories under directory `/etc/apt/sources.list.d`

  When install packages not in the repos listed in `sources.list` file, often a repo is added to directory `/etc/apt/sources.list.d` by default. For example, when I install `onedrive` package, a file `onedrive.list` is created which has one line of text

  ```
  deb [arch=amd64 signed-by=/usr/share/keyrings/obs-onedrive.gpg] https://download.opensuse.org/repositories/home:/npreining:/debian-ubuntu-onedrive/xUbuntu_22.04/ ./
  ```

- search packages by keywords with `apt-cache` .

  All information of packages in above repositories is cached in files in `/var/lib/apt/lists`.

  ```shell
  $ apt-cache search docx   # list packages related to docx file
  $ apt-cache search "pdf viewer"
  $ apt-cache show antiword # show more details of a package
  ```

  To view all packages stored in a file in `/var/lib/apt/lists`.

  ```shell
  $ vim /var/lib/apt/lists/us.archive.ubuntu.com_ubuntu_dists_jammy_main_binary-amd64_Packages
  ```

  The cached files in computer is updated to match online repository server with

  ```shell
  $ sudo apt update
  ```

  After update, we can upgrade packages that were installed with `apt` with

  ```shell
  $ sudo apt upgrade
  ```

- install packages

  ```shell
  $ sudo apt install pkg1 pkg2 pkg3
  ```

### Linux: delete packages and configurations

```shell
$ sudo apt purge xxxx # Remove a package completely
$ sudo apt autormove  # Remove dependencies that is not required by any other packages
$ sudo apt autoclean  # clean package .deb files saved in /var/cache/apt/archives/ but keep those not available on server
```

### comments best practice: 5 pieces of information to start a script

```
# Authour:
# Date Created:
# Last Modified:
# Description:
# Usage:
```

### Linux: exit status

0 for no error, 1 - 255 for an error

- 0: sucess
- 1: operation not permitted
- 2: no such file or directory
- 3: no such process
- more are [here](https://www.cyberciti.biz/faq/linux-bash-exit-status-set-exit-statusin-bash/#:~:text=The%20exit%20status%20is%20an,returns%20a%20status%20of%20127.)

### Linux: chmod, change file mode bits, file permission code

- Understand rwx in `$ ls -l`

  - d: directory, if `-`, then a file

  - r: read

  - w: write

  - x: execute

  ```
  # owner group others
  drwxr-xr-x  2 gl gl  4096 Oct  8 10:57 Videos
  -rw-rw-r--  1 gl gl 19842 Dec  4 17:30 weekly-EMA-200-20-9.tpl
  ```

- change permission with `chmod` exemple

  Let's start with a file permission as

  ```
  -rw-rw-r-- 1 gl gl    0 Dec  8 07:59 file1
  ```

  `$ chmod 760 file1` will change it to (7 for owner, 6 for group, 0 for others)

  ```
  -rwxrw---- 1 gl gl    0 Dec  8 07:59 file1
  ```

- `chmod` code

  - 0: no permision
  - 1: execute only
  - 2: write only
  - 3: write + execute (1 + 2)
  - 4: read only
  - 5: read + execute (4 + 1)
  - 6: read + write (4 + 2)
  - 7: read + write + execute (4 + 2 + 1)

### Linux: add new PATH in .profile

In addition to modify path for bash script in `.bashrc` [Linux: bash script run as command], we can add new path in `.profile` file.

```
# add to the end of .profile
export PATH="$PATH:$HOME/any/directory"
```

For the change to take effect, restart the terminal or source the file.

```shell
$ source ~/.profile
```

### bash: variables and shell expansion

**User-defined variables**: By tradition, use lower case for user defined variables.

```shell
student="Sarah"  # NO spaces around "=" sign, otherwise bash treat student as a command
echo "Hello ${student}"  # ${} called shell expansion
```

**Shell variables**: or called environment variables, usually in upper case, with special names. Common shell variables. $PATH, $HOME, $USER, $HOSTNAME, $PS1, $PWD, $OLDPWD, $?

- `$PATH`: or `${PATH}`, path of executables
- `$HOME`: absolute path to user's home directory
- `$USER`: current user username
- `$HOSTNAME`: computer name
- `$HOSTTYPE`: computer processor architecture type such as "x86-64"
- `$PS1`: string that defines what before the `$` sign in terminal, for exemple, `\[\e]0;\u@\h: \w\a\]${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[01;31m\] $(parse_git_branch)\[\033[00m\]\$`, very intimidating. It is defined in `.bashrc`. It is where to modify prompt style.
- `$PWD`: current working directory
- `$OLDPWD`: previous working directory
- `$?`: exit status of last command

### Bash: string upper case, lower case, string length, slice substring, concatinate strings, cut substring from stdin

- lower and upper cases

  ```shell
  name="John Doe"
  echo $name   # or ${name}, print name
  # to lower case
  echo ${name,} # first letter to lower case
  echo ${name,,} # all letters to lower case
  # to upper cse
  echo ${name^}  # first letter to upper case
  echo ${name^^}  # all to upper case
  ```

- length of the variable

  ```shell
  echo ${#name}  # "#" count the number of characters
  ```

- slice a string: position of characters in a string is index by 0, 1, 2, 3, ..., from 0

  ```shell
  echo ${name:3:5}  # substring of length 5 from the 4th character
  echo ${name:3}  # all from the 4th character, 3 + 1 = 4 so 4th
  echo ${name: -3:5} # start from 3rd counting from end. need a space before "-"
  echo ${name: -3}
  ```

- concatenate string: just treat each varaible as a string

  ```shell
  a="aaa"
  b="bbb"
  echo "$a/$b"   # print out aaa/bbb
  ```

- cut substring with a delimiter, input is from a file or stdout, not from a string. if have to use cut on a string, use `echo` to pipe the string to `cut` like `echo "$aaa_string" | cut -f 2 d ,`.

  ```bash
  echo "abc.def.ghi" | cut -f2 -d "."  # def
  cut xxx.txt -f2 -d "."               # cut line by line
  cut "abc.def.ghi" -f2 -d "."         # error, string not an argument
  ```

### Bash: command substitution `$(ls -ltr)`

use `()` instead of `{}`.

```shell
# command substitution
echo "Hello $USER, the time right now is $(date +%H:%m:%S)"

# assign command substitution to a variable
time=$(date +%H:%m:%S)  # use time to substitute command date and its format
echo "Hello $USER, the time right now is $time"
```

### Ubuntu: mount and unmount a drive

**View mounted drives**:

```shell
$ lsblk
```

**Unmount a partition**:

```shell
$ sudo umount /dev/sdb1
```

**Rename the directory name of a mounted system**: simply rename the directory

```shell
$ sudo mv oldname newnam
```

**Mount a new hard disk**: To mount a hard disk to  `/mnt/d/`.

```shell
# create directory
$ cd /mnt/
$ mkdir d
# check available disks to mount
$ sudo fdisk -l    # find disk such as /dev/sdb
# mount disk to direct d
$ sudo mount /dev/sdb1 /mnt/d    # sdb1, the first of sdb.
```

### Bash: math, arithmatic expansion, add, subtract, multiply, divide, module for integers

Math expression in `$(())`. **Only work with integers**.

- `+`, `-`, `*`, `/`, `**` (exponential), `%` (remainder),

```bash
echo $((3 + 4))   # print out 7
echo $(( (1 + 3) * 5 ))  # 20
x=9
y=3
echo $(($x / $y))  # print 3
echo $((x / y))  # save some typing
echo $((8 / 3))  # print 2 !!!!!!!!!!!!!!!!!!
```

### Bash: math, bc for decimal numbers, numerical values

**bc**: basic calculator

```bash
echo "scale=4; 5/3" | bc   # print 1.6666
```

### Bash: tilde expansion,  `~user`, `~+`, `~-`. ~ not in quote, use `$HOME` if have to

- `~` for home directory
- `~root` for user root's home directory
- `~+` for `$PWD`
- ~- for `$OLDPWD`

```shell
# to switch back and forth between two directories
$ cd ~-     # to previous
$ cd ~-     # switch back
```

### Bash: 5 steps to process command line

- tokenization
- command identification
- shell expansions
  - stage 1: brace expansion
  - stage 2:
    - parameter expansion like `$myname`
    - arithmetic expansion like `$((1 + 2))`
    - command substituion like `$(ls -ltr)`
    - tilde expansion like `~gl`, `~root`
  - stage 3: word splitting at `$IFS`
  - stage 4: globbing filepath with wildcards
- quote removal
- redirections

### Bash: quoting, single quote and double quote are different

Quoting is about removing special meanings. Backslash removes special meaning of next character, single quote remove all inside, double quote removes all but dollar sign $, backtick `, and those in **command substitution**.

- `\`: next character
- `''`: all characters inside
- `""`: all character except dollar signs `$` and backslash `\` and command substitution.

```shell
$ echo aaa \& bbb # print aaa & bbb, backslash \ removes special meaning of &
$ filepath="C:\Users\xxx"   # or single quote
$ echo "My $filepath"       # double quote only as we have $
```

### Bash: token, metacharacter, word, operator

- Token is a sequence of characters that is considered as a single unit by the shell.
- metacharacter: special character include `|`, `&`, `;`, `(`, `)`, `<`, `>`, space, tab, and newline.
- word: token that does not contain any unquoted metacharacters
- operator: token that contains at least one unquoted metacharater

```shell
# in the command below, the metacharacters include three spaces and one >.
# words include echo, $NAME, and file.txt
# operator is >
$ echo $NAME > file.txt
```

### Bash: simple command, compound command

**simple command**s: one line command, separeted by control operators

```shell
$ echo a b echo c d  # print a b echo c d. The command is terminated by newline
$ echo a b; echo c d # print a b then c d. The two command terminated by ";" and new line
```

**compound commands**: such as if statement and loops

```bash
if [[ 2 -gt 1]]; then
  echo "hello world"
fi
```

### Bash: shell expansion stages, brace expansion cannot contain any other expansions.

Expansion in later stage cannot be used in expansion in early stage.

- stage 1: brace expansion, which mean no other expansion used in brace expansion
- stage 2:
  - parameter expansion like `$myname`
  - arithmetic expansion like `$((1 + 2))`
  - command substituion
  - tilde expansion
- stage 3: word splitting
- stage 4: globbing

```bash
# exemple below will not print 1,2,...,10 as brace expansion occurrs before arithmetic expansion.
x=10
echo {1..$x}  # print {1..$x}
```

### Bash: word splitting, use quote if not wanting word splitting

**Word splitting** is a process the shell performs to split the result of some unquoted expansions into separate words at `$IFS`. Only  performed on the results of unquoted parameter expansions, command substitutions, and arithmetic expansions.

**IFS (internal field separator)**: by default it contains space, tab, and new line, so it is invisible if we check with `$ echo $IFS`.

```shell
# to view IFS, using @Q to view quoted output
$ echo ${IFS@Q}  # print ' \t\n'
```

**Examples**

```shell
# use default IFS
$ aaa="a b c"
$ touch $a.txt    # create 3 files named "a", "b", and "c.txt", ".txt only with c
                  # equivalent to $ touch a b c.txt
$ touch "$a.txt"  # "a b c.txt" as "$a.txt" is quoted
$ touch "a b c"  # only one file named "a b c". No word splitting as "a b c" is quoted

# change IFS
$ bbb="1,2,3"
$ touch $bbb   # create single file "1,2,3" with default IFS
$ IFS=","      # set IFS to "," for current session, but it does not change $IFS
$ touch $bbb   # create 3 files, 1, 2, and 3
$ unset IFS    # to cancel IFS=","
```

### Bash: globbing, wildcards in filenames

Globbing is only performed on words. see section [Linux command wildcards \*, ?, and [ in file / directory names]

### Bash: quote removal, use quote whenever possible

During quote removal, the shell removes all unquoted backslashes, single quote, and double quote that did not result from a shell expansion.

```shell
$ echo \$HOME  # print $HOME, unquoted backslash is removed
$ echo '\$HOME' # print \$HOME, single quote removed but backslash not as it is quoted
$ echo "C:\Users\gl\Downloads"  # correct path
```

### Bash: redirection, data streams, `<` `>` `>>` `tee`, stdin stout, sterr

**Three streams**: each command has stdin 0, stdout 1, stderr 2

**Examples**:

```shell
# 0< for direct input to command, where 0 or both 0< can be ignored
$ cat 0< file.txt  # ok
$ cat < file.txt   # ok
$ cat file.txt     # ok
$ 0< file.txt cat  # ok
$ < file.txt cat   # ok
$ file.txt cat     # not working, file.txt treated as command

# 1> 1>> redirects output to a file, one can be removed
$ echo "xxx" 1> output.txt
$ echo "xxx" > output.txt

# 2> 2>> redirect error message to a file
$ cd /root  # bash: cd: /root: Permission denied
$ cd /root 2> error.txt  # error message saved to error.txt
$ cd /root 2> /dev/null  # when a file is sent to /dev/null, it is just deleted

# &> redirect both stdout and sterror to a file
$ cd /root &> stdoutstderr.txt
```

### Bash: control operators `&&` `||` `;`  `\` `;;` `;&` `;;&` `|&` `(` `)` `|` new line

- `command1 && command2`: command2 runs only if comand1 has exit status 0.

  ```shell
  $ cd /root && echo ok  # bash: cd: /root: Permission denied
  ```

- `command1 || command2`: command2 runs only if command1 has non-zero exit status.

  ```shell
  $ cd /root || echo ok  # bash: cd: /root: Permission denied
                         # ok
  ```

- `command1; command2`: run command1 and then command2 no matter what the exit status of command1

  ```shell
  $ cd /root ; echo ok       # bash: cd: /root: Permission denied
                            # ok
  $ cd ~/Download; echo ok  # ok
  ```

- `command1 &`: when ampersand & is at the end of a command, the command will run at background while you use the terminal for new commands

  ```shell
  $ sleep 10 &     # sleep 10 sec and the terminal is ready for new command
  $ sleep 10 & ls  # ls does not wait for sleep to finish
  ```

- `command \`: end of line backslash indicates command input continuous on next line

  ```shell
  $ ls \
  > -ltr    # the two line equals to $ ls -ltr
  ```

  ### Linux: command date, +, %y, %Y, %b, %m, %d, %a, %H, %I, %M, %S, use plus sign and quote

  Check help page if not sure.

  ```shell
  # need + before format
  $ date %Y  # date: invalid date ‘%Y’
  $ date +%Y # 2022
  $ date +"%Y %b"  # 2022 Dec, format in quote
  ```

  ### Bash: positional parameters $1, $2, $3, ..., ${10}, ${11}, ...

  Allow bash script to read arguments from terminal. Note that when there are over 10 positional parameters, put double digit into {}. Otherwise the shell will interpret it as $1 and 0. Better keep the number of positional parameters no more than 9.

  **example**:

  Create file and save it as testPosParam

  ```bash
  #!/bin/bash
  echo "The first parameter is $1"
  echo "The second parameter is $2"
  echo "The third parameter is ${3:-999}"  # set default 999 if not given, otherwise empty
  ```

  Run from terminal

  ```shell
  $ bash testPosParam 123 abc  # The first parameter is 123
                               # The second parameter is abc
                               # The third parameter is 999
  ```

### Bash: special parameters $#, $0, $#, $@, "$@", $\*, "S\*" for script and positional paramers

**$#**: number of positional parameters, can be used to specify number of positional parameters as condition

**$0**: when used in a bash script, it is the name of the script file.

```bash
#!/bin/bash

if [[ $# -ne 2]]; then
	echo "You did not enter exactly 2 parameters"
	echo "Usage: $0 param1 param1"
	exit 1
fi
```

**$@**: refer to all positional parameters $1 $2 ... $N. Let's save the script below as `dollarat` and run it as

 `$ dollarat "monthly sales" "annual report"`

and see what we get.

```bash
#!/bin/bash

echo $@      # print monthly sales annual report
echo "$@"    # print monthly sales annual report
touch $@     # create four files: monthly, sales, annual, report, allow word splitting
touch "$@"   # create two files" `monthly sales` and `annual report`
```

**$\***:   unquoted $\* is the same as $@. Quoted "$\*" represent "S1,$2,..,$N"

```bash
echo $\*      # print monthly sales annual report
IFS=,
echo "$*"    # print monthly sales,annual report. Separate by IFS instead of space, no splitting
```

**application example**: calculate from command line. Save the script as `calculate` and run

`$ bash calculate 1 + 6 - 3`

```bash
#!/bin/bash

echo $(( $@ ))  # $@ replace whole thing 1 + 6 - 3
```

### Bash: read command for interactive input, options -p, -t, -s

The input is saved in `$REPLY` shell variable or custom variables

- `-p`: add prompt
- `-t`: time out option, expire if no input within the time limit
- `-s`: secret option, the user input will not show in terminal
- `-N 4`: input must be 4 characters.
- `-n 4`: input up to 4 characters

```shell
# saved in $REPLY
$ read        # presss enter and enter aaaaaaaaa
$ echo $REPLY # print aaaaaaaa

# saved in animal1 aninal2
$ read animal1 animal2 # press enter and input two values cat dog
$ echo $animal1        # print cat
$ echo $animal2        # print dog

# prompt
$ read -p "The first animal: " animal1
$ read -p "The second animal: " animal2
$ echo $animal1
$ echo $snimal2
```

### Linux: man, if not available, use help, info, internal external command

Check internal (builtin) or external command using `type`

```shell
$ type -a cd  # cd is a shell builtin   (internal)
$ type -a shellcheck # ls is aliased to `ls --color=auto'
                     # ls is /usr/bin/ls  (external)
                     # ls is /bin/ls
```

- help: for internal command

- man: for external command only, complete manual

  - search command by keywords in its role

    ```shell
    $ man -k compress    # search command by keywords compress
    ```

- info: for external command only, more information with links to other sources

### Bash: select ... in ... do ... done, space is the delimiter, not `,`

Select from options each separated by space, not `,`.  Keep in quote if not want to split a option.

```bash
PS3="What is the day of the week?: "       #$PS3 for the shell prompt of select
select day in Mon Tue Wed Thu Fri Sat Sun; do
	echo "The day of the week is $day"
	break    # to break the do - done loop, other wise loop back to select.
done
```

### Bash: auto indent whole file in vim, gg=G

gg to get to the beginning of the file, = is the indent command, G to go to the end of the file. The combination `gg=G` is to indent from beginning to the end of the file.

### Bash: list operators ;, &, &&, ||, list of command in one line

see [Bash: control operators]

### Bash: test commands and operators, [ space=around ] or [[ ... ]], `!` to negate

Return exit status 0 if true, exit status 1 if false.

**integer test operators** only for integers eq ne gt lt geq leq

```bash
$ [ 2 -eq 2 ] ; echo $?    # 0
$ [ ! 2 -eq 2 ] ; echo $?  # 1, ! for not
$ [ 2 -eq 3 ] ; echo $?    # 1
```

**string test operators**, =, !=, -z, -n

```bash
$ a=hello
$ b=world
$ [ $a = $b ] ; echo $?
$ [ -z $c ] ; echo $?       # -z to check if empty or null string
$ [ -n $a ] ; echo $?       # -n to check if string exists
```

**file test operators**, -e, -f, -d, -x, -r, -w

```shell
$ [[ -e file1 ]]   # check if file1 exits
$ [[ -f fname ]]   # check if a regular file, not directory
$ [[ -d fname ]]   # check for directory
$ [[ -x fname ]]   # check for executable file
```

### Bash: if ... elif ... else ... fi

if statement check the exit status of a  command

```bash
# dummy example
if [ 2 -gt 1 ] ; then
	echo "test passed"
elif [ 2 -eq 0 ]; then
	echo "test passed"
elif [ 4 -ne 6 ] && [ 7 -lt 8 ]; then   # each [] for one simple test
	echo "random try"
else
	echo "test failed"
fi

# example 2
a=$(cat file1.txt)
b=$(cat file2.txt)
c=$(cat file3.txt)
if [ $a = $b ] && [ $a = $c]; then
	rm file2.txt file3.txt
else
	echo "Files do not match"
fi
```

### Base: case ... esac, double quote, `;;` and `)`

Must put variable in double quote to prevent word splitting, each case must end with `;;`, which is a specific operator only for `case` statement.

```bash
read -p "Please enter a number: " number
case "$number" in
	[0-9]) echo "you have entered a single digit number";;
	[0-9][0-9]) echo "you have entered a double digit number";;
	[0-9][0-9][0-9]) echo "you have entered a three digit number";;
	*) echo "You have entered a number that is more than three digits";;
esac
```

### Bash: while loop

```bash
read -p "Enter your number: " num

while [ $num -gt 10 ]; do
	echo $num
	num=$(( $num - 1 ))
done
```

### Bash: getopts, define options for bash script

Save the code as fc_converter, which convert temperature between F and C. The command has two options: `-c` and `-f`. The value of the option is stored in `$OPTARG`.

```bash
#!/bin/bash

while getopts "f:c:" opt; do    # define options here, allow -f and -c
	case "$opt" in
		c) result=$(echo "scale=2; ($OPTARG * (9 / 5)) + 32" | bc);;
		f) result=$(echo "scale=2; ($OPTARG -32) * (5 / 9)" | bc);;
		\?) ;;   # for any other single character
	esac
done
echo "$result"
```

Run the command as

```shell
$ fc_converter -c 0   # 32
$ fc_converter -f 32  # 0
```

**example**: count down by seconds from an input minutes (-m) and seconds (-s).

`$ bash countdown -m 1 -s 9`

```bash
#!/bin/bash

time=0
while getopts "m:s:" opt; do
	case "$opt" in
		m) time=$(($time + 60 * $OPTARG));;
		s) time=$(($time + $OPTARG));;
	esac
done

while [ $time -gt 0 ]; do
	echo $time
	sleep 1s
	time=$(( $time - 1 ))
done

exit 0
```

### read-while loop to iterate over the lines of files or process substitution `<( command )`

process substitution: `<(command)`

This is fixed struct

```bash
# read line by line of a file
while read line; do
	# do whatever by each line here
	echo "$line"
done < "$1"      # put input file here

# read line by line of a command output using process substitution
while read line; do
	# do whatever by each line here
	echo "$line"
done < <( ls $HOME)
```

### Bash: indexed array

Index starts from 0.

```shell
$ number=(111 222 333 444)
$ echo $number    # only first element 111
$ echo ${number[2]} # third element 333
$ echo ${number[@]} # all element
$ echo ${number[@]:1}  # from second element to the end, slicing
$ echo ${number[@]:1:2} # two elements from the second
$ numbers+=(555)  # add new element to the end
$ unset number[2] # delete the third element, it also deleted the index 2.
                  # the remaining index are 0 1 3 4
$ echo ${!number[@]}  # check index
$ number[0]=999   # change a element

```

### Bash: readarray to generate index arrays

Read standard input into an array line by line. Each element has a `\n` at the end, which may mess up with string processing.

- `-t` to remove trailing `\n`

```shell
# read from a file
$ readarray days < weekday.txt
$ echo ${days[@]}
$ echo ${days[@]@Q}  # show the raw string, which ends with \n

# the prefered readarray
$ readarray -t days < weekday.txt

# read from other command output using process substitution
readarrays -t files < <(ls)
```

### Bash: for loop, on-the-fly list, array, no quote for index

When use the index of an array in for loop, do not quote it.

```bash
# using on-the-fly list
for element in first second third; do
	echo "This is $element"
done

# using array
readarray -t files < <(ls)
for file in "${files[@]}"; do
	if [ -f "$file" ]; then       # "" to avoid word splitting
		echo "     File: $file"
	elif [ -d "$file" ]; then
		echo "Directory: $file"
	else
		echo "  Invalid: $file"
	fi
done

```

**Example**, loop over array index

```bash
#!/bin/bash

readarray -t urls < urls.txt
readarray -t fnames < <(cut urls.txt -f 2 -d ".")
indeces=${!urls[@]}
for idx in ${indeces[@]}; do               # no quote, otherwise just "0 1 2 ...".
	fname="${fnames[$idx]}.txt"            # We want word split here
	curl --head "${urls[$idx]}" > "$fname"
done

exit 0
```

### Bash: debug with shellcheck for syntax error, error message structure

Do not blindly follow its suggestions, even it says an error. Use it for a warning.

```shell
$ sudo apt install shellcheck
$ shellcheck my_bash_script
```

Error message structure: command failed : what's wrong : reason of error

### Bash: at command to schedule task

The `at` is a daemon service runs at background for simple and one-time schedules.

```shell
$ sudo apt install at  # install at

$ service atd status   # check if at is running
$ sudo service atd start  # start atd service, stop to stop

# to schedule tasks to run at back ground
$ at 9:30am  # press enter
> echo "Hello world"   # one task
> cp xxx bbb           # another task
> <EOT>                # end of scheduled task by ctrl D

$ at -l     # list scheduled job
$ at -r 2   # remove job 2

$ at 10:05am -f my_bash_script    # schedule to run a bash script
$ at 10:05am Monday -f my_bash_script
$ at 10:05am -f 12/23/2022 my_bash_script
$ at now + 5 min -f my_bash_script    # 2 days ...
```

### Bash: cron directories

**System cron directories** are in `/etc`. All executable scripts in each directory runs as shown in names like `cron.daily`, `cron.hourly`, `cron.weekly` and `cron.monthly`. The exact time can be found in file `/etc/crontab`.

- no dot "." in script names

**Custom cron directories** are easy to create. Follow steps below to create a folder in home directory to hold scripts that run at 2am each day

```shell
# step one: create a directory to store scripts to be run at a given time
$ mkdir cron.daily.2am
# then add a line to crontab
$ crontab -e
```

```
00 02 * * * run-parts path/to/cron.daily.2am --report
```

### Ubuntu, reboot required after package upgrade

Check /var/run/reboot-required.pkgs for the list of packages that require reboot. For example, linux-base upgrade needs reboot.

We want to create a cron task that upgrades packages every day. We first need to create a bash script called `update_packages`:

```bash
#!/bin/bash
apt -y update
apt -y upgrade
# If `/var/run/reboot-required` file exists, reboot system after upgrade
if [ -f /var/run/reboot-required ]; then
	reboot
fi
```

As upgrade affect the whole system, we will modify `/etc/crobtab` to schedule this task by adding a line

```
0 0 * * * /path/to/update_script    # not use ~ for home as this file is in root.
```

For t his to take effect, run

```shell
$ sudo service cron restart
```

### Ubuntu: anacron

cron requires machine to be on at the scheduled time. anacron can pickup missed job when computer is turned on. anacron is scheduled in system file `/etc/anacrontab`. There is no user specific anacrontab. Edit this file

- change SHELL=/bin/bash

- add path to PATH

- cron.daily, cron.weekly, and cron.monthly are managed by anacron. So any missed tasks for scripts in these folders will run when computer is turned on.

- minimal time interval is a day, no hour and minute

- manually add a job

  run weekly (7), 30 mins after power on if missed, job identifier backup and script is backup_script, which must be in the PATH

  ```
  7 30 backup backup_script
  ```

- logs in `/var/spool/anacron`

### Linux, bash, remote server, ssh, scp

```shell
$ ssh usename@12.345.678.90
$ scp /path/to/local/file username@12.345.678.90:/path/to/remote/directory
$ scp username@12.345.678.90:/path/to/remote/file /path/to/local/directory
```

### Linux: upgrade error The following packages have been kept back

The reason is that the latest version of an installed package has new dependencies which are not installed and the system refuses to install those dependencies during `sudo apt upgrade`. To fix the problem, install the kept-back packages with

```shell
$ sudo apt install --only-upgrade package1 package2 package3
```
