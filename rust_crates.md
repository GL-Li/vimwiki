## proc_macro

**Tokens** are primitive productions in the grammar defined by rust language, includes:

- keywords such as `let` and `fn`
- identifiers such as variables `last_name` and `my_func`
- literals, such as 1234 and "abcd"
- lifetimes
- punctuations
- delimiters

**Token trees** are tokens organized into a structure such as `let x = 4;` which includes tokens `let`, `x`, `=`, `4`, and `;`.

**Token streams** are sequences fo token trees, usually used as the input and output of procedural macros.

## syn tbd

Syn is a parsing library for parsing a stream of Rust tokens into a syntax tree of Rust source code.
