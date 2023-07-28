## RStudio tricks =============================================================

### WHY: why install an old version of a package is much slower than install current version in Windows?

**Current version**: when installing a current version from CRAN, Windows download an `.zip` file that can be installed very quickly.

**Old version**: when install a past version, Windows 

### HOW: how to use package renv to manage package version in a project and work from terminal?

**project .Rprofile**: add a line to the project .Rprofile so disable global package cache so that all packages are saved within the project to enable quick restore if copied to another computer. 
    ```
    options(renv.config.cache.enabled = FALSE)
    ```
**renv::restore from the project directory**: assuming the project is copied to another computer, go to the project directory and run from terminal. Run `Rscript` from this directory, it automatically picks up `.Rprofile` and installs `renv` and other packages.
    ```shell
    $ Rscript -e "renv::restore()"
    ```
    
**renv::run() from terminal anywhere**: this requires `renv` package installed in the global library. If run from the project directory, the project `renv` package is used.
    ```shell
    $ Rscript -e 'renv::run("/path/to/main.R")`
    ```

### HOW: how to use renv package to restore package installed from local source file

After generating the `renv.lock` file, manually modify the source from "Unknown" to the path to the source file, for example
    ```
    "Source": "payParity_0.3.03.tar.gz"
    ```

### HOW: How to update R version in Windows?

Use `installr::updateR()` from a R console. Only works in Windows as `installr` is not available in Linux.

### lintr and RStudio session

**Problem found**: lintr may report issues in Bitbucker Pipelines that not found in local run. 

**Solution**: The reason is that a local RStudio session caches some objects that may affect lintr's outcome. For a safe lintr check, always start a new R session (not restart R).

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
