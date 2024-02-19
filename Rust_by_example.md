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
    
    

## 7: Expressions

The last value of a block statement `{...}` can be assigned to a variable. 

```rust
fn main() {
    let x = 5u32;
    let y = {
        let x = x * x;
        x  // no ; so it is assigned to y
    };  // end with ; because it is part of a let statement
    let z = {
        let x = x.pow(3);  // x^3 is not valid, ^ is reserved for bitwise OR
        x;  // with ;, a unit () is assigned to z
    };

    println!("x is {}, x^2 is {:?}, and x^3 is {:?}", x, y, z);
        // x is 5, x^2 is 25, and x^3 is ()
}
```



## 8. Flow of control

### 8.1 if ... else ...
Be aware that variables assigned in `if () else ()` stays within the scope defined by the branch. The following code does not work
```rust
fn main() {
    let x = 5;
    if x < 0 {
        let y = -x;
    } else if x > 0 {
        let y = x;
    } else {
        let y = 0;
    }
    println!("y is {y}");  // error, y is not available 
}
```

The correct way is:
```rust
fn main() {
    let x = 5;
    let y = if x < 0 {
        -x  // no ; as return, must be the same type at each branch
    } else if x > 0 {
        x
    } else {
        0
    };  // end ; as part of a let state ment.
    println!("y is {y}");
}
```


### 8.2 loop

covered in the rust book.


### 8.3 while

covered in the rust book

### 8.4 for and range

- range:
    - `1..5` is 1, 2, 3, 4
    - `1..=5` is 1, 2, 3, 4, 5

- for and iterators:
    - `iter()` borrows a collection
        ```rust
        fn main() {
            let names = vec!["aaa", "bbb", "ccc"];

            for name in names.iter() {
                match name {
                    &"aaa" => println!("aaa ---"),  // must be &str
                    _ => println!("not aaa ---"),
                }
            }
        }
        dbg!(names); // names still available
        ```
        
    - `into.iter()` moves and consumes the collection
        ```rust
        fn main() {
            let names = vec!["aaa", "bbb", "ccc"];

            for name in names.into_iter() {
                match name {
                    "aaa" => println!("aaa ---"),  // str not &str
                    _ => println!("not aaa ---"),
                }
            }

            // dbg!(names);  // names no available anymore.
        }
        ```
        
    - `inter_mut()` mutable borrowing of the collection
        ```rust
        fn main() {
            let mut names = vec!["aaa", "bbb", "ccc"];

            for name in names.iter_mut() {
                *name = match name {
                    &mut "aaa" => "AAA",
                    _ => "~~~",
                }
            }

            dbg!(names);
        }
        ```

### 8.5 match

`|` and `..` can be used in matching pattern

```rust
fn main() {
    let n = 13;
    match n {
        1 => println!("match single value"),
        2 | 3 | 6 | 10 => println!("match multiple values"),
        15..=20 => println!("match a range"),
        _ => println!("match the rest"),
    }
}
```

#### 8.5.1 Destructuring

- destructuring **tuples**: match by selected elements of a tuple
    ```rust
    fn main() {
        let tpl = (1, -2, 3);
        match tpl {
            (0, y, z) => println!("first is '0', y is {}, and z is {}", y, z),
            (1, ..) => println!("first is '1' and the rest doesn't matter"),
            (.., 4) => println!("last is '4' and the rest doesn't matter"),
            (3, .., 7) => println!("fisrt is '3' and last is '7'. The rest doesn't matter"),  
            (9, 1, _) => println!("firs 9 second 1 and the third doesn't matter"),
            _ => println!("doesn't matter"),
        }
    }
    ```
    
- destructuring **arrays/slices**: all that apply to tuple can be used for arrays, plus `@ ..` syntax for storing other elements
    ```rust
    fn main() {
        let arr = [3, 2, 3, 7];
        match arr {
            [1, 2, other @ ..] => println!("fisrt 1 second 2 and other {:?}", other),
            [3, middle @ .., 7] => println!("fisrt 3 last 7, the middle {:?}", middle),
            _ => println!("doesn't matter"),
        }
    }
```

