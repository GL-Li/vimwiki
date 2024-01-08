## Trusaic package workflow

### Develop package in WSL and end2end test in Windows

**The complication**:
- The package is development in WSL linux file system because RStudio docker container runs faster.
- The code calling the package functions run on Windows file system as the payParity platform runs on Windows file system.
- We need to build package source file in Linux file system and copy the source file to Windows file system and install in Windows.

**Method**:

- Create an executable bash script `build_payparity` in `~/bin/` to create the source file and copy it to Windows
    ```sh
    #! /usr/bin/env bash

    # delete tar.gz files in parent directory
    rm ../*tar.gz

    # build package source file payParity_xxxxx.tar.gz
    # into parent directory ../
    Rscript -e "devtools::build()"

    # copy the newly created tar.gz file to 
    # /mnt/d/DiversityEquityInclusion/payParity.tar.gz
    cp ../*tar.gz /mnt/d/DiversityEquityInclusion/payParity.tar.gz

    echo "copied to DEI and renamed as payParity.tar.gz"
    echo " --- "
    ```
    
- Create a batch file, `install_payParity.bat` in Windows to install package from source file
    ```bat
    "C:\Program Files\R\R-4.3.1\bin\Rscript.exe" -e "install.packages('payParity.tar.gz', repos = NULL, type='source')"

    del payParity.tar.gz
    ```
    
- How to use them:
    - `$ build_payparity` run from package directory in WSL
    - `> install_payparity.bat` run from DEI directory in Window.


### Manage installed packages

- delete installed third-party packages but keep all base packages

    ```r
    # all packages
    all_pkg <- installed.packages() |>
        rownames()
    # all base packages
    base_pkg <- installed.packages(priority = "high") |> 
        rownames()
    # non-base packages
    non_base_pkg <- setdiff(all_pkg, base_pkg)
    # remove non-base packages
    remove.packages(non_base_pkg)
    ```

- to install a set of packages in Windows from a file, run from command
    ```
    > "C:\program Files\R\R-4.3.1\bin\Rscript.exe" dependencies_windows.R
    ```

- have a test run
    ```
    > "C:\program Files\R\R-4.3.1\bin\Rscript.exe" pea1.R
    ```

- use `renv` to identify the minimal set of packages required for a project
    ```r
    # initialize renv project and check the env.lock file for installed
    # packages for this project. They are installed in the order.
    renv::init()
    ```



## General R settings =========================================================

### QA: Excel workbook: open save close to activate formulas

```r
#' This function equals to open a workbook, save it and then close it in Excel.
#' 
#' @description
#' The workbook created in selfSrv run has cells that populated by formula. 
#' These cells appears as NA when read with openxlsx::read.xlsx and other
#' packages. This issue can be fixed by opening the workbook in Excel and save
#' it with Ctr S manually. This function replaces this manual work.
#' 
#' @param data_dir string, directory where the workbook lives
#' @param workbook string, workbook name
#' 
#' @return no return

open_save_workbook <- function(data_dir,
                               workbook = "Regression Details Ingestion.xlsx") {
  # https://stackoverflow.com/questions/19404270/run-vba-script-from-r

  if (.Platform$OS.type != "windows") {
    stop("open_save_workbook only works on Window")
  }
  
  library(RDCOMClient) # RDCOMClient::COMCreate not working without the library
  
  workbook <- paste0(data_dir, workbook) |>
    stringr::str_replace_all("/", "\\\\")
  
  xlApp <- COMCreate("Excel.Application")  
  xlWbk <- xlApp$Workbooks()$Open(workbook)
  xlWbk$Save()
  xlWbk$Close(FALSE)
  xlApp$Quit()
  
  cat("\nThe workbook is opened, saved, and closed.\n")
}
```


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

- From RStudio running on a **Window machine**, run `devtools::build(binary = TRUE)` to generate a `.zip` file, which is a Windows binary.
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


## packages ======================================

### future package for parallel computing

- good for long time and small global objects
    - only globals used by `future({...})` are exported to new R session.
    - default limit if 500 MB that can be exported
    - edit the limit with `options(future.globals.maxSize = 1e9)`
    - memory not released in future session in interactive run
    - all memory release if run with `Rscript aaa.R`, takes time to release large memory.

