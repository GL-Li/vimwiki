# Rust By Example 
https://doc.rust-lang.org/rust-by-example/index.html

## 1. Hello World

### 1.1 Comments
- `//`: line comment
- `/* afdsaga sdafsaf */`: block comments
- `///`: doc comments
- `//!`: doc comments


### 1.2 Formatted print
Module `std::fmt` contains utilities for formatting and printing `String`s.

#### 1.2.1. Debug

- All `std` library types are automatically printable with `{:?}`.
- Other types can derive the `fmt::Debug` implementation with `#[derive(Debug)]`
- Pretty print with `println!("{:#?}", aaa)` for debug.

#### 1.2.2. Display

- `fmt::Display`, if implemented for a type, use `{}` to print a type. It is clean and compact than `fmt::Debug`.
    ```rust
    use std::fmt;
    
    struct TestDisplay {
        x: i32,
        y: String,
    }
    
    // follow the signature and the order in write!().
    impl fmt::Display for TestDisplay {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            // write! as a return statement, returns a fmt::Result
            write!(f, "x: {}\ny: {}", self.x, self.y)
        }
    }
    
    fn main() {
        let ttt = TestDisplay {
            x: 99,
            y: "abc".to_string(),
        };
        println!("{ttt}");
    }
    ```

- a more complicated implementation of `Display` using `write!()`
    ```rust
    use std::fmt;
    
    struct List(Vec<i32>);
    
    impl fmt::Display for List {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            // each write! write formatted  string into f, the formatter
            
            // as write! return a fmt::Result, use ? to handle error
            write!(f, "[")?;
    
            let vec = &self.0;
            for (i, v) in vec.iter().enumerate() {
                if i != 0 {
                    write!(f, ", ")?;
                }
                write!(f, "{}", v)?;
            }
            
            // final return, no semi-colon
            write!(f, "]")
        }
    }
    
    fn main() {
        let ttt = List(vec![1, 2, 3]);
        println!("{ttt}");
    }
    ```

### 1.2.3 Formatting
An even more complicated example for `Display`.
```rust
use std::fmt::{self, Formatter, Display};

struct City {
    name: &'static str,
    // Latitude
    lat: f32,
    // Longitude
    lon: f32,
}

impl Display for City {
    fn fmt(&self, f: &mut Formatter) -> fmt::Result {
        let lat_c = if self.lat >= 0.0 { 'N' } else { 'S' };
        let lon_c = if self.lon >= 0.0 { 'E' } else { 'W' };

        write!(f, "{}: {:.3}°{} {:.3}°{}",
               self.name, self.lat.abs(), lat_c, self.lon.abs(), lon_c)
    }
}

#[derive(Debug)]
struct Color {
    red: u8,
    green: u8,
    blue: u8,
}

fn main() {
    // for loop using array
    for city in [
        City { name: "Dublin", lat: 53.347778, lon: -6.259722 },
        City { name: "Oslo", lat: 59.95, lon: 10.75 },
        City { name: "Vancouver", lat: 49.25, lon: -123.1 },
    ] {
        println!("{}", city);
    }
    for color in [
        Color { red: 128, green: 255, blue: 90 },
        Color { red: 0, green: 3, blue: 254 },
        Color { red: 0, green: 0, blue: 0 },
    ] {
        // Switch this to use {} once you've added an implementation
        // for fmt::Display.
        println!("{:?}", color);
    }
}
```


## 2. Primitives

Rust primitives include

- scalar types:
    - signed integers
    - unsigned integers
    - floating point
    - char in single quote like `'a'`, `'b'`
    - bool, `true` or `false`
    - unit type `()`
- compound types:
    - arrays like `[1, 2, 3]`
    - typles like `(1, true)`

### 2.1 Literals and operators