- destrucruring **enums**: 
    ```rust
    #[allow(dead_code)]

    enum Color {
        Red,
        Blue,
        RGB(u32, u32, u32),
        HSV(u32, u32, u32),
    }
    fn main() {
        let color = Color::RGB(12, 26, 99);
        match color {
            Color::Red => println!("Red"),
            Color::Blue => println!("Blue"),
            Color::RGB(r, g, b) => println!("red {}, green {}, blue{}", r, g, b),
            Color::HSV(h, s, v) => println!("hue {}, saturation {}, value {}", h, s, v),
        }
    }
    ```

- destructuring **pointers/ref**: get the data a reference points to. Similar to dereferencing with `*`.
    ```rust
    fn main() {
        let aaa = &4; // create a reference to 4 with &
        match aaa {
            &bbb => println!("value is {}", bbb),
        }
    }
    ```
    
- destructuring **structs** all the way down to primitive types
    ```rust
    fn main() {
        struct Aaa {
            x: (u32, u32),
            y: u32,
        }

        let aaa = Aaa { x: (1, 2), y: 3 };
        match aaa {
            Aaa { x: (1, b), y: c } => println!("second of x is {b}, y is {c}"),
            _ => (),
        }
    }
    ```

#### 8.5.2 Guards
Guards are conditions added to the match arms.

    ```rust
    fn main() {
        let aaa = 4;
        match aaa {
            x if x > 0 => println!("greater than 0"),
            x if x == 0 => println!("is 0"),
            x if x < 0 => println!("less thant 0"),
            _ => println!("still need this arm even if it never happens"),
        }
    }
    ```


#### Binding
Assign values to names inside match branches.

```rust
fn main() {
    // example 1
    let age = 15;
    match age {
        0 => println!("I am a new born"),
        n @ 1..=12 => println!("I am a child of age {}", n),
        n @ 13..=19 => println!("I am a teenage of age {}", n),
        n => println!("I am too old of age {}", n),
    }

    // example 2
    let x = Some(42);
    match x {
        Some(n @ 42) => println!("The number is {}", n),
        Some(n) => println!("The number is {}", n),
        None => (),
    }
}
```


### if let

Matching `enum`s needs to be exaustive. It is more concise to use `if let` if we are only interested in match one variant of an enum.

```rust
fn main() {
    // good use of if let
    let number = Some(7);
    if let Some(i) = number {
        println!("matched {}", i);
    }

    // rather use match in this case
    let letter: Option<i32> = None;
    if let Some(i) = letter {
        println!("Matched to {}", i);
    } else {
        println!("Match failed");
    }
    
    // good for non-parameterized options
    enum Foo {
        Bar,
    }
    let a = Foo::Bar;
    if let Foo::Bar = a {
        println!("a is Foo::Bar");
    }
}
```


### 8.6. let else
Use `if let` to do something if the pattern match is successful. Use `let ... else ...` to **diverge** (break, return, panic!).

```rust
fn main() {
    let aaa: Option<i32> = None;
    let Some(_i) = aaa else {
        println!("match failed");
        panic!("not match")
    };
}
```

### 8.7. while let
Run the while loop until the option match fails.

```rust
fn main() {
    let mut aaa = Some(0);
    while let Some(i) = aaa {
        if i > 9 {
            println!("i is {i}");
            aaa = None;  // replace break
        } else {
            println!("i is {i}");
            aaa = Some(i + 1);
        }
    }
}
```



## 9. Functions

### 9.1. Associated functions & methods

**Associated functions** are function defined on a type generally. **Methods** are associated functions called on a particular instaance of a type.

