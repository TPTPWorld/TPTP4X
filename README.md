## How to build

The recommended way is to just run `make`, but see the `README` in the `JJParser` submodule,
and make sure `TPTP4X` (this project) and `JJParser` are siblings in your file system.

Alternatively, you can use CMake.
But this approach has not been fully tested.
At project root, run
```
cmake -B build
cmake --build build
```
CMake will automatically download the newest version of `JJParser` into the `build` directory,
and the executable binary `tptp4X` will also be located in `build`.