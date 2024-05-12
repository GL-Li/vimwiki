## sed

Tutorial:
- https://alexharv074.github.io/2019/04/16/a-sed-tutorial-and-reference.html

Manual:
- https://www.gnu.org/software/sed/manual/sed.html


### concepts
- pattern space: buffers each line (without the trailing new line "\n") that is read from the input stream. The pattern space is usually deleted between cycles (lines).
- hold space: keep data between cycles.

### Structure of a sed script
select lines followed by one or more commands and their options:
- `[ADDR]{X[OPTIONS];Y[OPTIONS];Z[OPTIONS]}`

- `[ADDR]`: optional, used to select lines
    - by line number: 
        - `sed -i 2d iris.csc` to delete the second line
        - `sed -i '$d' iris.csv` to delete the last line
        - `sed -n 2,5p iris.csv` to print lines 2-5
        - `seq 10 | sed -n '8,$p'` to print all lines from line 8.
        - `seq 10 | sed -n 2~3p` to print every 3 lines from the second line. `-n` for selected lines only
    - by matching condition 
        - `seq 30 | sed -n '/2$/p'`: print lines ending with "2"
        - `sed -n '/5.1/p' iris.csv`: print lines containing "5.1"
        - `seq 9 | sed -n '/6$/,/^1/p'`: print lines from the first match to the second match
            - if the second match does not exist, print from the first match to the end 
            - if the first match does not exist, no selection
            - repeated match: for example `seq 30 | sed -n '/8$/,/^1/p'` matches
                - 8, 9, 10: first round match
                - 18, 19: second round match
                - 18, 29, 30: third round match where the second match does not exist so print until end.
    - by offset from a line
            - `seq 10 | sed -n 5,+2p`  print line 5, 6, 7 
            - `seq 30 | sed -n '/2$/,+2p'` print three matches
                - 2, 3, 4 
                - 12, 13, 14
                - 22, 23, 24
    - exclude lines with `!` after the line range
        - `seq 10 | sed -i 2,8!p`   print 1, 9, 10
        - `sed -n '/^5/!p' iris.csv` print lines that do not start with "5"
        - `sed -i '/^5/!d' iris.csv` delete lines that do not start with "5"
  
- List of commonly used command
    - `=`
    - `a`
    - `c`
    - `d`
    - `e`
    - `i`
    - `l`
    - `n`
    - `p`
    - `q`
    - `Q`
    - `s///`
    - `y///`
    - `z`


### The `s` command (substitute)

Format:

- `s/REGEX/REPLACEMENT/[FLAGS]`  
- `s@REGEX@REPLACEMENT@[FLAGS]`  the seperator `/` can be replaced with other characters for example `@`.

**replace strings in a file**

```shell
# replace in-place, with or sometimes without single quote arount s///g
sed -i 's/Setosa/hahah/g' iris.csv

# replace a file and print all lines to stdout
sed 's/hahah/xxxx/g' iris.csv

# replace a file and print only replaced lines to stdout, need both -n and /p
sed -n 's/hahah/xxxx/p' iris.csv
```

**replace string from stdout**

```shell
echo "aaa bbb aaa" | sed s/aaa/xxx/g  # xxx bbb xxx, replace all
echo "aaa bbb aaa" | sed s/aaa/xxx/g   # xxx bbb aaa, replace the first occurrance
```

**flags**

- no flags: replace the first matched pattern in each line 
    - `sed 's/aaa/bbb/`
- `g`: replace all matched patterns
    - `sed 's/aaa/bbb/g`
- `n`: replace the nth match
    - `sed 's/aaa/bbb/2`   match the second match in each line
- `ng`: replace all matches from the nth one
- `i`: case-insensitive matching
    - `echo "Abc abc" | sed 's/a/xxx/gi'` print xxxbc xxxbc, both "a" and "A" matched and replaced 

**back reference** using capture group

- `$ echo "James Bond" | sed -E 's/(.*) (.*)/My name is \2, \1 \2./'`

**back reference** using matched pattern

- `echo "fox foot" | sed -E 's/o+/&&&/g'` print fooox foooooot, where `&` represent a matched pattern.

**case conversion**

- common converters in REPLACEMENT:
    - `\U`: all to upper case
    - `\u`: first character to upper case
    - `\L`: all to lower case
    - `\l`: first character to lower case

- examples
    - `echo "foo bar foo baz" | sed -E 's/(foo)/\uaaa/g'`
        - print Aaa bar Aaa baz. In the replacement section, `\u` convert 'aaa' to 'Aaa'
    - `echo "foo bar foo baz" | sed -E 's/(foo)/\u\1/g'`
        - print Foo bar Foo baz. In the replacement, `\1` back references to matched pattern. 


## awk

### awk field and separator
https://www.youtube.com/watch?v=oPEnvuj9QrI

- Records: each line is a record
- field: a line is seperated into multiple fields by space (default), that is, a word is a field.
    - example:
        - aaa.txt
            ```
            this is line1
            line2
            ```
        - `$ awk '{print $1,$3}'` prints the first and third fields. Empty if a field does not exist:
            ```
            this line2
            line2
            ```
