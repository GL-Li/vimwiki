## Outline

- QA
- Raw notes

## QA

### QA: why `extendr_api::wrapper::strings::Strings` method `.clone()` does not create a deep copy?
Q: The original question on stackoverflow:https://stackoverflow.com/questions/77815360/how-to-modify-a-r-string-in-rust-and-assign-it-to-a-new-r-variable-and-keep-the/77816394#77816394. 

A: The accepted answer goes deep in the concept of clone in rextendr:

- In Rust, `clone` can make a deep or shallow copy, depending on how it is implemented. The Rust String's clone makes a deep copy. This is not the case for extendr's Strings clone.

- R internals: https://cran.r-project.org/doc/manuals/r-release/R-ints.html#SEXPs
    - SEXP: S-expression pointer
    - SEXPREC: SEXP RECord structure SEXP points to. It is a C structure underlying every R object.
    - An R variable is bound to a value, which can be SEXP or SEXPREC.

- `extendr_api::wrapper::strings::Strings` is a wrapper of `extendr_api::robj::Robj`, which is a wrapper for an R SEXP.

- The `Clone` trait of `Robj` calls `Robj::from_sexp(self.get())`, which returns a SEXP, a pointer.

- example: the original R string changed with `.clone()`. 
    ```r
    library(rextendr)

    rust_source(code = '
    #[extendr]
    fn strings_elt(x: Strings) -> Strings {
        let mut y = x.clone();
        y.set_elt(0, "AAA".into());
        rprintln!("x is {:?} and y is {:?}", x, y);
        y
    }
    ')

    s <- c("aaa", NA, "bbb")
    s1 <- strings_elt(s)
    s1   # "AAA" NA    "bbb"
    s    # "AAA" NA    "bbb", also changed
    ```
    
- example: use `.duplicate()` to keep the original R string
    ```r
    rust_source(code = '
    #[extendr]
    fn strings_elt(x: Strings) -> Strings {
        let mut y = Strings::try_from(x.duplicate()).unwrap();
        y.set_elt(0, "AAA".into());
        rprintln!("x is {:?} and y is {:?}", x, y);
        y
    }
    ')

    s <- c("aaa", NA, "bbb")
    s1 <- strings_elt(s)
    s1 # "AAA" NA    "bbb"
    s # "aaa" NA    "bbb"
    ```
    

## Raw Notes

### Struct extendr_api::robj::Robj

- definition: wrapper for an R S-expression pointer (SEXP):
    ```rust
    pub struct Robj {
        inner: SEXP,
    }
    ```
    
- traits and methods:
    - `.clone()` to make a copy of Robj, which is a copy of SEXP
        ```rust
        impl Clone for Robj {
            fn clone(&self) -> Self {
                unsafe { Robj::from_sexp(self.get()) }
            }
        }
        ```
        
        - `from_sexp` to create an Robj from SEXP
            ```rust
            impl Robj {
                pub fn from_sexp(sexp: SEXP) -> Self {
                    single_treaded(|| {
                        unsafe { ownership::protect(sexp) };
                        Rojb { inner: sexp }
                    }
                }
            }
            ```
            
        - `get()` of trait GetSexp to return the SEXP of an Robj
            ```rust
            impl GetSexp for Robj {
                unsafe fn get(&self) -> SEXP {
                    self.inner
                }
            }
            ```
        
    - `duplicate()` to make a deep copy
        ```rust
        pub trait Rinternals: Types + Conversions {
            fn duplicate(&self) -> Robj {
                single_treaded(|| unsafe {Robj::from_sexp(Rf_duplicate(self.get())})
            }
        }
        ```
        
        - `Rf_duplicate(SEXP)` is a R internal function which makes a deep copy of a SEXP ??????????????????????????????????
        


