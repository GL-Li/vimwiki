## RStudio tricks =============================================================

### RStudio viewer max columns
The default `View(df1)` displays 50 columns. To increase the default, run from R console `> rstudioapi::writeRStudioPreference("data_viewer_max_columns", 500L)` to increase to 500 columns.

## package development ========================================================

### non-ASCII code in package
**Non-ASCII characters are not allowed in package**. They will have to be converted to unicode.
- use `stringi::stri_escape_unicode(c("ú", "ñ",  "ü"))` to convert non-ASCII code to unicode. Replace "\\" with "\" as shown in following example.
    ```R
    special_char <- c(
        # regex specials
        "+", "*", "?", "(", ")", "[", "]", "{", "}", "|", "^", "$", "!", "'", '"',
        "&", "%", "@", "#", "/",
        # Spanish letters, cannot use non-ASCII in package
        # "á", "é", "í", "ó", "ú", "ñ",  "ü",
        "\u00e1", "\u00e9", "\u00ed", "\u00f3", "\u00fa", "\u00f1", "\u00fc",
        # "Á", "É", "Í", "Ó", "Ú", "Ñ", "Ü"
        "\u00c1", "\u00c9", "\u00cd", "\u00d3", "\u00da", "\u00d1", "\u00dc"
    )
    ```

## third-party packages =======================================================

### readr
- `readr::read_csv(..., show_col_types = FALSE)` to suppress message.

### covr
- `covr::package_coverage(line_exclusions = "R/xxxx.R")` to exclude functions in a file from coverage check.
- `cover::package_coverage(line_exclusion = list("R/xxxx.R", "R/zzzz.R" = c(1:10, 15, 16)))` to exclude a file and selected lines of another file from coverage check.
