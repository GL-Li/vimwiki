## Outline

- Major reference
- Workflow and SOP
- Minimal examples
- QA
- Raw notes

## Major reference ============================================================

- The Rust Programing Language, The Book: https://doc.rust-lang.org/book/
    - with quiz: https://rust-book.cs.brown.edu/experiment-intro.html
- Rust By Example, RBE: https://doc.rust-lang.org/rust-by-example/
- Udemy: Ultimate Rust Crash Course: https://www.udemy.com/course/ultimate-rust-crash-course/

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

## QA ======= =================================================================

### QA: how to read input from terminal?
    
    ```rust
    use std::io;
    let mut aaa = String.new();
    io::stdin().read_line(&mut aaa).expect("Fail to read line");
    println!("aaa is {aaa}");
    ```



## Raw notes ==================================================================

### 1.1 Installation using rustup

**rustup** is a tool that manage Rust installation, update, and uninstallation.
    - `$ sudo apt purge rustc` to uninstall the old version that is not installed with rustup
    - reinstall Rust with rustup, may change overtime, always use the latest command. Just an example: `$ $ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh`
    - `$ rustup update` to update to a new version
    - `$ rustup self uninstall` to uninstall Rust and rustup.

### 1.2 Hello, World

**rustc is a compiler** that generates an executable file

    - create file `main.rs`
        ```rust
        fn main() {
            println!("Hello, world"); // ! indicates macro, not function
        }
        ```
    - compile the file with `$ rustc main.rs`, which create an executable file `main`.
    - run the executable with `$ ./main`. It can be run without Rust installed.

### 1.3 Hello, Cargo

- [x] create a project with Cargo
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

- [x] elements in the code
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
    - variables are immutable by default, but variable name can be re-declaired as a **new** variable. For example after we define `let x = 3`, we cannot simply reassign a value with `x = 9`. However, we can re-declair `x` with `let x = 9`, which is called **shadowning**.
    - constant can only be declaired once. After declair `const ABC: u32 = 24 * 3600` we cannot declair again with `const ABC: u32 = 24 * 60 * 60`. That is, we cannot assign any value to `ABC` again.
    - `let` can only be used in a function scope to declair variables but `const` can also be used in global scope.

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
        
#### 3.2 data types

**static data type**: As a statically typed language, Rust must know the types of all variables at compile time.

**Literals**: in computer science, a literal represents a fixed value that can be assigned to a variable or constant. For example, `12` is an integer literal, `"abc"` is a string literal, and `["cat", "dog"]` is a list (python) literal.

**scalar types**: A scalar type represents a single value. There four primary scalar types:
    - integer types like `u32`, `i32`, `u64`, `i64`:
        - **unsigned integer** has values like 0, 1, 2, .... No negative values.
        - **signed integer** can have negative, zero, and positive integers.
        - we can add underscore `_` to integer literals. For example, `12_34_567` is the same as `1234567`.
        - The default integer type is `i32`.
        - Integer division returns an integer, truncates toward zero.
    - floating-point types like `f32` and `f64`. 
        - Rust's default is `f64`. All floating-point types are signed.
        - Integer literals cannot be assigned to floating-point variables
            ```rust
            let x: f64 = 123; //wrong
            let x: f64 = 123.0; //correct
            ```
    - numerical operations must match types of two numbers
        ```rust
        let x: i32 = 5 / 3;  //return 1
        let x = -5 / 3; //return -1, Rust can guess data type for x, but slow
        let x = 5.0 / 3; //error, floating-point cannot be devied by integer 
        let x: f32 = 5 / 3; //error, types mismatch, 5 / 3 is integer
        let x: f32 = 5.0 / 3.0; //return 1.66666...
        ```
    - Boolean type with `true` or `false`
        ```rust
        let t: bool = true;
        ```
    - character type is a single character (can be unicode) in signle quote. It is NOT string type.
        ```rust
        let c: char = 'z';
        let heart_eyed_cat = '😻';
        let c: char = "z"; // error in double quote
        let c: char = 'abc'; //error, more than one character
       ```
       
**Compound types** contain multiple values
    - tuple type can group different types. A tuple has a fixed length once created. A empty tuple is called `unit` and is written in `()`.
        ```rust
        let aaa: (i32, f64, char) = (500, 3.14, 't');
        let (x, y, z) = aaa; // destructure the tuple into elements
        println!("aaa has {x}, {y}, {z}");
        
        // or use aaa.0, aaa.1, ..., to access element by index
        let ele_1 = aaa.0
        println!("aaa first element is {ele_1}");
        println!("aaa first element is {aaa.0}"); // error, {...} is a placeholder for a varaible, not a value
        ```
    - array type can only have elements of the same type and have a fixed length. Arrays live in stack, not heap as the length is fixed.
        ```rust
        // declair an array of type u32 and 5 elements
        let arr: [u32; 5] = [1, 2, 3, 4, 5];
        // array of 100 string "abc"
        let arr = ["abc": 100];
        // access array element
        let x = arr[0]
        ```
        
**Standard library types** from RBE  chapt 19.

#### 3.3 Functions

