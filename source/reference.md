# Reference

Header: `leaf_spy.hpp`. Namespace: `leaf_spy`.

## try_catch

```cpp
template <typename SpyHandler, typename ... DirectHandlers>
void try_catch(
    const leaf_spy_args_t& args,
    SpyHandler&& spy_handler,
    DirectHandlers&& ... direct_handlers
);
```

Runs `args.job_that_may_throw()` inside a leaf context. If the job throws:

1. `args.notify_that_job_executed_with_throw()` is called.
2. The direct handlers try first.
3. If none matched, `spy_handler.visit(...)` is called.
4. Whether anything leaves depends on the outcome and on `args.is_throw_out`. See [Direct handlers](handlers.md).

If the job does not throw, `args.notify_that_job_executed_without_throw()` is called.

A `captured_leaf_exception` thrown by the job is unpacked first, so its marks reach the handlers. See [Threads](threads.md).

## leaf_spy_args_t

An aggregate. Initialise it with designated initialisers.

| member | type | meaning |
| --- | --- | --- |
| `job_that_may_throw` | `const std::function<void ()>&` | the work that may throw |
| `notify_that_job_executed_with_throw` | `const std::function<void ()>&` | called if the job threw |
| `notify_that_job_executed_without_throw` | `const std::function<void ()>&` | called if the job did not throw |
| `is_throw_out` | `const bool`, default `true` | let an exception leave `try_catch` |

The first three are references. The functions they refer to must outlive the call. A lambda passed inline lives until the end of the full expression, which is enough.

## SpyHandlerConcept

```cpp
template <typename SpyHandler, typename Context>
concept SpyHandlerConcept;
```

A spy handler satisfies the concept when:

- every parameter of its `operator()` is a pointer, and
- `spy_handler.visit(ctx, id, exception)` is a valid call, with `ctx` of the context type, `id` a `leaf::error_id` and `exception` a `std::exception*`.

`try_catch` checks it with `static_assert`. See [The spy handler](spy.md).

## captured_leaf_exception

Header: `captured_leaf_exception.hpp`.

```cpp
struct captured_leaf_exception : std::exception
{
    std::shared_ptr<leaf::result<void>> result;

    explicit captured_leaf_exception(leaf::result<void> result);
    const char* what() const noexcept override;
};
```

Carries the marks of an exception from the thread that threw to the thread that catches. `what()` returns `"captured leaf exception"`. See [Threads](threads.md).

## CMake

| item | meaning |
| --- | --- |
| `leaf_spy::leaf_spy` | the `INTERFACE` target, C++20, links `Boost::leaf` |
| `BUILD_TESTS` | `ON` by default. Builds `leaf_spy_tests` |

When `Boost::leaf` is not a target yet, the library fetches `boostorg/leaf` `boost-1.92.0`.