```rust
#[derive(Debug)]
struct Point {
    x: f64,
    y: f64,
}

impl Point {
    // associated functions do not have self in arguments
    fn origin() -> Self {
        Point { x: 0.0, y: 0.0 }
    }
    fn new(x: f64, y: f64) -> Self {
        Point { x, y }
    }

    // method of a instance so arguments start with self, &self, or &mut self
    fn move_by(self, delta_x: f64, delta_y: f64) -> Self {
        Point {
            x: self.x + delta_x,
            y: self.y + delta_y,
        }
    }
    fn mute_by(&mut self, delta_x: f64, delta_y: f64) {
        self.x = self.x + delta_x;
        self.y = self.y + delta_y;
    }
}

fn main() {
    // associated functions used with the type like Point::xxx
    let aaa = Point::origin();
    let bbb = Point::new(1.1, 2.2);
    dbg!(aaa);
    dbg!(bbb);

    // methods are used on a instance like ccc.move_by
    let ccc = Point::origin();
    let ddd = ccc.move_by(0.1, 0.1);
    dbg!(ddd);

    let mut eee = Point::origin();
    eee.mute_by(0.1, 0.1);
    dbg!(eee);
}
```


### 9.2 Closures

#### 9.2.1. Capturing

Closures can capture variables in enclosing environment by reference `&T`, by mutatable reference `&mut T`, or by value `T`. 

- Capturing by reference is the default
    ```rust
    fn main() {
        let aaa = 3;
        let cls1 = |i| i + aaa; 
        println!("closure is {}, and aaa is {}", cls1(3), aaa);

        let bbb = "hello".to_string();
        let cls2 = || &bbb; // explicit reference to values in heap
        println!("closure is {:?}, and bbb is {}", cls2(), bbb);
    }
    ```
    
- Capturing by mutable reference
    ```rust
    fn main() {
        // both clousre and variables are mutable in definition
        let mut count = 0;
        let mut increment = || {
            count += 1;
            println!("New count is {}", count);
        };
        increment();

        let mut aaa = "hello".to_string();
        let mut change = || aaa.push_str(" world");
        change();
        println!("aaa is {}", aaa);
    }
    ```
    
- Capturing by taking ownership with `move`
    ```rust
    fn main() {
        let haystack = vec![1, 2, 3];

        // haystack is moved into closure f and f can be used repeatedly
        let f = move |needle| haystack.contains(needle);
        println!("{}", f(&1));
        println!("{}", f(&4));
        // println!("{:?}", haystack);  // haystack not available anymore
    }
    ```
    
#### As input parameters

- type annotation of closure as function input:
    - `Fn()`: captured variables only used by reference.
    - `FnMut()`: capture variables used by mutable reference
    - `FnOnce()`: captured variables used by value (owned)
    - `FnOnce` includes `FnMut` includes `Fn`.

- example
    ```rust
    fn apply<F>(f: F)
    where
        F: FnOnce(),
    {
        f();
    }

    fn apply_to_3<F>(f: F) -> i32
    where
        F: Fn(i32) -> i32, // take a parameter 
    {
        f(3)
    }

    fn main() {
        let greeting = "hello";
        // string slice is &str reference, to_owned make it an owned value
        let mut farewell = "goodby".to_owned();

        let diary = || {
            // use greeting by reference, which requires Fn
            println!("I said {}", greeting);
            // use farewell by mutable reference with push_str, requires FnMut
            farewell.push_str("!!!!");
            println!("Then I screemed {}", farewell);
            // drop the value of farewell, requires FnOnce
            std::mem::drop(farewell);
        };

        apply(diary); // closure diary requires FnOnce()
        // apply(diary); // error, diary can only be called once

        let aaa = 1;
        // aaa by reference only. x is a closure parameter
        let double = |x| 2 * x + aaa;
        let y = apply_to_3(double);
        println!("y is {y}");
    }
    ```
    
#### 9.2.3. Type anonymity
When a closure is used as a function parameter, it must be of one of the generic types: `Fn`, `FnMut`, or `FnOnce`. 

#### 9.2.4. Input functions
Like closure, a function can also be used as a function parameter with the same trait boundary `Fn`, `FnMut` and `FnOnce`.

```rust
// a generic function f bounded by trait Fn as input parameter
fn call_me<F: Fn()>(f: F) {
    f();
}

fn func() {
    println!("I am a function");
}

fn main() {
    let closr = || println!("I am a closure");
    call_me(closr);
    call_me(func);
}
```