**Statement** performs some action and does not return a value. Therefore a statement cannot be assigned to a variable. Examples
    ```rust
    let y = 6;
    let x = (let y = 6); //error: expected expressioin, found statement ('let')
    ```
    
**Expression** returns a value and can be assigned to a variable.
    ```rust
    // 123 is an expression in
    let x = 123;
    // a new scope block created with curly brackets is an expression
    let y = {
        let x = 3;
        x + 1 // not ending with ;
    };
    // calling a function or macro is an expression
    let z = my_functions();
    ```
    
**Function with return values**: rules to follow
    - types of argument must be specified 
    - type of return must be specified
    - last statement is the return, or return with `return` keyword
    - example
        ```rust
        // good function
        fn add_2(x: i32, y: i32) -> i32 {
            return x + y   // with or without return is ok
        }
        
        // error: return unit (), not i32 as declaired
        fn add_2(x: i32, y: i32) -> i32 {
            x + y;   // not an expression because of ending ;
        }
        ```
        
#### 3.4 control flow

**`if` must use `bool`** data type as condition
    ```rust
    fn main() {
        let number = 6;

        if number % 4 == 0 {
            println!("number is divisible by 4");
        } else if number % 3 == 0 {
            println!("number is divisible by 3");
        } else if number % 2 == 0 {
            println!("number is divisible by 2");
        } else {
            println!("number is not divisible by 4, 3, or 2");
        }
    }
    ```
    
**`if` is an expression** so can be assigned to a variable. The values of the final expressions of each arm must have the same data type, as the compiler needs to determine the variable data type at compiling, NOT at runtime.
    ```rust
    fn main() {
        let condition = true;
        let number = if condition { 5 } else { 6 };

        println!("The value of number is: {number}");
    }
    ```
    
**`loop` runs for ever** until stopped by `break`. We can also use `continue` to skip the rest and start from beginning.
    ```rust
    fn main() {
        let mut i: i32 = 0;
        loop {
            i = i + 1;
            if i > 10 {break};
            println!("again!");
        }
    }
    ```
    
**`loop` is an expression** that can be assigned to a variable. The return of a `loop` is the expression after `break`. The ending `;` does not matter.
    ```rust
    fn main() {
        let mut counter = 0;

        let result = loop {
            counter += 1;

            if counter == 10 {
                break counter * 2; // return, end with or without ; is ok
            } // ok with or without ending :
        };

        println!("The result is {result}");
    }
    ```
    
**`loop` labels** to disambiguate between multiple loops. If a loop is inside another loop, by default, `break` and `continue` apply to the inner-most loop where they are inside. To break their parent loop, we assign a label to that loop. A label starts with a single quote `'` followed by `:`, eg., `'xxxx: loop` in the example below.
    ```rust
    fn main() {
        let mut count = 0;
        'counting_up: loop {
            println!("count = {count}");
            let mut remaining = 10;

            loop {
                println!("remaining = {remaining}");
                if remaining == 9 {
                    break;
                }
                if count == 2 {
                    break 'counting_up;
                }
                remaining -= 1;
            }

            count += 1;
        }
        println!("End count = {count}");
    }
    ```
    
**`while` loop works the same** as in R
    ```rust
    fn main() {
        let mut number = 3;

        while number != 0 {
            println!("{number}!");

            number -= 1; // syntax like Python
        }

        println!("LIFTOFF!!!");
    }
    ```
    
**`for` loop works the same** as in R
    ```rust
    fn main() {
        let a = [10, 20, 30, 40, 50];

        for element in a {
            println!("the value is: {element}");
        }
    }
    ```
    
**range `(1..4)` for 1, 2, 3, 4**
    ```rust
    fn main() {
        for number in (1..4).rev() {  //.rev() to reverse the range
            println!("{number}!");
        }
        println!("LIFTOFF!!!");
    }
    ```
    
### 4. Understanding ownership

#### 4.1 What is ownership

**Safety is the absence of undefined behavior**: any unexpected action by the code potentially causses security problems. 
    - A foundational goal of Rust is to ensure codes never have undefined behavior.
    - A second goal of Rust is to prevent undefined behavior at compile-time, instead of run-time, which improves software reliability and performance.

**Frames are where variables live**: a frame is a mapping from variables to values within a single scope. A scope is defined by `{...}`, for example:
    ```rust
    fn main() {     // frame for function main holds the following:
        let n = 5;  //n:5 (L1, location 1)
        let y = plus_one(n); // y:6, (L3)
        println!("The value of y is: {y}");
    }

    fn plus_one(x: i32) -> i32 {  // frame for plus_one holds:
        x + 1  // x:5 as x is assigned with n in main() (L2)
    }
    ```

**Frames are organized into a stack of currently-called-functions**: in above example, two frames are involved in a stack when function `plus_one` is called, This stack can be shown as:
    ```
    main() frame, dropped after last line executed
    plus_one frame, dropped after return in main()
    ```

**Box stores data in heap**: box is a construct in Rust for putting data in heap. In the example below, only one copy of the one million array lives in heap, which is not in any stack frame. Instead, two pointers for a and b are in the stack frame of main.
    ```rust
    fn main() {
        let a = Box::new([0; 1_000_000]); // create a box in array with pointer a
        let b = a; // move the pointer to b. Pointer a deleted
    }
    ```
    
