## Outline

- Major reference
- Workflow and SOP
- Minimal examples
- QA
- Snippets 
- Raw notes

## Major reference ===

- The Rust Programing Language, The Book: https://doc.rust-lang.org/book/
    - with quiz: https://rust-book.cs.brown.edu/experiment-intro.html
- Rust By Example, RBE: https://doc.rust-lang.org/rust-by-example/
- Udemy: Ultimate Rust Crash Course: https://www.udemy.com/course/ultimate-rust-crash-course/
- Easy Rust: https://dhghomon.github.io/easy_rust/Chapter_21.html

## Workflow and SOP ===

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

## Minimal examples ===

## QA ===

### QA: what is the error "cannot move of out xxx which is behind a reference?

This happens when you try to move a variable you do not own and the data does not implement Copy trait..
```rust
fn main() {
    let sq = Square {r: 3.0};
    let bb = &sq;
    let sq1 = bb.enlarge(9.9); // bb is moved into the method and becomes self.
                               // self dererences all the way down to sq
                               // automatically and so sq will be deleted
                               // after the method completed. As bb does not
                               // own the data, Rust does not allow the deletion
                               // with an error message "Cannot move out of '*bb'
                               // which is behind a shared reference". move occurs
                               // because Square does not implement the 'Copy' trait
    println!("{:?}", sq1);
}

// if allow Copy with #[derive(Debug, Copy, Clone)], there will be
// no more "move out of ..." error
#[derive(Debug)] 
struct Square {
    r: f64,
}

impl Square {
   fn enlarge(self, x: f64) -> Square {
        Square {
            r: self.r + x,
        }
    } 
}
```

### QA: What are the differences between String and &str?  --- not completed

**String is a data structure stored in heap**

- used for dynamic string manipulation.
- access through a pointer (reference) whose size is 24 bytes, including pointer, length and capacity of the string. The actual string data have variable size.
- When talking about the size of a String object like in `let s = String::from("abc")`, which is 24 bytes, it is the size of the pointer in stack, not the actual data in heap. Actually `s` lives in stack and has a value of a pointer. The pointer points to the data in heap. `s` is just the pointer.
- create with ways like:
    - `let aaa = String::from("aaa")`
    - `let bbb = "bbb".to_string()`
    - using macro `format!`
        ```rust
        let a = "aaa";
        let b = "bbb";
        let c = "ccc";
        let abc = format!("String {}{}{}", a, b, c)
            // a new String with data "String aaabbbccc"
        ```

**&str is a borrowed reference to a string literal or a slice of String**. 
- use it whenever possible as it is faster and simple than `String`.
- It does not own the data and is immutable.
    ```rust
    let x = "xxx";  // a &str type with fixed data and lives in stack
    let xs - "xxx".to_string(); // a string type and "xxx" is in heap
    ```

### QA: what are the ways of print with formatter {}?

- `println!("x is {x}");` - the basic if x is a varaible
- `println!("x is {}", x);` - the basic if x has trait of display
- `println!("x is {:?}", x);` - debugging print, print as is
- `println!("x is {:?}", x);` - beautified print
- `println!("x is {:p}", x);` - print memory address
- `println!("x is {} and y is {}", x, y);` - print two object
- `println!("x is {1} and y is {0}", y, x);` - print two objects with order, the index starts from 0, **not** 1.
- name expressions in print
    ```rust
    let x = 1;
    let y = 2;
    println!("xplus is {xplus} and yplus is {yplus}",
             xplus = x + 1,  // name expression x + 1 with xplus
             yplus = y + 1);
    ```
- format inside {}
    ```rust
    let x = 1;
    let y = 2;
    println!("three * at each side of x: {:*^3} and 3 = to the left of y: {:=>3}", x, y)
    ```

### QA: how to read input from terminal?
```rust
use std::io;
let mut aaa = String.new();
io::stdin().read_line(&mut aaa).expect("Fail to read line");
println!("aaa is {aaa}");
```


## Snippets ===

### snippet: s.as_bytes().iter().enumerate() to iterate bytes of a String
The byte of a character is an integer, for example, `b'a'` is 97, `b' '` is 32. In the example, `b` and single quote of a charater gives the byte representation of the character.

```rust
let s = String::from("abc def ghi");
for (i, &item) in s.as_bytes().iter().enumerate() {
  println!("{i}: {item}");
};
```

### snippet: std::env::args().nth(1).unwrap_or_else(||{println!("something")}) to read the first argument in `cargo run arg1 arg2`

```rust
fn main() {
  let arg: String = std::env::args().nth(1).unwrap_or_else(|| {
    println!("Please supply an argument to this program")
  };
  println!("The input is {arg}");
}
```

# The Rust Book ============

## 1. Getting started

### 1.1 Installation using rustup

**rustup** is a tool that manage Rust installation, update, and uninstallation.

- `$ sudo apt purge rustc` to uninstall the old version that is not installed with rustup
- reinstall Rust with rustup, may change overtime, always use the latest command. Just an example: `$ $ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh`
- `$ rustup update` to update to a new version
- `$ rustup self uninstall` to uninstall Rust and rustup.

### 1.2 Hello, World

**rustc is a compiler** that generates an executable file

- create file `main.rs`

    ```[rust](rust)
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

## 2. Programming a guessing game

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

## 3 Common programming concepts

### 3.1 variables and mutability

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

### 3.2 data types

**static data type**: As a statically typed language, Rust must know the types of all variables at compile time.

**Literals**: in computer science, a literal represents a fixed value that can be assigned to a variable or constant. For example, `12` is an integer literal, `"abc"` is a string literal, and `["cat", "dog"]` is a list (python) literal.

**scalar types**: A scalar type represents a single value. There four primary scalar types:
- integer types like `u32`, `i32`, `u64`, `i64`:
    - **unsigned integer** has values like 0, 1, 2, .... No negative values.
    - **signed integer** can have negative, zero, and positive integers.
    - we can add underscore `_` to integer literals. For example, `12_34_567` is the same as `1234567`.
    - The default integer type is `i32`.
    - Integer division returns an integer, truncates toward zero. For example, 5/3 is 1.
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

### 3.3 Functions

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

### 3.4 control flow

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

## 4. Understanding ownership

### 4.1 What is ownership

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

**`&String` type is a reference to a `String`**
- It has no ownership. It exists as long as the `String` it points to is not out of scope.

### 4.2 referrence and borrowing

**When ownership moved?** 
- Generally speaking, move happens to types that does not implement copy trait.
- Custom Struct will be moved if copy trait is not implemented.
- For the built-in types, those lives in heap will be moved, not those in stack
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

**Dereferencing a pointer to get its value** of a mutable heap variable. 

- A variable `x` has two parts: a pointer and a value: `&x` is the pointer and `*x` is the value. 

  ```rust
  fn main() {
      let x = Box::new(5);  // &x is in stack and *x is in heap
      let r1 = &x;  // r1 is in stack, which is a copy of the pointer of x
      let r2 = r1;  // r2 is a copy of r1
      let r3 = &&x; // r3 is a reference to the pointer of x
      let r4 = &r1; // r4 is a reference to r1
      println!("{:p}", r1);  // r1 and r2 are at the same memory address
      println!("{:p}", r2);  
      println!("{:p}", r3);  // r3 has its own address
      println!("{:p}", r4);  // r4 has its own address
      
      println!("{:p}", x);   // x has its address before move
      let y = x;
      println!("{:p}", y);   // y has the same address as x before x's move
  }
  ```

  

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
- example with error
    ```rust
    fn main() {
        let mut vec: Vec<i32> = vec![1, 2, 3];
        let num: &i32 = &vec[2];
        printlin!("num is {num}")
            // no problem to use num here
        
        vec.push(4);
        println!("print num again: {num}");
            // error when access num again, vec has been altered by the
            //   push and num may lose reference to &vec[2]
    }
    ```

**Read, Write, and Own permisions**: relationship between variables and their data
- The relationship is checked within compiler, not during runtime.
- example
    ```rust
    fn main() {
        let mut vec: Vec<i32> = vec![1, 2, 3];
            // initiate vec with RWO permision
        
        let num: &i32 = &vec[2];
            // initiate num with RO as immutable. It borrows data from vec
            // so vec becomes R only
            
        println!("Third element of vec is {}", *num);
            // This is the line where num is last used, returning borrowed
            // data from vec and then dropped. After the return, vec re-gain
            // RWO of the data.
            
        println!("This is the last time use vec, whose first element is {}",
                 vec[0]);
            // vec dropped as it will out of scope.
    }
    ```
- permissions are defined on paths that include variables. Paths are things like `x`, `&x`, `*x`, `y[0]`, `z.1`, and `*&x`, ...
    ```rust
    // when vec is inmmutable
    let vec: Vec<i32> = vec![1, 2, 3];
    let mut x1: i32 = vec[2];
    x1 += 100;
        // x1 is a regular mutable i32 varaible
        
    let x2: &i32 = &vec[2];
        // &i32 is reference, which is immutable as vec is immutable
    
    // when vec is mutable
    let mut vec: Vec<i32> = vec![1, 2, 3];
    let x3: &mut i32 = &mut vec[2];
    *x3 += 100;
        // &mut indicate mutable reference 
        // *x3 has write permission to data. 
        // both x3 and vec[2] changed to 103.
        // vec is not accessible until x3 is dropped
    ```

**Borrow checker finds permission violations**: Creating an immutable reference to a data cause the data to be temporarily read-only until the reference dropped.
- in plain English: I borrow it with no intention to change it. You can use it but do not change it either until I return it.

**Mutable references allows mutation but prevents aliasing**: When a mutable reference is created the original variable loses access to the data until the reference dropped.
- in plain English: I borrow it and will change it. You please do not touch it until I return it.

**Data must outlive all of its references** especially when a function returns a reference, make sure the data the reference point to is still alive after function return.


### 4.3 Fixing ownership errors

**When creating a reference, always keep in mind when the value is going to be returned**. Most ownership errors are caused by fogetting this.

**Error Case study: returning a reference to the stack**
```rust
fn return_a_string() -> &String {
    let s = String::from("Hello world");
    &s  // s is dropped after function return so the returned reference
        // points to nowhere
}
```
**Error case study: not enough permissions**
```rust
fn stringify_name_with_title(name: &Vec<String>) -> String {
    name.push(String::from("Esq."));  // name is immutable
    let full = name.join(" ");
    full
}
```

### 4.4 the slice type

**Range syntex** `&s[..3]`, `&s[2..5]`, `&s[3..]`, `&s[..]`
- To get the `&str` type of whole String `s`, we can use `&s` and `&s[..]` interchangebly. `&s[..]` is explicitly as a `&str` and so is prefered.
- example 1
    ```rust
    let s = String::from("hello world!");
    let len = s.len();
    let s1 = &s[3..len];
    let s2 = &s[3..]
    
    let aaa = &s[..];  // type &str
    let bbb = &s;      // type &String but also &str
    ```
- example 2 in function
    ```rust
    fn test(s: &String) -> &str {
      let s1 = &s[..];  // type &str
      let s2 = &s;      // type &&String also &tr
      s1
    }
    ```

 **&String vs &str**
 - A `&String` is a normal reference to a String and takes 8 bytes in stack
 - A `&str` is a reference to a slice of a String. It takes 16 bytes in stack, 8 for the reference and 8 for the length of the slice.

### 4.5 Ownership recap

**Example showing various pointers**
- code:
    ```rust
    fn main() {
      let mut a_num = 0;   // lives in stack
      inner(&mut a_num);
    }
    
    fn inner(x: &mut i32) {  // x is a ponter poiting to the value of a_num
      let another_num = 1;  // lives in stack
      let a_stack_ref = &another_num;  // ponter to value of another_num
    
      let a_box = Box::new(2);  // a pointer poiting to a heap data
      let a_box_stack_ref = &a_box;  // a pointer poiting to the pointer of a_box
      let a_box_heap_ref = &*a_box;  // a pointer poiting to the heap value, it has
                                     // no ownership and is a immutable borrowing
    
      *x += 5;
    }
    ```
    
## 5. Use Structs to structure related data

### 5.1 Defining and instantialing Structs

**What is a struct**: a collection of different types of data.

- **Define a struct**
    ```rust
    struct User {
        active: bool,
        username: String,
        email: String,
        sign_in_count: u64,
    }
    ```

- **Instantiate a struct**
    ```rust
    let user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };
    ```
- **Mutate a struct instance**: must initiate the whole struct as mutable, not individual field.
    ```rust
    let mut user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };
    user1.email = String::from("aaa@gmail.com");
    ```

- **Use field init shorthand**: when function parameters have the same name as the struct fields, they can be passed to the struct fields directly
    ```rust
    fn build_user(email: String, username: String) -> User {
            User {
                active: true,
                username,     // no need to be username: username
                email,        // no need to be email: email
                sign_in_count: 1,
            }
        }
    ```
- **Use struct update to create instance from other instance**: be aware that if the heap field of the old instance is moved, it cannot be used anymore.
    ```rust
    let user2 = User {
        email: String::from("another@example.com"),
        ..user1   // the rest fields moved in from user1, including username, which is
                  // a String in heap, therefore user1 is no longer available.
    };
    ```
    

**Special structs**

- Using **tuple structs** without named fields to create different types: a tuple struct looks like a tuple. It is a named tuple. This struct has no field names.
    ```rust
    struct Color(i32, i32, i32);
    struct Point(i32, i32, i32);
    
    let black = Color(0, 0, 0);   // black and origin are different types
    let origin = Point(0, 0, 0);
    ```

- **uint-like** structs without any fields: it is useful to implement a trait on a type but don't have any data to strore in the type.
  
    ```rust
    struct AlwaysEqual;
    let aaa = AlwaysEqual;
    ```
    

**Borrowing fields of a struct**
- borrow checker will track ownership at both the struct and field level.
    ```rust
    struct Point {x: i32, y: i32};
    
    fn main() {
        let mut p = Point {x: 0, y: 0};
        let x = &mut p.x;
        let y = &p.y;
        println!("({}, {})", x, y);
        
        // error, mutable borrow above
        let x1 = &p.x;
        println!("({}, {})", x, y);
    }
    ```

### 5.2 An example programming using stucts

**Debug printing** with `#[derive(Debug)]`
```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

let rect = Rectangle {
    sidet: 30,
    height: 20,
};

println!("The rectangle is {:?}", rect);
// or
dbg!(&rect)  // dbg! moves ownership while println! only use reference.
```