#### 9.2.5. As output parameters

Closures can be used as function outputs. As their type is unknown, they are bounded by trait `Fn`, `FnMut` or `FnOnce`. All captured variables must be `move`d to the closure so returned closure can live by itself.

```rust
fn create_fn() -> impl Fn() {
    let text = "I am a Fn".to_string();
    // text used by reference
    move || println!("{}", text)
}

fn create_fnmut() -> impl FnMut() {
    let mut text = "I am a FnMut".to_string();
    text.push_str(" and I can change!");
    move || println!("{}", text)
}

fn create_fnonce() -> impl FnOnce() {
    let text = "I am a FnOnce. I will be moved".to_string();
    let text_moved = text;
    move || println!("{}", text_moved)
}

fn main() {
    let fn_ref = create_fn();
    let mut fn_mut = create_fnmut();  // don't forget mut
    let fn_once = create_fnonce();

    fn_ref();
    fn_mut();
    fn_once();
}
```


#### 9.2.6. Examples in std
**9.6.2.1 Iterator::any**

- definition in standard library: this function takes a closure that returns `bool` as input. The function returns a `bool`.

    ```rust
    pub trait Iterator {
        type Item;

        fn any<F>(&mut self, f: F) -> bool where
            F: FnMut(Self::Item) -> bool,
    }
    ```
    
- application examples
    ```rust
    fn main() {
        let a = [1, 2, 3];
        println!("{}", a.iter().any(|&x| x > 2));   // true
        println!("{}", a.iter().any(|&x| x > 5));   // false
    }
    ```
    
**9.2.6.2. Searching throug iterators**

- definition in standard library. It takes a closure that returns `bool`. The function returns a `Option`, which is `Some(first_element)` if found and `None` if not found.

    ```rust
    pub trait Iterator {
        type Item;

        fn find<P>(&mut self, predicate: P) -> Option<Self::Item> where
            P: FnMut(&Self::Item) -> bool;
    }
    ```
    
- application examples
    ```rust
    fn main() {
        let a = [1, 2, 3];
        // why double reference &&x?
        // - a.iter() is reference as &mut self in find<p>(&mut self, ...)
        // - P: FnMut(&Self::Item) takes reference again.
        println!("{:?}", a.iter().find(|&&x| x > 2)); // Some(3)
        println!("{:?}", a.iter().find(|&&x| x > 5)); // None
    }
    ```
    
### 9.3. Higher order functions
The functional flavor of Rust language. A simple example

```rust
fn is_odd(n: u32) -> bool {
    n % 2 == 1
}

fn main() {
    println!("Find the sum of all the numbers with odd squares under 1000");
    let upper = 1000;

    // Functional approach
    let sum_of_squared_odd_numbers: u32 =
        (0..).map(|n| n * n)                             // All natural numbers squared
             .take_while(|&n_squared| n_squared < upper) // Below upper limit
             .filter(|&n_squared| is_odd(n_squared))     // That are odd
             .sum();                                     // Sum them
    println!("functional style: {}", sum_of_squared_odd_numbers);
}
```

### 9.4. Diverging functions and expressions

Diverging functions never return and therefore there is no type restriction on return type. The return type of diverging functions is specified by `-> !`. It is different from unit `()`, which itself is a type. 

Example of divering functions:

    - `panic!()`
    - `exit()`
    - `unimplemeted!()`, which is a wrap around `panic!()`
    
Examples of diverging expressions

    - `continue`
    - `break`
    - `return`

Applications of diverging functions and expressions: `!` can be coreced into any type so it can be used in `if ... else ...` and `match` branches:

- example 1:
    ```rust
    fn main() {
        let x = -1;
        // panic!() is coreced into i32
        let y = if x > 0 { x + 1 } else { panic!() };
        println!("{y}");
    }
    ```
    
- example 2:
    ```rust
    fn main() {
        let x = vec![1, 2, 3, 4, 5, 6];
        for i in x {
            let y = match i % 2 == 0 {
                true => i,
                false => continue,  // coerced to type i32
            };
            println!("{y}");
        }
    }
    ```