- example code:
    ```R
    library(future)
    # use miltisession for parallel computing
    plan(multisession)
    # print out process ID just to show that different session are used
    print(Sys.getpid())

    # create a future which takes at least 10 sec to finish
    f1 <- future({
      Sys.sleep(10)
      print(Sys.getpid())
      cat("f1: Hello World!\n")
      "111"
    })

    # in multisession, no need to wait 10 sec for f1
    print("f1 created")

    # another future taking 10 sec
    f2 <- future({
      Sys.sleep(10)
      print(Sys.getpid())
      cat("f2: Hello World!\n")
      "222"
    })

    # no need to wait for f2
    print("f1 and f2 futures created ------")
    
    # after 10 sec, both f1 and f2 are completed at background so we
    # evaluate v1 and v1. Multisession saves 10 secs compared to sequential
    v1 <- value(f1)
    print(v1)
    v2 <- value(f2)
    print(v2)

    print("end of the test")
    ```
    
- same code put in function and check time:
    ```r
    library(future)
    plan(multisession)
    print(Sys.getpid())

    fff <- function() {
      f1 <- future({
        Sys.sleep(10)
        print(Sys.getpid())
        cat("f1: Hello World!\n")
        "111"
      })

      print("f1 created")

      f2 <- future({
        Sys.sleep(10)
        print(Sys.getpid())
        cat("f2: Hello World!\n")
        "222"
      })

      print("f1 and f2 futures created ------")
      v1 <- value(f1)
      print(v1)
      v2 <- value(f2)
      print(v2)

      print("end of the test")

      return(list(f1 = v1, f2 = v2))
    }

    # finish in 10.3 secs instead of 20+
    system.time({
      f <- fff()
    })
    ```
    
- code to test memory exported to future session
    ```R
    options(future.globals.maxSize = 5e9)
    library(future)
    plan(multisession)
    print(Sys.getpid())

    # generate a large global object
    x <- rnorm(5e8)

    Sys.sleep(10)
    print("x created")

    fff <- function() {
      f1 <- future({
        # this future session need this large object
        # x is exported to this session
        xxx <- length(x)
        print(xxx)
        Sys.sleep(10)
        print(Sys.getpid())
        cat("f1: Hello World!\n")
        "111"
      })

      print("f1 created")

      f2 <- future({
        # this future session does not need this large object
        Sys.sleep(10)
        print(Sys.getpid())
        cat("f2: Hello World!\n")
        "222"
      })

      print("f1 and f2 futures created ------")
      v1 <- value(f1)
      print(v1)
      v2 <- value(f2)
      print(v2)

      print("end of the test")

      return(list(f1 = v1, f2 = v2))
    }

    system.time({
      f <- fff()
    })
    ```
    
    
### doFuture package to parallelize for loops

- basic example:
    ```R
    options(future.globals.maxSize = 5e9)
    library(doFuture)
    plan(multisession)


    # just to check time, can be removed
    # completed in 10.5 seconds instead of 40+ seconds
    system.time({
      # run each in parallel, return a list
      y <- foreach(x = 3:6) %dofuture% {
        Sys.sleep(10)
        c(sqrt(x), x, x^2)  # final return
      }
    })

    # y is a list y[[1]], y[[2]], ... for the return of each x
    y
        # [1] 1.732051 3.000000 9.000000
        #
        # [[2]]
        # [1]  2  4 16
        #
        # [[3]]
        # [1]  2.236068  5.000000 25.000000
        # 
        # [[4]]
        # [1]  2.44949  6.00000 36.00000
    ```


### Use future with for loops without foreach

To break a large data into small pieces and feed each small piece into a thread, we can use the future function, which is more flexible than `foreach ... %dofuture%`. 

```R
options(future.globals.maxSize = 5e9)
library(doFuture)
plan(multisession)
library(dplyr)

N <- 20
dat_raw <- data.frame(
  group = rep(letters[1:N], 100000),
  value = rnorm(2000000)
)

# set up futures
t0 <- Sys.time()
for (grp in letters[1:N]) {
  dat <- dat_raw |>
    filter(group == grp)
  assign(paste0("group", grp), future({
    Sys.sleep(10)
    c(total = sum(dat$value), avg = mean(dat$value))
  }))
}
print(Sys.time() - t0)

# collect values
res <- list()
t0 <- Sys.time()
for (x in letters[1:N]) {
  res[[x]] <- value(get(paste0("group", x)))
}
print(Sys.time() - t0)
```
