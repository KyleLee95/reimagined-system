# Group Theory -> Functional Programming

Associative Property ensures:

- Predictability of output
- Type safety

## Monoid

- A structure with:
  - an associate binary operation
  - identity element

## Group

- A Monoid that also has the additional property of having an **inverse element** for each element

## Semigroup

- Similar to a Monoid but does not necessarily have an identity element.

```haskell
-- Monoids are implemented in Haskell as an extension of a semigroup

class Semigroup a => Monoid a where
  mempty :: a                     -- Identity element
  mappend :: a -> a -> a          -- Binary operation
  mconcat :: [a] -> a             -- Fold (apply mappend over a list)

  mconcat = foldr mappend mempty



```

## Real Software Example: Payments

Defining a sum type for payments. As in, what primitive types can Payment take on.

```haskell
data Payment
  = Cash Double
  | CreditCard String Double
  | BankTransfer { account :: String, amount :: Double }
  | None -- Identity Element
  deriving (Show, Eq)
```

`<>` is an alias for the `mappend` operation defined above.

We now see that the type of `Payment` is useful when defining how to aggregate
payments.

### Monoid

```haskell
instance Monoid Payment where
  -- identity law.
  mempty = None
  mappend = (<>)
```

### Semigroup

```haskell

instance Semigroup Payment where
-- Defining how mappend should be handled between None and some type 'b'/'a'
  None <> b = b
  a <> None = a

-- Defining how mappend should be handled for the defined types of Payments
  (Cash x) <> (Cash y) = Cash (x + y)
  (CreditCard card x) <> (CreditCard _ y) = CreditCard card (x + y)
  (BankTransfer acc x) <> (BankTransfer _ y) = BankTransfer acc (x + y)

-- Payments of different types cannot be combined directly
  a <> _ = a
```

How does this preserve the identity law?

```haskell

-- Cash payments only

(Cash x <> Cash y) <> Cash z = Cash (x + y) <> Cash z = Cash (x + y + z)
Cash x <> (Cash y <> Cash z) = Cash x <> Cash (y + z) = Cash (x + y + z)


-- None: Cash is always preserved
(None <> Cash 10) <> Cash 20 == Cash 10 <> Cash 20
None <> (Cash 10 <> Cash 20) == Cash 10 <> Cash 20


-- Mixed payment types: The first element (in this case Cash) is always preserved
(Cash 10 <> CreditCard "1234" 20) <> BankTransfer "Account" 30
= Cash 10 <> BankTransfer "Account" 30
= Cash 10

Cash 10 <> (CreditCard "1234" 20 <> BankTransfer "Account" 30)
= Cash 10 <> CreditCard "1234" 20
= Cash 10
```

### Putting it to use:

Define a function aggregates payments by type

```haskell
processPayment :: Payment -> String -- function definition
--haskell uses "pattern matching" to match input cases (types) to operations
--pattern matching based on types is how haskell ensures type safety
processPayment (Cash amount) = "Paid in cash: $" ++ show amount
processPayment (CreditCard card amount) = "Paid $" ++ show amount ++ " with card: " ++ card
processPayment (BankTransfer account amount) = "Transferred $" ++ show amount ++ " to account: " ++ account
processPayment None = "No payment made"
```

## Group-like function

"inverse" would be to refund the amount

```haskell

inversePayment :: Payment -> Payment
inversePayment (Cash amount) = Cash (-amount)
inversePayment (CreditCard card amount) = CreditCard card (-amount)
inversePayment (BankTransfer acc amount) = BankTransfer acc (-amount)
inversePayment None = None
```
