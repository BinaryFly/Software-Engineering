# Questions

## _1_

What is the type of `map` applied to the type `List<a>`?

**A**:

```ts
let mapList = <a, b>(_: Fun<a, a>) => Fun<List<a>, List<a>>`
```

**B**:

```ts
let mapList = function<a>(): Fun<List<a>, List<a, b>>
```

**C**:

```ts
let mapList = <a, b>(_: Fun<a, b>) => Fun<List<a>, List<b>>;
```

**D**:

```ts
let mapList = function<a, b>(): Fun<List<a>, List<b>>
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

## _2_

Consider the data structure

```ts
type Triplet<a> = {
  x: a;
  y: a;
  z: a;
};
```

Which of the following implementations of map is correct?

**A**:

```ts
let map_Triplet = function <a, b>(f: Fun<a, b>): Fun<Triplet<a>, Triplet<b>> {
  return Fun((t: Triplet<a>) => {
    return {
      x: f.f(t.x),
      y: t.y,
      z: t.z,
    };
  });
};
```

**B**:

```ts
let map_Triplet = function <a, b>(f: Fun<a, b>): Fun<Triplet<a>, Triplet<b>> {
  return Fun((t: Triplet<a>) => {
    return {
      x: f.f(t.x),
      y: f.f(t.y),
      z: f.f(t.z),
    };
  });
};
```

**C**: All the given implementations are correct

**D**:

```ts
let map_Triplet = function <a, b>(f: Fun<a, b>): Fun<Triplet<a>, Triplet<b>> {
  return Fun((t: Triplet<a>) => {
    return {
      x: f.f(t.x),
      y: f.f(t.y),
      z: t.z,
    };
  });
};
```

<details><summary>Show Answer</summary>
<p>

```
B
```

</p>
</details>
<br>
<hr>

## _3_

Given the following Functor

```ts
type Constant<a, s> = s;
```

choose the correct statement

**A**:
It is a valid functor and

```ts
let map_Constant<a, b, s>(f: Fun<a, b>):
  Fun<Constant<a, s>, Constant<b, s>> = id<Constant<a, s>>()
```

**B**:
The following map implementation

```ts
let map_Constant<a, b, s>(f: Fun<a, b, s>):
  Fun<Constant<a, s>, Constant<b, s>> = id<Constant<a, s>>()
```

violates the identity law and the associative law of map.

**C**:
It is a valid functor and

```ts
let map_Constant<a, b, s>(f: Fun<a, b, s>): Fun<Constant<a, s>, Constant<b, s>> = id<s>()
```

**D**: It is not a valid functor.

<details><summary>Show Answer</summary>
<p>

```
A
```

</p>
</details>
<br>
<hr>

## _4_

Given the following function definitions

```ts
let incr = (x: number) => x + 1;
let crazy = (x: number) => str(x + 25);
```

and the following expression

```ts
map_Option(incr.then(crazy));
```

Which one of the following expressions is equivalent to the one above?

**A**:

```ts
mapOption(crazy).then(map_Option(incr));
```

**B**:

```ts
crazy.then(map_Option(incr));
```

**C**:

```ts
mapOption(incr).then(map_Option(crazy));
```

**D**:

```ts
incr.then(map_Option(crazy));
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

## _5_

Implement a List Functor `List<a>` where its data structure is defined with a discriminate union made of `Cons` and `Empty`. `Cons` contains a field `head` of type `a` for the first element in the list, and a field `tail` of type `List<a>`. `Empty` does not contain anything and marks only an empty list.

```ts
type List<a> =
  | {
      kind: "Cons";
      head: a;
      tail: List<a>;
    }
  | {
      kind: "Empty";
    };
```

Define a `List<string>` of symbols (single characters) and apply a transformation `encode` to the symbol using map that adds a constant number (shift) to the ASCII code of the character:

```ts
let encode: Fun<List<string>, List<string>>;
```

You can use `charCodeAt(0)` to get the integer number corresponding to the symbol and then add a constant value.

Generalize `encode` with the following type

```ts
let encode: Fun<number, Fun<List<string>, List<string>>>;
```

so that the shift value can be specified as input.

<hr>

## _6_

Implement an Exception Functor `Exception<a>` where its data structure is defined with a discriminate union made of `Result` and `Error`. `Result` contains an element of type `a` while `Error` contains a `string` reporting an error message.

<hr>

## _7_ (hard)

Consider a tile game where each tile can be either terrain, a town, or an army. The player can build a town converting a terrain tile to a town, destroy a town or an army reverting a town back to a terrain tile, or moving an army changing a terrain tile into an army tile. Any other combination keeps the tile as it is.

1. Implement the tile as a Tile Functor whose data structure is `Tile<a>`, where `a` defines the kind of tile.

2. Implement the **composite functor** `List<Tile<a>>` that converts all the tiles of type `a` in a different tile.

<hr>

## _8_ (hard)

Define a pair as a bi-functor `Pair<a, b>`. A bifunctor contains a function

```ts
let map_Pair_left = function<a, b, c>(f: (_: a) => c): Fun<Pair<a,b>, Pair<c, b>>
let map_Pair_right = function<a, b, c>(f: (_: b) => c): Fun<Pair<a,b>, Pair<a, c>>
let map2_Pair = function<a, b, c, d>(f: (_: a) => c, g:(_: b) => d):
  Fun<Pair<a, b>, Pair<c, d>>
```

Is it possible to define both the `left` and `right` variants from `map2_Pair`?
