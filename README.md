# ocl-include

[![Crates.io][crates_badge]][crates]
[![Docs.rs][docs_badge]][docs]
[![Travis CI][travis_badge]][travis]
[![Appveyor][appveyor_badge]][appveyor]
[![Codecov.io][codecov_badge]][codecov]
[![License][license_badge]][license]

[crates_badge]: https://img.shields.io/crates/v/ocl-include.svg
[docs_badge]: https://docs.rs/ocl-include/badge.svg
[travis_badge]: https://api.travis-ci.org/nthend/ocl-include.svg
[appveyor_badge]: https://ci.appveyor.com/api/projects/status/github/nthend/ocl-include?branch=master&svg=true
[codecov_badge]: https://codecov.io/gh/nthend/ocl-include/graphs/badge.svg
[license_badge]: https://img.shields.io/crates/l/ocl-include.svg

[crates]: https://crates.io/crates/ocl-include
[docs]: https://docs.rs/ocl-include
[travis]: https://travis-ci.org/nthend/ocl-include
[appveyor]: https://ci.appveyor.com/project/nthend/ocl-include
[codecov]: https://codecov.io/gh/nthend/ocl-include
[license]: #license

Simple preprocessor that implements #include mechanism for OpenCL source files.

## About

OpenCL API doesn't provide mechanism for including header files into the main one, like in C and C++. This crate is a simple preprocessor that handles `#include ...` and `#pragma once` directives in source files, collects them over filesystem or memory, and gives a single string to the output that could be passed to OpenCL kernel builder.

## Documentation

+ [`crates.io` version documentation](https://docs.rs/ocl-include)
+ [`master` branch documentation](https://nthend.github.io/ocl-include/target/doc/ocl_include/index.html)

## Examples

Let you have `main.c` and `header.h` files in `./kernels/` folder:

`main.c`:
```c
#include <header.h>

int main() {
    return RET_CODE;
}
```

`header.h`:
```c
#pragma once

static const int RET_CODE = 0;
```

### Filesystem only

The follwong code takes `main.c` from the filesystem and includes `header.h` into it.

```rust
use std::path::Path;
use ocl_include::*;

fn main() {
    let hook = FsHook::new()
    .include_dir(&Path::new("./examples")).unwrap();

    let node = collect(&hook, Path::new("main.c")).unwrap();

    println!("{}", node.collect());
}
```

### Filesystem and memory

The follwong code takes `main.c` source from the memory and includes `header.h` into it from the filesystem.

```rust
use std::path::Path;
use ocl_include::*;

fn main() {
    let main = r"
    #include <header.h>
    int main() {
        return ~RET_CODE;
    }
    ";

    let hook = ListHook::new()
    .add_hook(
        MemHook::new()
        .add_file(&Path::new("main.c"), main.to_string()).unwrap()
    )
    .add_hook(
        FsHook::new()
        .include_dir(&Path::new("./examples")).unwrap()
    );

    let node = collect(&hook, Path::new("main.c")).unwrap();

    println!("{}", node.collect());
}
```

## Hooks

Hook is a handler that retrieve files by their names.

The crate contains the following hooks now: 

+ `FsHook`: takes files from the filesystem.
+ `MemHook`: retrieves the source from the memory.
+ `ListHook`: contains a list of other hooks and tries to retrieve source from them subsequently.

## License

Licensed under either of

 * Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
 * MIT license ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.

### Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be dual licensed as above, without any
additional terms or conditions.