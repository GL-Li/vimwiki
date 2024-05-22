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
    - arrays like `[1, 2, 3]`, must be the same type
    - typles like `(1, true)`, can be different types

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
An **arrays** is a collection of objects of the same type, whose length is known at compiling time. A **slice** is a reference to a section of an array, whose length is unknown at compiling.

- create an array:
    - `let x: [i32; 5] = [1, 2, 3, 4, 5];`
    - `let y: [i32; 500] = [99; 500];`. The `[T; N]` type T must implements `Copy` trait.
    - `let aaa: [i32; 0] = [];` empty array, must specify type.
- array methods:
    - length: `x.len()`
    - get an element with index i: `x.get(i)`, which returns a `Result`. Good to handle out of boundary error. 
- slice must be a reference to an array with `&`:
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
            // to explecitly specify type for num_2 as 99i32 may be convert to 
            // more than one types.
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
    
    
## 10. Modules

### 10.1. Visibility
By default, a function is only available within the module defined the functions. Using `pub` to declair its visibility to other modules.

```rust
// mod_a is at the same level as fn main() and is visible to main(),
// no pub declaration needed
mod mod_a {
    pub fn print_a() {
        println!("a");
    }

    // mod_a1 is within the scope of mod_a, not visible to main,
    // need pub declaration
    pub mod mod_a1 {
        pub fn print_a1() {
            println!("a1");
        }

        pub mod mod_a2 {
            pub fn print_a2() {
                println!("a2");
            }
        }
    }
}

fn main() {
    // every modules and function in the chain of :: must be visible to main()
    mod_a::print_a();
    mod_a::mod_a1::print_a1();
    mod_a::mod_a1::mod_a2::print_a2();
}
```

### 10.2. Struct visibility
The fields of a Struct, if used outside of the module where it is defined, need to declaired `pub`.

```rust
mod mod_a {
    #[derive(Debug)]
    pub struct Point {
        pub x: i32,
        pub y: i32,
    }

    // no pub before impl
    impl Point {
        pub fn get_x(&self) -> i32 {
            self.x
        }
    }
}

fn main() {
    let x = mod_a::Point { x: 1, y: 2 };
    dbg!(x);

    let y = mod_a::Point { x: 11, y: 99 };
    // get_x is restricted to the scope of y, so no path to get_x needed.
    println!("x is {}", y.get_x());
}
```


### 10.3. The `use` declairation

To avoid full path to a function. The declairation only accessible within the scope where it is declaired.

### 10.4. super and self

- `self::func()`: call function `func` within current module scope.
- `super::func()`: call function in parent scope.

### 10.5. File hierarchy

A sample project structure
```
$ tree .
.
├── my
│   ├── inaccessible.rs
│   └── nested.rs
├── my.rs
└── split.rs
```

How to understand the structure with `my`:

- `my.rs` is a module.
- `inaccessible.rs` and `nested.rs` are sub modules of `my.rs` as they are under the directory of `my`. Their code are inserted into `my.rs` with `mod inaccessible` and `pub mod nested`.
- code example:
    - `./my/inaccessbiel.rs`
        ```rust
       fn privat_function() {
           println!("This is a private function"); 
       }
       ```
    - `./my/nested.rs`
        ```rust
        pub fn func() {
            println!("This is a public function");
        }
        
        pub mod aaa {
            pub func() {
                println!("This is a nested public functions"):
            ]
        }
        ```
    - `./my.rs`
        ```rust
        //import code from above files
        mod inaccessible;
        pub mod nested;
        
        //skip
        ```
- How to use all functions in `my.rs`? Just add `mod my;` to, for example, `split.rs`.


## 11. Crates

A crate is a compilation unit. This chapter describe a way to convert a single crate into a library and how to use the library. This is a more obsolete method compared to library structure created by cargo.

### 11.1. create a library

A library can be created from a single `.rs` file. Here is a simple example:

- `aaa.rs` is a collection of functions and modules
    ```rust
    pub fn print_aaa() {
        println!("aaa");
    }
    
    pub mod mod1 {
        pub fn print_mod1() {
            println!("module 1");
        }
    }
    ```

