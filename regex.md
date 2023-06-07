## Outline
    - Major reference
    - Workflow and SOP
    - Minimal examples
    - Glossaries
    - Raw notes

## raw notes

### regex resources

- https://regex101.com/

### regex meta characters

- In R, the escape backslash is `\\` instead of `\`. For example, to escape `+`, we use `\\+`. That is, `\` itself needs escape, which is different from other languages.

- special characters
    - `\\d` matches digits
    - `\\D` matches non-digits
    - `\\w` matches any word characters (alphanumeric and underscore)
        - only `a-zA-Z0-9`, not other language letters
    - `\\W` matches any non-word characters
    - `\\s` matches any whitespace character (spaces, tabs, line breaks)
    - `\\S` matches any non-whitespace characters

- non printable characters
    - `\\t` for tab
    - `\\n` for new line
    - `\\r` carriage return, used togeter with `\\n` on Windows as `\\r\\n`. It moves cursor down first and then move all the way to the left.

- character set and ragne in `[]`
    - `[aAcdF]it` matches ait, Ait, cit, dit, and Fit.
    - character range like `[1-7]`, `[a-h]` and `[1-7A-J]`
    - `[^xxx]` to negate character set or range
    - meta characters inside `[]` will become literal character except for `^`. To be safe, better still to escape them.

### regex concepts, mode, anchor, quantifier, greedy, lazy, possessive

- modes / flags
    - global mode: search across text for all occurrence, otherwise search for first occurrence
    - case insensitive mode: ignore cases
    - multiline
    - single line

- anchors / positional anchors
    - `^` and `$`: start and end of a line
    - `\b`: word boundary, `"\bcat\b"` matches "this is a cat" but not "these are bobcats"
    - `\B`: non-word boundary, `"\Bcat\B"` matches "these are bobcats" instead of "this is a cat"
    - `\A` and `\Z`: start and end of the whole file.

- repetition with quantifiers `?`, `*`, `+`
    - `?` optional:
        - `cats?` matches "cat" or "cats", zero or one "s"
            - use "cat(s)?` to make "s" optional explicitly.
        - `max(imum)?` matches "max" and "maximum".
    - `*` repeat zero or more times
    - `+` repeats one or more times

- limit repetiton
    - `{5}`: repeat 5 times
    - `{3,5}` repeats 3, 4, or 5 times
    - `{3,}` repeats 3 or more times

- greedy lazy possessive repetition
    - `?` `*` and `+` are greedy, returns all matches
    - `??` `*?` and `+?` are lazy, returns few matches as possible
        ```R
        x <- "<h1> heading one </h1>"
        # .+> matches all until hitting the last >
        str_extract_all(x, "<.+>")
        # [[1]]
        # [1] "<h1> heading one </h1>"

        # .+?> matches all until hitting the first >
        str_extract_all(x, "<.+?>")
        # [[1]]
        # [1] "<h1>"  "</h1>"
        # being lazy means `.+?` matches as few characters as possible before find `>`.

        ```
    - `?+`, `*+`, and `++` are possessive that does not backtrack. It is faster than lazy and greedy search.
        ```R
        y = "this is a dog"
        str_extract(y, ".+dog")
        # [1] "this is a dog"
        # .+ includes dog but then track back for dog to complete the match

        str_extract(y, ".++dog")
        # [1] NA
        # .++ include dog but will not tack back so cannot find match for dog in the patern and returns nothing.

        ```

### regex concepts: capturing groups, backreference

- capturing groups in `(xxx)`
    ```R
    x <- c("123 456 789", "987 654 321 - 987 654")
    str_match(x, "^(\\d{3})\\s(\\d{3})")
    # each element include an overall match followed by each capture groups
    #      [,1]      [,2]  [,3]
    # [1,] "123 456" "123" "456"
    # [2,] "987 654" "987" "654"

    str_match(x, "^(\\d{3})\\s(\\d{3})\\s(\\d{3})(\\s-\\s\\1\\s\\2)?")
    # \\1 and \\2 backreference to the first and second capture
    # groups for repeated matching
    #      [,1]                 [,2]  [,3]  [,4]  [,5]
    # [1,] "123 456 789"        "123" "456" "789" NA
    # [2,] "987 654 321 - 987 " "987" "654" "321" " - 987 654"
    ```

