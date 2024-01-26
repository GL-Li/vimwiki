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