### 5.3 Method syntax

**Use impl to implement a method within the scope of a struct**

- `self` as function input: differences between `self`, `&self` and `&mut self`
    ```rust
    #[derive(Debug)]
    struct Rectangle {
        width: u32,
        height: u32,
    }
    
    impl Rectangle {
        fn area(&self) -> u32 {      // reference to borrow value
            self.width * self.height
        }
    }
    
    // ok to put multiple method in one impl
    impl Rectangle {
        fn grow(&mut self, x: u32) {  // mutable reference to change value
            self.width += x;  // u32 so do not dereference
            self.height += x;
        }
        
        fn delete(self) {  // move ownership to delete
            // do nothing but self is deleted
        }
    }
    
    fn main() {
        let mut rect1 = Rectangle {
            width: 30,
            height: 50,
        };
    
        rect1.grow(10);
        println!(
            "The rectangle is now {:?}, which area is {}.",
            rect1,
            rect1.area()
        );
    
        rect1.delete();
    }
    ```

- `Self` as return to generate an instance
    ```rust
    impl Rectangle {
        fn square(size: u32) -> Self {
            Self {
                width: size,
                height: size,
            }
        }
    }
    
    let sq = Rectangle::square(3);   // call function square
    ```
    

**struct method will automatically referencing and derefencing to match the self parameters**

```rust
fn main() {
    let rect = Box::new(Rectangle {
        width: 1,
        height: 2,
    });
    println!("Area is {}", rect.area()); // auto dereferenncing from Box to Rectangle
}

struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}
```

## 6. Enums and pattern matching

### 6.1 Defining an enum

**enum defines a type that has multiple variants**

- example: IP address has two versions: v4 and v6. V4 is represented by 4 integers like those in 192.168.0.3, which can be written in tupple (192, 168, 0, 3); V6 IP is a string such as 2001:db8:3333:4444:CCCC:DDDD:EEEE:FFFF. We can define an enum for IP address which has V4 and V6 variants:
    ```rust
    enum IpAddr {
      V4(u8, u8, u8, u8),
      V6(String)
    }
    ```
- create IP address variables
    ```rust
    let ip_v4 = IpAddr::V4(192, 168, 0, 3);
    let ip_v6 = IpAddr::V6(String::from("2001:db8:3333:4444:CCCC:DDDD:EEEE:FFFF"));
    ```
    
- define methods for enum just like for struct
    ```rust
    impl IpAddr {
      fn do_sth(&self) {
        // do something here
      }
    }
    ```
    

**option enum - a special builtin enum**

- why need it: Option enum is used to handle `null` cases as Rust does not have `null`. Option enum avoids errors when using null as a non-null value.

- definition of option enum: it is defined in standard library below, where `<T>` is a generic type parameter that can be any type.
    ```rust
    enum Option<T> {
      None,
      Some(T),
    }
  ```

- use of option enum. Option enum, `Some` and `None` are included in prelude when Rust is launched.
    ```rust
    let some_number = Some(5);  // type Option<i32>
    let some_string = Some(String::from("hello"));  // type Option<String>
    let absent_number: Option<i32> = None;  // define type for None
    ```
    
- Option enum is its own type
    ```rust
    let x = 5;   // i32
    let y = Some(8); // Option<i32>
    let sum = x + y; // error, different types
    ```
    
- builtin method for Option enum: https://doc.rust-lang.org/std/option/enum.Option.html
    - check if an option is Some or None and return a boolean. If a Some, we can add additional conditions.
        ```rust
        let x = Some(2);
        assert_eq!(x.is_some(), true);
        assert_eq!(x.is_none(), false):
        assert_eq!(x.is_some_and(|x| x > 1), true) // |x| x > 1 is a closure
        ```
   - extract value containted in an option and handle the case of None 
        ```rust
        assert_eq!(x.expect("x should be 2"), 2);) 
            // if x is None, panic with the message "x should be 2"
        
        assert_eq!(Some("stuff").unwrap(), "stuff"); //returns the contained Some value
        let z: Option<&str> = None;
        assert_eq!(z.unwrap_or("sth"), "sth")  // used to handle None
        
        let t = 5;
        assert_eq!(None.unwrap_or_else(|| 2 * t), 10);
        ```
        
### 6.2 The match control flow construct

**an enum is often used with match** to explore each variants

 - basic example: the match must exhaust all possibilities
    ```rust
    enum Coin {
      Peny,
      Nickel,
      Dime,
      Quarter,
    }
    
    // match all variants
    fn value_in_cents(coin: Coin) -> u8 {
      match coin {
        Coin::Peny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
      }
    }
    
    // if we only care about peny and quater
    fn value_in_peny_quarter(coin: Coin) -> u8 {
      match coin {
        Coin::Peny => 1,
        Coin::Quarter => 25,
        _ => (),  // catch all other cases
      }
    }
    ```
    
- match patterns that bind to values to extract values out of enum variants
    ```rust
    fn main() {
        let q1 = Coin::Quarter("Indiana".to_string());
        let q1_cents = value_in_name(q1);
        println!("value in cents is {}", q1_cents);
    }
    
    enum Coin {
        Peny,
        Nickel,
        Dime,
        Quarter(String),  // value of the String can be extracted, example below
    }
    
    fn value_in_name(coin: Coin) -> String {
        match coin {
            Coin::Peny => "one".to_string(),
            Coin::Nickel => "five".to_string(),
            Coin::Dime => "ten".to_string(),
            Coin::Quarter(state) => state,  // state is the String in Quarter
        }
    }
    ```
    
- matching with `Option<T>`
    ```rust
    fn main() {
        let x = Some(5);
        let y = plus_one(x);
        println!("y is {:?}", y);
    }
    
    fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            None => None,
            Some(i) => Some(i + 1),  // add inside Some()
        }
    }
    ```
    

