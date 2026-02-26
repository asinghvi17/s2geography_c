# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

C API wrapper for [s2geography](https://github.com/paleolimbot/s2geography), providing an opaque-handle C interface suitable for FFI consumption from Julia, Rust, Python, etc. All functions use the `s2geog_` prefix.

## Build

This library builds as a subdirectory of s2geography. The expected layout is:

```
parent/
  s2geography/      # clone of paleolimbot/s2geography
  s2geography_c/    # this repo
```

Build from the s2geography directory:

```bash
cd s2geography
mkdir build && cd build
cmake .. -DS2GEOGRAPHY_BUILD_C_API=ON -DS2GEOGRAPHY_BUILD_TESTS=ON -DCMAKE_CXX_STANDARD=17
cmake --build .
```

### Run tests

```bash
# GTest (C++) tests
./s2geography_c/s2geography_c_test

# Pure C API test
./s2geography_c/s2geography_c_api_test

# Or via ctest (discovers both)
ctest -T test --output-on-failure .
```

### Dependencies

Same as s2geography: CMake 3.14+, s2geometry, Abseil, OpenSSL. Use conda (`mamba install cxx-compiler s2geometry libabseil cmake openssl -c conda-forge`) or Homebrew.

## Architecture

### API pattern

Opaque-handle C API. All types are forward-declared typedefs (e.g., `S2GeogGeography*`). Internally these are `reinterpret_cast` wrappers around s2geography C++ types.

- **Error handling**: Functions returning `int` use 0=success, non-zero=error. Functions returning pointers return `NULL` on error. Call `s2geog_last_error()` for the error message (thread-local).
- **Memory**: Caller owns all returned objects. Free with the matching `_destroy` or `_free` function. `s2geog_make_collection` takes ownership of its child geographies.
- **Coordinate input**: `_lnglat` variants take interleaved `[lng0, lat0, lng1, lat1, ...]` in degrees. `_xyz` variants take interleaved `[x0, y0, z0, ...]` unit-sphere coords. All input arrays are borrowed (caller retains ownership).

### File layout

- `include/s2geography_c.h` — public C header (all declarations)
- `src/s2geography_c.cc` — implementation (single translation unit)
- `test/s2geography_c_test.cc` — GTest C++ tests
- `test/s2geography_c_api_test.c` — pure C tests (proves header/linkage work from C)

### Operation categories exposed

Geometry construction (`s2geog_make_*`), WKT/WKB/GeoArrow IO, scalar accessors, ShapeIndex (prepared geometry), predicates, distance, geometry ops, boolean ops, coverings, linear referencing, cell/point ops, aggregators, GeographyIndex, ArrowUDF, projections.
