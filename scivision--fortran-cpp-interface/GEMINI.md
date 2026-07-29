## fortran-cpp-interface

> when building the project code, use CMake presets


* build and test project code: `cmake --workflow default`
* only build the project code: `cmake --workflow build`

* Notice that the test/*/CMakeLists.txt files have directory property LABELS set that can be used to run a subset of tests with `ctest -L <label>` (e.g. `ctest -L allocate` to run only the tests in test/allocate)

---
> Source: [scivision/fortran-cpp-interface](https://github.com/scivision/fortran-cpp-interface) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-07-24 -->
