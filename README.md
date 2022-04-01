# Checked Math

A convenience macro for changing the checking properties of math expressions without having to change those expressions (mostly).

## Some Background

By default Rust's math expressions (e.g., 2 * 9) panic on overflow in debug builds and overflow silently in release builds. While this is a fine default, you may want different behaivor. For instance, you may want to panic on overflow even in release builds. In that case you'll want to reach for the various APIs available for Rust numbers (e.g., [`checked_mul`](https://doc.rust-lang.org/std/primitive.u32.html#method.checked_mul)). The issue with this however, is that you can no longer use normal math notation, instead needing to use cumbersome methods like `a.checked_add(b).unwrap()` instead of `a + b`.

Checked Math lets you keep using normal math notation by performing the rewrite in a proc macro.

## Example

By default the following may silently overflow in release builds at runtime:

```rust
(x * y) + z
```

In order to to panic on error instead, people often rewrite to:

```rust
x.checked_mul(y).unwrap().checked_add(z).unwrap()
```

Or you could use Checked Math and write the following:

```rust
use checked_math::checked_math as cm;
cm!((x * y) + z).unwrap()
```

The macro call converts the normal math expression into an expression returning `None` if any of the checked math steps return `None`, and `Some(_)` on success.

Projects may want to wrap the rewriting macro to automatically include their error
reporting, for example:
```rust
#[macro_export]
macro_rules! my_checked_math {
    ($x: expr) => {
        checked_math::checked_math!($x).unwrap_or_else(|| panic!("math error"))
    };
}
```

## Limitations

Checked Math is currently limited in the following:

* Because math operations can more easily propogate type inference information than method calls, you may have to add type information when using the macros that were not neceesary before.
* The syntax the macro accepts is intentionally limited to binary expressions, parentheses, argument-less calls and some extras. The background is that it would be confusing if inner expressions like in `checked_math!(foo(a + b))` or `checked_math!(if a + b { b + c } else { d })` were silently not transformed.
* Conversion of `pow` is extremely naive so if you call a `pow` method on some type, this will be converted to `checked_pow` even if that makes no sense for that type.

## History

This is a modified version of `overflow` from https://github.com/rylev/overflow/ originally by Ryan Levick.
See LICENSE.
