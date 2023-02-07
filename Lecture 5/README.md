<!-- index-start -->

## Index

- [Error management
  ](#error-management)
- [Example application
  ](#example-application)
- [From `Option` to `Either`
  ](#from-option-to-either)
- [Sketch of practical applications
](#sketch-of-practical-applications)
<!-- index-end -->

# Introduction

In this chapter we will discuss the first practical application of monads in order to solve, or simplify, a concrete problem. We will begin with a traditional example which is simple enough to be simple to oversee, but complex enough to show a powerful feature that improves the quality of our software by showcasing all that monads are powerful at.

## Error management

Consider a computation which might succeed, or fail. We want to model failure in a way that is type safe, so that the compiler will be able to help us check all error conditions, and ensure that we are making no dangerous assumptions such as "This method will always succeed, no need to check for errors!", or "I am in a hurry, I will add the check later!".

Invariably, these faulty assumptions will turn against us: methods that always succeeded will suddenly start failing at Friday, 17.55, and checks that were meant to be added later slowly turn to history, then legend, and finally become myth.

Let us therefore define a new datatype which forces us to perform the proper checks and steers us clear of faulty assumptions. This datatype will model the possible results of a method which might return some value of some arbitrary type `a`, or might return no value at all:

```ts
type Option<a> = { kind: "none" } | { kind: "some"; value: a };
```

In order to simplify working with a polymorphic datatype such as `Option`, it is considered common courtesy and practice to also define some handy ways to construct it, one function per concrete shape:

```ts
let none = <a>(): Fun<Unit, Option<a>> => Fun((_) => ({ kind: "none" }));
let some = <a>(): Fun<a, Option<a>> => Fun((a) => ({ kind: "some", value: a }));
```

Notice that we are defining functions that are compatible with our previous framework, given that we do not want to step out of it and rather extend it. This means that invoking, for example, `some<number>()`, gives us back the `Fun<number,Option<number>>` which will then actually know how to create an `Option<number>` from a `number`.

Let us now consider the `map_Option` method. This method must transform an `Option<a>` into an `Option<b>`, by checking whether or not it is a `none`, and if this is not the case, transforming the value and re-wrapping the content in a new `some`.

```ts
let map_Option = <a, b>(f: Fun<a, b>): Fun<Option<a>, Option<b>> =>
  Fun((x) => (x.kind == "none" ? none<b>() : f.then(some<b>())));
```

Notice that, thanks to our `Fun`, we can simply concatenate `f` and `some` together, into a reasonably linear and elegant implementation.

The monadic operators are then `unit_Option`, and `join_Option`. Let us begin with the simplest of the two:

```ts
let unit_Option = <a>() : Fun<a,Option<a>> = some<a>()
```

The `join` operator is a bit more challenging, as it requires checking for `none` in order to simplify the `none<Option<Option<a>>` into `none<Option<a>>`:

```ts
let join_Option = <a>(): Fun<Option<Option<a>>, Option<a>> =>
  Fun((x) => (x.kind == "none" ? none<a>() : x.value));
```

Of course we could define `bind_Option` trivially from `map_Option` and `join_Option`, and augment `Option<a>` with a method `then`. Armed with these (trivial) additions, we would end up with the following implementation:

```ts
let bind_Option = function<a, b>(opt: Option<a>, k: Fun<a, Option<b>>) : Option<b> {
return map_Option(k).then(join_Option()).f(opt)
}

type Option<a> = ({
kind: "none"
} | {
kind: "some",
value: a
}) & {
then: <b>(f: (\_: a) => Option<b>) => Option<b>
}
```

This implementation would then allow us to define methods which gracefully fail such as, for example, safe division:

```ts
let safe_div = (a:number, b:number) : Option<number> =>
b == 0 ? none<number>() : some<number>(a // b)
```

Suppose now that we wanted to perform safe division on the result of a safe division, therefore implementing patterns such as `(x / y) / z`. Then `safe_div` would be reformulated as:

```ts
let safe_div = (a: Option<number>, b: Option<number>): Option<number> =>
  a.then((a_val) =>
    b.then((b_val) =>
      b_val == 0 ? none<number>() : unit_Option(a_val / b_val)
    )
  );
```

Thanks to this new formulation, we would be able to define complex nested patterns such as:

```ts
let div3 = (
  x: Option<number>,
  y: Option<number>,
  z: Option<number>
): Option<number> => safe_div(safe_div(x, y), z);
```

## Example application

As a concrete example of application, we might consider that `Option` has found its way in most mainstream languages.

.Net has had `Nullable` for quite a while, `Option` is available in Rust, ML, `Maybe` is part of Haskell, Java has seen `Optional` added to the latest versions of the standard library.

The typical applications of `Option` are found when something might fail or to represent invalid state. For example, consider a webpage which must draw a list of customers. When the page is instantiated, there are no customers to show, as they must yet be loaded. We might be tempted to store an empty array of customers, but this would be mistaken, as we would then be unable to distinguish between a list being loaded, and an actually empty list of customers. The proper representation would then become `Option<List<Customer>>`. Rendering such a component would then be forced to check the `Option` as follows:

```ts
function render(customers: Option<List<Customer>>) {
  if (customer.kind == "none") {
    render_loading_screen();
    // customer.value is not available
    // accessing it causes a compiler error
  } else {
    // customer.value is simply available
    // render the list of customers found there
    render_customers(customer.value);
  }
}
```

## From `Option` to `Either`

Suppose now that we wanted to extend our `Option` to also carry extra information together with `none`. This extra information could be considered an extra payload, for example useful if we want to store, for example, error data or some additional optional attachment of sorts.

This would lead us to the following datatype definition, which is a strict generalization of `Option`:

```ts
type Either<a, b> = { kind: "left"; value: a } | { kind: "right"; value: b };
```

The two utility functions to initialize the concrete shapes of `Either` will both look like `some`, but with both type arguments. Traditionally they are called `inl` and `inr`, meaning "left embedding" and "right embedding":

```ts
let inl = <a, b>(): Fun<a, Either<a, b>> =>
  Fun((a) => ({ kind: "left", value: a }));
let inr = <a, b>(): Fun<b, Either<a, b>> =>
  Fun((b) => ({ kind: "right", value: b }));
```

The structure preserving transformation is then:

```ts
let map_Either = <a, b, c, d>(
  f: Fun<a, c>,
  g: Fun<b, d>
): Fun<Either<a, b>, Either<c, d>> =>
  Fun((x) =>
    x.kind == "left"
      ? f.then(inl<a, b>()).f(x.value)
      : g.then(inr<a, b>()).f(x.value)
  );
```

We can now show how to implement `Option` only in terms of `Either`. `Option` then becomes just a shell implementation, only calling existing methods from `Either`. This clearly shows that `Option` is no more than an _instance_ of `Either`:

```ts
type Option<a> = Either<Unit, a>;
let none = <a>(): Option<a> => inl<Unit, a>().f({});
let some = <a>(): Fun<a, Option<a>> => inr<Unit, a>();

let map_Option = <a, b>(f: Fun<a, b>): Fun<Option<a>, Option<b>> =>
  map_Either<a, Unit, b, Unit>(f, id<Unit>());
```

In order to complete the generalization of the previous definition of `Option`, let us define the monadic operators for `Either. unit_Either` is trivially just the right injection (thus we assume that the binding happens on the right hand of `Either`):

```ts
let unit_Either = <a, b>(): Fun<a, Either<b, a>> => inr<b, a>();
```

Joining is also quite similar, but it only requires the realization that the monad is thus all focused on the right side of the `Either`, and that the left side contains information that is simply propagated, without further processing:

```ts
let join_Either = <a, b>(): Fun<Either<b, Either<b, a>>, Either<b, a>> =>
  Fun((x) => (x.kind == "left" ? inl<b, a>().f(x.value) : x.value));
```

## Sketch of practical applications

Just like `Option` denoted absence of value (for example while loading something from a server), `Either` denotes absence of value and presence of other information, such as for example exception information. Thus, we could define a simple exception system as:

```ts
type Exceptional<a> = Either<string, a>;
```

A function returning `Exceptional<Customer>` would therefore contain a `Customer`, if the computation has succeeded, and a `string` (containing an error description) if the computation has failed.
