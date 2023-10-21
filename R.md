## General R settings =========================================================

### QA: how to set up options and environment variables for R?

- Set up options in `.Rprofile` and uses options
    - set options by adding line
      ```r
      options(warning.length = 100)
      ```
    - view options with `getOptions(warning.length)`
    - view all options with `options()`.
    
- Specify environment variables in `.Renviron` file and use them.
    - add line:
        ```
        AAA=abcdefg
        ```
    - access the environment variable with `Sys.getenv("AAA")`

- There are three locations for these files: 
    - project root
    - user home, in Linux it is `~/`.
    - R home, check with `R.home()`.

- When a R session starts, it first look them at the project root. If not find, look at home, and then R home.

- If they are important to a project, it is recommended to place them in project root so can be shared on other computers.

### QA: how to access system info from R?

`> .Platform$OS.type` to show OS, return "windows", "unix", or ...
`> R.home()` to show where is R installed.

### QA: how to install packages from local files for Windows?

**Use windows binary files** is much faster and does not need RTools
- Window binaries can be downloaded from CRAN. There are three versions for r-devel, r-release, and r-oldrel, for different R releases, for example, R-4.4, 4.3, and 4.2. 
- No older Windows binaries, but can be converted from source file using https://win-builder.r-project.org/
- Install with `install.packages("/path/to/my_package.zip", repos = NULL, type = "win.binary")`

**Use source file** is slow and need RTools if the build contains `C/C++/Fortran` code as they need `make` to compile.

### QA: how to create Windows binary files for private packages?

- `devtools::build_win_release(email = "xxxx@yyyy.com", manual = FALSE)`
- The command create a `tar.gz` source file and sends the file to https://win-builder.r-project.org/, where Window binary file is built
- Once the Window binary file is ready, an email will be sent to the provided email address.
- Follow the link in  the address to download the binary file.


### QA: how to use package renv to manage package version in a project and work from terminal?

**project .Rprofile**: add a line to the project .Rprofile so disable global package cache so that all packages are saved within the project to enable quick restore if copied to another computer. 
    ```
    options(renv.config.cache.enabled = FALSE)
    ```
**renv::restore from the project directory**: assuming the project is copied to another computer, go to the project directory and run from terminal. Run `Rscript` from this directory, it automatically picks up `.Rprofile` and installs `renv` and other packages.
    ```sh
    $ Rscript -e "renv::restore()"
    ```

**renv::run() from terminal anywhere**: this requires `renv` package installed in the global library. If run from the project directory, the project `renv` package is used.

    ```shell
    $ Rscript -e 'renv::run("/path/to/main.R")`
    ```

### QA: how to use renv package to restore package installed from local source file

After generating the `renv.lock` file, manually modify the source from "Unknown" to the path to the source file, for example
    ```
    "Source": "payParity_0.3.03.tar.gz"
    ```

### QA: How to update R version in Windows?

Use `installr::updateR()` from a R console. Only works in Windows as `installr` is not available in Linux.

### QA: why local lintr run and Bitbucket Pipelines lintr run give different reports although using the same Docker image?

**Problem found**: lintr may report issues in Bitbucker Pipelines that not found in local run. 

**Solution**: The reason is that a local RStudio session caches some objects that may affect lintr's outcome. For a safe lintr check, always start a new R session (not restart R).


### QA: how to control the RStudio server working directory at start?

The starting working directory can be managed by file `/etc/rstudio/rsession.conf` by adding lines:
    ```
    # change ~/working to whatever you like. It is created automatically if not exist
    # in docker container, use full path instead of ~/
    session-default-working-dir=/home/rstudio/working
    ```
    
If copy `rstudio-prefs.json` to containers `/home/rstudio/.config/rstudio/` make sure the line
```
"initial_working_directory": "/home/rstudio/project/` matches.
```

See https://docs.posit.co/ide/server-pro/rstudio_pro_sessions/directory_management.html#:~:text=The%20default%20working%20directory%20for,conf%20config%20file.


### QA: how to view more columns in RStudio viewer?

The default `View(df1)` displays 50 columns. To increase the default, run from R console

 `> rstudioapi::writeRStudioPreference("data_viewer_max_columns", 500L)` 

to increase to 500 columns.

## package development ========================================================

### QA: how to suppress print in unit test?

Use `expect_output(res <- my_function(...))` to hide print out.

### QA: how to use and write package vignettes?

- `> browseVignettes(package = "dplyr")` to open all vignettes of a package in web brower
- `browseVignettes()` to open vignettes of all installed packages
- Vignettes are R markdown file in subdirectory `vignetters/`. Example: https://github.com/tidyverse/dplyr/tree/main/vignettes

### QA: how to handel  non-ASCII code in package

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

### QA: how to exclude files and lines from coverage check using covr?

- `covr::package_coverage(line_exclusions = "R/xxxx.R")` to exclude a file from coverage check.
- `cover::package_coverage(line_exclusion = list("R/xxxx.R", "R/zzzz.R" = c(1:10, 15, 16)))` to exclude a file and selected lines of another file from coverage check.

-
-
-
## R coding ==================================================

### QA: R is a very flexible with types, which potentially causes unexpected results without showing any errors or warnings. What are the common functions have this kind of danger?

- `all(character(0) == 0)` is `TRUE`. This is so dangerous, as empty case should not be simply treated as TRUE. We want to identify some real cases that make sense.

- `length(levels(x))`: the code is supposed to check the number of levels of a factor. It also works when `x` is a character. This will give incorrect results as shown in the code snippet below:
    ```r
    x <- c("aaa", "bbb", "ccc")
    if (length(levels(x)) < 2) {   # expect nothing happen but since levels(x) is NULL
        # do something             # and length(NULL) is 0. The code will do sth.
    }
    ```
    

### QA: what are some readr::read_csv tricks? 1) how to suppress message in readr::read_csv functon? 2) check all rows to determine column type.

- `readr::read_csv(..., show_col_types = FALSE)` to suppress message.
- `readr::read_csv(..., guess_max = Inf)` to read all rows then determine column type. Default is 1000 rows. So if the first 1000 rows are empty or in different type, the reading result might be wrong.

### QA: how to check memory usage in a R process?

Use `bench::bench_process_memory()` to retrieve current and maximum memory from the R process. The reported number include all memory relevant to the R process.