- Literals: 1, 1.2, 'x', "abc", true, (), 1e6, 7.8e-4
- operators:
    - `99u32 + 78`
    - `99i32 - 100`
    - `true && false`, using double `&&` and `||`. Single for bitwise AND and OR
    - `true || false`
    - `!true`
    - complete list: https://doc.rust-lang.org/book/appendix-02-operators.html

### 2.2 Tuples
Tuples can have mixed types as elements.

- practice 1: reverse the order of tuple element
    ```rust
    fn reverse(pair: (i32, bool)) -> (bool, i32) {
        let (aaa, bbb) = pair;
        (bbb, aaa)
    }
    ```

- practice 2: transpose a matrix
    ```rust
    use std::fmt;
    #[derive(Debug)]
    struct Matrix(f32, f32, f32, f32);

    impl Matrix {
        fn transpose(&self) -> Matrix {
            Matrix(self.0, self.2, self.1, self.3)
        }
    }

    impl fmt::Display for Matrix {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            write!(f, "[")?;
            write!(f, "{} {}\n", self.0, self.1)?;
            write!(f, " {} {}", self.2, self.3)?;
            write!(f, "]")
        }
    }

    fn main() {
        let mtx = Matrix(1.1, 1.2, 2.1, 2.2);
        println!(
            "mtx is\n{}\nand it's transpose is\n{}",
            mtx,
            mtx.transpose()
        );
    }
    ```

### 2.3 Arrays and slices
An **arrays** is a collection of objects of the same time, whose length is known at compiling time. A **slice** is a reference to a section of an array, whose length is unknown at compiling.

- create an array:
    - `let x: [i32; 5] = [1, 2, 3, 4, 5];`
    - `let y: [i32; 500] = [99; 500];`. The `[T; N]` type T must implements `Copy` trait.
    - `let aaa: [i32; 0] = [];` empty array, must specify type.
- array methods:
    - length: `x.len()`
    - get an element with index i: `x.get(i)`, which returns a `Result`. Good to handle out of boundary error. 
- slice:
    - `&x` all array as a slice
    - `&x[1..4]` elements of index  1, 2, and 3
    - `&[] as &[i32]` empty slice, must specify type or type can be inferred.


## 3. Custom types
struct, enum, constant and static.

### 3.1 Structures

- tuple structs: `Struct Pair(i32, i32)`. 
    - To assign elements of `p = Pair(11, 99)` to new variables `x` and `y`:
        - `x = p.0`, `y = p.1` or
        - destructure: `Pair(x, y) = p`
    - Example
        ```rust
        #[derive(Debug)]
        struct Pair(i32, String);
        fn main() {
            let p = Pair(3, "apple".to_string());
            // access element
            println!("{}, {}", p.0, p.1);
            // destructure the tuple struct
            let Pair(x, y) = p;
            println!("{}, {}", x, y);
        }
        ```
  
- unit structs, used in generics??????????????:
    ```rust
    #[derive(Debug)]
    struct UnitXyz;  // unit struct has no field 
    fn main() {
        let _unit = UnitXyz;
        println!("{:?}", _unit);
    }
    ```

- classic C structs
    ```rust
    #[derive(Debug)]
    struct Point {
        x: i32,
        y: i32,
    }

    #[derive(Debug)]
    struct Rectangle {
        bottom_left: Point,
        top_right: Point,
    }

    fn main() {
        let p1 = Point { x: 0, y: 0 };
        let p2 = Point { x: 1, y: 2 };
        let rec = Rectangle {
            bottom_left: p1,
            top_right: p2,
        };
        println!("{:?}", rec);

        // update the top_right point
        let rec2 = Rectangle {
            top_right: Point { x: 3, y: 3 },
            ..rec // the rest of rec is moved or copied here
        };
        println!("{:?}", rec2);
        
        // access elements
        println!("{:?} and {:?}", rec.top_right, rec.top_right.x);
        
        // error, rec.bottom_left moved into rec2 above. To solve the problem,
        // give Copy trait to Point with #[derive(Debug, Clone, Copy)]
        println!("{:?}", rec.bottom_left);

        // destructure
        let Rectangle {
            bottom_left: aaa,
            top_right: bbb,
        } = rec2;
        println!("{:?} and {:?}", aaa, bbb);
    }
    ```

