### Breaking changes

- [PR#111](https://github.com/biojppm/c4core/pull/111) - Rename formatting overloads accepting `c4::append`:
   - `catrs(append_t, ...) -> catrs_append(...)`
   - `catseprs(append_t, ...) -> catseprs_append(...)`
   - `formatrs(append_t, ...) -> formatrs_append(...)`
- [#PR101](https://github.com/biojppm/c4core/pulls/101): As part of the `substr` ctor cleanup, the `to_substr(char (&arr)[N])` overload no longer decays to `char*` inside. This changes calling code by now returning a `substr` with length equal to `N-1` instead of `strlen(arr)` as before:
```c++
// longer than "foo", ie longer than {'f', 'o', 'o', '\0'}:
char arr[] = "foo\0\0\0\0\0\0";
assert(strlen(arr) == 3);
assert(sizeof(arr) == 9);

// previously:
assert(to_substr(arr).len == 3);
// now:
assert(to_substr(arr).len == 9);

// the breaking change happens only with arrays:
assert(to_substr((char*)ptr).len == 3); // as before
```

### New features

- [#PR101](https://github.com/biojppm/c4core/pulls/101): For `substr` and `csubstr`:
  - add simultaneous ctors from `char[]` and `char*`. Using SFINAE to narrow the `char*` overload prevents it from overriding the `char[]` overload. Thanks to @huangqinjin for the idea (see [#97](https://github.com/biojppm/c4core/issues/97)).
  - remove unneeded constructors of `csubstr` from non-const chars.
  - to each single-argument ctor, add corresponding functions `to_csubstr()` and `to_substr()` to enable clients coercing their types in generic code such as `c4::cat()` and `c4::format()`.
  - Add interop with `std::string_view` when the standard is at least C++17 ([#PR101](https://github.com/biojppm/c4core/pulls/101)):
    - provided in the header [`c4/std/string_view.hpp`](src/c4/std/string_view.hpp)
    - similarly to existing interop headers, this is opt-in and requires explicit inclusion
    - implemented:
      - `to_csubstr()` (since `std::string_view` is not writeable, cannot provide `to_csubstr()`)
      - `to_chars()` (since `std::string_view` is not writeable, cannot provide `from_chars()`)
      - comparison operators
- `substr`: split `.first_not_of()` and `.last_not_of()` into different overloads, removing the defaulted `start` parameter:
  - `.first_not_of(T, start=0)` -> `.first_not_of(T)` , `.first_not_of(T, start)`
  - `.last_not_of(T, start=npos)` -> `.first_not_of(T)` , `.first_not_of(T, npos)`
  This may or may not result in a speedup.
- [PR#105](https://github.com/biojppm/c4core/pull/105): Add macros in `c4/language.hpp` for compile-time flow of exceptions:
  - `C4_EXCEPTIONS`: defined when exceptions are enabled
  - `C4_IF_EXCEPTIONS(code_with_exc, code_without_exc)`: select statements for exceptions enabled/disabled
  - `C4_IF_EXCEPTIONS_(code_with_exc, code_without_exc)`: select code tokens for exceptions enabled/disabled
- [PR#105](https://github.com/biojppm/c4core/pull/105): Add macros in `c4/language.hpp` for compile-time flow of RTTI:
  - `C4_RTTI`: defined when rtti is enabled
  - `C4_IF_RTTI(code_with_rtti, code_without_rtti)`: select statements for rtti enabled/disabled
  - `C4_IF_RTTI_(code_with_rtti, code_without_rtti)`: select code tokens for rtti enabled/disabled
- [PR#109](https://github.com/biojppm/c4core/pull/109): Add partial support for XTENSA processors (missing implementation of `c4::aalloc()`). See [rapidyaml#358](https://github.com/biojppm/rapidyaml/issues/358).



### Fixes

- [PR#115](https://github.com/biojppm/c4core/pull/115) - Refactor of `c4::blob`/`c4::cblob`. Use SFINAE to invalidate some of the constructors.
- [PR#110](https://github.com/biojppm/c4core/pull/110)/[PR#107](https://github.com/biojppm/c4core/pull/107) - Update fast_float.
- [PR#108](https://github.com/biojppm/c4core/pull/108) - Fix preprocessor concatenation of strings in `C4_NOT_IMPLEMENTED_MSG()` and `C4_NOT_IMPLEMENTED_IF_MSG()`.
- [PR#106](https://github.com/biojppm/c4core/pull/106) - Fix include guard in the gcc 4.8 compatibility header, causing it to be missing from the amalgamated header.
- [PR#123](https://github.com/biojppm/c4core/pull/123) - Ensure the gcc 4.8 compatibility header is installed (fixes [#103](https://github.com/biojppm/c4core/issues/103)).
- [PR#105](https://github.com/biojppm/c4core/pull/105) - Fix existing throw in `c4/ext/sg14/inplace_function.h`. Ensure tests run with exceptions disabled and RTTI disabled. Add examples of exceptional control flow with `setjmp()/std::longjmp()`.
- [PR#104](https://github.com/biojppm/c4core/pull/104)/[PR#112](https://github.com/biojppm/c4core/pull/112) - Fix pedantic warnings in gcc, clang and MSVC
- [PR#104](https://github.com/biojppm/c4core/pull/104) - Fix possible compile error when `__GNUC__` is not defined
- Inject explicit `#include <charconv>` on the amalgamated header. The amalgamation tool was filtering all prior includes, thus causing a compilation error. Addresses [rapidyaml#364](https://github.com/biojppm/rapidyaml/issues/364).
- [PR#117](https://github.com/biojppm/c4core/pull/117): Windows: fix compilation with MSVC/clang++.
- Windows: add missing `C4CORE_EXPORT` to `c4::base64_valid()`, `c4::base64_encode()` and `c4::base64_decode()`.
