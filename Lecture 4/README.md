<!-- index-start -->

## Index

- [Monads: (endo)functors with monoidal structure
  ](#monads-endofunctors-with-monoidal-structure)
  - [Operators
    ](#operators)
- [A first example: the `Fun` monad
  ](#a-first-example-the-fun-monad)
- [A data container: `Pair` as two monads
  ](#a-data-container-pair-as-two-monads)
  - [The right version
    ](#the-right-version)
- [The identity monad
  ](#the-identity-monad)
- [Another perspective on `bind`
  ](#another-perspective-on-bind)
  - [Monoid laws, revisited
  ](#monoid-laws-revisited)
  <!-- index-end -->

# Introduction

Whenever we are working with a functor which, at the same time, exhibits the monoidal structure of having `unit` and `join`, and the associative and identity laws, then this functor is also called a _monad_, and plays a very important role in building advanced design patterns. We will see more technical details about this, but first let us take a brief philosophical detour.

The importance of monads cannot be understated. Monads arise everywhere: concurrency via `async/await` in C# and JavaScript, LINQ in C#, streams in Java, `Promise` in JavaScript are only some of the examples of mainstream application of monads. Monads also arise in most advanced languages: Scala, F#, Haskell all feature monads prominently.

The power of monads arises from their combination of structure and flexibility. Monads are not applicable to a single domain, but rather are a meta pattern that tells us how to build libraries which "make sense", in many different domains. This way, instead of having to reinvent the wheel at each turn, we will be able to recognize the underlying monad, implement its operators, and be done. This allows us to leverage lots of structure and knowledge, at minimal cost, in domains that include (but are not limited to):

- exception handling;
- list comprehensions;
- state management;
- concurrency;
- backtracking ("classic AI").

Moreover, monads can be composed into new monads, "just" like functions can be composed into new functions by means of `then`. This means that a standard toolkit of monads is not the final stop, and is actually just the beginning: most problems will be solvable with some composition of different basic monads.

## Monads: (endo)functors with monoidal structure

Consider a functor `F` (actually an _endo_-functor, given that it goes from types to types, and therefore does not change domain). The functor implements the following "module":

```ts
type F<a> = ...
let map_F : <a,b>(f:Fun<a,b>) => Fun<F<a>, F<b>>
```

The functor `map_F` guarantees the important properties of preserving the internal structure, and preserving the identities. This means that `map_F` will only transform the `a`'s into `b`'s, and do nothing more to the rest of the information available.

A functor can also have a monoidal structure. This means that it would also feature the following functionality:

```ts
let unit: <a>() => Fun<a, F<a>>;
let join: <a>() => Fun<F<F<a>>, F<a>>;
```

Seen from this perspective, a monad would simply seem to be **a generic datatype `F` which can be transformed (`map_F`), constructed (`unit`) and flattened (`join`)**.

### Operators

The power of monads arises from the fact that, even within the vague boundaries that we have just set up, it possible to define quite a lot of useful methods on monads, without ever having to think about the concrete monad. Let us begin with the two most used such operators, `bind` and `kleisli`.

The `bind` operator allows us to link two instances of a monad, where the second instance depends on the _content_ of the first. `bind` results in a new instance of the monad, which is then often used as input of the following `bind` operation:

```ts
let bind = <a, b>(p: F<a>, q: Fun<a, F<b>>): F<b> =>
  map_F<a, F<b>>(q).then(join<b>()).f(p);
```

The first `map_F(q)` goes from `F<a>` to `F<F<b>>`. `join` will then flatten the `F<F<b>>` into a single `F<b>`. Of course the chain of functions needs to be called with `p`.

We can use `bind` in order to write code that will look as follows:

```ts
bind(p, x =>
bind(q, y =>
bind(r, z =>
...unit(...)...)
```

Where `p`, `q`, and `r` are all instances of the monad `F`, but `q` might be computed from `x`, `r` might be computed from `x` and `y`.

`bind` is usually not defined and used so explicitly. A typical trick would be to augment `F` so that it contains another method, `then`, which performs a `bind`. This version of `bind` simply uses a lambda instead of `Fun` as second argument for better readability.

```ts
then: <a, b>(f: (_: a) => F<b>): F<b>
```

`then` can simply call `bind` by wrapping its lambda argument into a `Fun` and then passing it to `bind`. Note that `then` takes only one argument (while `bind` takes two where the first argument is `F<a>`), because it is a method and `this`, which has type `F<a>`, is implicitly passed.

Thanks to such a method, the code above would turn into:

```ts
p.then(x =>
q.then(y =>
r.then(z =>
...unit(...)...)
```

The attentive reader might have noticed that this pattern is very common, in this very same format, in the JavaScript world: `Promise`, the fundamental data structure used to perform remote calls, has a `then` method which is used exactly as we just described. The only minor difference is that instead of `unit`, we would then use `Promise.resolve`, which has the very same meaning but another name.

## A first example: the `Fun` monad

Let us explore some examples of basic monads. These monads are not particularly exciting or mind-blowingly useful by themselves, but they are self contained and complex enough to let us acquaint with the reality of the practical application of monads.

We begin with a monad which we actually already know: `Fun`. Let us consider functions from a fixed type, for example `number`:

```ts
type Fun_n<a> = Fun<number, a>;
```

This interface is trivially a functor:

```ts
let map_Fun_n = <a, b>(f: Fun<a, b>, p: Fun_n<a>): Fun_n<b> => p.then(f);
```

The `unit` and `join` operators are also quite simple, as they just involve calling and creating functions:

```ts
let unit_Fun_n = <a>() => Fun<a, Fun_n<a>>(x => Fun(i => x))
let join_Fun_n = <a>() => Fun<Fun_n<Fun_n<a>>>,Fun_n<a>>(f => Fun(i => f.f(i).f(i)))
```

The `Fun_n` monad is therefore a monad which encapsulates the composition, but also the creation and simplification, of functions. It is, in a sense, a strict extension of the original definition of function, because that definition did not provide facilities to create new functions (`unit_Fun_n` creates a constant function, that is a function that always returns a given constant), nor to flatten functions (`then` only made it possible to compose functions at the same level, but not simplify nested functions).

## A data container: `Pair` as two monads

Recall the `Pair` datatype which contains values of two arbitrary types:

```ts
type Pair<a, b> = { x: a; y: b };
let fst = <a, b>(): Fun<Pair<a, b>, a> => Fun((p) => p.x);
let snd = <a, b>(): Fun<Pair<a, b>, b> => Fun((p) => p.y);
let map_Pair = <a, b, a1, b1>(
  f: Fun<a, a1>,
  g: Fun<b, b1>
): Fun<Pair<a, b>, Pair<a1, b1>> => Fun((p) => ({ x: f.f(p.x), y: g.f(p.y) }));
```

From `Pair` it is possible to effortlessly generate two monads, one per generic parameter. Let us fix the second parameter, say to `number` (any other type would do) and thus define the `WithNum` generic container:

```ts
type WithNum<a> = Pair<a, number>;
```

It might occur to the reader that this is not really different from the `Countainer` we worked with when introducing functors, and indeed we are now working with a generalization of that sort of datatype.

The structure preserving transformation will therefore map the whole pair, while preserving the right side of the pair with the identity function:

```ts
let map_WithNum = <a, b>(f: Fun<a, b>): Fun<WithNum<a>, WithNum<b>> =>
  map_Pair(f, id<number>());
```

The monadic operators arise quite easily, by taking advantage of the underlying monoidal structure of `number`:

```ts
let unit_WithNum : <a>() : Fun<a,WithNum<a>> => Fun(x => { x:x, y:0 })
let join_WithNum : <a>() : Fun<WithNum<WithNum<a>, WithNum<a>> =>
  Fun(x => { x:x.x.x, y:x.y + x.x.y })
```

We could have given an alternate definition that uses another monoid on numbers, for example with `1` and `*` in place of `0` and `+,` obtaining yet another monad.

### The right version

Notice that the monad just presented is based on the left-hand side of the pair. It is possible to swap `x` and `y`, therefore obtaining another (almost identical) monad which varies the second argument (the `b`) instead of the first (the `a`). We leave this as a trivial exercise to the reader.

## The identity monad

Just like functions and monoids have an identity, which is a sort of safe starting point of many chains of computations, monads have an identity as well. This identity is interesting as a case study, as it illustrates

The identity monad is based on the identity functor, which does (on purpose) absolutely nothing:

```ts
type Id<a> = a;
let map_Id = <a, b>(f: Fun<a, b>): Fun<Id<a>, Id<b>> => f;
```

The monadic combinators are also quite straightforward:

```ts
let unit_Id = <a>() : Fun<a,Id<a>> = id<a>()
let join_Id = <a>() : Fun<Id<Id<a>>, Id<a>> = id<a>()
```

## Another perspective on `bind`

The `bind` operator allows us to merge together (the "content") of a monad to a function which turns that content into another monad. The result of this merging operation is a monad itself.

`bind` has one minor issue though: it is completely disconnected from our original principles of _composing through functions_, which we have seen has shown its usefulness ubiquitously. We can reformulate `bind` in a way which, while being substantially equivalent, emphasizes that this sort of operator is nothing but a way of composing specially defined functions:

```ts
let then_F = <a, b>(f: Fun<a, F<b>>, g: Fun<b, F<c>>): Fun<a, F<c>> =>
  f.then(map_F<b, F<c>>(g)).then(join_F<c>());
```

This new formulation of function composition when the return value is encapsulated in an instance of a monad, is also known as _Kleisli composition_. Its importance arises from the fact that this new composition operator clearly shows how binding different instances of a monad together is no more than a fancy composition. Both `bind` and `then_F` will be used throughout the rest of the text, depending on which of the two operators best matches the given problem.

### Monoid laws, revisited

Given Kleisli composition, we can reformulate the monoid laws in a much more elegant way. The elegance arises from the fact that these laws are now precisely mirroring the monoidal laws, but using Kleisli composition in place of the monoidal composition operation.

The identity laws (recall: `x <+> e = e <+> x = x`) replaces `<+>` with `then_F` and the identity element `e` with `unit()`:

```ts
then_F<a,b,b>(f, unit<b>()) : Fun<a,F<b>> ==
then_F<a,a,b>(unit<a>(), f) : Fun<a,F<b>> ==
f
```

Intuitively, `f` will take us from a to `F<b>`. `unit` will pack either the `b` or the `a` in an `F<b>` or `F<a>`, but `then_F` will take care of the flattening for us, therefore annulling the packing performed by `unit`.

The association laws (recall: `f <+> (g <+> h) = (f <+> g) <+> h`) replaces `<+>` with `then_F` as well, leading us to:

```ts
then_F(f, then_F(g, h)) == then_F(then_F(f, g), h);
```

Intuitively, both sides of the equation will pass the result of `f g`, which then produces a result which is fed into `h`. In the first case (to the left) we pass the result of `f` to `then_F(g,h)`, which does nothing but propagating the chain. In the second case we pass the result of _directly_ to `g`, but we delegate to the outer `then_F` the role of passing it on to `h`.

We omit a more formal verification of these properties, given the scope of this text.
