## packages

### readr
- `readr::read_csv(..., show_col_types = FALSE)` to suppress message.

### covr
- `covr::package_coverage(line_exclusions = "R/xxxx.R")` to exclude functions in a file from coverage check.
- `cover::package_coverage(line_exclusion = list("R/xxxx.R", "R/zzzz.R" = c(1:10, 15, 16)))` to exclude a file and selected lines of another file from coverage check.
