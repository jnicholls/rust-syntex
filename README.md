Syntex Code Generation Framework
================================

[![Build Status](https://api.travis-ci.org/erickt/rust-syntex.png?branch=master)](https://travis-ci.org/erickt/rust-syntex)
[![Latest Version](https://img.shields.io/crates/v/syntex.svg)](https://crates.io/crates/syntex)

`syntex` is a library that enables compile time syntax extension expansion.
This allows users to use libraries like [regex\_macros]() in a rust 1.0
compatible context.

Configuring with Cargo
======================

To create a package:

```toml
[package]
name = "hello_world_macros"
version = "0.2.0"
authors = [ "erick.tryzelaar@gmail.com" ]

[dependencies]
syntex = "*"
syntex_syntax = "*"
```

To use it:

Cargo.toml:

```toml
[package]
name = "hello_world"
version = "0.3.0"
authors = [ "erick.tryzelaar@gmail.com" ]
build = "build.rs"

[build-dependencies]
syntex = "*"
```

build.rs:

```rust
extern crate syntex;
extern crate hello_world_macros;

use std::env;
use std::path::Path;

fn main() {
    let mut registry = syntex::Registry::new();
    hello_world_macros::register(&mut registry);

    let src = Path::new("src/main.rs.in");
    let dst = Path::new(&env::var("OUT_DIR").unwrap()).join("main.rs");

    registry.expand("hello_world", &src, &dst).unwrap();
}
```

src/main.rs:

```rust
// Include the real main
include!(concat!(env!("OUT_DIR"), "/main.rs"));
```

src/main.rs.in:

```rust
fn main() {
    let s = hello_world!();
    println!("{}", s);
}
```

Limitations
===========

Unfortunately because there is no stable plugin support in Rust yet, there are
some things that syntex cannot do:

* The code generated by syntex reports errors in the generated file, not the
  source file.
* Syntex macros cannot be embedded in macros it doesn't know about, like the
  builtin `vec![]`, `println!(...)`, and etc. This is because those macros
  may override the `macro_name!(...)` to mean something different.
