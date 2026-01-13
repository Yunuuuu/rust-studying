# Syntax

## Namespace
A
[namespace](https://doc.rust-lang.org/reference/names/namespaces.html#namespaces)
is a logical grouping of declared names. Names are segregated into separate
namespaces based on the kind of entity the name refers to. Namespaces allow the
occurrence of a name in one namespace to not conflict with the same name in
another namespace.

## Patterns
[identifier
patterns](https://doc.rust-lang.org/reference/patterns.html#identifier-patterns)
bind the value they match to a variable in the [value
namespace](https://doc.rust-lang.org/reference/names/namespaces.html#r-names.namespaces.kinds).

The variable will **shadow** any variables of the same name in scope. The scope
of the new binding depends on the context of where the pattern is used (such as
a `let` binding or a `match` arm).

By default, [identifier
patterns](https://doc.rust-lang.org/reference/patterns.html#identifier-patterns)
bind a variable to a copy of or move from the matched value depending on whether
the matched value implements
[Copy](https://doc.rust-lang.org/reference/special-types-and-traits.html#copy).
This can be changed to bind to a reference by using the `ref` keyword, or to a
mutable reference using `ref mut`.

```rust,ignore
match a {
    None => (),
    Some(value) => (),
}

match a {
    None => (),
    Some(ref value) => (),
}
```

In the first match expression, the value is copied (or moved). In the second
match, a reference to the same memory location is bound to the variable value.
This syntax is needed because in destructuring subpatterns the `&` operator
can't be applied to the value's fields.

`ref` is not something that is being matched against. Its objective is
exclusively to make the matched binding a reference, instead of potentially
copying or moving what was matched.

To service better ergonomics, patterns operate in different [binding
modes](<https://doc.rust-lang.org/reference/patterns.html#binding-modes>) in
order to make it easier to bind references to values. 

* Each time a reference is matched using a non-reference pattern, it will
  automatically dereference the value and update the default binding mode.

* When a reference value is matched by a non-reference pattern, it will be
  automatically treated as a `ref` or `ref mut` binding. 

```rust
let x: &Option<i32> = &Some(3);
if let Some(y) = x {
    // x was automatically derefrenced
    // y was converted to `ref y` and its type is &i32
}
```

## Expressions
[Expressions](<https://doc.rust-lang.org/reference/expressions.html#place-expressions-and-value-expressions>)
are divided into two main categories: place expressions and value expressions:

* A place expression is an expression that represents a memory location.

* A value expression is an expression that represents an actual value.

The following contexts are place expression contexts:

  - The left operand of a [compound assignment](https://doc.rust-lang.org/reference/expressions/operator-expr.html#compound-assignment-expressions) expression.
  - The operand of a unary [borrow](https://doc.rust-lang.org/reference/expressions/operator-expr.html#borrow-operators), [raw borrow](https://doc.rust-lang.org/reference/expressions/operator-expr.html#raw-borrow-operators) or [dereference](https://doc.rust-lang.org/reference/expressions/operator-expr.html#the-dereference-operator) operator.
  - The operand of a field expression.
  - The indexed operand of an array indexing expression.
  - The operand of any implicit borrow.
  - The initializer of a let statement.
  - The [scrutinee](https://doc.rust-lang.org/reference/glossary.html#scrutinee) of an if let, match, or while let expression.
  - The base of a functional update struct expression.

When a place expression is evaluated in a value expression context, or is bound
by value in a pattern, it denotes the value held in that memory location. If the
type of that value implements Copy, then the value will be copied. In the
remaining situations, if that type is Sized, then it may be possible to move the
value. After moving out of a place expression that evaluates to a local
variable, the location is deinitialized and cannot be read from again until it
is reinitialized.

## Dereference
The `*`
([dereference](<https://doc.rust-lang.org/reference/expressions/operator-expr.html#the-dereference-operator>))
operator is also a unary prefix operator. 

* When applied to a
  [pointer](https://doc.rust-lang.org/reference/types/pointer.html) it denotes
  the pointed-to location.

* If the expression is of type `&mut T` or `*mut T`, and is either a local
  variable, a (nested) field of a local variable or is a mutable place
  expression, then the resulting memory location can be assigned to.

## Dot Operator
The [dot operator](<https://doc.rust-lang.org/nomicon/dot-operator.html>) will
perform a lot of magic to convert types. It will perform auto-referencing,
auto-dereferencing, and coercion until types match. The detailed mechanics of
method lookup are defined
[here][https://rustc-dev-guide.rust-lang.org/hir-typeck/method-lookup.html#method-lookup],
but here is a brief overview that outlines the main steps.

Suppose we have a function `foo` that has a receiver (a `self`, `&self` or `&mut
self` parameter). If we call `value.foo()`, the compiler needs to determine what
type `Self` is before it can call the correct implementation of the function.
For this example, we will say that `value` has type `T`.

We will use `fully-qualified syntax` to be more clear about exactly which type
we are calling a function on.

- First, the compiler checks if it can call `T::foo(value)` directly. This is
  called a "by value" method call.
- If it can't call this function (for example, if the function has the wrong
  type or a trait isn't implemented for `Self`), then the compiler tries to add
  in an automatic reference. This means that the compiler tries
  `<&T>::foo(value)` and `<&mut T>::foo(value)`. This is called an "autoref"
  method call.
- If none of these candidates worked, it dereferences `T` and tries again. This
  uses the `Deref` trait - if `T: Deref<Target = U>` then it tries again with
  type `U` instead of `T`. If it can't dereference `T`, it can also try
  _unsizing_ `T`. This just means that if `T` has a size parameter known at
  compile time, it "forgets" it for the purpose of resolving methods. For
  instance, this unsizing step can convert `[i32; 2]` into `[i32]` by
  "forgetting" the size of the array.