- `rustc --crate-type=lib aaa.rs` to compile `aaa.rs` to a library. The library name is `libaaa.rlib` by default, which is at the same directory.
- `libaaa.rlib` is not installed in the system, so we cannot delcair it with `use aaa;`

### 11.2. Using a library

To use this library, create `bbb.rs` in the same directory. `bbb.rs` has a `main()` function.

- `bbb.rs`
    ```rust
    fn main() {
        aaa::print_aaa();
        aaa::mod1::print_mod1();
    }
    ```

- compile and run `bbb.rs`
    - `rustc bbb.rs --extern aaa=libaaa.rlib` to  compile to executable `bbb`. Need to specify what is `aaa` in `bbb.rs`.
    - `./bbb` to run the file.
- I'd rather copy `aaa.rs` to `bbb.rs` and compile `bbb.rs` into an executable if `libaaa.rlib` is not used in other executables.


## Cargo

### 12.1. Dependencies

The dependencies can be from multiple sources. Here is an example `cargo.tmol` file.

```toml
[package]
name = "foo"
version = "0.1.0"
authors = ["mark"]

[dependencies]
clap = "2.27.1" # from crates.io
rand = { git = "https://github.com/rust-lang-nursery/rand" } # from online repo
bar = { path = "../bar" } # from a path in the local filesystem
```

### 12.2. Conventions

Cargo support more than one binaries in one project. Besides `main.rs`, other binaries are in `./src/bin/` directory:

```
foo
├── Cargo.toml
└── src
    ├── main.rs
    └── bin
        └── my_other_bin.rs
```

To compile a specific binary, run `$ cargo run --bin my_other_bin`. 

### 12.2. Tests

Unit tests are in the file with the module to be tested. Integration tests have its own subdirectory `./tests/`.

### 12.3. Build scripts

Not now


## 13. Attributes
Attributes are metadata applied to some module, crate, or item.

- #[outer_attribute] applies to the item immediatedly following it. Examples:
    ```rust
    #[derive(Debug)]
    struct Point {x: i32, y: i32}
    ```
    
- #[inner_attribute] applies to enclosing item, typically a module or a crate:
    ```rust
    #![allow(unused_variables)]
    
    fn main() { 
        let x = 3; // disable warning about unsed variables
    }
    ```
    
### 13.1. dead_code

`#[all(dead_code)]` turns off dead code lint waring about an unused function:

```rust
fn used_function() {}

// `#[allow(dead_code)]` is an attribute that disables the `dead_code` lint
#[allow(dead_code)]
fn unused_function() {}  // no warning for this function

fn noisy_unused_function() {}  // warning for this one

fn main() {
    used_function();
}
```

### 12.2. crates

Limited usage. No effect on cargo.

### 12.3. cfg
Configuration conditional checks.

- `#[cfg()]` vs `cfg!()`:
    - `#[cfg(...)]` is attribute applied to function below it.
    - `cfg(...)` returns a bool in a boolean expression.
- Example:
    ```rust
    // This function only gets compiled if the target OS is linux
    #[cfg(target_os = "linux")]
    fn are_you_on_linux() {
        println!("You are running linux!");
    }
    
    // And this function only gets compiled if the target OS is *not* linux
    #[cfg(not(target_os = "linux"))]
    fn are_you_on_linux() {
        println!("You are *not* running linux!");
    }
    
    fn main() {
        are_you_on_linux();
    
        println!("Are you sure?");
        if cfg!(target_os = "linux") {
            println!("Yes. It's definitely linux!");
        } else {
            println!("Yes. It's definitely *not* linux!");
        }
    }
    ```
    
    
## 14. Generics

**generic struct**

```rust
#[derive(Debug)]
struct A;  // concrete type
#[derive(Debug)]
struct Single(A)  // concrete type, typple-like struct
#[derive(Debug)]
struct SingleGeneric<T>(T)  // generic type, tupple-like struc

fn main() {
    let _s = Single(A);
    println!("{:?}", _s)  // Single(A)
    let _char = SingleGeneric('a')
    println!("{:?}", _char)  // SingleGeneric('a')
    let_i32 = SingleGendric(99)
    println!("{:?}", _i32)   // SingleGeneric(99)
}
```

