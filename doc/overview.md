# Overview

Monty is a compiled, statically-typed, multi-paradigm programming language
primarily influenced by Python and Rust. It features significant whitespace,
reference types, duck typing, uniform function call syntax (UFCS), restricted
multiple dispatch, sum types, pattern matching, and garbage collection.

## Status

Monty is still being designed. There is currently no working compiler or
standard library.

# Example Code

## Hello, world!

```monty
fn main() -> Void:
    print("Hello, world!")
```

## FizzBuzz

```monty
use std.math

fn main() -> Void:
    for n in 1.up_to(100):
        match 3.divides(n), 5.divides(n):
            true, true => print("FizzBuzz")
            true, false => print("Fizz")
            false, true => print("Buzz")
            false, false => print(n)
```

## Fibonacci

```monty
fn fibonacci() -> Iter[Int]:
    var a, b = 0, 1
    while true:
        yield b
        a, b = b, a + b

fn main() -> Void:
    for n in fibonacci().take(10):
        print(n)
```

## Linked List

```monty
struct ListNode[T]:
    next: ListNode[T]?
    value: T

struct LinkedList[T]:
    var head: ListNode[T]? = None

    @static
    fn empty() -> Self:
        Self()

    fn push_front(value: T) -> Void:
        self.head = Some(ListNode(self.head, value))

    fn pop_front() -> T?:
        if self.head is Some(node):
            self.head = node.next
            Some(node.value)
        else:
            None
```

# Features

## Planned Features

### Attributes

Attributes, written `@attribute_name`, are used to add metadata to
syntax elements. For now they primarily serve to reduce the number of keywords
in the language.

```monty
trait Foo:
    @static
    fn bar(@kw baz: Int) -> Int
```

### Inline Tests

> 📝 Note: tests may also appear inside of functions.

```monty
fn foo(a: Int) -> Int
    a + 5

test foo_adds_5:
    expect foo(5) == 10
```

Attributes:

* `@manual`: do not run the test automatically.

### Constructors

Constructor syntax is used to create instances of types, and to pattern match
those types. Constructors cannot be overridden.

> 📝 Note: these examples all use tuple structs, which are structs with no
> named fields. Struct fields can be named and have default values.

```monty
struct Foo:
    Int
    Int

fn main() -> Void:
    let foo = Foo(1, 2)
    match foo:
        Foo(a, b) => print(a, b)  # prints "1 2"
```

Attributes:

* `@opaque`: Restrict construction, pattern matching, and field access to the
current module.

### Tuples

> 📝 Note: parentheses are usually optional around tuples.

```monty
fn div_rem(a: Int, b: Int) -> Int, Int:
    a / b, a % b

fn main() -> Void:
    let result = div_rem(35, 7)
    print("Quotient: ${result.0}")
    print("Remainder: ${result.1}")
```

### Operator Overloading

```monty
struct MyInt:
    Int

    @static
    fn add(a: Self, b: Self) -> Self:
        Self(a.0 + b.0)

fn main() -> Void:
    let a, b = MyInt(5), MyInt(10)
    print(a + b)  # prints "MyInt(15)"
```

### Implicit Conversion

Types may opt-in to automatic, infallible conversions.

```monty
struct MyInt:
    Int

    @static
    @implicit
    fn from(value: Int) -> Self:
        Self(value)

    @static
    fn add(a: Self, b: Self) -> Self:
        Self(a.0 + b.0)

fn main() -> Void:
    let a: MyInt = 5
    print(a + 10)  # prints "MyInt(15)"
```

### Duck Typing

```monty
struct MyInt:
    Int

    fn double() -> Self:
        Self(2 * self.0)

trait MyTrait:
    fn double() -> Self

fn foo(a: MyTrait) -> Void:
    print(a.double())

fn main() -> Void:
    foo(MyInt(5))  # prints "MyInt(10)"
```

### Equality

> 📝 Note: partial equality is implemented by `fn try_eq() Bool?`.

```monty
struct MyInt:
    Int

    @static
    fn eq(a: Self, b: Self) -> Bool:
        a.0 == b.0

fn main() -> Void:
    let a, b = MyInt(5), MyInt(10)
    if a == b:
        print("a equals b")
    else:
        print("a does not equal b")
```

### Comparisons

> 📝 Note: partial order is implemented by `fn try_cmp() Ord?`.

```monty
struct MyInt:
    Int

    @static
    fn cmp(a: Self, b: Self) -> Ord:
        match:
            if a.0 < b.0 => Lt
            if a.0 > b.0 => Gt
            _ => Eq

fn main() -> Void:
    let a, b = MyInt(5), MyInt(10)
    if a > b:
        print("a is greater than b")
    else:
        print("a is not greater than b")
```

### UFCS

```monty
fn foo(a: Int) -> Int:
    a + 5

fn main() -> Void:
    let x = 10.foo()
    print(x)  # prints "15"

```

### Sum Types

```monty
enum Foo:
    Bar(Int)
    Baz(Int, Int)

fn main() -> Void:
    let x = Foo.Bar(5)
    match x:
        Bar(y) => print(y)
        Baz(a, b) => print(a, b)
```

### Error Handling

> 📝 Note: the type `Option[T]` may be written `T?` and `Result[V, E]` may be
> written `V ? E`.

The try operator `?` can be used to unwrap values, returning errors immediately.

```monty
fn safe_div(dividend: Int, divisor: Int) -> Int ? Str:
    if divisor == 0:
        return Err("division by zero")
    if divisor == -1 and dividend == Int.min_value:
        return Err("integer overflow")
    Ok(dividend / divisor)

fn foo() -> Int ? Str:
    Ok(5.safe_div(3)?.safe_div(0)?)
```

### Generics

```monty
fn sum[V -> Acc: Zero + AddAssign[V] = V](it: Iter[V]) -> Acc:
    var result = Acc.zero()
    for value in it:
        result += value
    result

fn main() -> Void:
    let a = List[Int32].from(1.up_to(10))
    let x = a.sum()         # x has type Int32
    let y = a.sum[Int64]()  # y has type Int64
```

Let's break this down.

* `V`: The type being summed.
* `->`: A separator. Calls match separators from the right, so `sum[Int32]`
binds `Acc` to `Int32`, while `sum[Int32 ->]` binds `V`. Typically inferrable
types are placed on the left while required types are placed on the right.
* `Acc`: The result (accumulator) type.
* `: Zero + AddAssign[V]`: `Acc` must implement the traits `Zero` and
`AddAssign[V]`.
* `= V`: If not specified, the default value of `Acc` is `V`.

### Closures

```monty
fn foo(bar: Fn[Int -> Int]) -> Int:
    bar(5)

fn baz() -> Void:
    let a = 10

    fn add_a_1(b: Int) -> Int:
        a + b

    let add_a_2 = |b: Int| a + b

    print(foo(add_a_1))  # prints "15"
    print(foo(add_a_2))  # prints "15"
```

## Hopeful Features

* Range syntax
* Bitfields
* Threads
* Async (via stackful coroutines)
* List, map, and set literals
* Bidirectional type inference
* Arrays
* SIMD
* CTFE
