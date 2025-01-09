# Group Theory -> Functional Programming: A story of Algebraic Structures and Type Safety

## Monoid

- A structure with:
  - an associate binary operation
  - identity element

## Group

- A Monoid that also has the additional property of having an **inverse element** for each element

## Semigroup

- Similar to a Monoid but does not necessarily have an identity element.

Implementation of the above in `Rust`

```rust
pub trait Semigroup {
    fn combine(self, other: Self) -> Self;
}

pub trait Monoid: Semigroup {
    fn empty() -> Self;
}

```

## Real Software Example: Payments

```rust
//definition of the types of payments
enum Payment {
    Cash(f64),
    CreditCard(String, f64),
    None,
}
```

The above implements a type for `Payments` composed of different primitives
(Cash, Credit Card, or None). Combining data types to form a new type is an example of Algebraic Data types at work.

The concept of a **semigroup** can be applied to aggregating payments.

```rust
impl Semigroup for Payment {
    fn combine(self, other: Self) -> Self {
        match (self, other) {
            (Payment::Cash(a), Payment::Cash(b)) => Payment::Cash(a + b),
            (Payment::CreditCard(card, a), Payment::CreditCard(_, b)) => Payment::CreditCard(card, a + b),
            _ => self,  // Non-matching types cannot combine directly
        }
    }
}
```

- Payments of different types don't combine
- Associative property holds true (A + B) + C = a + (b+c)

```rust
impl Payment {
    fn inverse(self) -> Self {
        match self {
            Payment::Cash(a) => Payment::Cash(-a),
            Payment::CreditCard(card, a) => Payment::CreditCard(card, -a),
            _ => Payment::None,
        }
    }
}


```