**data structure Vec, String, and HashMap are boxes**: they are stored in heap when created with a pointer stored in stack frames.

**Garbage collection** is used by many other languages to manage memory, which regularly looks for no-longer-used memory while running. The programmer must explicitly allocate and free the memory.

**stack, heap, pointer**: memory available to code at **runtime**
    - **stack stores values of known fixed size** in memory in first-in-first-out order. Adding data is called **pushing on** to the stack. Removing data is called **popping off** the stack.
    - **heap allocates a space in memory identified by a pointer to an unknown or changing value**. The pointer, whose size is known and fixed, is stored in a stack. To add data, the program first locate the pointer in stack, which then lead to the heap.

**Ownership rules**
    - Each value in Rust has an owner.
    - There can only be one owner at a time.
    - When the owner goes out of scope, the value will be dropped.

**string literals** are fixed string with type `&str`
    - used when the value of a string is known at compile time
    - is hard-coded like `"abc"`, must inside double quote!!!!
    - do not use `mut` as by definition, string literal are immutable
    ```rust
    let s1 = "Hello";
    let s2: &str = "World";
    let mut s3 = "whatever";   //warning, mut not needed but not an error
    ```

**String type** provided by standard library
    - https://linuxhint.com/strings-in-rust/
    - ways to create String object
    - `let s = string::from("hello")`: generated from a string literal
    - methods for string type
        - `s.push_str(", world")`: to append a string literal to a string
        - 

**`&String` type is a reference to a `String`**
    - It has no ownership. It exists as long as the `String` it points to is not out of scope.

### 4.2 referrence and borrowing

**When ownership moved?** 
    - only applied to varaibles in heap not in stack
        ```rust
        // for varaibles in stack, ownership not moved
        let x = 5;
        let y = x; // value of x, 5, is copied to y. Two copies of 5.  Both x and y exist
        println!("x is {x} and y is {y}")  // no problem
        
        // for variables in heap, ownership moved
        let a = String::from("xyz");
        let b = a;  // ownership of "xyz" moved to b. Only one "xyz" in heap. 
                    // variable a is not valid anymore
        printlin!("a is {a} and b is {b}")  // error as a does not exists anymore
        ```
    - heap varaible ownership move happens when reassigned or used in a function call

**References are non-owning pointers** prefixed with ampersand operator `&`. It borrows value from a heap variables but does not take the ownership of the value.
    ```rust
    fn main() {
        let m1 = String::from("Hello");
        let m2 = String::from("world");
        greet(&m1, &m2); // note the ampersands
        let s = format!("{} {}", m1, m2);
    }

    fn greet(g1: &String, g2: &String) { // note the ampersands
        println!("{} {}!", g1, g2); // as g1 and g2 does not own "Hello" and 
                                    // "World", when they dropped after the 
                                    // funcdtion call in main, the values persist
    }
    ```
    
**Dereferencing a pointer to get its value** of a mutable heap variable
    - explicit dereferencing with an asterisk `*`, not used very often but need to understand how it works
       ```rust
      let mut x: Box<i32> = Box::new(5);  // use Box to put 5 into heap
      let a = *x;  
        // x is a pointer-value pair. The * removes the pointer, which is called
        //   dereferencing, so *x is the value only
        // let a = *x; assigns the value to a. As a has no pointer, it lives in stack.
        //   This assignment is possible as x is a i32 value. Not valid if x is a
        //   str type.
      *x += 1;  // modify the value of x to 6
      
      let r1: &Box<i32> = &x;  // shared borrowing 
        // &Box<i32> means reference to a value of type Box<i32>.
        // &x is the value of x through reference of the pointer of x.
        // let r1 = &x; assign x's value to r1 through reference.
        // The operator `=` assigns value, not pointer. `&x` means assign
        //   value through pointer.
        // r1's value is 6 but does not own the value.
      
      let b: i32 = **r1;  
        // *r1 --> *&x --> x so **r1 --> *x
        // b is 6 and lives in stack
                          
      let r2: &i32 = &*x; 
        // r2 is a reference to a i32 object, which has value. 
        // &* create an immutable reference. A heap value cannot have two or
        //   more mutable references but can have multiple references, 
        //   including 0 or 1 mutable reference.
      ```
      
**Inexplicit dereference and reference** in `.` method. Depends on how the method and function are defined. Below are just a few specific examples.
    ```rust
    let x: Box<i32> = Box::new(-1);
    let abs1 = i32::abs(*x);
        // explicit dereference, otherwise type error
        // function i32::abs() only takes i32 integers
        
    let abs2 = x.abs(); 
    let abs3 = &x.abs();
        // inexplicit deferencing, the .abs() will corce the type to
        //   match the method
    
    let abs4 = *x.abs();
        // error, *x is a value without any reference so it cannot be 
        //  dereferenced
    ```
    
**Ponter safety principle**: data should never be aliased and mutated at the same time.
    - Alasing is accessing the same data with different variable.
    - when a data is mutated, its reference may also change, so the alias could loses reference.