### 14.4. Functions

To define a generic function, the function must be followed by `<T>` or like explicityly. The generic type `T` usually need to be bounded by traits for the code to work.

```rust
struct SGen<T>(T); // a generic type
fn gen_spec_t(x: SGen<i32>) {}  // not a generic function
fn generic<T>(x: SGen<T>) {}  // generic function

fn generic<T>(x: T)
where
    T: std::fmt::Display,
{
    println!("{x}");  // require T to have Display trait
}

fn main() {
    generic(123);
    generic("abc")
} 

```


### 14.2. Implementation

To implement methods for generic struct, we need `impl<T>` like definition.

```rust
use std::fmt::Display;

struct GenVal<T> {
    val: T,
}

impl<T> GenVal<T> {
    fn value(&self) -> &T {
        &self.val
    }
}

// start another impl if a method requres different traits of T
impl<T: Display> GenVal<T> {
    fn prt(&self) {
        println!("{}", self.val);
    }
}

fn main() {
    let x = GenVal { val: 99 };
    x.prt();
    let y = x.value();
    println!("{}", y);
}
```


### 14.3. Traits
A trait can be generic, which has generic parameters.

```rust
struct Empty;
struct Null;

// trait with a generic parameter.
// Its method will have two parameters
trait DoubleDrop<T> {
    fn double_drop(self, _: T);
}

// impl has two generic parameters,
// one for the type U implementing the trait,
// one for the parameter of the trait itself.
impl<T, U> DoubleDrop<T> for U {
    fn double_drop(self, _: T) {
        // nothing required
        println!("Dropped");
    }
}

fn main() {
    let aaa = Empty;
    let bbb = Null;
    
    // both aaa and bbb are dropped
    aaa.double_drop(bbb);
}
```


### 14.4 Bounds
Two pupurses of trait bound restriction:

- Restric the generic to types that conform to the bounds.
- Allow the generic to access the methods of the traits.

### 14.5. Multiple bounds

Use `+` to apply multiple bounds, for example, .
```rust
fn aaa<T: Debug + Display>(t: &T) {
    // pass
}
```

### 14.6. Where clause
Above example can be written as
```rust
fn aaa<T>(t: &T)
where T: Debug + Display,
{
    // pass
}
```

### 14.7. New tuype idiom

The idea is to create and use new types instead of using common types such as `i32` and `f64`. The purpose is to reduce accidental errors.

```rust
struct ProductID(u32);
fn print_productid(id: ProductID) {
    println!("{}", id.0);
}

fn main() {
    let pid = ProductID(13579);
    print_productid(pid); // 13579, the same as an integer but need type ProductID

    // the following is incorrect
    // print_productid(13579);  // error, wrong type.
}
```

### 14.8. Associated items

**What is associated items?**

- An item is a component of a crate, such as:
    - nodules
    - `use` declarations
    - function definitions
    - type definitions
    - struct definitions
    - constant items
    - trait definitions
    - implementations
    - and more
- An associated item is an extension to trait generics, which allows trait to internally define new items.
    - One such associated items is assocated type, providing simpler usage patterns whe nthe trait is generic over its container type. 


**Why use associated types in trait definition**