- field seperator: we can set it to anything instead of the default space
    - `echo "Ihahaamhahahappy" | aws -F 'haha' '{print $1,$2,  $3, "!!!"}'` print I am happy !!!. The default seperator between printed fields is space.
- output field separator, `OFS`. We can change `OFS` so the output fields is seperated by other strings
    - `echo "Ihahaamhahahappy" | awk -F "haha" 'BEGIN {OFS="---"} {print $1,$2,$3}'` prints I---am---happy.
 

### awk built-in variables
https://github.com/adrianlarion/simple-awk

- `FILENAME`: the name of the current file being processed.
- `NR`: number of record, counted from all files
- `FNR`: file number of record, line number of current file being processed.
- `FS`: field seperator, default to space ' '.
- `IGNORECASE`: ignore case if 1, no if 0.
- `NF`: number of field (word) in a line
- `RS`: record seperator, `\n` by default.


### awk select lines

- `awk '/this/' aaa.txt`  print lines containing "this" in aaa.txt.
- `awk '$2 == "is"' aaa.txt` print lines whose second field is "is"
- `awk '$1 ~ /i/' aaa.txt` print lines whose first field matches pattern 'i'. Use `~` for pattern match and `!~` for not match.


### awk `printf` to format output

- `awk -F ',' '{printf "%15s %-15s %-10s\n", $5, $2, $3}' iris.csv`
    - `%15s`: total width 15 character, right aligned
    - `%-15s`: left aligned
    - line ending `\n` is required for new lines


### awk `BEGIN` and `END` block
The general pattern is `awk 'BEGIN {setup OFS and initialize variables} {process line by line} END {summarize results}' file.txt`.

- `awk -F "," 'BEGIN {count = 100} {count ++} END{print count}' iris.csv` total number of lines
    - set initial `count` in `BEGIN` block
    - increment line by line in middle block
    - print out final result in `END` block
    - `BEGIN` and `END` blocks are optional



### awk math

- `echo "4,2,5" | awk -F "," '{sum = $1 + $2 + $3; print sum}'` print 11
- `awk -F ',' '{cumsum = cumsum + $3; print cumsum}' iris.csv` cumulative sum of field 3, no need to initialize cumsum.
- `awk -F ',' '{cumsum = cumsum + $3} END {print "average =", cumsum / NR}' iris.csv` print average of field 3

### awk functions

- `substr($3, 1, 5)` to extract character 1 - 5 of a string field `$3`.
    - `echo "aaabbbccc" | awk '{print substr($0, 1, 5)}'`  print aaabb
    - `awk -F ',' '{printf "%15s %-15s %-10s\n", substr($5, 1, 5), $2, $3}' iris.csv` substring of field 5


## tr
translate or delete **characters**.

**translate characters**

- `echo "Abc Def" | tr [a-z] [A-Z]`  to upper case ABC DEF
- `echo "Abc Def" | tr be XY` to AXc DYf, that is, treat be and XY as two arrays and map b -> X and e -> Y

**delete characters**

- `echo "Abc Def" | tr -d [a-z]` to delete all lower case characters
- `echo "Abb Effff" | tr -s bf` to squeeze duplicated b and f to a single character


## pass
password manager, https://www.passwordstore.org/

Following steps detailed in https://www.youtube.com/watch?v=FhwsfH2TpFA

- create GPG key pair if not already have one
    - `$ gpg --gen-key`
        - remember the master passphrase and never share with anyone else
        - a directory `$HOME/.gnupg/openpgp-revocs.d` created
        - `$gpg -K` to view the public key
        - `$ gpg --edit-key 01D414DB308B6CDD6D93AAABDE70199F0F16EE9D` to edit the key. Replace the public key from the output of `gpg -K`.
        - `gpg> expire` and select never expire. Master passphrase needed.
        - `gpg> save` and exit
- installa and initialize password store
    - `$sudo apt install pass`
    - `$ pass init 01D414DB308B6CDD6D93AAABDE70199F0F16EE9D`
        - the command above created subdirectory `.password-store`
    - `$ pass git init` to git version control of passwords
- generate password, for example, for github account
    - `$ pass generate githut/GL-Li`
        - a randomly generated password is created and stored in a subdirectory `.password-store/github/GL-Li`.
        - to view the password, `$ pass show github/GL-Li`, master passphrase required.
        - a git commit is aumatically added to the history
            - `pass git log` to view commit history
            - `pass git xxx` to run git command on the password repo. No need to switch to the repo. You can `$ cd .password-store`.
            - make a private repo on github or bitbucket for the repo
- use a password
    - `pass show -c github/GL-Li` to copy the password to clipboard and ready to paste. 
    
- use the password store on another computer
    - export gpg keys into files
        - `$ gpg --output public.gpg --armor --export lglforfun@gmail.com`
        - `$ gpg --output private.gpg --armor --export-secret-keys lglforfun@gmail.com`, require master passphrase
    - import gpg keys to another computer
        - copy file `public.gpg` and `private.gpg` to the new computer
        - `$ gpg --import private.gpg` to import private key
        - `$ gpg --import public.gpg` to import public key
    - update trust level on the new computer
        - `$ gpg --edit-key lglforfun@gmail.com` to start gpg
        - `gpg> trust` and select `5` the ultimate trust and save
    - clone `.password-store` to the new computer. Must have exactly the same directory name.