**How matches interact with ownership**

- match consumes x if x is not copyable
```rust
    fn main() {
        let x = Some(String::from("hello"));
        let y = plus_one(x);
        println!("y is {:?}", y);
        // println!("x is {:?}", x);  // x is moved to plus_one as it has String
    }

    fn plus_one(x: Option<String>) -> Option<String> {
        match x {
            None => None,
            Some(s) => Some(s + "_add_1"),
        }
    }
```

- match on a reference to avoid moving 
    ```rust
    fn main() {
        let opt: Option<String> = 
            Some(String::from("Hello world"));
    
        // opt became &opt to avoiding moving
        match &opt {  
            Some(s) => println!("Some: {}", s),
            None => println!("None!")
        };
    
        println!("{:?}", opt);
    }
    ```
    
### 6.3 Concise control flow with `if let`

- a less verbose way to handle values that match one pattern while ignoring the rest:
    ```rust
    //the following two code blocks behaves the same
    
    // block 1
    let config_max = Some(3u8);
    match config_max {
        Some(max) => println!("The maximum is configured to be {}", max),
        _ => (),  // have to add this line to conver other possibilies though not interested in
    }
    
    // block 2
    let config_max = Some(3u8);
    if let Some(max) = config_max {   // ignore other possibilities
        println!("The maximum is configured to be {}", max);
    }
    ```
    
- `if let ... else ...` is not concise compared to `match`. Simply use match if need to check multiple possibilities.
    ```rust
    // these two blocks are equivalent
    
    // block 1
    let mut count = 0;
    match coin {
        Coin::Quarter(state) => println!("State quarter from {:?}!", state),
        _ => count += 1,
    }
    
    // block 2
    let mut count = 0;
    if let Coin::Quarter(state) = coin {
        println!("State quarter from {:?}!", state);
    } else {
        count += 1;
    }
    ```


## 7. Managing growing project with packages, crates, and modules

### 7.1 Packages and Crates

**A create is the smallest amount of cde that the Rust compiler considers at a time**

- binary crate: compiled to an executable such as command-line program. Must have a function called `main()`.
- library crate: do not have main and do not compile to executable. They define functions to be shared with other projects. 
- a package is a bundle of one ore more crates that provides a set of functionality. A package contains a `cargo.toml` file that describes how to build thoese crates. A package can contain at most one library crate and multiple binary crates.

### 7.2 Defining modules to control scope and privacy

**Modules cheat sheet**: how compiler works
- start from the crate root: which is `src/main.rs` for binary crate and `src/lib.rs` for library crate.
- declaring modules in the root file
- declaring submodules in any file other than the root file
- path to code in modules like `crate::garden::vegetables::Asparagus`.
- private vs public: default to private, use `pub` to make it public
- the `use` keyword
- example: a binary crate named `backyard`:
    - structure
        ````
        backyard
        |-- Cargo.lock
        |-- Cargo.toml
        |-- src
            |-- garden
            |   |-- vegetables.rs
            |-- garden.rs
            |-- main.rs
        ```
    - `src/main.rs`
        ```rust
        use crate::garden::vegetables::Asparagus;
        
        pub mod garden;
        
        fn main() {
            let plant = Asparagus {};
            println!("I'm growing {:?}!", plant):
        }
        ```
        

**Grouping related code in modules**: example restaurant library
- create a library crate project with:
    - `$ cargo new restaurant --lib`    where `--lib` for library crate. It create files:
        ```
        ├── Cargo.toml
        └── src
            └── lib.rs
        ```
    - add the following code in file `src/lib.rs`. The code defines a nested module structure.
        ```rust
        mod front_of_house {
            mod hosting {
                fn add_to_waitlist() {}
        
                fn seat_at_table() {}
            }
        
            mod serving {
                fn take_order() {}
        
                fn serve_order() {}
        
                fn take_payment() {}
            }
        }
        ```
    - module tree of the crate:
        ```
        crate
         └── front_of_house
             ├── hosting
             │   ├── add_to_waitlist
             │   └── seat_at_table
             └── serving
                 ├── take_order
                 ├── serve_order
                 └── take_payment
        ```
        
### 7.3 Paths for referring to an item in the module tree

**absolute and relative path to a function** not working if the module of the function is private, or the function is private
- absolute path starts from `crate`
- relative path starts by default from current module
- example. In file `src/lib.rs` we have code:
    ```rust
    // front_of_house is accessible to function eat_at_restaurant as they are in
    // the same scope
    mod front_of_house {
        // hosting is private in a scope not accessible to function eat_at_resturant
        mod hosting {  
            fn add_to_waitlist() {}
        }
    }
    
    pub fn eat_at_restaurant() {
        // Absolute path not working for private hosting module
        crate::front_of_house::hosting::add_to_waitlist();
    
        // Relative path not working for private hosting module
        front_of_house::hosting::add_to_waitlist();
    }
    ```
    

**Exposing paths with the pub keyword** to expose modules and functions that are private to functions calling them.
- example
    ```rust
    mod front_of_house {
        // both hosting module and add_to_waitlist function need to be public
        pub mod hosting {
            pub fn add_to_waitlist() {}
        }
    }
    
    pub fn eat_at_restaurant() {
        // Absolute path
        crate::front_of_house::hosting::add_to_waitlist();
    
        // Relative path
        front_of_house::hosting::add_to_waitlist();
    }
    ```
    

**Starting relative paths with super** to access modules and functions in parent module
- one `super::` to parent module
    ```rust
    fn deliver_order() {}
    
    mod back_of_house {
        fn fix_incorrect_order() {
            cook_order();
            super::deliver_order();
        }
    
        fn cook_order() {}
    }
    ```

- double `super::super::` to grandparent model
    ```rust
    // the main.rs file
    mod grandparent_module {
        pub fn grandparent_function() {
            println!("This is the grandparent function");
        }
    }
    
    mod parent_module {
        pub mod child_module {
    
            pub fn access_grandparent_function() {
                // Accessing the grandparent function using super::super::
                super::super::grandparent_module::grandparent_function();
            }
        }
    }
    
    fn main() {
        parent_module::child_module::access_grandparent_function();
    }
    ```
    

**Making structs and enums public**: 

- struct need to make public field-by-field case
    ```rust
    mod back_of_house {
        pub struct Breakfast {      // public as a whole
            pub toast: String,
            seasonal_fruit: String, // keep private from access individually
        }
    
        impl Breakfast {
            pub fn summer(toast: &str) -> Breakfast {
                Breakfast {
                    toast: String::from(toast),
                    seasonal_fruit: String::from("peaches"),
                }
            }
        }
    }
    
    pub fn eat_at_restaurant() {
        // Order a breakfast in the summer with Rye toast
        let mut meal = back_of_house::Breakfast::summer("Rye");
        // Change our mind about what bread we'd like
        meal.toast = String::from("Wheat");
        println!("I'd like {} toast please", meal.toast);
    
        // The next line won't compile if we uncomment it; we're not allowed
        // to see or modify the seasonal fruit that comes with the meal
        // meal.seasonal_fruit = String::from("blueberries");
    }
    ```

- enum made public as a whole
    ```rust
    mod back_of_house {
        pub enum Appetizer {  // all variants are public 
            Soup,
            Salad,
        }
    }
    
    pub fn eat_at_restaurant() {
        let order1 = back_of_house::Appetizer::Soup;
        let order2 = back_of_house::Appetizer::Salad;
    }
    ```
    

### 7.4 Bring paths into scope with  the use keyword

**why**: The path `a::b::c::d()` is repetitive. We can use `use` to bring `d()` into a scope.

**Creating idiomatic use paths**

- For functions, `use` goes all the way down to the module containing the functions, so we know here the function is NOT locally defined with an managiable path. It also avoids namespace confilict.
    ```rust
    mod mod1 {
      pub mod2 {
        pub fn f1() {}
        pub fn f2() {}
      }
    }
    use crate::mod1::mod2;  // path to the parent module of functiions
    pub fn my_fun() {
      mod2::f1();
      mod2::f2();
    }
    ```
- For structs and enums, `use` the full path, probably rarely two structs or enums having the same name.
    ```rust
    use std::collections::HashMap;  // full path to the struct
    
    fn main() {
        let mut map = HashMap::new();
        map.insert(1, 2);
    }
    ```
    

**Providing new names with the `as` keyword**

- We can also use `as` to rename a function, struct, or enums to a shorter one or to avaoid name conflict
    ```rust
    use std::fmt::Result;
    use std::io::Result as IoResult;  // rename to avoid conflict
    
    fn function1() -> Result {
        // --snip--
    }
    
    fn function2() -> IoResult<()> {
        // --snip--
    }
    ```
    

**Re-exporting names with pub use**

- `pub use` brings names from other module to current one and use these names as if from current module.
    ````rust
    mod restaurant {
        pub mod front_of_house {
            pub mod hosting {
                pub fn add_to_waitlist() {}
            }
        }
    
        // pub use re-export hosting mod to under mode restaurant
        pub use crate::restaurant::front_of_house::hosting;
        
        // re-export mod room from other mod to current on
        pub use crate::hotel::room;
    
        pub fn eat_at_restaurant() {
            hosting::add_to_waitlist();
        }
    }
    
    mod hotel {
        pub mod room {
            pub fn desk();
        }
    }
    
    pub fn eat_at_restaurant() {
        // skip front_of_house in the parh as hosting is exported under restaurant
        restaurant::hosting::add_to_waitlist();
        // use mod room from restaurant mod instead of hotel
        restaurant::room::desk();
        // of course we can still use desk from hotel
        hotel::room::desk();
    }
    ```
    

**Using external packages**: to use an external package, take package rand for example:

- first add the package to `Cargo.toml` under `[dependencies]` so that the package and its dependencies will be downloaded for the project.
    ```toml
    [package]
    ...
    ...
    ...
    
    [dependencies]
    rand = "0.8.5"
    ...
    ```

- then bring the package to scope with `use rand::xxx`:
    ```rust
    // Rng is a trait of rand package. use rand::rng brings all the trait
    // into scope, such as rand::thred_rng.gen_range() function.
    use rand::Rng;
    fn main() {
        let secret_number = rand::thread_rng().gen_range(1..=100);
    }
    ```
    