- capture groups with names for easy reference using `(?<aaa>xxx)` inside
    ```R
    names = c("Jone Smith", "Jay F Brown")
    str_match(names, "(?<firstname>\\w+)\\s?(?<middlename>\\w+)?\\s(?<lastname>\\w+)")
    #                    firstname middlename lastname
    # [1,] "Jone Smith"  "Jone"    NA         "Smith"
    # [2,] "Jay F Brown" "Jay"     "F"        "Brown"
    ```

- non-capturing group `(?:xxx)` which match is not returned as an seperate object but is in the overall matched string.
    ```R
    cars <- c("this is a green car", "this is a red car")
    str_match(cars, "(?:red|green)\\s(?<car>car)")
    # the first group is used but not captured
    #                  car
    # [1,] "green car" "car"
    # [2,] "red car"   "car"
    ```

### regex positive and negative lookahead and lookbehind

Not returned and not in the overall matched string.

- lookahead with `(?=xxx)` to look ahead of "xxx"
    ```R
    x <- c("abc def ghi jkl", "123 ghi 456")

    # look for word ahead of "ghi"
    str_match(x, "(?<word>\\w+)\\s?(?=ghi)")
    #             word
    # [1,] "def " "def"
    # [2,] "123 " "123"
    ```

- lookbehind with `(?<=xxx)` to look behind "xxx"
    ```R
    # look for word after "ghi"
    str_match(x, "(?<=ghi)\\s?(?<word>\\w+)")
    #             word
    # [1,] " jkl" "jkl"
    # [2,] " 456" "456"

    ```

- negative lookahead with `(?!xxx)` to find match not before "xxx"
    ```R
    # find ghi that is not before "456"
    str_match(x, "(?<word>ghi)(?!(?:\\s?456))")
    #            word
    # [1,] "ghi" "ghi"
    # [2,] NA    NA
    ```

- negative lookbehind with `(?<!xxx)` to find match not behind "xxx"
    ```R
    # find ghi that is not after "def"
    str_match(x, "(?<!(?:def\\s?))(?<word>ghi)")
    #            word
    # [1,] NA    NA
    # [2,] "ghi" "ghi"
    ```

### regex atomic group with `(?>xxxx|yyy|zz)` to exit the group after the first match inside

[ref](https://www.abstractsyntaxseed.com/blog/regex-engine/atomic-groups)

To make it useful, order the alternatives from longest to shortest. In this order, we are sure that if `xxxx` matches, `yyy` must be `xxx` in order to match, therefore no need to try again.

Atomic group is rarely used by normal regex users.

### practices

#### extract all 4 factors

The output should be "rcethn.male", "jobTitle.Sr. Manager", "tenure", and "Number of report". This is a good practice that help understanding greedy match and lazy match, and also lookahead.

```R
aaa <- c("rcethn.male (20)", "jobTitle.Sr. Manager (17)",
         "tenure X10", "Number of report")

# use lazy match, which will pickup |$ as alternative.
# - with greedy match, .+ will match all till line end and
#   then check for the alternatives in the lookahead patterns.
#   It will find the last one "$" and return all string as a factor.
#
# - with lazy match, .+? try to match as few as possible that
#   also matches a pattern in the lookahead alternatives and have
#   a valid return.
str_match(aaa, "(?<factor>^.+?)(?=\\s\\(\\d+\\)$|\\sX\\d{2,}$|$)")
# or
str_extract(aaa, "(?<factor>^.+?)(?=\\s\\(\\d+\\)$|\\sX\\d{2,}$|$)")
# str_extract returns overall matching. In this case, it is the
# same as the capturing group
```
