---
title: "Interfacing with Rust from C/C++"
date: 2023-08-16T16:34:36+02:00
tags: ["whatilearned"]
draft: false
---

The Rust programming language is considered as one of the potential successors
for C++ when it comes to systems programming. A reasonable first step, when
adapting to a new language, is replacing small parts of a system while the
remaining parts are maintained in the former language.

To achieve this, an interoperability between the new and old language needs to
be established. In the following, I will describe how to implement a dynamic
Rust library that exposes its interface to C/C++.

## Make a Rust library accessible from C/C++

Assume working in a cargo directory that was created e.g. by

```
cargo new --lib mylib
```

### Use the FFI

Rust relies on the Foreign Function Interface (FFI) to interface with other
languages. The FFI mainly targets C as language.

To make functions in Rust code accessible via the FFI, they need to be
externalized and declared as `no_mangle`:

```rust
// src/lib.rs

#[no_mangle]
pub extern "C" fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

- `extern "C"` ensures the C ABI is used for calling functions in shared
  libraries
- `#[no_mangle]` ensures the function names don't experience
  [mangling](https://en.wikipedia.org/wiki/Name_mangling)

Both conditions must be fulfilled to enable interfacing via this function using
the FFI.

### Build a dynamic library

By default, cargo does [not build dynamic system
libraries](https://doc.rust-lang.org/reference/linkage.html#linkage). To
generate a dynamic library (.dylib on Mac, .so in Linux and .dll on Windows),
the `Cargo.toml` has to be extended by:

```toml
# Cargo.toml
[lib]
crate-type   = ["cdylib"]
```

This ensures `rustc` actually compiles a dynamic library.

### Generate a C/C++ header file

To access functions in third-party libraries, C/C++ code needs to include a
header file that declares the interface towards the library.

The header file can be written and maintained by hand, or be generated
automatically by using `cbindgen`. Add `cbindgen` to the build dependencies in
`Cargo.toml`:

```toml
# Cargo.toml
[build-dependencies]
cbindgen = "0.24.3"
```

And place a custom `build.rs` file in the project root with the following
content:

```rust
// build.rs

extern crate cbindgen;

use std::env;

fn main() {
    let crate_dir = env::var("CARGO_MANIFEST_DIR").unwrap();
    println!("Generating C/C++ header");
    cbindgen::Builder::new()
      .with_crate(crate_dir)
      .generate()
      .expect("Unable to generate bindings")
      .write_to_file("include/mylib.h");
}
```

This will ensure when calling `cargo build`, `cbindgen` is used to
auto-generate a C/C++ header file that declares the interface of the Rust
library.

After invoking `cargo build --release`, the generated header looks like this:

```cpp
// include/mylib.h

#include <cstdarg>
#include <cstdint>
#include <cstdlib>
#include <ostream>
#include <new>

extern "C" {

int32_t add(int32_t a, int32_t b);

} // extern "C"
```

## Building the C++ project

Assume we want to use the Rust library in a C++ project:

```cpp
// main.cpp
#include <add_rs.h>
#include <iostream>

int main()
{
    int result = add(3, 5); // `add` is implemented in Rust!
    std::cout << "Result: " << result << std::endl;
    return 0;
}
```

To compile this project with GCC, e.g. use:

```
g++ main.cpp -o add -Imylib/include -Lmylib/target/release/ -ladd_rs
```

Now, if you execute the program, the C++ program will interface with the
library written in Rust using a C interface to calculate the desired result:

```
Result: 8
```
