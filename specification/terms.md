Some terms, defined by @ubsan and @arielb1. Please debate on them, or correct them, as you see fit :)

## §1 Types

A type is a construct in Rust which defines certain parameters, and which turns an array of bytes into a value.

A type may have type, lifetime, or (in the future) constant parameters. If any of these parameters are generic, then the type is a "generic type". Otherwise, the type is a "concrete type".

A type is any of the following:

### §1.1 Primitives

#### §1.1.1 Numeric types

 * `u8`
 * `u16`
 * `u32`
 * `u64`
 * `usize`
 * `i8`
 * `i16`
 * `i32`
 * `i64`
 * `isize`
 * `f32`
 * `f64`

#### §1.1.2 Pointers

 * `<T> *const T`
 * `<T> *mut T`
 * `<'a, T> &'a T`
 * `<'a, T> &'a mut T`

#### §1.1.3 Arrays

`<T, const N: usize> [T; N]`

#### §1.1.4 Slices

 * `str`
 * `<T> [T]`

#### §1.1.5 Tuples

 * `<T> (T,)`
 * `<T, U> (T, U)`
 ...

#### §1.1.6 Function Pointers

 * `fn()`
 * `<T, U> fn(T) -> U`
 ...


### §1.2 `struct`s

(TODO)

### §1.3 `enum`s

(TODO)

## §2 Expressions

### §2.1 Location Expressions (lexprs)

A typed expression that evaluates to an object, as opposed to a value. The LHS of an assignment and the examinee of a pattern are examples of lexprs.

If a vexpr is used where an lexpr is expected (other than the LHS of an assignment, where using a vexpr is illegal), it is wrapped in an implicit temporary lexpr, the rules of which are given in §2.3.

### §2.2 Value Expression (vexpr)

A typed expression that evaluates to an value, as opposed to an object. The RHS of an assignment and the arguments of a function are examples of vexprs.

If a lexpr is used where an vexpr is expected, the memory is read from the object referred to by the lexpr to create a vexpr *if* either T: Copy, or the mutability of the lexpr allows for moving.

### §2.3 Temporary lexpr

An implicit lexpr that wraps a vexpr when an lexpr is expected.

#### §2.3.1 The reference operator on the RHS of let statements
(TODO: figure out the rule)

This case only exists when, on the right hand side of a let statement, a reference operator (`&`, `&mut`) takes the address of a vexpr. I don't know the rule. ping @nikomatsakis to actually say the rule.

#### §2.3.2 Other temporary lexprs

 When evaluated, it:

 * creates a temporary object, scheduled for destruction at the end of the statement.
 * evaluates the vexpr and stores it within the object.
 * evaluates the lexpr of the temporary object.

A common example occurs when the result of `format!` is borrowed - it is wrapped by an implicit temporary lexpr.

```Rust
fn send(message: &str) {
    /* .. */
}

fn main() {
    let message = &format!("1+1={}", 1+1);
    send(message);
}
```

### §2.4 Note on the terms lvalue and rvalue

These terms aren't used in this document, because they are used in conflicting ways in various places. However, lvalue is equivalent to lexpr, while rvalue is equivalent to vexpr.

## §3 Objects

An object is an untyped, contiguous block of memory with a certain size and a certain alignment.
 
### §3.1 Capabilities

All objects are able to be read, but you may not write to all objects.

A lexpr associated with an object, as a vexpr, will evaluate to a value (see §4) that is equivalent to the memory of the object, interpreted as the type of the lexpr.

A write to an object will set the memory of that object equivalent to the value of the write, without the interpretation of typing.

### §3.2 Allocation.

An object may be allocated in one of three ways:

In static memory:

```rust
static x: i32 = 0i32; // a static, read-only object of size >= 4, alignment >= 4
static mut x: i32 = 0i32; // a static, read-write object of size >= 4, alignment >= 4
static x: AtomicI32 = 0i32; // a static, read-write object of size >= 4, alignment >= 4
```

on the stack:

```rust
let x: i32 = 0i32; // a stack, read-only object of size >= 4, alignment >= 4
// you get the point
```

or on the heap:

```rust
let x: Box<i32> = Box::new(0i32); // a stack, read-only object pointing to a heap,
                                  // read-write object of size >= 4, alignment >= 4
```

### §3.3 Note on types:

Objects *are not* typed. No, not even effective types ^-^

## §4 Value

A value is a specific inhabitant of a type. Values are pure mathematical objects, and do not live in memory. They only exist to be stored in objects, and to be evaluated.

### §4.1 Examples

```rust
true
[0, 0, 0, 0]
0xcccccccc as *const u32
{
    let foo: u32 = 0;
    &foo // <-
}
Some((4u32, 2usize))
```

### §4.2 Undef

A special case of any value is "undef". Any type may take on the value "undef", excepting zero sized types. Evaluating an "undef" value from a lexpr is Undefined Behavior, unless one is reading undef bytes from memory into a struct's padding bytes. The act of storing "undef" is not UB, only reading from an lexpr.

### §4.3 Conversion to Memory

A value, in order to get stored into memory, must first undergo the process of transformation from pure mathematical object into a computer-friendly form. Memory shall be represented as a byte array. Integers will be converted to base-2, 2's complement; floats shall be converted to IEEE754; etc. Any padding shall take on the value "undef".

### §4.4 Conversion from Memory

Conversion from memory will be a conversion to memory, but in reverse, going from computer-friendly form to pure mathematical object.