- avoid declair explicitly the generic type when using the trait, therefore make the code more readable???????????
- example
    ```rust
    struct Container(i32, i32);
    
    // trait defined explicitly by generic type A and B
    trait Contains1<A, B> {
        // explicity <A, B>
        fn first(&self) -> i32;
        fn last(&self) -> i32;
    }
    impl Contains1<i32, i32> for Container {
        fn first(&self) -> i32 {
            self.0
        }
        fn last(&self) -> i32 {
            self.1
        }
    }
    fn difference1<A, B, C>(container: &C) -> i32
    where
        // explicity <A, B, C>
        C: Contains1<A, B>,
    {
        container.last() - container.first()
    }
    
    // trait with associated types
    trait Contains2 {
        type A;  // declair associated types A and B in trait definition
        type B;  // They are still generic here
    
        fn first(&self) -> i32;
        fn last(&self) -> i32;
    }
    impl Contains2 for Container {
        type A = i32;  // specify associated type when implement method for a type
        type B = i32;  // be explicit
    
        fn first(&self) -> i32 {
            self.0
        }
        fn last(&self) -> i32 {
            self.1
        }
    }
    fn difference2<C: Contains2>(container: &C) -> i32 {  // use trait bound for C, 
        container.last() - container.first()              // no more A and B
    }                                                     // they are inferred from C
    
    fn main() {
        let number_1 = 3;
        let number_2 = 10;
    
        let container1 = Container(number_1, number_2);
        println!("The difference1 is: {}", difference1(&container1));
    
        let container2 = Container(number_1, number_2);
        println!("The difference2 is: {}", difference2(&container2));
    }
    ```
    
### 14.9. Phantom type parameters

**What is phantom type parameters**

- Do not show up at runtime but are checked statically at compile time.
- Are used as markers or to perform type checking at compile time.
- `use std::marker::PhantomData`

**Example**: 
```rust
use std::marker::PhantomData;
#[derive(PartialEq)]
struct PhantomTuple<A, B>(A, PhantomData<B>);

fn main() {
    let aaa: PhantomTuple<char, i32> = PhantomTuple('Q', PhantomData);
    let aaa1: PhantomTuple<char, i32> = PhantomTuple('S', PhantomData);
    let bbb: PhantomTuple<char, f64> = PhantomTuple('R', PhantomData);

    println!("aaa is the same as aaa1: {}", aaa == aaa1); // false
    // println!("aaa is the same as bbb: {}", aaa == bbb); // error, type mismatrch
}
```

#### 14.9.1. Testcase: unit clarification
This is a more realistic use case where a number is associated with a unit.

```rust
use std::marker::PhantomData;
use std::ops::Add;

// define unit type
#[derive(Debug, Copy, Clone)]
enum Inch {}
#[derive(Debug, Copy, Clone)]
enum Mm {}

// define a f64 Length type with Phantom type parameter "Unit" so that the f64
// number may have a different unit.
#[derive(Debug, Copy, Clone)]
struct Length<Unit>(f64, PhantomData<Unit>);

// implement Add trait, which defines the behavior of `+`
impl<Unit> Add for Length<Unit> {
    // type Output = Self; works the same as below. Length<Unit> can be replace
    // with Self in the function in all occurances
    type Output = Length<Unit>; // Output is an associated type of trait Add

    // this function allows adding two Length types using "+"
    // f64 already implemented + so we can self.0 + rhs.0 in this function.
    // After implement add for Length type, we can run L1 + L2
    fn add(self, rhs: Length<Unit>) -> Length<Unit> {
        Length(self.0 + rhs.0, PhantomData)
    }
}

fn main() {
    let one_foot: Length<Inch> = Length(12.0, PhantomData);
    let one_meter: Length<Mm> = Length(1000.0, PhantomData);

    println!("Two feet is {:?}", one_foot + one_foot);
    println!("Two meters is {:?}", one_meter + one_meter);
    // error: mismatched type
    // println!("Two feters is {:?}", one_meter + one_foot);
}


// The print out is
// Two feet is Length(24.0, PhantomData<main::Inch>)
// Two meters is Length(2000.0, PhantomData<main::Mm>)
```

**Trait std::ops::Add** is defined as

```rust
pub trait Add<Rhs = Self> {
    type Output;

    // Required method
    fn add(self, rhs: Rhs) -> Self::Output;
}
```

where `Rhs = Self` means the default type the generic parameter `Rhs` is `Self`, which is the type to implementing trait `Add`. It can take other type if specified.


## 15. Scoping rules

### 15.1. RAII

**RAII: Resource Acquisition Is Initialization**: Rust enforces RAII so whenever an object goes out of scope, its destructor is called and its owned resources are freed.