- run `$ cargo run` to install rand and its dependencies if not already install, and run the code.

**Using nested paths to clean up large use lists**

- using nested paths to convert multiple `use` statements into one. Examples:
    ```rust
    // before
    use std::cmp::Ordering;
    use std::io;
    // after converting into nested path
    use std::{cmp::Ordering, io};
    
    // before
    use std::io;
    use std::io::Write;
    //after
    use std::io::{self, Writing};
    ```
    
- to bring all public modules and functions into scope, use the glob operator:
    ```rust
    use std::collectiions::*;
    ```

### 7.5 Separating modules into different files

- basic structure: the root file `lib.rs` or `main.rs` lists the names of the top modules that can be called with `crate::xxx`. Then define seperate files `xxx.rs` with the same names as the modules. for example
    - root file `src/lib.rs`
        ```rust
        // list all modules
        mod restaurant;
        mod hotel;
        
        // use the modules
        use crate::restaurant::kitchen;
        
        pub fn cooking () {
            kitchen::knife();
        }
        ```
    - `src/restaurant.rs`
        ```rust
        pub mod kitchen {
            pub fn knife() {)
        }
        ```

- further seperate mod kitchen into a file `kitchen.rs`. In this case we need to create a sub directory `restaurant` so the file is `src/restaurant/kitchen.rs` and the files changed to:
    - `src/restaurant.rs`:
        ```rust
        pub mod kitchen;
        ```
    - `src/restaurant/kitchen.rs`:
        ```rust
        pub fn knife() {}
        ```
        



## 8. Common collections

### 8.1 Storing lists of values with vectors

**vector basics Vec<T>**

- A vector only stores values of the same type.
- two ways to create a vector
    ```rust
    // method 1: create an empty vector
    let v1: Vec<i32> = Vec::new();
    
    // method 2: create a vector with initial values, type inferred
    let v2 = vec![1, 2, 3];
    ```
- updating a vector with `.push`
    ```rust
    let mut v = Vec::new();
    v.push(1);
    v.push(2);
    ```
- reading elements of Vectors: two ways, which to use depends on whether you want to programming crash or not when index is out of boundary.
    ```rust
    // use index
    let v = vec![1, 2, 3];
    println!("{}", v[1]);  // print number 2
    println!("{}", v[99]); // panic, out of boundary
    
    // use get to return an option
    println!("{:?}", v.get(1));  // print Some(2)
    println!("{:?}", v.get(9));  // print None
    ```

**Iterating over the values in a vector**

- use each value in iteration:
    ```rust
    let v = vec![100, 32, 57];
    for n_ref in &v {                    // use reference to avoid moving
        // n_ref has type &i32
        let n_plus_one: i32 = *n_ref + 1; // dereference to access value
        println!("{n_plus_one}");
    }
    ```
- modify each value in iteration:
    ```rust
    let mut v = vec![100, 32, 57];
    for n_ref in &mut v {
        // n_ref has type &mut i32
        *n_ref += 50;
    }
    ```
- non-copyable elements cannot be moved out of a vector by indexing:
    ```rust
    let v = vec![String::from("Hello ")];
    let mut s = v[0];    // cannot move String out of the vector by indexing to
    s.push_str("world"); // avoid the String being owned by both v and s.
    println!("{s}");
    ```

**Using a enum to store multiple types**

- example:
    ```rust
    enum SpreadsheetCell {
        Int(i32),
        Float(f64),
        Text(String),
    }
    
    // this vector has all element of type SpreadsheetCell, but different variants
    let row = vec![
        SpreadsheetCell::Int(3),
        SpreadsheetCell::Text(String::from("blue")),
        SpreadsheetCell::Float(10.12),
    ];
    ```
    

**Quiz**

- quiz 2:
    ```rust
    fn main() {
      let mut v: Vec<i32> = vec![1, 2, 3];
      let mut v2: Vec<&mut i32> = Vec::new();
      for i in &mut v {
        v2.push(i);  // as v2 is Vec<&mut i32>, i has to be &mut i32,
      }
      *v2[0] = 5;   // v2[0] point to v1[0] which point to a value. 
                    // the assignment changes the value but v2[0] and v1[0]
                    // have the same value
      let a = *v2[0];
      let b = v[0];
      println!("{a} {b}");  // print 5 5
    }
    ```
    
### 8.2 storing UTF-8 encoded text with Strings

**string basics**

- creating a new string: 3 methods:
    ```rust
    // 1. new empty string
    let s1 = String::new();
    
    // 2. with initial value
    let s2 = String::from("any value");
    
    // 3. convert from a &str
    let s3 = "any value".to_string();
    ```
    
- updating a string:
    ```rust
    // 1. appending with push_str
    let mut s1 = String::from("Hello ");
    s1.push_str("World!");  // update s1, no return. s1 has to be mutable
    let s3 = " I come";
    s1.push_str(s3);  // s3 is &str and is not moved
    println!("s3 is not deleted. It is still {s3}");  // no problem
    
    // 2. push to add a single character
    let mut s4 = "lo".to_string();
    s4.push('l');  // s4 update to "lol"
    
    // 3. concatenate with + operator
    let s5 = String::from("aaa");
    let s6 = String::from("bbb");
    let s7 = s5 + &s6;  // s5 moved to s7, in the form String + &str + &str + ...
    let s8 = String::from("aaa") + "bbb" + "ccc"; // get aaabbbccc
    
    // 4. concatenate with format! macro, which does not take ownership
    let c1 = "aaa".to_string();
    let c2 = "bbb".to_string();
    let c3 = "ccc".to_string();
    let c = format!("{c1}-{c2}-{c3}");  // get aaa-bbb-ccc and c1, ... still live
    ```
    

**Indexing into strings**: String does not support indexing with s[2]

- internal representation of String: in UTF-8 encoding, a character may be represented by multiple bytes and return by indexing of bytes may not make sense.
    ```rust
    // 1 byte a ascii character
    let hello = String::from("Hola");
    
    // 2 bytes a character
    let hello = String::from("Здравствуйте");
    
    // 4 bytes a character
    let hello = String::from("你好");
    ```

- bytes and scalar values and grapheme clusters: there are multiple ways to represent a String. Hard to indexing.

**Slicing Strings**: it has the same issue as indexing with character boundaries. Use with caution. The good thing is that the compiler can catch the issue.

- examples:
    ```rust
    // safe with ascii  characters
    let s = "abcdefg";
    let s1 = &s[0..3];  // s1 is "abc"
    
    // be careful when a character has more than one byte
    let c = "Здравствуйте";
    let c1 = &c[0..4];  // c1 is the first two chacaters "Зд"
    let c2 = &c[0..5];  // panic, cut into the third character,
    ```
    

**Methods for iterating over strings** 

- using `chars()` method:
    ```rust
    for c in "Здabcравствуйте".chars() {
        println!("c");  // print each character З, д, a, b, ...
    }
    ```
    
- using `bytes()` method: may never need it in my life
    ```rust
    for c in "Здabcравствуйте".byte() {
        println!("c");  // print UTF8 code 208, 151, 208, 180, ...
    }
    ```
    
### 8.3 Storing keys with associated values in hash map

**hash map basics**

- creating a hash map: all keys must be of the same type and all values of the same type.
    ```rust
    // HashMap is not included in prelude
    use std::collections::HashMap;
    
    let mut scores = HashMap::new();
    
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);
    dbg!(&scores);  // use reference, otherwise moved by dbg!
    ```

- accessing values in hash map with `get()` method, which returns an `Option<&V>`.
    ```rust
    // 1. access a single value
    let team_name = String::from("Blue");
    let score = scores.get(&team_name).copied().unwrap_or(0)
        // get() returns a Option<&i32>, not good for unwrap_or(0) as 0 is i32
        // use copied() to convert it to Option<i32>
        
    // 2. iterate over a hash map
    for (k, v) in &scores {
        println!("{k}: {v});
    }
    ```
    

**hash maps and ownership**

- copy and move:
    ```rust
    let k = String::from("Blue");
    let v = 10;
    let mut scores = HashMap::new();
    scores.insert(k, v);  // String k is moved to scores, i32 v is copied to scores
    ```
    

**updating a hash map**

- overwriting a value of an existing key use `.insert()`
    ```rust
    use std::collections::HashMap;
    
    let mut scores = HashMap::new();
    
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Blue"), 25); // replace line above
    
    println!("{:?}", scores);
    ```
    
- adding a key-value pair only if the key does not exist with `.entry().or_insert()`
    ```rust
    use std::collections::HashMap;
    
    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);
    
    scores.entry(String::from("Yellow")).or_insert(50);  // new, added
    scores.entry(String::from("Blue")).or_insert(50);    // existing, no change
    
    println!("{:?}", scores);
    ```
    
- updating a value based on the old value
    ```rust
    use std::collections::HashMap;
    
    let text = "hello world wonderful world";
    
    let mut map = HashMap::new();
    
    for word in text.split_whitespace() {
        // or_insert() returns a mutable reference of the value of word to
        // count, so that the value can be updated by dereferecing.
        // the mutable reference goes out of scope before next round
        let count = map.entry(word).or_insert(0);
        *count += 1;
    }
    
    println!("{:?}", map);
    ```
    



## 9. Error Handling

### 9.1 Unrecoverable errors with panic!

**using a `panic!` backtrace**

- two ways to cause panic:
    - error in code
        ```rust
        fn main() {
            let v = vec![1, 2, 3];
            v[99];  // out of boundary error
        }
        ```
    - use `panic!()` to stop run
        ```rust
        fn main() {
            panic!();
        }
        ```

- display backtrace at panic: i
    - `$ RUST_BACKTRACE=1 cargo run` to show main backtrace
    - `$ RUST_BACKTRACE=full cargo run` to show all backtrace


### 9.2 recoverable errors with Result

**Result enum**

- definition:
    ```rust
    enum Result<T, E> {
        Ok(T),  // contains the success value, small letter k
        Err(E), // contains the error value
    }
    ```
    
- examples
    ```rust
    let good_result: Result<i32, i32> = Ok(10);
    let bad_result: Result<i32, i32> = Err(10);
    ```
    
