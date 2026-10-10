# The spy handler

The spy is the object that reads the marks of an exception that nobody handled. It is the second argument of `try_catch`.

A spy has two members.

## `operator()`: which marks to keep

```cpp
void operator()(
    const exception::domain::library_A*,
    const exception::domain::library_B*,
    const exception::subdomain::library_A::part_1*,
    const exception::e_what_info*
) const
{
}
```

The body is empty. Only the parameter list matters: Boost.LEAF looks at it and reserves a slot for every type in it. A mark that is not listed here is not kept, and `visit` will not find it.

Every parameter must be a pointer. `try_catch` checks this at compile time with `SpyHandlerConcept`.

## `visit`: read them

```cpp
template <typename Context>
void visit(
    Context& ctx,
    const leaf::error_id id,
    std::exception* exception
) const
```

`ctx.template get<T>(id)` returns a pointer to the mark of type `T`, or `nullptr` if the exception does not have it.

```cpp
const auto* domain_a = ctx.template get<exception::domain::library_A>(id);
if (domain_a)
{
    ss << "domain: library_A\n";
}
```

`exception` is the caught `std::exception`. It is `nullptr` if the thrown object was not derived from `std::exception`.

## What a spy usually does

A spy for a project built with domains and subdomains (see [A typical project](project.md)) usually does four things:

1. prints the `std::exception` text
2. prints every domain and subdomain it finds, and an error line if there is none of one kind
3. prints the payloads, such as the `e_what_info` text
4. prints Boost.LEAF's own diagnostic info

The check of the agreement "an exception must contain a domain and a subdomain" lives here. It is the pointer handler from [Catching any set of types](any-set.md), written once:

```cpp
const auto* domain_a = ctx.template get<exception::domain::library_A>(id);
const auto* domain_b = ctx.template get<exception::domain::library_B>(id);
const auto* domain_c = ctx.template get<exception::domain::library_C>(id);

if (!domain_a && !domain_b && !domain_c)
{
    out << "ERROR: no domain in exception!\n";
}
```

## Output goes where you choose

The spy writes where you want. It can hold a `std::ostringstream` by reference and let you print it later, write to `std::cout`, or send the text to a log.

## One spy per project

All spies in a program usually share the same list of marks, so declare one and reuse it. The unit tests use a small one of their own, see [Testing](testing.md).
