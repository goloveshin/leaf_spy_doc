# Getting started

## Requirements

- C++20 (concepts and fold expressions)
- Boost.LEAF. If the target `Boost::leaf` does not exist when the library is configured, the library fetches `boostorg/leaf` itself.

The library is header-only. The only target is the `INTERFACE` library `leaf_spy::leaf_spy`.

## CMake

The library lives at [github.com/goloveshin/leaf_spy](https://github.com/goloveshin/leaf_spy). Fetch it with `FetchContent`:

```cmake
include(FetchContent)

FetchContent_Declare(
    leaf_spy
    GIT_REPOSITORY https://github.com/goloveshin/leaf_spy.git
    GIT_TAG main
)
FetchContent_MakeAvailable(leaf_spy)

target_link_libraries(my_app PRIVATE leaf_spy::leaf_spy)
```

Replace `main` with a release tag to pin a version. To use a local copy instead, replace `GIT_REPOSITORY` and `GIT_TAG` with `SOURCE_DIR <path>`.

The library has an option `LEAF_SPY_BUILD_TESTS`. It builds the unit tests and the example of the library. It is `ON` only when `leaf_spy` is the top-level project, so a project that uses the library does not build them. You can set it yourself before `FetchContent_MakeAvailable`:

```cmake
set(LEAF_SPY_BUILD_TESTS OFF)
```

## The first program

```cpp
#include <leaf_spy.hpp>

#include <boost/leaf/exception.hpp>

#include <iostream>
#include <string>

namespace leaf = boost::leaf;


struct library_A{};

struct part_1{};

struct e_what_info
{
    std::string what;
};

struct spy_t
{
    // the slot types the context keeps
    void operator()(const library_A*, const part_1*, const e_what_info*) const
    {
    }

    // reads the slots of the exception
    template <typename Context>
    void visit(Context& ctx, const leaf::error_id id, std::exception*) const
    {
        if (ctx.template get<library_A>(id))
        {
            std::cout << "domain: library_A\n";
        }
        if (ctx.template get<part_1>(id))
        {
            std::cout << "subdomain: part_1\n";
        }
        if (const auto* info = ctx.template get<e_what_info>(id))
        {
            std::cout << "what: " << info->what << '\n';
        }
    }
};

int main()
{
    leaf_spy::try_catch(
        {
            .job_that_may_throw = []
            {
                leaf::throw_exception(library_A{}, part_1{}, e_what_info{ "disk is full" });
            },
            .notify_that_job_executed_with_throw = []{},
            .notify_that_job_executed_without_throw = []{},
            .is_throw_out = false
        },
        spy_t{}
    );
}
```

Output:

```text
domain: library_A
subdomain: part_1
what: disk is full
```

Three parts, in the order they appear:

1. **Marks.** `library_A`, `part_1`, `e_what_info` are plain structs, as in [A typical project](project.md#declare-the-marks-in-one-header).
2. **Spy.** `spy_t` has an `operator()` that lists the marks to keep, and a `visit` that reads them. See [The spy handler](spy.md).
3. **`try_catch`.** It runs the job. Nothing handled the exception, so the spy runs. `is_throw_out = false` keeps the exception inside.
