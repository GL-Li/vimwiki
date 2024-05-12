## base

**options()**

Set or change global options:

```r
# change or add option aaa to value "AAA" but return the original options
orig_ops <- options(aaa = "AAA")

# retrive option aaa
x <- getOption("aaa")  # x is "AAA"

# retore to original options
options(orig_ops)

# now the aaa is NULL as it does not exist
getOption("aaa")
```

**on.exit()**

Record expression to be run when a function exits (success or failure). It can be used to restore options and working directory.

```r
# restore options
print_2_digits <- function(x) {
  # keep a copy of original options and then change digits
  op <- options(digits = 2)
  # retore options when exiting function, even function failed. So place
  # on.exit() right after the option changed and before possible failure
  on.exit(options(op))
  print(1 / x)
}
print(3.14159265)           # all digits
print_2_digits(3.14159265)  # 3.1
print(3.14159265)           # all digits
print_2_digits("abcdefg")   # error
print(3.14159265)           # all digits

# restore working directory
read_file <- function(dir_path, file_name) {
    wd_oroginal <- getwd()
    setwd(dir_path)
    on.exit(setwd(wd_original)
    
    read.csv(file_name)
}
```

**.onLoad()**

In package development, `.onLoad` function is used to run code when `library(aaa)` a package. The function is usually placed in `R/zzz.R` (yes, zzz.R) like

```r
.onLoad <- function(libname, pkgname) {
    # load required packages
    if (!require(ggplot2)) {
        install.packages("ggplot2")
        library(ggplot2)
    }
    
    # some other actions
}
```

**tryCatch**

`tryCatch` is used to continue the code execution in case of an error (or warning). It is slow so do not use it in very large for loop.

The standard use case: if the expression is successful, returns the output of the expression. If there is error or warning, returns `NULL`  or specified values.

```r
beera <- function(expr){
  tryCatch(expr,
         error = function(e){
           message("An error occurred:\n", e)
           return("hahaha")  # without this, returns NULL in case of error
         },
         
         # the waring section is optional. Without it, tryCatch just return the output
         # of the expr, for example, as.numeric(c("1", "one")) returns c(c, NA)
         warning = function(w){
           # if a warning, print out this message and return nothing
           message("A warning occured:\n", w)
           return(99999)  # without this, returns NULL in case of warning
         },
         
         # The finally section can be used for cleaning up database connections and
         # logging information to a file. Never use it for return value as it will
         # overwrite any previous return.
         finally = {
           message("Finally done!")
         })
}

beera(1 + 1)  # 2
beera(1 / 0) # Inf
beera(as.numeric("1", "one"))  # NULL, with warning message
```


## cli

https://github.com/r-lib/cli/

Used to format output on R console. It allows R code in the messages.

```
> library(cli)
> pkgs <- c("aaa", "bbb", "ccc")
> cli_alert_success("Downloaded {length(pkgs)} packages.")
✔ Downloaded 3 packages.

> f <- "aaa.txt"
> cli_alert_danger("Failed to open the file {f}")
✖ Failed to open the file aaa.txt

> cli_h1("heading 1")

── heading 1 ───────────────────────────────────────────────────────────────────────────────────────────────────
> cli_h2("heading 2")

── heading 2 ──

> cli_h3("heading 3")
── heading 3 
```

Progress bar

```r
clean <- function() {
  cli_progress_bar("Cleaning data", total = 100)
  for (i in 1:100) {
    Sys.sleep(5/100)
    cli_progress_update()
  }
}
clean()
```

## withr
https://cran.r-project.org/web/packages/withr/vignettes/changing-and-restoring-state.html

This package is used to temporarily change global state within a scope. The global state is restored when out of the scope.It has two set of functions

- `with_xxx(...)`: changes state of `xxx` inside this function
- `local_xxx(...)`: usually called inside a custom function and change the state inside the custom function

**Examples** graphics parameters

```r
# the default global color and pch for graphic parameters are "black" and 1
# which can be checked with par("col") and par("pch")
plot(mtcars$hp, mtcars$wt) # the plot takes global par parameters

# with_par changes parameters in function par() inside the scope
# of with_par(...)
withr::with_par(
    list(col = "red", pch = 19), # change color and pch
    plot(mtcars$hp, mtcars$wt) # the plot takes par parameters inside this scope
)

# out of the scope, restore to global par parameters
plot(mtcars$hp, mtcars$wt) # the plot takes global par parameters


# local_par changes state inside my_plot function
my_plot <- function() {
  withr::local_par(list(col = "red", pch = 19))
  plot(mtcars$hp, mtcars$wt) # the plot takes par parameters inside this scope
}
my_plot()
```

**Examples** working directory

```r
# change working directory in a function using local_dir function so we eo not 
# need to using getwd and setwd to change directory back and forth
run_selfSrv <- function(selfSrv_dir) {
    withr::local_dir(selfSrv_dir)
    system("Rscript pea1.R")
}
```


## processx

The package provide enhanced functionality (`processx::run()`) compared  base R's `system()` and `system2()` function. Use this package if the two base functions fail to do. Otherwise stick to the base R functions.