- `std::fs::File::open()` returns a Result type
    ```rust
    use std::fs::File;
    
    fn main() {
        let greeting_file_result = File::open("hello.txt");
    
        let greeting_file = match greeting_file_result {
            Ok(file) => file,
            Err(error) => panic!("Problem opening the file: {:?}", error),
        };
    }
    ```
    
- examples of `File.open()` failures:
    - NotFound when file does not exist: 
      ```
      [src/main.rs:7] &greeting_file_result = Err(
            Os {
                code: 2,
                kind: NotFound,
                message: "No such file or directory",
            },
        )
      ```
    - PermissionDenied, for example, read a write-only file
        ```
        [src/main.rs:7] &greeting_file_result = Err(
            Os {
                code: 13,
                kind: PermissionDenied,
                message: "Permission denied",
            },
        )
        ```
    

**matching on different errors**

- use `error.kind()` to extract specific result of `Result::Err(error)` variant
    ```rust
    use std::fs::File;
    use std::io::ErrorKind;
    
    fn main() {
        let greeting_file_result = File::open("hello.txt");
    
        let greeting_file = match greeting_file_result {
            Ok(file) => file,
            // .kind() method to get ErrorKind of error in Err(error)
            Err(error) => match error.kind() {
                ErrorKind::NotFound => match File::create("hello.txt") {
                    Ok(fc) => fc,
                    Err(e) => panic!("Problem creating the file: {:?}", e),
                },
                other_error => {
                    panic!("Problem opening the file: {:?}", other_error);
                }
            },
        };
    }
    ```
    
- enum ErroKind has 40 variants and is growing
    ```rust
    pub enum ErrorKind {
        NotFound,
        PermissionDenied,
        ConnectionRefused,
        ...,
        OutOfMemory,
        Other,
    }
    ```
    

**shortcuts for panic on Error: unwrap and expect**

- why: less verbose than `match`. Most programmer use them instead of `match`.

- `.unwrap()` of a `Restult` enum returns its content if successful or panic if failure
    ```rust
    use std::fs::File;
    
    fn main() {
        let greeting_file = File::open("hello.txt").unwrap();
    }
    ```
    
- `.expect()` method does the same but gives a customized  error message
    ```rust
    use std::fs::File;
    
    fn main() {
        let greeting_file = File::open("hello.txt")
            .expect("hello.txt should be included in this project");
    }
    ```
    

**propagating errors** to function calling something that might fail

- normal example:
    ```rust
    use std::fs::File;
    use std::io::{self, Read};
    
    // use Result enum to wrap different type for return
    fn read_username_from_file() -> Result<String, io::Error> {
        let username_file_result = File::open("hello.txt");
    
        let mut username_file = match username_file_result {
            Ok(file) => file,
            Err(e) => return Err(e),
        };
    
        let mut username = String::new();
    
        match username_file.read_to_string(&mut username) {
            Ok(_) => Ok(username),
            Err(e) => Err(e),
        }
    }
    ```

- short version using `?` opearator. The code bedlow equals to those above
    ```rust
    use std::fs::File;
    use std::io::{self, Read};
    
    fn read_username_from_file() -> Result<String, io::Error> {
        let mut username = String::new();
    
        // ? means that move on if successful, Err(e) if failed
        // read_username_from_file must return Result or Option or
        // other types implementing FromResidual
        File::open("hello.txt")?.read_to_string(&mut username)?;
    
        Ok(username)
    }
    ```
    
- use `?` in functions returning Option
    ```rust
    fn last_char_of_first_line(text: &str) -> Option<char> {
        text.lines().next()?.chars().last()
    }
    ```

- return of `main()` function: can return a Result type
    ```rust
    use std::error::Error;
    use std::fs::File;
    
    // ignore success type by unit (), return any error with Box<dyn Error>
    fn main() -> Result<(), Box<dyn Error>> {
        let greeting_file = File::open("hello.txt")?;
    
        Ok(())   // a way to use unit ()
    }
    ```
    
### 9.3 to panic! or not to panic!
Revisit when having more experiences.


## 10 Generic types, traits, and lifetimes

**generic types vs concrete types**

- generics are abstract stand-ins for concrete types or other properties. A generic type is a placeholder for multiple concrete types.

- concrete types are types like i32, String, or other specific types.

### 10.1 generic data types

**define generic data types**

- in struct definitions:
    ```rust
    // Point allow differernt types in field but all fields have the same type
    struct Point<T> {
        x: T,
        y: T,
    }
    
    fn main() {
        let integer = Point { x: 5, y: 10 };  // T is i32
        let float = Point { x: 1.0, y: 4.0 }; // T is f64
    }
    
    // each field can have their own type
    struct Point<T, U> {
        x: T,
        y: U,
    }
    
    fn main() {
        let both_integer = Point { x: 5, y: 10 };
        let both_float = Point { x: 1.0, y: 4.0 };
        let integer_and_float = Point { x: 5, y: 4.0 };
    }
    ```
    
- in enum definitions. We already have the examples in Option and Result enums
    ```rust
    enum Option<T> {
        Some(T),
        None,
    }
    
    enum Result<T, E> {
        Ok(T),
        Err(E),
    }
    ```

- in method definitions
    - simple case
        ```rust
        struct Point<T> {
            x: T,
            y: T,
        }
        
        // use impl<T> to define a generic method
        impl<T> Point<T> {
            fn get_x(&self) -> &T {
                &self.x   // it is &(self.x), not (&self).x
            }
        }
        
        // we can also define a method for a specific type of generic struct
        impl Point<f32> {
            fn distance_from_origin(&self) -> f32 {
                (self.x.powi(2) + self.y.powi(2)).sqrt()
            }
        }
        
        fn main() {
            let p = Point { x: 5, y: 10 };
        
            println!("p.x = {}", p.get_x());
            println!("distance to origin is {}", p.distance_from_origin());
        }
        ```
    
    - mixed types:
        ```rust
        struct Point<X1, Y1> {
            x: X1,
            y: Y1,
        }
        
        impl<X1, Y1> Point<X1, Y1> {
            // self in method is Point<X1, Y1> above
            // add new signature <X2, Y2> to mixup to make it generic
            fn mixup<X2, Y2>(self, other: Point<X2, Y2>) -> Point<X1, Y2> {
                Point {
                    x: self.x,  // x has type X1
                    y: other.y, // y has type Y2
                }
            }
        }
        
        fn main() {
            let p1 = Point { x: 5, y: 10.4 };
            let p2 = Point { x: "Hello", y: 'c' };
        
            let p3 = p1.mixup(p2);
        
            println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
        }
        ```

### 10.2 traits: define shared behavior

**what is trait**: a trait is a method or functionality shared by multiple types. It has the same method signature or the format to be used when being called. The implemetation, however, is different in different types.

- defining a trait: the definition only provides signature and is very brief as the specific implementation depends on type. A method in a trait can also have simple default body, which is overwritten when re-defined for specific types
    ```rust
    // a trait can have multiple methods, all must be impleted 
    pub trait Summary {
        // take reference to a variable of certain types and return a String.
        // This is all the definition required
        fn summarize(&self) -> String; 
        
        // define another method that mutate the variable of types with this trait
        fn hide_author(&mut self);
    }
    ```
    
- implement a trait on a type. All trait items must be implemented if no default definition. A working example:
    ```rust
    // src/main.rs
    fn main() {
        // example of Tweet
        let mut tweet = Tweet {
            username: String::from("horse_ebooks"),
            content: String::from("of course, as you probably already know, people"),
            reply: false,
            retweet: false,
        };
        tweet.hide_author();
        println!("1 new tweet: {}", tweet.summarize());
    
        // example of NewsArticle
        let mut article = NewsArticle {
            headline: String::from("Breaking New: xxxx"),
            location: String::from("Boston"),
            author: String::from("Tom"),
            content: String::from("It just happened, ..."),
        };
        article.hide_author();
        println!("News: {}", article.summarize());
    }
    
    
    pub trait Summary {
        fn summarize(&self) -> String;
        fn hide_author(&mut self);
    }
    
    pub struct NewsArticle {
        pub headline: String,
        pub location: String,
        pub author: String,
        pub content: String,
    }
    
    impl Summary for NewsArticle {
        // remember to implement all items in trait Summary
        fn summarize(&self) -> String {
            format!("{}, by {} ({})", self.headline, self.author, self.location)
        }
    
        fn hide_author(&mut self) {
            self.author = String::from("anonimous");
        }
    }
    
    pub struct Tweet {
        pub username: String,
        pub content: String,
        pub reply: bool,
        pub retweet: bool,
    }
    
    impl Summary for Tweet {
        fn summarize(&self) -> String {
            format!("{}: {}", self.username, self.content)
        }
    
        fn hide_author(&mut self) {
            self.username = String::from("Anonimous");
        }
    }
    ```

- default implementations: it is like a placeholder for a trait item. It is used to pass compiler before implement for each type. It will be replaced after the item is implemented for a type but the default still works for types without specific implementation.
    ```rust
    pub trait Summary {
        // default to a very simple message
        fn summarize(&self) -> String {
            println!("To be completed ...");
            String::from("(Read more ...)")
        }
    }
    ```

**traits as parameters** of functions

- what is it: trait is used to specify types of a parameter
    ```rust
    // instead of specifying type for item, here the signature means item can be any
    // type that has trait Summary implemented. So far we now have three ways to 
    // specify the type of a parameter in functions:
    // - concrete type such as x: i32
    // - generic type such as x:T
    // - type with trait such as x: &impl Xxx
    pub fn notify(item: &impl Summary) {
        println!("Breaking news! {}", item.summarize());
    }
    ```

- trait bound syntax: trait bound is the long form signature above:
    ```rust
    // the following signature looks like restricted generic function -- T is restricted to those with trait Summary.
    pub fn notify<T: Summary>(item: &T) {
        println!("Breaking news! {}", item.summarize());
    }
    
    // trait bound is more concise when multiple parameters have the same trait
    pub fn notify<T: Summary>(item1: &T, item2: &T) {//pass}
    // better than 
    pub fn notify(item1: &impl Summary, item2: &impl Summary) {//pass}
    ```
    
