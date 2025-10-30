# Syntax

<https://doc.rust-lang.org/reference/patterns.html#identifier-patterns>

* By default, identifier patterns bind a variable to a copy of or move from the
  matched value depending on whether the matched value implements `Copy`. This
  can be changed to bind to a reference by using the `ref` keyword, or to a
  mutable reference using `ref mut`.

```
match a {
    None => (),
    Some(value) => (),
}

match a {
    None => (),
    Some(ref value) => (),
}
```

* `ref` is not something that is being matched against. Its objective is
  exclusively to make the matched binding a reference, instead of potentially
  copying or moving what was matched.


<https://doc.rust-lang.org/reference/patterns.html#binding-modes>:

* Each time a reference is matched using a non-reference pattern, it will
  automatically dereference the value and update the default binding mode.

* When a reference value is matched by a non-reference pattern, it will be
  automatically treated as a `ref` or `ref mut` binding. 

```
let x: &Option<i32> = &Some(3);
if let Some(y) = x {
    // x was automatically derefrenced
    // y was converted to `ref y` and its type is &i32
}
```

<https://doc.rust-lang.org/reference/expressions.html#place-expressions-and-value-expressions>

Expressions are divided into two main categories: place expressions and value expressions:

* A place expression is an expression that represents a memory location.

* A value expression is an expression that represents an actual value.

The following contexts are place expression contexts:

  - The left operand of a [compound assignment](https://doc.rust-lang.org/reference/expressions/operator-expr.html#compound-assignment-expressions) expression.
  - The operand of a unary [borrow](https://doc.rust-lang.org/reference/expressions/operator-expr.html#borrow-operators), [raw borrow](https://doc.rust-lang.org/reference/expressions/operator-expr.html#raw-borrow-operators) or [dereference](https://doc.rust-lang.org/reference/expressions/operator-expr.html#the-dereference-operator) operator.
  - The operand of a field expression.
  - The indexed operand of an array indexing expression.
  - The operand of any implicit borrow.
  - The initializer of a let statement.
  - The scrutinee of an if let, match, or while let expression.
  - The base of a functional update struct expression.

When a place expression is evaluated in a value expression context, or is bound
by value in a pattern, it denotes the value held in that memory location. If the
type of that value implements Copy, then the value will be copied. In the
remaining situations, if that type is Sized, then it may be possible to move the
value. After moving out of a place expression that evaluates to a local
variable, the location is deinitialized and cannot be read from again until it
is reinitialized.

<https://doc.rust-lang.org/reference/expressions/operator-expr.html#the-dereference-operator>

* The `*` (dereference) operator is also a unary prefix operator.

* When applied to a [pointer](https://doc.rust-lang.org/reference/types/pointer.html) it denotes the pointed-to location.

* If the expression is of type &mut T or *mut T, and is either a local variable, a (nested) field of a local variable or is a mutable place expression, then the resulting memory location can be assigned to.

<https://doc.rust-lang.org/nomicon/dot-operator.html>