### 3.2 Enums

**Type aliases**: rename an enum or struct to be shorter or more meaningful. `Self` is the most common alias used in `impl` block.
    
```rust
#[derive(Debug)]
enum Aaaaaaaaaaaaaa {
    Add,
    Substract,
}
// alias
type Aaa = Aaaaaaaaaaaaaa;

impl Aaa {
    fn run(&self, x: i32, y: i32) -> i32 {
        match self {
            Self::Add => x + y,  // Self is an alias to Aaa or Aaaaaaaaaaaaa
            Self::Substract => x - y,
        }
    }
}

#[derive(Debug)]
struct Bbbbbbbbbbbbbbb {
    x: i32,
    y: i32,
}
// alias
type Bbb = Bbbbbbbbbbbbbbb;

fn main() {
    let a = Aaa::Add;
    println!("{:?}", a);
    
    // ways to call functions of enum
    let x = a.run(2, 7);
    let y = Aaa::Substract.run(2, 7);
    println!("{x} and {y}");

    let b = Bbb {x: 11, y: 99};
    dbg!(b);  // dbg! moves b
}
```

#### 3.2.1 `use`

The `use` declaration is used to skip mnanual scoping with `::`

```rust
#![allow(dead_code)]

enum Status {
    Rich,
    Poor,
}

enum Work {
    Civilian,
    Soldier,
}

fn main() {
    // explicitly "use" enum variants
    use crate::Status::{Poor, Rich};
    use crate::Work::*;

    let status = Poor; // instead of Status::Poor
    let work = Civilian; // instead of Work::Civilian
    match status {
        Rich => println!("Rich means a lot of money"),
        Poor => println!("Poor has little money"),
    }

    match work {
        Civilian => println!("I am a civilian."),
        Soldier => println!("I am a soldier."),
    }
}
```

#### 3.2.2 C-like

```rust
#![allow(dead_code)]

// enum with implicit discriminator starting at 0
enum Aaa {
    Ax,
    Ay,
    Az,
}

// enum with explicity discriminator
enum Bbb {
    Bx = 111,
    By = 0x00ff00,  // can be converted to integer in main()
    Bz = 333,
}

fn main() {
    // convert enum variants to index
    println!("Ax's index is {}, Ay is {}", Aaa::Ax as i32, Aaa::Ay as i32);
    
    // convert enum initial value to i32
    println!("Bx's value is {}, By is {}", Bbb::Bx as i32, Bbb::By as i32);
}
```

### 3.3 Constant

- `const`: An unchangeable value.
- `static`: A possibly mutable variable with `static` lifetime.
- examples:
```rust
#![allow(dead_code)]

// define constant in global scope, accessible to any scope
static AAA: &str = "Rust";
const BBB: i32 = 10;

fn is_big(n: i32) -> bool {
    n > BBB
}

fn main() {
    let n = 16;
    println!("AAA is {AAA}");
    println!("{} is {}", n, if is_big(n) {"big"} else {"small}"});
}
```


## 4. Variable bindings
section 4.1-4.3 are included in Rust Book

### 4.4 Freezing

When data is bound by the same name immutably, it also freezes. Frozen data can't be modified until the immutable binding goes out of scope.

```rust
fn main() {
    let mut aaa = 99;

    {
        let aaa = aaa;     // make aaa immutable in this scope
        println!("{aaa}");
        // aaa = 11;       // error, aaa is immutable in this scope
    }
    aaa = 11;
    println!("{aaa}");
}
```


## 5 Types
Most are in the Rust Book.

### 5.1 Casting

Using keyword `as` to convert types

