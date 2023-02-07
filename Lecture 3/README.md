<!-- index-start -->

## Index

- [Numbers
  ](#numbers)
- [More similar types
  ](#more-similar-types)
- [Towards a generalization
  ](#towards-a-generalization)
- [Monoid definition, revisited
  ](#monoid-definition-revisited)
- [Generalizing from types to functors
](#generalizing-from-types-to-functors)
<!-- index-end -->

# Introduction

Some datatypes have even more structure than just functors. This structure appears in different forms. Some forms are very primitive, for example in the sense of some builtin operators that mirror our mathematical understanding of a given structure. Some forms are more abstract, and, when combined with functors, give rise to some very powerful design patterns that play a crucial role in the definition of higher order meta programming abstractions.

## Numbers

Consider a simple example, `number`. We know that `number` supports some operations with important properties and also has some special elements which behave in a very specific way.

The first such operation would be `+,` together with its special element `0`, which is called the _identity_ of `+.`

We know that, for all numbers `a`, `b`, and `c`, the following holds:

```
a + (b + c) = (a + b) + c
```

(this is known as _association law_). Moreover, the _identity law_ states that, for all numbers `x`, the following holds:

```
x + 0 = 0 + x = x
```

Similarly, we could observe that, for the product:

```
a _ (b _ c) = (a _ b) _ c
```

and

```
x _ 1 = 1 _ x = x.
```

## More similar types

`number` is not the only datatype supporting such a structure. Consider for example another datatype, `string`. Just like numbers, strings can be added together into new strings, and the empty string `""` acts as the identity. Given arbitrary strings `a`, `b`, and `c`, then the following always holds:

```ts
a + (b + c) = (a + b) + c
a + "" = "" + a = a
```

Arrays of arbitrary types also support this structure, but instead of `+` we use the `concat` operation, and the identity is the empty array `[]`. Consider arbitrary arrays `a`, `b`, and `c` (all of type `Array<a>`, thus with the same content), then the following always holds:

```ts
a.concat(b.concat(c)) = a.concat(b).concat(c);
a.concat([]) = [].concat(a) = a;
```

In short, we can find this structure in many datatypes, always following the same set of underlying rules, even if the underlying datatypes are very different from each other.

## Towards a generalization

We can generalize these rules as follows:

- we have a datatype `T`;
- we have a composition operation `<+> : (T,T) => T` that takes as input two values of type `T` and returns a new `T`;
- we have an identity element `e:T`;
- the following holds for all `a:T`, `b:T`, `c:T`:
  -- `a <+> (b <+> c) = (a <+> b) <+> c`
  -- `a <+> e = e <+> a = a`

Such a structure is called a **monoid**, and plays a fundamental role in the rest of our narrative.

Keep in mind that, in this case, the operator <+> does not have specific semantics, like + for numbers, but stands for any possible associative operator. The same holds for the identity, which is not to be intended with a specific meaning, like 0 for numbers.

## Monoid definition, revisited

We stated before that monoids are characterized by a structure `T`, a binary operation `<+>`, and an identity element `e` (with associativity and identity laws in place).

This definition is a bit at odds with our previous work: we have built a framework of functions and ways to combine them together into new functions with `then`, but our definition of monoid does not really mix well with functions and composition.

The identity element `e`, in particular, is problematic. In order to be able to take advantage of our composition framework, everything needs to be a function. `e` is just a value, and is therefore not composable and thus "isolated" from the rest of our discoveries.

We can redefine the identity as a special function though, which takes no input and always returns `e`:

```ts
let zero: () => T = () => e;
```

Unfortunately, our composition framework requires functions to always accept an input, but `zero` does not. We can thus further reformulate `zero` to take a "fake" input which does not carry any real information, which we will call the `Unit` type:

```ts
type Unit = {}

let zero: (_:Unit) : T => e
```

Now `zero` is fully composable. The binary operation though, is not, as function composition requires functions in a single argument and never two. We can thus wrap the arguments in an appropriate container, the pair:

```ts
type Pair<a,b> = { fst:a, snd:b }

let plus: (p:Pair<T,T>) : T => ...
```

Now plus is also composable. We redefine thus a monoid, in a way that is compatible with our composition framework, as follows:

- a type `T`;
- a function `plus : Pair<T,T> => T`;
- a function `zero : Unit => T`;
- the identity law: `plus(zero({}), x) = plus(x, zero({})) = x`;
- the associative law: `plus({ fst:a, snd:plus({fst:b, snd:c})}) = plus({ fst:plus({ fst:a, snd:b }), snd:c })`.

## Generalizing from types to functors

Our discussion on monoids started from very concrete data types which programmers manipulate on a daily basis (strings, numbers, lists). Whenever confronted with a repeating pattern, we try to capture and understand its general properties.

_This way, next time we encounter an instance of it, we will quickly be able to understand what we can expect from it without reinventing the wheel._

The more general our definition of the pattern, the more we will be able to apply it. Monoids are indeed a structure which comes back in a very large number of contexts. For this reason, we take one last extra abstraction step. Instead of defining monoids in terms of a concrete type T, we will define them in terms of a functor `F`.

The monoid functions and laws must therefore be reformulated so that they work with functor `F` instead of type `T`. Let us begin with `zero : Unit => T`: instead of `T`, we expect to find an `F`, but `F` is a functor, and as such it cannot be used for a standalone declaration. This means that it must take as input a type parameter, which we add to the definition, obtaining a first working draft:

```ts
unit : <a>(?) => F<a>
```

The input of the monoidal identity on functors is also a bit challenging. Recall that the input of the `zero` function had been chosen as `Unit`, because we did not want to inject any extra structure. When we do the same for functors, we obtain the functor which carries no information, the identity functor:

```ts
type Id<a> = a;
let map_Id = function <a, b>(f: Fun<a, b>): Fun<Id<a>, Id<b>> {
  return f;
};
```

We can use this functor as the equivalent of `Unit`, therefore obtaining the following definition:

```ts
unit : <a>(Id<a>) => F<a>
```

Since `Id<a>` is just `a`, we can reformulate the definition one last time into the final version:

```ts
unit: <a>(a) => F<a>;
```

The beauty of this definition is that, even though the path to get to it was quite tortuous, it now clearly shows that `unit` gives us a way to embed a value into a functor, and is therefore the familiar concept of a **constructor**.

The same considerations and refactoring can be performed on the `plus : Pair<T,T> => T` operation. First of all, we will need to replace `T` with `F<a>` for some `a`. Let us do so for the return type:

```ts
join: <a>(Pair<T,T>) => F<a>
```

`Pair<T,T>` is a composition of two types (`T` and `T`) into a single type. In order to translate it faithfully, we would like to take two functors (`F` and `F`) and compose them into a single functor. Functors can easily be composed, leading us to:

```ts
join : <a>(F<F<a>>) => F<a>.
```

A monoid is therefore a functor `F`, equipped with two extra operations in addition to `map_F`:

- `unit : a => F<a>`, which takes as input a value of an arbitrary type `a` and embeds it into its simplest possible representation within `F`;
- `join : F<F<a>> => F<a>`, which takes as input `F` nested twice, and flattens it into a single `F`.
  The associativity and identity laws can also be translated in this functional framework in terms of function compositions.

The _associativity law_ becomes the following equivalence:

- `join<F<a>> == map_F<F<F<a>>, F<a>>(join<a>)`, where both sides are functions with type `Fun<F<F<F<a>>>, F<a>>`.
  Although it might seem odd, the type is indeed made of three nested functors. In order to avoid confusion, let us rename the generic argument of the definition of `join` to `a1`. The definition thus becomes `join : <a1>(F<F<a1>>) => F<a1>`. When we use join, it is possible to instantiate its generic argument with any type, including another type which requires generic arguments. This is the case of `F<a>` used when calling `join`. In the definition of `join` we replace `a1` with `Fun<a>` thus obtaining `join : <F<a>>(F<F<F<a>>>)`. In the definition of the identity law, the scope of `a` in the `join` definition and in `Fun<a>` is different.

The identity law becomes the following equivalence:

- `unit<F<a>>.then(join<a>) == map_F<a, F<a>>(unit<a>).then(join<a>) == id<F<a>>`, where both sides are functions with type `Fun<F<a>, F<a>>`.