- specifying multiple trait bounds with `+` syntax
    ```rust
    // either way is ok but the later is preferred
    pub fn notify(item: &(impl Summary + Display)) {
    // or
    pub fn notify<T: Summary + Display>(item: &T) {
    ```
    
- more readable trait bounds with `where` clauses
    ```rust
    // hard to read if a signature has too many traits, foe example
    fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
    
    // it can be rewritten as below. It is easy to read and edit
    fn some_function<T, U>(t: &T, u: &U) -> i32
        where
            T: Display + Clone,  
            U: Clone + Debug,
    {
    ```
    

**returning types that implement traits**

- still the function need to return only a type that implements a given trait
    ```rust
    fn returns_summarizable() -> impl Summary {
        Tweet {
            username: String::from("horse_ebooks"),
            content: String::from(
                "of course, as you probably already know, people",
            ),
            reply: false,
            retweet: false,
        }
    }
    ```

- this is wrong as a function cannot return two different types
    ```rust
    // compiler does not know what type to return, NewArticle or Tweet?
    fn returns_summarizable(switch: bool) -> impl Summary {
        if switch {
            NewsArticle {
                headline: String::from(
                    "Penguins win the Stanley Cup Championship!",
                ),
                location: String::from("Pittsburgh, PA, USA"),
                author: String::from("Iceburgh"),
                content: String::from(
                    "The Pittsburgh Penguins once again are the best \
                     hockey team in the NHL.",
                ),
            }
        } else {
            Tweet {
                username: String::from("horse_ebooks"),
                content: String::from(
                    "of course, as you probably already know, people",
                ),
                reply: false,
                retweet: false,
            }
        }
    }
    ```
    

**using trait bounds to conditionally implement methods** that have given traits. It is a way to avoid compiling errors when defining methods on generic types.

- `PartialOrd` trait to compare
  
    ```rust
    use std::fmt::Display;
    
    struct Pair<T> {
        x: T,
        y: T,
    }
    
    // no restriction on method new()
    impl<T> Pair<T> {
        fn new(x: T, y: T) -> Self {
            Self { x, y }
        }
    }
    
    // cmp_display() only works for type implementing Display and PartialOrd.
    // Otherwise >= or println! will report a cmpliler error
    impl<T: Display + PartialOrd> Pair<T> {
        fn cmp_display(&self) {
            if self.x >= self.y {
                println!("The largest member is x = {}", self.x);
            } else {
                println!("The largest member is y = {}", self.y);
            }
        }
    }
    ```
    
- if a trait is implemented to a type, all methods of the trait are applied to the type. By using trait bound, we can make sure that these methods work.
    ```rust
    // ToString is a trait in standard library. It has method to_sring().
    // This method applies to all types that has Display trait. That is,
    // any type that can be printed can be coverted to string
    impl<T: Display> ToString for T {
        // -- snip--
    }
    
    // inter has Display so can be converted to string
    let s = 3.to_string()   // same as String::from("3")
    ```
    

**Quizz**

- quiz 1: the code below does not compile:
    ```rust
    use std::fmt::Display;
    
    // the return is some type with Display trait, but not neccesarily a String
    // so .push_str method may not work for it.
    fn displayable<T: Display>(t: T) -> impl Display { t }
    fn main() {
      let s = String::from("hello");
      let mut s2 = displayable(s);
      s2.push_str(" world");
      println!("{s2}");
    }
    ```
    
    
### 10.3 Validating references with lifetimes

**lifetimes basics**

- why lifetime: when a function returns a reference type, this return depends on the reference to  one or more parameters that are also references (A function cannot return a reference to a owned or local variable). Lifetimes tell compiler that these parameters lives long enough for the returned reference. In other words, lifetimes prevent dangling references.

- example:
    ```rust
    // the return is a string slice &str that depends on x and y. So x and y
    // must live at lease as long as the return. The lifetime specifier 'a tells
    // compiler that they have the same lifetime.
    fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
        if x.len() > y.len() {
            x
        } else {
            y
        }
    }
    
    // when calling the function and using its return, make sure the parameters
    // live long enough. Here is a failed example:
    fn main() {
        let string1 = String::from("long string is long");
        let result;
        {
            let string2 = String::from("xyz");
            result = longest(string1.as_str(), string2.as_str());
        }
        // string2 is dead when printing result, compiler error
        println!("The longest string is {}", result);
    }
    ```
    

**other topics on lifetime**

- lifetime annotations in struct definition: in case a struct field holds reference, we need to specify lifetime in struct definition, for example:
    ```rust
    // simply give required lifetime specifier `a in definition, which means that
    // the field part has the same lifetime as the variable of type ImportantExcerpt.
    struct ImportantExcerpt<'a> {
        part: &'a str,
    }
    
    fn main() {
        let novel = String::from("Call me Ishmael. Some years ago...");
        let first_sentence = novel.split('.').next().expect("Could not find a '.'");
        let i = ImportantExcerpt {
            // first_sentence is a string slice &str
            part: first_sentence,
        };
    }
    ```

- lifetime elision rules: usually every reference in function signature needs a lifetime specifier. Some common patterns, however, are so common that the Rust team decided to code the pattern into compiler and lift the lifetime specifier. There are three rules the compiler follows if the lifetime specifiers are not provided:
    - rule 1: the compiler assigns a different lifetime specifier for each input reference type. For example, 
        - `fn foo(x: &i32, y: &i32)` is treated like `fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`
    - rule 2: If there is only one input lifetime parameter, that lifetime is assigined to all output lifetime parameters:
        - `fn foo(x: &i32) -> &i32 {` becomes `fn foo<'a>(x: &'a i32) -> &'a i32 {`.
    - rule 3: If `&self` or `&mut self` is one of the input parameters, its lifetime is assigned to all output lifetime parameters.
    
- lifetime annotations in method definitions
    ```rust
    // declair lifetime after impl and ImportantExceerpt
    impl<'a> ImportantExcerpt<'a> {
        // output is not a reference so no lifetime needed
        fn level(&self) -> i32 {
            3
        }
    }
    
    impl<'a> ImportantExcerpt<'a> {
        // rule 3
        fn announce_and_return_part(&self, announcement: &str) -> &str {
            println!("Attention please: {}", announcement);
            self.part
        }
    }
    ```
    
- the static lifetime `'static` denotes a reference that lives for  the entire duration of the program. All string literals have `'static` lifetime.Do not abuse it as the compiler may suggest it.

**generic type parameters, trait bounds, and lifetimes together**

- an example having them all:
    ```rust
    use std::fmt::Display;
    
    // lifetime and generic typ in the same bracket <'a, T>
    fn longest_with_an_announcement<'a, T>(
        x: &'a str,
        y: &'a str,
        ann: T,
    ) -> &'a str
    where
        T: Display,
    {
        println!("Announcement! {}", ann);
        if x.len() > y.len() {
            x
        } else {
            y
        }
    }
    
    fn main() {
        longest_with_an_anouncement("abcd", "xyz", "ann: any type with Display");
    }
    ```
    
## 11. Writing automated tests

### 11.1 How to write tests

**the anatomy of a test function**

- when we make a new library project with `cargo new xxx --lib`, a test module with a test function is automatically generated in file `./src/lib.rs`:
    ```rust
    pub fn add(left: usize, right: usize) -> usize {
        left + right
    }
    
    #[cfg(test)]
    mod tests {
        use super::*;
    
        #[test]     // indicate this is a test function
        fn it_works() {
            let result = add(2, 2);
            assert_eq!(result, 4);
        }
    }
    ```
    
- run test with `$ cargo test`

**checking results with `assert!` and it families**
- `assert!(xxx)` xxx is true or false
- `assert_eq!(x, y)` to check x and y are equal.
- `assert_ne!(x, y)` to check x and y are not equal.
- adding custom message to failure message: arguments specified after required arguments in `assert!` families are passed to `format!()`. Examples
    ```rust
    // when the following assertion fails, all the argument afte y are passed to 
    // format! as format!("{} does not equal to {}", x, y) and printed out in 
    // failure message.
    assert_eq!(x, y, "{} does not equal to {}", x, y)
    ```
    

**checking for panics with should_panic** when panic is expected

- example `src/lib.rs`:
    ```rust
    pub struct Guess {
        value: i32,
    }
    
    impl Guess {
        pub fn new(value: i32) -> Guess {
            if value < 1 || value > 100 {
                panic!("Guess value must be between 1 and 100, got {}.", value);
            }
    
            Guess { value }
        }
    }
    
    #[cfg(test)]
    mod tests {
        use super::*;
    
        #[test]
        #[should_panic]  // indicate panic is expected. test failed if not panic
        fn greater_than_100() {
            Guess::new(200);
        }
    }
    ```
    
- adding custom message to `#[should_panic]` when fail at not panic
    ```rust
    #[should_panic(expected = "whatever message you want to add")]
    ```
    

**using Result<T, E> in tests**, no `assert!` family used and no `should_panic` either.

- example:
    ```rust
    #[cfg(test)]
    mod tests {
        #[test] // this is a fully functioning test function
        fn it_works() -> Result<(), String> {
            if 2 + 2 == 4 {
                Ok(())
            } else {
                Err(String::from("two plus two does not equal four"))
            }
        }
    }
    ```
    
### 11.2 Controlling how test are run

**cargo test options**

- running test in parallel or consecutively: by default, cargo test functions run in parallel. So make sure the test functions are independent from each other. Turn of the parallel run with flag --test-threads=1 if one test function depends one the output of other functions.
    ```sh
    # not a typo, double -- in the command
    $ cargo test -- --test-threds=1
    ```
    
- showing function output: by default, cargo test does not display any `println!` output if the test succeeds. It prints when a test fails.To display the print even when the test succeed, add flag `--show-output`:
    ```sh
    $ cargo test -- --show-output
    ```

**running a subset of tests**

- running a single test function `test_func_1`:
    ```sh
    $ cargo test test_func_1
    ```
    
