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
