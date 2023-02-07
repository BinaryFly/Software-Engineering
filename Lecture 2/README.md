<!-- index-start -->

## Index

- [Structures
  ](#structures)
  - [Generic structures
    ](#generic-structures)
  - [Transforming containers
    ](#transforming-containers)
  - [Harmonizing the interface
    ](#harmonizing-the-interface)
  - [Composition of maps
    ](#composition-of-maps)
  - [A generalization
    ](#a-generalization)
  - [Extra constraints
    ](#extra-constraints) - [Identity
    ](#identity) - [Distribution
    ](#distribution)
  - [Functors
    ](#functors) - [An example
    ](#an-example)
- [Composition of structures
  ](#composition-of-structures)
- [Composition of functors
  ](#composition-of-functors)
  - [A concrete example
  ](#a-concrete-example)
  <!-- index-end -->

# Introduction

In this lecture we discuss the structure inherent to container data structures. We then discuss how such data structures all support a special sort of transformation, which is meant to transform the content of the data structure without destroying its structure. This sort of transformation, which we call a _structure preserving transformation_, allows us to build programs which formally match our notion of _reliability_ and _correctness_.

Moreover, we will notice how this notion of structural preservation is entirely based on the concepts we had seen previously of function composition.

## Structures

Structures, or (data) structures are a fundamental pillar of modern computing. They allow us to think in terms of _what exists in our program at any given time_, and (in a statically typed programming language, which is our focus at the moment) also guarantee that _nothing else_ exists outside the given data types. So, types are all and only the allowed shapes of data in our programs.

Containers can be simple. For example, we could define a container such as a `Point`, which contains two coordinates:

```ts
type Point = { x: number; y: number };
```

Such a container is simple. It certainly performs a useful function, but does not offer anything besides its strictly defined, predetermined structure. It cannot be adapted, nor changed to support, for example, two strings, or two dates, or even more. Our original goal was to improve the quality of our programs, by building flexible and composable building blocks which are defined (and debugged!) once and then used in many different contexts, without having to reinvent the wheel for each new context.

### Generic structures

Data structures are called generic when they can change the way they behave (intuitively, the data they contain or process) depending on some parameters. This is called generic programming, or parametric polymorphism.

For example, we might want to define a container which might store data of arbitrary type, together with a counter of sorts. Since we want to store data of any type, then we need to define a parametrically polymorphic data structure which can be instantiated to contain values of different types as follows. We will call this datatype `Countainer` as it is a _counting container_:

```ts
type Countainer<a> = { content: a; counter: number };
```

The definition of `Countainer` is not of a single, concrete datatype. Rather, `Countainer` defines, in one go, infinitely many data structures, one for each possible type `a`. We could define, and use, concrete instances of `Countainer` by passing it different concrete types as input:

```ts
let c_i:Countainer<number>({ content:10, counter:0 })
let c_s:Countainer<string>({ content:"Howdy!", counter:0 })
let c_ss:Countainer<Array<string>>({ content:["Howdy", "!"], counter:0 })
```

and so on.

It is worthy of notice that the choice of many modern languages of `<` and `>` as delimiters for generic parameters and arguments is not random. `Countainer`, like all other generic datatypes, is a compile-time function: it takes as input a type, and returns as output the counting container for that type!

### Transforming containers

Suppose we had a function that turns numbers into booleans, say `is_even:Fun<number,boolean>`. We can of course invoke this function by passing it a `number`, and this will inevitably produce a `boolean` as result.

Consider now the scenario of having indeed the `is_even` function, but instead of a number,a `Countainer<number>`. A `Countainer<number>` is not very different from just a `number`: it only carries a bit of extra information (the `counter`).

This suggests that we might want to perform the `is_even` transformation on the `content` of the `Countainer`. A well behaved transformation on the `content` could take as input the whole `Countainer<number>`, transform its `content` into a `boolean`, and then repackage the whole thing into a `Container<boolean>`. This extra repackaging step is crucial: this way we _preserve_ the extra information (the `counter`) which we have no reason to want to throw away! A simple implementation for this pattern would be:

```ts
let transform_countainer_content_num_to_bool = function (
  f: Fun<number, boolean>,
  c: Countainer<number>
): Countainer<boolean> {
  return { content: f.f(c.content), counter: c.counter };
};
```

This sort of transformation is often called `map` in modern programming languages. Notice though that such a function could very well be defined for any type of container content, not only `number` and `boolean`. We might indeed reformulate the above function in a more generic way, so that it can accept inputs, and produce outputs, of arbitrary types. We achieve this result with a generic function, which is specified not on concrete types such as `number` or `string`, or `Array<string>`, but on generic types (in our case `a` and `b`) to be specified later:

```ts
let map_countainer = function <a, b>(
  f: Fun<a, b>,
  c: Countainer<a>
): Countainer<b> {
  return { content: f.f(c.content), counter: c.counter };
};
```

The function above could be invoked as follows:

```ts
let c: Countainer<number> = { content: 3, counter: 0 };
let l: Countainer<boolean> = map_countainer(is_even, c);
```

`l` would then become `{ content:false, counter:0 }`. Of course, since containers can be mapped by passing them arbitrary instances of `Fun`, nothing stops us from invoking `map_countainer` with a `Fun` produced by composing other `Fun`'s:

```ts
let c: Countainer<number> = { content: 3, counter: 0 };
let l: Countainer<boolean> = map_countainer(incr.then(is_even), c);
```

This results in `l.content` becoming `4`, as `map` will first increment, and then check whether or not the value is even.

A function such as `map_countainer` is known as a _structure preserving transformation_. We will later see what this means more precisely. For the moment, our intuition is that such a transformation will:

- apply the given `Fun` to all available values of type `a`, so that they become of type `b` (the `content`);
- repackage the `b`'s into a new structure, by just copying over the remaining information (the `counter`).

### Harmonizing the interface

Let us take a moment to observe the interface of `map_countainer`:

```ts
map_countainer : <a,b>(f:Fun<a,b>, c:Countainer<a>) : Countainer<b>
```

The function has a big issue: it does somehow interrupt our ability to make chains through function composition. This means that the powerful elegance and simplicity of the `then` operator is now lost!

In order to improve the situation, we might reformulate the function into having a new signature:

```ts
map_countainer :<a,b>(f:Fun<a,b>) :Fun<Countainer<a>, Countainer<b>>
```

According to this new formulation, `map_countainer` simply takes as input a function on arbitrary datatypes, and elevates it in level of abstraction by making it able to work on `Countainer`'s. We could implement it as follows:

```ts
let map_countainer =
function<a,b>(f:Fun<a,b>) : Fun<Countainer<a>, Countainer<b>> {
return Fun(c =>{ content:f.f(c.content), counter:c.counter })
}
```

This apparently minor change has a profound impact on our ability to chain computations: namely, we can now produce new functions by combining existing ones, in two ways:

- we can compose functions in a pipeline via then;
- we can elevate functions to the domain of Countainer via map_countainer.

For example, we could simply define:

```ts
let incr_countainer: Fun<
  Countainer<number>,
  Countainer<number>
> = map_countainer(incr);
let is_countainer_even: Fun<
  Countainer<number>,
  Countainer<boolean>
> = map_countainer(is_even);
```

and so on.

### Composition of maps

Container transformation functions are now `Fun`'s. This means that, just like all other functions, they can be composed together, as long as the output of one function has the same type as the input of the next. This means that we could compose together the functions we just defined into a new pipeline of `Countainer` transformations:

```ts
let my_f = incr_countainer.then(is_countainer_even);
```

`my_f` would take as input a `Countainer<number>`, increment its content, check whether or not it is even, and store the resulting boolean into the final `Countainer<boolean>`. Notice that the transformation preserves the extra structure, meaning that `map_countainer` guarantees that the `counter` will not be unduly modified. We could of course define a function to increment the counter of a container though:

```ts
let tick: Fun<Countainer<number>, Countainer<number>> = Fun((c) => ({
  ...c,
  counter: c.counter + 1,
}));
```

(Note: `{...c, counter:c.counter+1}` copies `c` over and overwrites the `counter` of the result; this is called a _spread operator_).

We can now make use of our `tick` function as part of our pipelines, as follows:

```ts
let my_g = incr_countainer.then(is_countainer_even).then(tick);
```

### A generalization

Let us generalize the constructs we have just put to use. We started with `Countainer`, which is a generic type. Then we defined a generic function that lifted an existing function to the domain of `Countainer`'s, transforming the generic content, and preserving the rest.

The same process of defining such a transformation could be applied to more than just our `Countainer`. Suppose we had an arbitrary generic datatype:

```ts
type F<a> = ...
```

We use "..." to denote that we really do not care about its content or shape.

We could then define a transformation function which takes as input an arbitrary function, and lifts it to work with `F`'s:

```ts
map_F : <a,b>(f:Fun<a,b>) : Fun<F<a>, F<b>>
```

Of course we cannot just implement `F` here, since we do not know how `F` really is defined, but we can at least postulate what its interface is supposed to be.

(Note: unfortunately, we cannot really (easily) define such an interface in modern languages such as TypeScript: `F` itself would be a generic parameter of this interface, but it is not a type (rather a generic type itself!). Most modern languages do not support generic types as parameters of another generic type.)

### Extra constraints

When we have both a generic type `F`, and its corresponding structure preserving transformation `map_F`, then we want to put a few extra constraints in order to guarantee that `F` is well behaved. Mostly, these constraints ensure that `map_F` does not do anything surprising (surprises go, by definition, against our intuition, and as such cause unexpected behavior and, inevitably, bugs).

#### Identity

The first, unsurprising behavior is that `map_F` preserves the identity function. Specifically,if we invoke `map_F` with `id` as input, given that `id` will not change the content, then we expect the final F to be the same as the original. This can be stated as:

```ts
map_F<a, a>(id<a>()) = id<F<a>>();
```

Suppose that, in the case of the `Countainer`, `map_countainer` increased the `counter` (which would change the surrounding structure, and thus not be a valid structure preserving transformation). Then the above equality would not hold, because `id<Countainer<a>>()` would return exactly the same value, `counter` included, whereas `map_Countainer<a,a>(id<a>())` would increase the underlying `Counter`.

#### Distribution

The second constraint is more articulated, and just as interesting. When composing functions, we expect that the resulting function behaves as the sum of the composed functions, and nothing more. What about composing the result of structure preserving transformations though?

Consider two composable functions `f:Fun<a,b>` and `g:Fun<b,c>`. We can of course lift them to `F` via `map_F`:

```ts
let f_F: Fun<F<a>, F<b>> = map_F<a, b>(f);
let g_F: Fun<F<b>, F<c>> = map_F<b, c>(g);
```

We implicitly obtain two paths between `F<a>` and `F<c>`, one passing directly through values of type `F` via `F_f` then `g_F`, the other passing through `f` then `g`, but lifting the resulting composition:

```ts
let path1: Fun<F<a>, F<c>> = f_F.then(g_F);
let path2: Fun<F<a>, F<c>> = map_F<a, c>(f.then(g));
```

`path1` will run `f` on the content of the first input of type `F<a>` per definition of `map_F` (how we obtained `f_F`), and then it will run `g` on the transformed content. `path2` will do the same, but instead of packaging and repackaging twice, it will unpack the input of type `F` once, apply `f` then `g` right away, and repackaging once and for all.

### Functors

A type constructor `F` and its transformation `map_F` are called a **functor** if the following holds for all `f:Fun<a,b>` and `g:Fun<b,c>`:

```ts
map_F : (f:Fun<a,b>) : Fun<F<a>,F<b>>
map_F<a>(id<a>()) = id<F<a>>()
map_F<a,b>(f).then(map_F<b,c>(g)) = map_F<a,c>(f.then(g))
```

A functor is a well behaved data structure that always behaves in a reliable, intuitive way. It will be the foundation of the course.

#### An example

Let us now check a couple of examples of actual functors, in order to see how they translate into practice.

- `Option`
  The `Option` functor offers a type-safe alternative to managing values which might, for whatever reason, be absent. Think of a computation that might either produce a number, or fail: its return type should represent the fact that there might be no number at all!

Option is defined as a discriminated union, that is a union of two different types which must be explicitly checked by means of a shared constant:

```ts
type Option<a> = { kind: "none" } | { kind: "some"; value: a };
```

When given a value of type `Option<a>`, the only value accessible will be the `kind`, which has type `"none" | "some"`. We cannot read the `value` right away, so the following code would give a compile-time error:

```ts
function DOES_NOT_WORK(x: Option<number>): number {
  return x.value;
}
```

On the other hand, the following function would work because it first checks for availability of `value`:

```ts
function print(x: Option<number>): string {
  if (x.kind == "some") return `the value is ${x.value}`;
  else return "there is no value";
}
```

Inside the then branch, the type of `x` is adjusted, because we know that it is not just an `Option<number>`, but the more specific `{ kind:"some", value:number }`.

The map function of `Option` will have to check for availability of the `value`, and if a `value` is present, then we transform it, otherwise we simply repackage a `none`:

```ts
let none : function<a>() : Option<a> { return { kind:"none" } }
let some : function<a>(x:a) : Option<a> {
return { kind:"some", value:x } }

let map_Option = function<a,b>(f:Fun<a,b>) : Fun<Option<a>,Option<b>> {
return Fun(x => x.kind == "none" ? none<b>() : some<b>(f.f(x.value)))
}
```

Thanks to `Option` and its `map_Option`, we can safely manipulate values which might be absent by simply chaining operations on the content, and then mapping the result:

```ts
let pipeline: Fun<Option<number>, Option<boolean>> = map_Option(
  incr.then(double.then(is_greater_than_zero))
);
```

Notice that we do not need to check whether or not the original value was present, as this is all done implicitly by `map_Option`. Moreover, the property that:

```ts
map_Option(f).then(map_Option(g)) = map_Option(f.then(g));
```

is also very useful for optimization: performing `map_Option` just once will evaluate one conditional less, meaning that the right hand side with a single `map_Option` produces the same result as the left hand side, but requires a bit less computation and as such is expected to be slightly faster.

## Composition of structures

An important property of abstractions is that they must be composable. The ability to compose allows us to define basic building blocks, test them, ensure they work, and then "glue" them together along some predefined composition mechanisms which are guaranteed to preserve the essential properties of the individual elements in a logical way.

Following this philosophy, we defined the concept of `then` to compose functions, in a way that results in a new function that, at its core, is entirely based on the original two. Composition also usually features a sort of starting point, which when composed with other constructs, does nothing but yield them back. The identity function, which returns the input intact, was the starting point of function composition: `f.then(id()) = id().then(f) = f`.

## Composition of functors

Just like functions, functors can also be composed. Let us consider two functors, `F` and `G`, together with their transformations `map_F : Fun<a,b> => Fun<F<a>, F<b>>` and `map_G : Fun<a,b> => Fun<G<a>, G<b>>`.

We would like to compose both `F` and `G` together, so that we get a new functor `F . G` which encapsulates the properties of both `F` and `G`.

Fortunately, this is surprisingly easy to do. The resulting functor will have type:

```ts
type F_G<a> = F<G<a>>;
```

Its transformation will simply transform all elements of the external `F` by using `map_F`, and the internal elements of `G` with `map_G`:

```ts
let map_F_G = function<a,b>(f:Fun<a,b>) => Fun<F_G<a>, F_G<b>> {
  return map_F<G<a>,G<b>>(map_G<a,b>(f))
}
```

It is possible to prove, but we will not do it here for the sake of brevity, that functor composition preserves all properties of a functor, such as identity and distribution of composition. Suffice here to state that the composed functor will be a fully valid functor as well.

The "basic" functor which preserves composition will, unsurprisingly, be the identity functor which adds no structure whatsoever:

```ts
type Id<a> = a;
let map_Id = function <a, b>(f: Fun<a, b>): Fun<Id<a>, Id<b>> {
  return f;
};
```

### A concrete example

Let us consider both functors `Countainer` and `Option`. We might use them to define a possibly empty `Countainer` as follows:

```ts
type CountainerMaybe<a> = Countainer<Option<a>>
let map_Countainer_Maybe = function<a,b>(f:Fun<a,b>) :
  Fun<CountainerMaybe<a>, CountainerMaybe<b>> {
  return map_Countainer<Option<a>,Option<b>>(map_Option<a,b>(f)));
}
```

Our `CountainerMaybe` will have a counter associated with a container which might be empty. Transforming the composed functor will first transform the `content` of the external `Countainer`, turning it from `Option<a>` into `Option<b>`. This transformation is achieved by transforming `Option` via its own map.
