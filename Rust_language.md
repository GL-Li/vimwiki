## Outline

- Major reference
- Workflow and SOP
- Minimal examples
- Glossaries
- Raw notes

## Major reference ============================================================

- The Rust Programing Language, The Book: https://doc.rust-lang.org/book/

## Workflow and SOP ===========================================================

### SOP: manage project with cargo

- commands
    - `$ cargo new xxx` to create a new project.
    - `$ cargo run` to run the project for test purpose
    - `$ cargo check` to quickly check if the project is compilable.
    - `$ cargo build --release` to build executable for release. It is an optimized build and is slow than simple `$ cargo build`.
    - `$ cargo update` to update dependencies
    - `$ cargo doc --open` to generate and view documentation of the project.
- files
    - `Cargo.toml`: specify dependencies and versions. `$ cargo update` will only update to the lastes of the major version. For example, if the version is `8.5.2` in the file, the update only update it to the latest `8.x.x` but not `9.x.x`.
    - `Cargo.lock`: locks the dependency versions of the project when the project was first build or run. The future `$ cargo run` and `$ cargo build` will use the versions locked in this file unless run `$ cargo update`.
    - to update the major version, must manually modify `Cargo.toml` file.

## Minimal examples ===========================================================

## Glossaries =================================================================

## Raw notes ==================================================================

### 1.2 Hello, World

- [x] writing and running a Rust program

    - create file `main.rs`
        ```rust
        fn main() {
            println!("Hello, world"); // ! indicates macro, not function
        }
        ```
    - compile the file with `$ rustc main.rs`, which create an executable file `main`.
    - run the executable with `$ ./main`. It can be run without Rust installed.

### 1.3 Hello, Cargo

- [ ] create a project with Cargo
    - Cargo is Rust's build system and package manager
    - `$ cargo new hello_cargo` to create a new project and change directory to this project.
    - The initial files and directories

        |-- Cargo.toml
        |-- src/
            |-- main.rs
        |-- .git/
        |-- .gitignore

    - file `Cargo.tmol`, the configuration for build
        ```toml
        [package]
        name = "hello_cargo"
        version = "0.1.0"
        edition = "2021"

        # See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

        [dependencies]
        ```

    - `$ cargo run` to **build** and **run** the project. The resulting directory is shown below. The build results are in `target/`.
        .
        ├── Cargo.lock
        ├── Cargo.toml
        ├── src
        │   └── main.rs
        └── target
            ├── CACHEDIR.TAG
            └── debug
                ├── build
                ├── deps
                │   ├── hello_cargo-509b9f0437440349
                │   └── hello_cargo-509b9f0437440349.d
                ├── examples
                ├── hello_cargo
                ├── hello_cargo.d
                └── incremental
                    └── hello_cargo-3126xnd42m8w0
                        ├── s-gli30nzpvg-lrek9b-2d30rzle7wwfe
                        │   ├── 2i9jyl3pxdf5etxo.o
                        │   ├── 31zlg2245xt8ksz7.o
                        │   ├── 3np6xw9hm19hjnfl.o
                        │   ├── 4fk6laxkr9oabmrl.o
                        │   ├── 4kt3ygatlq124166.o
                        │   ├── dep-graph.bin
                        │   ├── mn88feykr2k9mmk.o
                        │   ├── query-cache.bin
                        │   └── work-products.bin
                        └── s-gli30nzpvg-lrek9b.lock

    - `$ cargo check` to quickly check if the project can be compiled. It is fast as it does not generate executable.

    - `$ cargo build --release` to compile with optimization and create an executable in `target/release`, which contains several files and directories. But the executable `hello_cargo` can be copied and run anywhere.

        └── release
            ├── build
            ├── deps
            │   ├── hello_cargo-57332a145d3af13d
            │   └── hello_cargo-57332a145d3af13d.d
            ├── examples
            ├── hello_cargo
            ├── hello_cargo.d
            └── incremental

    - what are in the `release` directory?
        - only the executable is required to run
        - all others are artifacts for quick rebuild.

### 2. Programming a guessing game

- [ ] elements in the code
    - `use std::io;`  use library `io` from standard library `std`
    - `std::prelude` defines items that are automatically imported
    - `io` and many standard libraries are also prelude but needs to be imported with `use std::xxx`.
    - `let mut guess = String::new();`
        - `let` to create a variable
        - `mut` specifies a mutable variable. By default a variable is immutable in Rust
        - `String` is a **string type** provided by `std::prelude` and function `String::new()` generate a new instance of string.
    - `io::stdin().read_line($mut guess).expect("Failed to read line");`
        - `io::stdin()` creates an instance of `Stdin` for standard input handle
        - `read_line(&mut guess)` is a method of standard input handle that read the standard input and append the input in the mutable variable `guess`.
            - `&` indicates the this argument is a reference to `mut guess`
        - `expect("Failed to read line")` to handle potential failure
            - `read_line` also return a **Result** (called **enum**) that can be one of multiple states such as `Ok` and `Err`. Each state is called a **variant**.
            - `expect()` is a method of Restult type
        - `println!("You guessed: {guess}");` where `{}` is a placeholder for the values of `guess`
            - other use of place holder, empty placeholder holds value of expression after it.
                ```rust
                let x = 5;
                let y = 10;
                println!("x = {x} and y + 2 = {}", y + 2);
                ```
    - crate is a collection of Rust source code files.
        - binary crate: this guessing game project is to create a binary crate, which is an executable
        - library crate: like R packages, containing code to be used in other programs.
    - use library crates
        - declare dependencies in `Cargo.toml` file
            ```toml
            [dependencies]
            rand = "0.8.5"
            ```
        - `$ cargo build` to download crate `rand` and its dependencies
    - `Cargo.lock` file lock the versions of dependencies the first time they are compiled
        - the version remains the same as specified by this file in all later build unless the command says otherwise explicitly
        - `$ cargo update` to update the version of dependencies
    - `$ cargo doc --open`: Generate documentation for the project, which list help documentation of functions and dependencies of the project.

### 3 Common programming concepts

#### 3.1 variables and mutability

**concurrent programming** is a form of programming in which several processes run concurrently during overlapping time periods.

**constant vs immutable variables**: constant will never change
    - variables are immutable by default, but variable name can be re-declaired. For example after we define `let x = 3`, we cannot simply reassign a value with `x = 9`. However, we can re-declair `x` with `let x = 9`, which is called **shadowning**.
    - constant can only be declaired once. After declair `const ABC: u32 = 24 * 3600` we cannot declair again with `const ABC: u32 = 24 * 60 * 60`. That is, we cannot assign any value to `ABC` again.

**shadowing and scope**: shadowing is only valid within its scope
    - curly braces `{...}` define a scope. A shadowing inside it will **NOT** affect the variables outside it
    - example: in the code below, `let x = x * 2` inside `{}` does not change x value out side it.
        ```rust
        fn main() {
            let x = 5;

            let x = x + 1; //now 6

            {
                let x = x * 2; // here x is 12
                println!("The value of x in the inner scope is: {x}");
            }

            println!("The value of x is: {x}"); // x is still 6 !!!!!
        }
        ```
