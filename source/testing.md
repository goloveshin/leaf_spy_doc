# Testing

Two things are worth testing: that your code throws the right marks, and that handlers react as you expect. Both use `try_catch` with a small spy that records what it saw.

The unit tests of the library use [doctest](https://github.com/doctest/doctest), but nothing here depends on it.

## A recording spy

```cpp
struct mark_a { int value = 0; };
struct mark_b { int value = 0; };

// a type the exception never has: a handler for it never matches
struct bottom_skip_t{};

struct seen_marks_t
{
    bool saw_a = false;
    int a = 0;
};

struct bind_t
{
    seen_marks_t* marks = nullptr;

    void operator()(const mark_a*, const mark_b*) const
    {
    }

    template <typename Context>
    void visit(Context& ctx, const leaf::error_id id, std::exception*) const
    {
        if (const auto* a = ctx.template get<mark_a>(id))
        {
            marks->saw_a = true;
            marks->a = a->value;
        }
    }
};
```

## A helper for the arguments

```cpp
leaf_spy_args_t make_args(
    const leaf_spy_args_t::job_that_may_throw_ft& job,
    const bool is_throw_out
)
{
    return leaf_spy_args_t{
        .job_that_may_throw = job,
        .notify_that_job_executed_with_throw = on_throw,
        .notify_that_job_executed_without_throw = on_ok,
        .is_throw_out = is_throw_out
    };
}
```

`on_throw` and `on_ok` are `const` empty functions of the matching types. They must outlive the call, because the arguments hold references.

## A test

```cpp
TEST_CASE("is_throw_out false swallows when no direct handler matches")
{
    seen_marks_t marks;

    const leaf_spy_args_t::job_that_may_throw_ft job = []
    {
        leaf::throw_exception(std::runtime_error("job"), mark_a{ 11 }, mark_b{ 22 });
    };

    CHECK_NOTHROW((try_catch(
        make_args(job, false),
        bind_t{ &marks },
        [](const bottom_skip_t&){}
    )));

    CHECK(marks.saw_a);
    CHECK(marks.a == 11);
}
```

Wrap the call in an extra pair of parentheses inside `CHECK_NOTHROW` and `CHECK_THROWS`. The macro would otherwise split it at the commas.

## What the library tests cover

The tests in `tests/test_leaf_spy.cpp` cover each row of the table in [Direct handlers](handlers.md):

- no handler matches, with `is_throw_out` true and false
- a handler returns
- a handler does `throw;`
- a handler throws `std::runtime_error`, with `is_throw_out` true and false
- two levels of `try_catch`, where the upper one catches what the lower one rethrew
