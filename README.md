# rgo - Rust Scope Guard Implementation

A scope guard will run a given closure when it goes out of scope, even if the code between panics (as long as panic doesn't abort).

## Installation

Add this to your `Cargo.toml`:

```toml
[dependencies]
rgo = "1.0"
```

## Usage

### Basic Example

```rust
extern crate rgo;

fn f() {
    let _guard = rgo::guard((), |_| {
        println!("Hello Scope Exit!");
    });

    // rest of the code here

    // Here, at the end of `_guard`'s scope, the guard's closure is called
    // It is also called if we exit this scope through unwinding instead
}
```

### `defer!` Macro

```rust
#[macro_use(defer)] extern crate rgo;

use std::cell::Cell;

fn main() {
    let drop_counter = Cell::new(0);
    {
        defer! {
            drop_counter.set(1 + drop_counter.get());
        }

        // Do regular operations here
        assert_eq!(drop_counter.get(), 0);
    }
    assert_eq!(drop_counter.get(), 1);
}
```

### Scope Guard with Value

```rust
extern crate rgo;

use std::fs::File;
use std::io::{self, Write};

fn try_main() -> io::Result<()> {
    let f = File::create("newfile.txt")?;
    let mut file = rgo::guard(f, |f| {
        // ensure we flush file at return or panic
        let _ = f.sync_all();
    });
    file.write_all(b"test me\n").map(|_| ())
}
```

## Crate Features

- `use_std` (enabled by default): Enables the `OnUnwind` and `OnSuccess` strategies
- Disable `use_std` to use `no_std`

## Rust Version

This crate requires Rust 1.20 or later.