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

### concepts
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
 

### built-in variables
https://github.com/adrianlarion/simple-awk

- `FILENAME`: the name of the current file being processed.
- `NR`: number of record, counted from all files
- `FNR`: file number of record, line number of current file being processed.
- `FS`: field seperator, default to space ' '.
- `IGNORECASE`: ignore case if 1, no if 0.
- `NF`: number of field (word) in a line
- `RS`: record seperator, `\n` by default.