- filtering to run multiple test functions using keyword. So we can group tests by including keywords in test function names.
    - test example:
        ```rust
        #[cfg(test)]
        mod tests {
            use super::*;
        
            #[test]
            fn add_two_and_two() {
                assert_eq!(4, add_two(2));
            }
        
            #[test]
            fn add_three_and_two() {
                assert_eq!(5, add_two(3));
            }
        
            #[test]
            fn one_hundred() {
                assert_eq!(102, add_two(100));
            }
        }
        ```
        
    - run different tests:
        ```sh
        # all three test
        $ cargo test  
        
        # run add_two_and_two only
        $ cargo test add_two_and_two  
        
        # run tests having "and" in test function names
        $ cargo test and
        
        # run tests having letter "h" in names
        $ cargo test h
        ```
    
- ignoring some tests unless specifically requested by add `#[ignore]` attribute:
    - test code example:
        ```rust
        #[cfg(test)]
        mod tests {
            use super::*;
        
            #[test]
            fn add_two_and_two() {
                assert_eq!(4, add_two(2));
            }
        
            #[test]
            #[ignore]  // ignored in $ cargo test
            fn add_three_and_two() {
                assert_eq!(5, add_two(3));
            }
        
            #[test]
            fn one_hundred() {
                assert_eq!(102, add_two(100));
            }
        }
        ```
    - cargo test:
        ```sh
        # ignore tests with ignore attritube
        $ cargo test
        
        # ONLY run tests with ignore attribute
        $ cargo test -- --ignored
        ```
        
### 11.3 Test organization

**unit tests** 

- unit tests are tests wihtin the package, small and focused.

- the tests module and configuration attribute `#[cfg(test)]`: 
    - `#[cfg(test)]` tells compiler that this module only runs in `$ cargo test` and is ignored in `$ cargo build`.
    - This module stays in the file where the functions to be tested are defined.

**integration tests**:  integration tests are tests that uses the package as external users do.

- the tests directory: as many test files under `./tests/` directory as needed. These files only run in `cargo test`.
    ```
    package_xxx
    ├── Cargo.lock
    ├── Cargo.toml
    ├── src
    │   └── lib.rs
    └── tests
        │── integration_test_1.rs
        └── integration_test_2.rs
    ```
    
- an example test file:
    ```rust
    // need to load the package
    use package_xxx;
    
    #[test]
    fn test_func_1() {
        // -- snippet calling package function -- 
    }
    ```
    
- run integration test:
    - `$ cargo test` runs all tests including integration test
    - `$ cargo test --test test_func_1` runs only integration tests in `test_func_1`.

- submodules in integration tests: each integration test file is compiled into a separate crate when runing `cargo test`. We often need helper functions that are shared by these test files. We can place the helper functions in a file, **whose name must be** `mod.rs`, in a subdirectory under `tetsts/`:
    - file structure:
        ```
        package_xxx
        ├── Cargo.lock
        ├── Cargo.toml
        ├── src
        │   └── lib.rs
        └── tests
            ├── subdir_1
            │   └── mod.rs
            ├── integration_test_1.rs
            └── integration_test_2.rs
        ```
    - how to use sub module in `subdir_1`, example:
        ```rust
        use package_xxx
        mod subdir_1  // contains a file must named mod.rs
        
        #[test]
        fn test_func_1() {
            subdir_1::anyfunc();  // calls a function in subdir_1/helper_func.rs
            // other code
        }
        ```
    
- integration tests only works for library crates, not for binary crates.


## 12. An I/O project: building a commmand line program

### 12.1 Accepting command line arguments

- reading command line arguments with `std::env::args()`
    ```rust
    use std::env;
    
    fn main() {
        // env::args returns a String and .collect() returns an iterator
        let args: Vec<String> = env::args().collect();
        dbg!(args);
    }
    ```
    
- run the program:
    - `$ cargo run -- aaa bbb ccc` prints out as follows. The double `--` indicate the parameters after it are for the program, not for `cargo run`.
        ```
        [src/main.rs:5] args = [
            "./target/debug/minigrep",
            "aaa",
            "bbb",
        ]
        ```
    - after comilation, we can run `$ ./path/to/minigrep aaa bbb`, which print out the same results.

### 12.2 Reading a file

- read a file into String with `std::fs::read_to_string(file_path)`:
    ```rust
    use std::fs;
    
    fn main() {
        // poem.txt is at the project root. Whole file read into a Results<String>
        let contents = fs::read_to_string("poem.txt")
            .expect("Should have been able to read the file");
    
        println!("With text:\n{contents}");
    }
    ```
    
    
### 12.3 Refactoring to improve modularity and error handling

**separation of converns for binary projects** following these steps

- split your program into a `main.rs` and a `lib.rs` and move your program logic into `lib.rs`.

- As long as your coammand line parsing logic is small, it can remain in the `main.rs`.

- When the command line parsing logic starts getting complicated, extract it from `main.rs` and move it to `lib.rs`.

- The responsibilities that remain in the main function after this process should be limited to the following:
    - calling the command line parsing logic with the argument values
    - setting up any other configuration
    - calling a `run` function in `lib.rs`
    - handling the error if `run` returns an error.

**grouping configuration values** in a struct

- This minigrep command always have a query and a filepath. To make the code more organized, we can place them in a construct:
    ```rust
    struct Config {
        query: String,
        file_path: String,
    }
    ```
    
- creating a constructor for Config struct:
    ```rust
    impl Config {
        fn new(args: &[String]) -> Config {
            // ok to use clone for small Strings
            let query = args[1].clone();
            let file_path = args[2].clone();
    
            Config { query, file_path }
        }
    }
    ```
    

**fixing the error handling** when query or filepath not provided in terminal

- When one or two arguments are missing, for example, running `$ cargo run -- star`, we will have index out of boundary error as `args[2]` is not provided.

- use `panic` to give a better error message:
    ```rust
    impl Config {
        fn new(args: &[String]) -> Config {
            if arg.len() < 3 {
                panic!("not enough arguments"); 
            }
            let query = args[1].clone();
            let file_path = args[2].clone();
    
            Config { query, file_path }
        }
    }
    ```
    
- `painc!()` is better used for programming problem, not for usage problem, as users do not need to see all the error message such as `thread 'main' panicked at 'not enough arguments', src/main.rs:26:13`. Another reason is that it is better to process failure in `main()` function.

- returning a `Result` instead of calling `panic!`:
    - change function name from `new` to `build`. As a convention, `new` is expected to never fail.
    - the `build` method:
        ```rust
        impl Config {
            fn build(args: &[String]) -> Result<Config, &'static str> {
                if args.len() < 3 {
                    return Err("not enough arguments");
                }
        
                let query = args[1].clone();
                let file_path = args[2].clone();
        
                Ok(Config { query, file_path })
            }
        }
        ```
    
- calling `Config::build()` and hadling errors in `main()`:
    ```rust
    use std::process;
    
    fn main() {
        let args: Vec<String> = env::args().collect();
    
        // use a closure to handle error message and exit program
        let config = Config::build(&args).unwrap_or_else(|err| {
            // users only see the printed error message
            println!("Problem parsing arguments: {err}");
            // an exit code for additional processing
            process::exit(1);
        });
    
        // --snip--
    ```
    

**extracting logic from main**

- code snipets:
    ```rust
    // read terminal arguments
    let args: Vec<String> = std::env::args().collect()
    
    // exit with a status
    std::process::exit(1)
    
    // work with multi-line text
    for line in article {
        if line.contains("xyz") {
            // ...
        }
    }
    ```
    
### 12.4 Developing the library's functionality with test-driven development

**test-driven development** following these steps

- write a test that fails and run it to make sure if fails for the reason you expected.
- write or modify just enough code to make the new test pass.
- refactor the code youjust added or changed and make sure the test continue to pass.
- repeat above process


### 12.5 Working with evnironment variables

- example: how to use environment variable `AAA_BBB`
    - `$ AAA_BBB=1 cargo run -- to poem.txt`: This is how an environment variable is presented in terminal
    - To use it in Rust code:
        ```rust
        use std::env;
        // env::var returns a Result<String, VarError>
        // is_ok returns true or false
        let aaa_bbb = env::var("AAA_BBB").is_ok();
        ```
        
### 12.6 Writing error messages to standard error instead of standard output

- When running a programm, we often send the output to a log file for a record, for example `$ cargo run > log.txt`, and keep the terminal clean. It is, however, desirable to print the error message on stout so we know what's wrong.

- `eprintln!()` prints error message to stout and the error message will not be saved to `log.txt`. Example of using `eprintln!`:
    ```rust
    fn main() {
        let args: Vec<String> = env::args().collect();
    
        let config = Config::build(&args).unwrap_or_else(|err| {
            eprintln!("Problem parsing arguments: {err}");
            process::exit(1);
        });
    
        if let Err(e) = minigrep::run(config) {
            eprintln!("Application error: {e}");
            process::exit(1);
        }
    }
    ```
    
### chapter summary: working code

`src/main.rs`
```rust
use std::env;
use std::process;

use minigrep::Config;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::build(&args).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {err}");
        process::exit(1);
    });

    if let Err(e) = minigrep::run(config) {
        eprintln!("Application error: {e}");
        process::exit(1);
    }
}
```

`src/lib.rs`
```rust
use std::error::Error;
use std::{env, fs};

pub struct Config {
    pub query: String,
    pub file_path: String,
    pub ignore_case: bool,
}

impl Config {
    pub fn build(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let file_path = args[2].clone();

        let ignore_case = env::var("IGNORE_CASE").is_ok();

        Ok(Config {
            query,
            file_path,
            ignore_case,
        })
    }
}

pub fn run(config: Config) -> Result<(), Box<dyn Error>> {
    let contents = fs::read_to_string(config.file_path)?;

    let result = if config.ignore_case {
        search_case_insensitive(&config.query, &contents)
    } else {
        search(&config.query, &contents)
    };

    for line in result {
        println!("{line}");
    }

    Ok(())
}

pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.contains(query) {
            results.push(line);
        }
    }

    results
}

pub fn search_case_insensitive<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let mut results = Vec::new();

    let query = query.to_lowercase(); // to_lowercase returns a String
    for line in contents.lines() {
        if line.to_lowercase().contains(&query) {
            results.push(line);
        }
    }

    results
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn case_sensitive() {
        let query = "duct";
        // \ for continue the line, not a new line
        let contents = "\
Rust:
safe, fast, productive.
Pick three.
Duct tape.";

        assert_eq!(vec!["safe, fast, productive."], search(query, contents));
    }

    #[test]
    fn case_insensitive() {
        let query = "rUsT";
        let contents = "\
Rust:
Safe, fast, productive,
Pick Three.
Trust me.";
        assert_eq!(
            vec!["Rust:", "Trust me."],
            search_case_insensitive(query, contents)
        );
    }
}
```


