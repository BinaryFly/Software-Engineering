# Questions

## _1_

Given a set `M` with an operation `<+>` and two elements `x` and `y` from `M`, what are the properties that must be satisfied by `M` to have the structure of Monoid?

**A**:

1. There must be an identity element `e` such that `x <+> y = e`
2. `(x <+> y) <+> z = x <+> (y <+> z)`

**B**:

1. There must be an identity element `e` such that `x <+> y = e`
2. `x <+> y = y <+> x`

**C**:

1. There must be an identity element `e` such that `e <+> x = x <+> e = x`
2. `(x <+> y) <+> z = x <+> (y <+> z)`

**D**:

1. There must be an identity element `e` such that `e <+> x = x <+> e = e`
2. `x <+> y = y <+> x`

<details><summary>Show Answer</summary>
<p>

```
C
```

</p>
</details>
<br>
<hr>

## _2_

Consider the type `String`, an operation `invertAdd(s1, s2)` that **first** inverts each string and **then** appends the second string to the first, and the `element e = ""` (empty string).

Which of the following statements is false?

**A**: The element `e` is a valid identity for `String`

**B**: `invertAdd` is not commutative, that is

```ts
invertAdd(s1, s2) != invertAdd(s2, s1);
```

**C**: The operation `invertAdd` is not associative

**D**: `String` with the operation `invertAdd` and `e` as identity is a valid Monoid

<details><summary>Show Answer</summary>
<p>

```
D
```

</p>
</details>
<br>
<hr>

## _3_

Given a monoid `Monoid<a>`, what is the correct definition of its `join_Monoid` function?

**A**:

```ts
let join_Monoid = <a, b>(f: Fun<a, b>): Fun<Monoid<a>, Monoid<b>> => {...}
```

**B**:

```ts
let join_Monoid = <a>(): Fun<a, Monoid<a>> => {...}
```

**C**:

```ts
let join_Monoid = <a>(): Fun<Monoid<Monoid<a>>, Monoid<a>> => {...}
```

**D**:

```ts
let join_Monoid = <a, b>(m: Monoid<a>, f: Fun<a, Monoid<b>>): Monoid<b> => {...}
```

<details><summary>Show Answer</summary>
<p>

```
C
```

</p>
</details>
<br>
<hr>

# Practical Exercises

## _4_

Define strings in terms of the string monoid, characterized by:

```ts
let zero: Fun<unit, string>;
let plus: Fun<Pair<string, string>, string>;
```

## _5_

Define the list as a monoid with the following monoidal operations:

```ts
let zero: <a>() => Fun<unit, List<a>>;
let plus: <a>() => Fun<Pair<List<a>, List<a>>, List<a>>;
```

where `plus` is the list concatenation.

## _6_

Implement the identity functor `Identity<a>` and extend it with the monoid operations for functors:

```ts
let id: <a>() => Fun<a, Identity<a>>;
let join: <a>() => Fun<Identity<Identity<a>>, Identity<a>>;
```

## _7_

Extend the `Option` functor with the monoid operations for functors:

```ts
let unit: <a>() => Func<a, Option<a>>;
let join: <a>() => Func<Option<Option<a>>, Option<a>>;
```

## _8_

Extend the `List` functor with the monoid operations for functors:

```ts
let unit: <a>() => Fun<a, List<a>>;
let join: <a>() => Fun<List<List<a>>, List<a>>;
```

Hint: the `plus` function you defined for lists in a previous exercise might come in handy!
