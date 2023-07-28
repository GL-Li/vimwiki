
## General R settings =========================================================

### HOW: how to install packages from local files for Windows?

**Use windows binary files** is much faster and does not need RTools
    - Window binaries can be downloaded from CRAN. There are three versions for r-devel, r-release, and r-oldrel, for different R releases, for example, R-4.4, 4.3, and 4.2. 
    - No older Windows binaries, but can be converted from source file using https://win-builder.r-project.org/
    - Install with `install.packages("/path/to/my_package.zip", repos = NULL, type = "win.binary")`

**Use source file** is slow and need RTools if the build contains `C/C++/Fortran` code as they need `make` to compile.

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

### WHY: why local lintr run and Bitbucket Pipelines lintr run give different reports although using the same Docker image?

**Problem found**: lintr may report issues in Bitbucker Pipelines that not found in local run. 

**Solution**: The reason is that a local RStudio session caches some objects that may affect lintr's outcome. For a safe lintr check, always start a new R session (not restart R).

### HOW: how to view more columns in RStudio viewer?

The default `View(df1)` displays 50 columns. To increase the default, run from R console `> rstudioapi::writeRStudioPreference("data_viewer_max_columns", 500L)` to increase to 500 columns.

-
-
-
## package development ========================================================

### HOW: how to handel  non-ASCII code in package

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

### HOW: how to exclude files and lines from coverage check using covr?

- `covr::package_coverage(line_exclusions = "R/xxxx.R")` to exclude a file from coverage check.
- `cover::package_coverage(line_exclusion = list("R/xxxx.R", "R/zzzz.R" = c(1:10, 15, 16)))` to exclude a file and selected lines of another file from coverage check.

-
-
-
## R coding ===================================================================

### HOW: how to suppress message in readr::read_csv functon?

- `readr::read_csv(..., show_col_types = FALSE)` to suppress message.