## 13 Functional Language Features: Iterators and Closures

> Programming in a functional style often includes using functions as values by passing them in arguments, returning them from other functions, assigning them to variables for later execution, and so forth.

### 13.1. closure: anonymous functions that capture variables from their environment

**closure definition**: a closure can take in environment variables without setting them as arguments as normal functions do.

- often appear on the fly in method `.unwrap_or_else()`:
    ```rust
    fn main() {
        let x1 = Some("hello world");
        // Closure arguments are placed inside ||. In this case, there is no
        // arguments so || is empty
        let y1 = x1.unwrap_or_else(|| "empty string");
        let x2: Option<&str> = None;
        let y2 = x2.unwrap_or_else(|| "empty string");
        println!("{y1}, {y2}");  // hello world, empty string
    }
    ```
    
- can be assigned to a varaible:
    ```rust
    // annotation is optional if the compiler can infer the types
    fn main() {
        let x = 1;
        // the type of parameters and result are optional
        // closure can take vaviable x from environment
        let f = |y| x + y;
        let res = f(2);
        println!("{res}");  // 3
    }
    
    
    // if you want annotation, here is how
    fn main() {
        let x: u8 = 1;
        // format to specify types, curly bracket is required
        let f = |y: u8| -> u8 { x + y };
        let res = f(2);
        println!("{res}");  // 3
    }
    ```
    

**reference and ownership of environment variables in closure**

- Base on the how the closure is defined, the closure automatically decide ???? how an environment variables is used in three ways:
    - immutable reference
    - mutable reference
    - taking ownership

- immutable reference:
    ```rust
    fn main() {
        let x = "hello world".to_string();
        let f = || println!("{}", x); // immutable borrow of x in closure
        println!("{x}");  // immutable borrow, at this moment another immutable
                          // borrow lives above because of f() below. It is ok to
                          // to have multiple immutable borrows
        f();  // immutable borrow, ok
    }
    ```
    
- mutable reference:
    ```rust
    // ok
    fn main() {
        // need to add mut before both x and f
        let mut x = "hello world".to_string();
        let mut f = || {
            x.push_str("!");  // mutable borrow
        };
        f();  // mutable borrow completed after function run within its scope
              // so it does not affect subsequent borrowing
        println!{"{x}");  // immutable borrow only at this moment
    }
    
    // not ok as mutable and immutable borrow live the same time
    fn main() {
        // need to add mut before both x and f
        let mut x = "hello world".to_string();
        let mut f = || {
            x.push_str("!");  // mutable borrow
        };
        println!{"{x}");  // immutable borrow, but the same time we have a mutable
                          // borrow above, which lives because of f() below.
        f();  // mutable borrow
    }
    ```
    
- take ownership with keyword `move`. It is useful when passing the closure to a new thread as the new thread needs to own the data. Threads will be discussed in chpt 16.A brief example:
    ```rust
    use std::thread;
   
    fn main() {
        let list = vec![1, 2, 3];
        println!("{:?}", list);
   
        // this new thread may finish after the main thread finished so if does not
        // own the list, the list will be a dangling reference.
        thread::spawn(move || println!("from thread: {:?}", list))
            .join()
            .unwrap();
    }
   ```
   

**moving capture values out of closures and the Fn traits**

- `FnOnce` trait applies to closures that can be called once. All closures at least implement `FnOnce`. A closure that moves captured values out of it body will only implement `FnOnce`. All ananimous functons are only called once, for example in `unwrap_or_else()`, which is defined as:
    ```rust
    // definition of unwrap_or_else()
    impl<T> Option<T> {
        pub fn unwrap_or_else<F>(self, f:F) -> T
        where
            F: FnOnce() -> T  // so f can be any closure without parameters
        {
            match self {
                Some(x) => x,
                None => f(),
            }
        }
    }
    
    // in the case below, the captured value t is sent out of the closure
    fn main() {
        let s = None;
        let t = "empty".to_string();
        let x = s.unwrap_or_else(|| t);
        println!("{x}");
    }
    ```
    
- `FnMut` applies to closure that don't move captured values out of their body but that might mutate the captured values. Take `sort_by_key` for example:
    ```rust
    // definition of sort_by_key on Vec.
    pub fn sort_by_key<K, F>(&mut self, mut f: F)
    where
        F: FnMut(&T) -> K,
        K: Ord,
    {
        stable_sort(self, |a, b| f(a).lt(&f(b)));
    }
    
    // use case. In this case f = |s| s.width return the width of an element.
    // It is used in another closure |a, b| f(a).lt(&f(b)) to compare two elements.
    #[derive(Debug)]
    struct Rectangle {
        width: u32,
        height: u32,
    }
    
    fn main() {
        let mut list = [
            Rectangle { width: 10, height: 1 },
            Rectangle { width: 3, height: 5 },
            Rectangle { width: 7, height: 12 },
        ];
    
        // s represent an element in list, can be any symbol.
        list.sort_by_key(|s| s.width);  // this closure mutates list
        println!("{:#?}", list);
    }
    ```
    
- `Fn` applies to closures that don't move values out and don't mutate values or captures nothing from their environment.

**closures must name captured lifetimes**

- lifetimes needed when a normal function accepts or returns a closure. Case with error:
    ```rust
    // This function returns a closure that returns a String from a &str.
    // It has a compiler error for missing lifetime.
    fn make_a_cloner(s_ref: &str) -> impl Fn() -> String {
        move || {
            s_ref.to_string()
        }
    }
    
    // If the function passed the compiler, this use case shows the problem.
    fn main() {
        let s_own = String::from("Hello world");
        let cloner = make_a_cloner(&s_own);
        drop(s_own);  // s_own dropped
        cloner(); // Undefined behavior: pointer used after its data is freed.
    }
    ```

- correct case:
    ```rust
    // pay attention to how lifetime is added in three locations
    fn make_a_cloner<'a>(s_ref: &'a str) -> impl Fn() -> String + 'a {
        move || {
            s_ref.to_string()
        }
    }
    
    fn main() {
        let s_own = String::from("Hello world");
        let cloner = make_a_cloner(&s_own);
        drop(s_own);  // It is a simple coding error. Not an undefined behavior.
        cloner();     // Undefined behaviors are unsafe and should be avoided.
    }
    ```
    
- simplify the function with lifetime elision: as there is only one parameters, this function can be defined as:
    ```rust
    fn make_a_cloner(s_ref: &str) -> impl Fn() -> String + '_ {
        move || s_ref.to_string()
    }
    ```
    

**Quiz**

- Determine whether the program will pass the compiler. If it passes, write the expected output of the program if it were executed.
    ```rust
    fn main() {
        let mut s = String::from("Hello");
        // s is not a captured environment varaible in the following closure.
        // It is just a normal function parameter and can be any other symbol
        // like x or t. No reference to any environment variable in the closure.
        let add_suffix = |s: &mut String| s.push_str(" world");
        println!("{s}");  // only immutable refernce to s at this moment
        add_suffix(&mut s);  // only mutable reference to s at this moment
        
        // so the programming compiles. The printout is Hello.
    }
    ```
    
## 13.2 Processing a series of items with iterators

**the `Iterator` trait and the `next` method**

- `Iterator` trait is defined in the standard library. Users need to define own `next` method when assign this trait to a struct.
    ```rust
    pub trait Iterator {
        type Item;  // associated type for the method, more in chpt 19
        fn next(&mut self) -> Option<Self::Item>; // require definition by users 
        // -- other methods with default implementation --
    }
    ```
    
- Example - different ways to convert a vector into an iterator 
    ```rust
    #[test]
    fn iterator_demonstration() {
        // use .iter() method
        let v1 = vec![1, 2, 3];  // Vector has IntoIterator trait
        let mut v1_iter = v1.iter();  // v1_iter has Iterator trait
        assert_eq!(v1_iter.next(), Some(&1)); // v1_iter reference to the values
        assert_eq!(v1_iter.next(), Some(&2));
        assert_eq!(v1_iter.next(), Some(&3));
        assert_eq!(v1_iter.next(), None);
    }
    
    #[test]
    fn into_iterator_demonstration() {
        // use into_iter() method
        let v2 = vec![1, 2, 3];
        let mut v2_iter = v2.into_iter(); // into_iter owns the value
        assert_eq!(v2_iter.next(), Some(1));
        assert_eq!(v2_iter.next(), Some(2));
        assert_eq!(v2_iter.next(), Some(3));
        assert_eq!(v2_iter.next(), None);
    }
    ```
    

**iterator methods provided by the standard library**

- methods that consume the iteratori are called **consuming adaptors**: these methods use `next` in their definition and thus consume the iterator:
    ```rust
    #[test]
    fn iterator_cosumers() {
        let v1 = vec![1, 2, 3];
        let it = v1.iter();
        let total: i32 = it.sum();  // explicit type required for sum
        assert_eq!(total, 6);
    
        assert_eq!(it.max(), Some(&3));  // max returns an option, different from sum.
    }
    ```
    
- methods that produce other iterators are called **iterator adaptors**: 
    ```rust
    #[test]
    fn iterator_map() {
        let v1 = vec![1, 2, 3];
        let it = v1.iter();
        let v2: vec<_> = it.map(|x| x + 1).collect(); // type needed
    
        assert_eq!(v2, vec![2, 3, 4]);
    }
    
    #[test]
    fn iterator_filter() {
        let v1 = vec![1, 2, 3];
        let it = v1.iter();
    
        // very complicated reference and ownership, dig in later
        let v2: Vec<_> = it.filter(|&&x| x > 1).cloned().collect();
        assert_eq!(v2, vec![2, 3]);
    }
    ```