### 15.2. Ownership and moves

#### 15.2.1. Mutability
Mutability can be changed when ownership is transferred.
```rust
fn main() {
    let aaa = Box::new(999i32);
    // *aaa = 4;  // error, immutable
    
    let mut bbb = aaa;  // move and change mutability
    *bbb = 111;
    println!("{bbb}");
}
```

#### 15.2.2. Partial moves
```rust
fn main() {
    #[derive(Debug)]
    struct Person {
        name: String,
        age: Box<u32>,
    }

    let aaa = Person {
        name: "Tom".to_string(),
        age: Box::new(37),
    };

    // partially destructure aaa. Destructure name only
    let Person { name: xxx, ref age } = aaa;
    println!("The unnamed person's name is {} and age is {}", xxx, age);
    // aaa.name is not available but aaa.age is available
    // println!("{:?}", aaa.name);  // error, borrow of moved value
    println!("{:?}", aaa.age);
}
```

### 15.3. Borrowing

#### 15.3.1. Mutability
Already know mutable borrow `$mut`.

#### 15.3.2. Aliasing

Only allow one mutable borrowing when no other borrowing, mutable or not.

#### 15.3.3. The ref pattern

**keyword** `ref`

- `ref` is used at the left side of a `let` binding, equivalent to `&` on the right side.
- The `ref` keyword can be used to take references to the fields of a struct or tuple when doing pattern matching or destructuring via the `let` binding.

**Example**

```rust
fn main() {
    let aaa = "tom".to_string();
    // abb and acc both borrow from aaa
    let abb = &aaa;
    let ref acc = aaa;
    println!("{:?} and {:?}", abb, acc);

    #[derive(Debug)]
    struct Point {
        x: i32,
        y: i32,
    }

    let p1 = Point { x: 11, y: 22 };

    let copy_p1_x = {
        // destructure
        let Point { x: ref xxx, y: _ } = p1;
        *xxx // xxx is a reference to p1.x
    };
    println!("{}", copy_p1_x);

    // ref mut for mutable reference at left side.
    let mut p2 = p1;
    {
        let Point {
            x: _,
            y: ref mut yyy,
        } = p2;
        *yyy += 33;
    }
    println!("{}", p2.y);
}
```

### 15.4. Lifetimes
Most examples can be simplified by elision. Do not pay much attentions to the details. Rust Book has better discussion on lifetimes.

#### 15.4.2. Explicity annotation
already know

#### 15.4.2. Functions
already know

#### 15.4.3. Methods
already know

#### 15.4.4. Structs
Similar

#### 15.4.5. Traits
Similar

#### 15.4.6. Bounds
Generic types can be bounded by lifetimes. Exmaple:

```rust
fn print_ref<'a, T>(t: &'a T) 
    where T: Debug + 'a
{
   println!("{:?}", t); 
}
```

#### 15.4.7. Coercion
Do not see why need lifetime coercion from the examples. The examples can be simplified.


#### 15.4.8. Static
Need a better intro. The examples here are not sufficient to understand static.

#### 15.4.9. Elission
Refer to the Rust Book.

## 16. Traits
A trait is a collection of methods defined for an unknown type: `Self`.

### 16.1. Derive
A list of traits can be `#[derive(...)]`ed.

- `std::cmp::PartialEq`: for a **partial equivalence relation**, which is symetric and transitive, but not reflexive.
    - symetric: `a == b` inmplies `b == a`
    - transitive: `a == b` and `b == c` implies `a == c`
    - reflexive: `a == a`. This is not true, for example, for `NaN` in type `f32` and `f64`
    - definition.
        ```rust
        pub trait PartialEq<Rhs = Self>
        where
            Rhs: ?Sized,
        {
            // Required method, x.eq(y) is x == y
            fn eq(&self, other: &Rhs) -> bool;
        
            // Provided method, x.ne(y) is x != y
            fn ne(&self, other: &Rhs) -> bool { ... }
        }
        ```

- `std::cmo::Eq`: for equivalence relation, which is symmetric, transitive, and reflexive.

