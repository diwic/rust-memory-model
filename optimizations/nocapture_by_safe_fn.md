### Example

```rust
fn escape_simple(x: &i32) -> usize {
    22
}

fn escape_as_usize(x: &i32) -> usize {
    x as usize
}

fn escape_as_ptr(x: &i32) -> *const i32 {
    x
}

pub struct MyType(*const i32);

fn escape_as_newtype(x: &i32) -> MyType {
    MyType(x)
}
```

### Explanation

To be very LLVM specific, we'd like to add "nocapture" attributes.
More generally, we'd like to be able to assume that if you have an
argument of type `fn foo<'a>(x: &'a T) -> U`, and `'a` (nor any
lifetime related to `'a`) does not appear in `U`, then we can assume
that no alias of `x` escapes `foo`. However, it's not clear that we
can always assume that.

- `escape_simple` -- no capture would be appropriate
- `escape_as_newtype` -- same types as previous example, but in this case
  one might imagine that the user expects to coerce the result back to
  a pointer and use it. Is that legal?
- `escape_as_ptr` -- same as above, but they were more explicit in their types.
- `escape_as_newtype` -- same as above, but there is a newtype wrapping this
  `*const` -- maybe it forms a kind of abstraction boundary?
  
### Source

IRC conversation: https://botbot.me/mozilla/rustc/2016-02-24/?msg=60823650&page=1