```rust
fn main() {
    let decimal = 33333.14;
    let integer: u8 = decimal as u8;
    println!("{integer}"); // 255, u8 overflow from 3333.14

    let character: char = decimal as u8 as char; // f32 cannot be cast to char
    println!("{character}");
}
```

### 5.4 Aliasing

Use keyword `type` to rename a type.

```rust
fn main() {
    type Aaa = u64;
    type U64 = u64;

    let a: Aaa = 2002;
    let b: U64 = 99999;

    println!("{a} and {b}")
}
```




## 6. Conversion

### 6.1. From and Into

- The `From` trait allows for a type to define how to create itself from another type, hence providing a very simple mechanism for converting between several types.
    - example: `String::from("abc")` to create a `String` from a `str`.
    - implement trait `From` for a custom type:
        ```rust
        // this example create a type Number from a type i32
        use std::convert::From;
        
        #[derive(Debug)]
        struct Number {
            value: i32,
        }
        
        impl From<i32> for Number {
            fn from(item: i32) -> Self {
                Number { value: item }
            }
        }
        
        fn main() {
            let num = Number::from(30);
            println!("My number is {:?}", num);
            
            // into() is working if From is implemented but need
            // to explecitly specify types
            let num_2: Number = 99i32.into();  // i32 not required as type of num_2 defined
            println!("Number from into is {:?}", num_2);
        }
        ```
    
- The `Into` trait is the reciprocal of the `From` trait. 
    - need to know the type into which to convert, as there may be multipe conversion destination.
    - example:
        ```rust
        use std::convert::Into;
        
        #[derive(Debug)]
        struct Number {
            value: i32,
        }
        
        impl Into<Number> for i32 {
            fn into(self) -> Number {
                Number { value: self }
            }
        }
        
        fn main() {
            let int = 5;
            // neet to specify type of destination
            let num: Number = int.into();
            println!("My number is {:?}", num);
        }
        ```
        
        
### 6.2. TryFrom and TryInto

- Used for conversion that may fail. Return option `Result`s
- Example:
    ```rust
    use std::convert::{TryFrom, TryInto};
    
    #[derive(Debug)]
    struct EvenNumber(i32);
    
    impl TryFrom<i32> for EvenNumber {
        type Error = ();
        fn try_from(x: i32) -> Result<Self, Self::Error> {
            if x % 2 == 0 {
                Ok(EvenNumber(x))
            } else {
                Err(())
            }
        }
    }
    
    fn main() {
        // use try_from
        let aaa = EvenNumber::try_from(3);
        let bbb = EvenNumber::try_from(4);
        println!("aaa is {:?} and bbb is {:?}", aaa, bbb);
    
        // as TryFrom is implemented, we can use try_into after
        // specify explicitly types
        let ccc: Result<EvenNumber, ()> = 7i32.try_into();
        let ddd: Result<EvenNumber, ()> = 8i32.try_into();
        println!("ccc is {:?} and ddd is {:?}", ccc, ddd);
    }
    ```

### 6.3. To and from Strings

- **Converting to String**: the prefered method of converting any type to a String is to implement `fmt::Display` trait which automatically provides trait `ToString` and `to_string()` method.

    ```rust
    use std::fmt;
    
    struct Circle {
        radius: i32,
    }
    
    // follow the template, only change write! line
    impl fmt::Display for Circle {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            write!(f, "Circle of radius {}", self.radius)
        }
    }
    
    fn main() {
        let aaa = Circle { radius: 99 };
        println!("{}", aaa.to_string());  # to displayed string
    }
    ```
    
- **Parsing a String**: to convert a string into a number
    ```rust
    fn main() {
        // specify type for aaa
        let aaa: i32 = "5".parse().unwrap();
        // or use turbofish syntax
        let bbb = "10".parse::<i32>().unwrap();
        println!("Sum is {:?}", aaa + bbb);
    } 
    ```