- `std::cmp::PartialOrd`: is used when not all two values of a type can be compared, for example, we cannot compare two `NaN`s of type `f32` or `f64`.
    ```rust
    pub trait PartialOrd<Rhs = Self>: PartialEq<Rhs>
    where
        Rhs: ?Sized,
    {
        // Required method
        fn partial_cmp(&self, other: &Rhs) -> Option<Ordering>;
    
        // Provided methods
        fn lt(&self, other: &Rhs) -> bool { ... }  // for a < b
        fn le(&self, other: &Rhs) -> bool { ... }  // for a <= b
        fn gt(&self, other: &Rhs) -> bool { ... }  // for a > b
        fn ge(&self, other: &Rhs) -> bool { ... }  // for a >= b
    }
    ```
    
- `std::cmp::Ord` to compare any two pairs. Applicable to type like `i32` and `char`.
    ```rust
    pub trait Ord: Eq + PartialOrd {
        // Required method
        fn cmp(&self, other: &Self) -> Ordering;
    
        // Provided methods
        fn max(self, other: Self) -> Self
           where Self: Sized { ... }
        fn min(self, other: Self) -> Self
           where Self: Sized { ... }
        fn clamp(self, min: Self, max: Self) -> Self // return max or min if
           where Self: Sized + PartialOrd { ... }    // self is out of the range
    }
    ```
    
- example
    ```rust
    #[derive(PartialEq, PartialOrd)]
    struct Aaa(f64);
    
    fn main() {
        let a1 = Aaa(3.14);
        let a2 = Aaa(9.82);
        if a1 > a2 {
            println!("{}", a1.0);
        } else {
            println!("{}", a2.0);
        }
    }
    ```
    
### 16.2. Returning Traits with `dyn`
When a function returns different types that implements a common trait Xxxx, we use `-> Box<dyn Xxxx>` as the function return.

- example:
    ```rust
    struct Sheep {}
    struct Cow {}
    
    trait Animal {
        fn noise(&self) -> &'static str;
    }
    
    impl Animal for Sheep {
        fn noise(&self) -> &'static str {
            "baaaah!"
        }
    }
    
    impl Animal for Cow {
        fn noise(&self) -> &'static str {
            "mooooo!"
        }
    }
    
    // a function whose return signature is -> impl Animal can only possible return
    // one type, which must be pre-determine in function body at compile time.
    fn one_animal() -> impl Animal {
        Sheep {} // only return type Sheep
    }
    
    // The return type Cow or Sheep is determine in run time. In compile time the
    // function still return one type Box. Yes, Box is a type that is stored in
    // stack and points to values in heap.
    fn random_animal(random_number: f64) -> Box<dyn Animal> {
        if random_number < 0.5 {
            Box::new(Sheep {})
        } else {
            Box::new(Cow {})
        }
    }
    
    fn main() {
        let animal = random_animal(0.1);
        println!("{}", animal.noise());
    
        let aaa = one_animal();
        println!("{}", aaa.noise());
    }
    ```
    
### 16.3 Operator overloading

Many operators can be overloaded via traits. For example, if a type implemented trait `Add`, then `a + b` is equivalent to `a.add(b)`. The operator `+` acts differently based on the method `add` implemeted for each type.

- trait `std::ope::Add`:
    - signature: the default Rhs is Self, but can be other types. 
        ```rust
        pub trait Add<Rhs = Self> {
            type Output;
        
            // Required method
            fn add(self, rhs: Rhs) -> Self::Output;
        }
        ```
    - example
        ```rust
        use std::ops::Add;
        
        // create a set of zero-sized types (ZST) for demonstration
        struct Foo;
        struct Bar;
        
        #[derive(Debug)]
        struct FooBar;
        #[derive(Debug)]
        struct BarFoo;
        
        // Add trait most used to add two variables of the same type. But it can be
        // to joint two different types, for example String + &str.
        impl Add<Bar> for Foo {
            type Output = FooBar;
            fn add(self, _: Bar) -> FooBar {
                FooBar
            }
        }
        
        // a + b can be different from b + a
        impl Add<Foo> for Bar {
            type Output = BarFoo;
            fn add(self, _: Foo) -> BarFoo {
                BarFoo
            }
        }
        
        fn main() {
            let x = Foo;
            let y = Bar;
            println!("{:?}", x + y); // operator + moves both x and y
        
            let x = Foo;
            let y = Bar;
            println!("{:?}", x.add(y)); // another way of using add
        
            let x = Foo;
            let y = Bar;
            println!("{:?}", Add::add(y, x)); // one more way of using add
        }
        ```
        
