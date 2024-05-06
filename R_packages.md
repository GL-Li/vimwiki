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