### 16.4. Drop
Rust automatically drop a variable when it goes out of scope by calling `Drop::Drop`. Users, however, can still implement the trait for type to, for example, print out message when a variable is dropped.

- signature of trait `std::ops::Drop`
    ```rust
    pub trait Drop {
        // Required method
        fn drop(&mut self);
    }
    ```
    
- example
    ```rust
    struct Droppable {
        name: &'static str,
    }
    
    impl std::ops::Drop for Droppable {
        fn drop(&mut self) {
            println!("{} is dropped!", self.name);
        }
    }
    
    fn main() {
        let x = Droppable { name: "Tom" };
        {
            let _y = Droppable { name: "Jerry" };
        }  // print message when _y goes out of scope here
        
        drop(x);  // print message when x is dropped
    }
    ```
    
### 16.5. Iterators

**Iterator basics**

- `for` construct turns some collections into iterators using the `.into_iter()` method implicitly.
- `.take(n)` method keep the first n element, `.skip(n)` method skips the first n element.
- example
    ```rust
    fn main() {
        let x = 1..5;
        for i in x {
            println!("{i}");
        }
    
        let y = 1..10;
        for i in y.take(3) {
            println!("{i}");
        }
    
        let y = 1..10;
        for i in y.skip(2).take(3) {
            println!("{i}");
        }
    }
    ```
    

**Fibonacci sequence**

    ```rust
    #[derive(Debug)]
    struct Fibonacci {
        cur: u32,
        next: u32,
    }
    
    impl Iterator for Fibonacci {
        type Item = u32;a // must use name Item
    
        // .next() returns an Option
        fn next(&mut self) -> Option<Self::Item> {
            let current = self.cur;
            self.cur = self.next;
            self.next = current + self.next;
    
            // Fibonacci never ends, so None is never returned.
            Some(current)
        }
    }
    
    fn fib() -> Fibonacci {
        Fibonacci { cur: 0, next: 1 }
    }
    
    fn main() {
        let mut x = fib();
        println!("{:?}", x.next());  // Some(0)
        println!("{:?}", x.next());  // Some(1)
        println!("{:?}", x.next());  // Some(1)
        println!("{:?}", x.take(3)); // Take { iter: Fibonacci { cur: 2, next: 3 }, n: 3 }
    
        let x = fib();
        for i in x.take(6) {
            println!("{}", i);
        }
    }
    ```

### 16.6. impl trait

**Normal use**: Declair function argument or return type by traits like `fn aaa(x: TraitA) -> impl TraitB {}`.

**use impl with closures**: `Fn`, `FnMut`, and `FnOnce` are traits for closures.

    ```rust
    // This function returns a closure that has one parameters
    fn make_adder_function(y: i32) -> impl Fn(i32) -> i32 {
        let closure = move |x: i32| x + y;
        closure
    }
    
    fn main() {
        let aaa = 99;
        // add_9 is a closure that returns x + 9
        let add_9 = make_adder_function(9);
        println!("{}", add_9(aaa));
    }
    ```


### 16.7. Clone
use method `.clone()` to make a copy if the type implements `Clone` trait.


### 16.8. Supertraits

A supertrait B is like a trait bound to a trait A. When implementing A for a type T, we need the type is also implementetd with trait B.Simply speaking, A can only be implemeted to type that implements A's super trait B.

```rust
use std::fmt;

// supertrait fmt::Display is a trait bound on trait OutlinePrint
trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        // to_string() require self to have Display trait, so we add 
        // fmt::Display as the super trait of trait OutlinePrint
        let output = self.to_string(); 
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", output);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}

struct Point {
    x: i32,
    y: i32,
}

// must implement fmt::Display for Point before implementing OutlinePrint
// for Point.
impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}

impl OutlinePrint for Point {}

fn main() {
    let p = Point { x: 11, y: 22 };
    p.outline_print();
}
```

### 16.9. Disambiguating overlapping traits

When a type is implemeted with two or more traits and the traits have the same method name like `.get()`, how to resolve the conflict in names? Using **Fully Qualified Syntax**.

```rust
trait NameGetter {
    fn get(&self) -> String;
}

trait AgeGetter {
    fn get(&self) -> u8;
}

struct Aaa {
    name: String,
    age: u8,
}

impl NameGetter for Aaa {
    fn get(&self) -> String {
        self.name.to_string()
    }
}

impl AgeGetter for Aaa {
    fn get(&self) -> u8 {
        self.age
    }
}

fn main() {
    let aaa = Aaa {
        name: "Tom".to_string(),
        age: 77,
    };
    
    // use fully qualified syntax
    let name = <Aaa as NameGetter>::get(&aaa);
    let age = <Aaa as AgeGetter>::get(&aaa);
    println!("Name: {}, Age: {}", name, age);
}
```


## 17. macro_rules

Macros generate source code that gets compiled with the rest of program.

### 17.1. syntax

#### 17.1.1. Designators

**list of common designators**

- `expr` for expressions
- `ident` for variable/function names
- `ty` for type
- `stmt` for statement
- `literal` for literal constant
- `tt` for token tree
- ...

**examples**

- a macro that generats functions
    ```rust
    macro_rules! create_function {
        ($func_name:ident) => {
            fn $func_name() {
                // stringify! is a macro that converts an ident into a string
                println!("You are calling function {:?}", stringify!($func_name));
            }
        };
    }
    
    fn main() {
        create_function!(func1);
        create_function!(func2);
        func1();
        func2();
    }
    ```

- a macro that process expressions
    ```rust
    macro_rules! print_result {
        ($x:expr) => {
            println!("{:?} = {:?}", stringify!($x), $x);
        };
    }
    
    fn main() {
        // output "{ let x = 1u32; x * x + 2 * x - 1 }" = 2
        print_result!({
            let x = 1u32;
            x * x + 2 * x - 1
        });
    
        // output "9 + 2" = 11
        print_result!(9 + 2);
    }
    ```
    
#### 17.1.2. Overload

Macros can be overloaded to accept different combinations of arguments.

    ```rust
    macro_rules! aaa {
        // --- can be replaced with a random string. It is used as a identifier
        // for this branch.
        ($left:expr; --- $right:expr) => {
            println!(
                "{:?} and {:?} is {:?}",
                stringify!($left),
                stringify!($right),
                $left && $right
            );
        };
        
        // similarly, orrr can be replaced with another random string.
        ($left:expr; orrr $right:expr) => {
            println!(
                "{:?} or {:?} is {:?}",
                stringify!($left),
                stringify!($right),
                $left || $right
            );
        };
    }
    
    fn main() {
        aaa!(true; --- false);
        aaa!(3 + 5 == 8; orrr false);
    }
    ```

#### 17.1.3. Repeat

Use `+` and `*` like in regular expression but repeat an argument.

    ```rust
    macro_rules! bbb {
        // base case
        ($x:expr) => {
            $x
        };
    
        // repeat "$($y:expr)," one or more times.
        ($x:expr, $($y:expr),+) => {
            // call bbb! recursively
            std::cmp::min($x, bbb!($($y),+))
        }
    }
    
    fn main() {
        println!("{}", bbb!(3, 2, 5));
    }
    ```

### 17.2. DRY (Don't Repeat Yourself)

skip for now

### 17.3. DSL (Domain Specific Languages)

Use rust macros to construct the syntax of another language that is compiled into Rust.


### 17.4. Variadic interfaces

A variadic interface takes an arbitrary number of arguments.
