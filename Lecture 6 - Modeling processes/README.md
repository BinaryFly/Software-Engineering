<!-- index-start -->

## Index

- [Stateful processes
  ](#stateful-processes)
- [A practical application
  ](#a-practical-application)
- [Case study: modeling a tiny DSL
](#case-study-modeling-a-tiny-dsl)
<!-- index-end -->

# Introduction

Datatypes have been, so far, almost exclusively used to model state. State is, inherently, static, and models _what things are_, not _what they do_.

This leaves a weakness in our ability to model _dynamic processes_. Such processes are also, in a sense, _things_, so they should be modeled with datatypes. At the same time, they _perform actions_, so they should also include dynamic parts in their definition.

In this chapter we will focus our discussion on the modelling of simple processes, and how monads arise naturally as a complement to such models. We will then, in later chapters, explore richer models of more complex processes.

## Stateful processes

Let us consider a process that works on state. Such a process will be able, at all times, to both **read** from and **write** to the state. A process also represents a computation, and it will therefore produce a result, of an arbitrary type.

Since the process performs a computation that will give us a result of an arbitrary type, let us start from this definition and enrich it in steps until we reach a satisfactory formulation. Our starting point therefore only encapsulates the performing of a computation:

```ts
type Producer<a> = Fun<Unit, a>;
```

Of course, this computation needs the ability to access the state. This means that the state, of some arbitrary type, must be passed to this computation:

```ts
type Reader<s, a> = Fun<s, a>;
```

Of course, we want the process to also be able to redefine the value of the state (remember: we want to be referentially transparent, so we will not mutate `s` in place!), and we can achieve this by letting our process return a new value of the updated state. This is usually called `State` monad, and we will conform to this definition:

```ts
type State<s, a> = Fun<s, Pair<a, s>>;
```

In order to complete the monadic definition, we need (as usual) the `map_State` function, the `unit_State`, and `join_State`. Let us begin with the functoriality of our `State`, which takes advantage of the fact that the stateful process is no more than a function, and as such it can be concatenated with the inner transform `f`:

```ts
let map_State = <s,a,b>(f:Fun<a,b>) : Fun<State<s,a>, State<s,b>> =>
  Fun(s => s.then(map_Pair(f,id<s>()))
```

Of course creating a state via `unit_State` will require us to simply create a function which always returns the same value `a`. Such a function is the closest thing to a whole process built for modeling a constant:

```ts
let unit_State = <s,a>() : Fun<a,State<s,a>> =>
  Fun(x => Fun(s => ({ x:x, y:s }))
```

Joining a nested process is also similarly simple,

```ts
let apply = <a, b>(): Fun<Pair<Fun<a, b>, a>, b> =>
  Fun((fa) => fa.fst.f(fa.snd));

let join_State = function <s, a>(): Fun<State<s, State<s, a>>, State<s, a>> {
  return Fun<State<s, State<s, a>>, State<s, a>>(
    (p: State<s, State<s, a>>): State<s, a> => {
      return p.then(apply());
    }
  );
};
```

As a very last addition to the library, consider two basic functions to read and write to the state as a whole:

```ts
let get_state = <s>(): State<s, s> => Fun((s) => ({ x: s, y: s }));
let set_state = <s>(s: s): State<s, Unit> => Fun((_) => ({ x: {}, y: s }));
```

Thanks to these functions, stateful processes will actually be able to access the state when needed!

## A practical application

Let us now get acquainted with the state monad in a simple case: a text-based renderer.

A renderer takes a rendering buffer as input, and produces a new rendering buffer (remember: referential transparency! The actual drawing is done as late as possible) with the extra drawing operations somehow applied to it. Eventually we put the rendering buffer on the actual screen.

This description mirrors the signature of the state monad quite closely, with the only minor difference that we do not use the result parameter but only the state. Moreover, let us assume that rendering in our case means ASCII-rendering, that is drawing figures with special characters on strings. This will keep things visual, but simple.

```ts
type RenderingBuffer = string;
type Renderer = State<RenderingBuffer, Unit>;
```

Let us begin with a very simple primitive renderer, which renders nothing:

```ts
let render_nothing: Renderer = Fun((b) => ({ x: {}, y: b }));
```

Let us then add a very simple primitive renderer, which just adds a string to the buffer:

```ts
let render_string = (s: string): Renderer => Fun((b) => ({ x: {}, y: b + s }));
```

We can then use this function to render some fixed primitives:

```ts
let render_asterisk = render_string("*");
let render_space = render_string(" ");
let render_newline = render_string("\n");
```

Consider now the rendering of a sequence of asterisks of length n:

```ts
let render_line = (n: number): Renderer =>
  n == 0 ? render_asterisk.then(render_line(n - 1)) : render_nothing;
```

We could actually generalize this pattern of repetition of a computation on the state, on the state monad itself, as follows:

```ts
let repeat =
  <s, a>(n: number, f: (_: a) => State<s, a>): ((_: a) => State<s, Unit>) =>
  (a) =>
    n == 0 ? unit_State(a) : f(a).then((a) => repeat(n - 1, f)(a));
```

This would make it possible to elegantly restructure our repeated renderer as follows:

```ts
let render_line = (n: number): Renderer =>
  repeat<RenderingBuffer, Unit>(n, (_) => render_asterisk)({});
```

Similarly, we could define a renderer of, for example, a square, by repetition of the line renderer:

```ts
let render_square = (n:number) : Renderer =>
  repeat<RenderingBuffer,Unit>(n, _ =>
    render_line(n).then(_ => render_newline)({})
```

The actual rendering would then be performed by running a renderer (thus calling its function with some initial buffer) and then printing the state from the resulting tuple:

```ts
console.log(render_square(10).f("").x);
```

## Case study: modeling a tiny DSL

Let us now use the state monad to model a tiny programming language. While a language at this scale is not really useful for much, the structure presented is the core of many complex libraries, making it didactically quite useful.

An instruction produces either a result, or a change in memory, and is always able to read from memory to perform its action. This is modeled with a generic instruction as follows:

```ts
type Memory = Immutable.Map<string, number>;
type Instruction<a> = State<Memory, a>;
```

Basic instructions (unsafe, but we will fix this in the coming chapters) would manipulate variables as follows:

```ts
let get_var = (v:string) : Instruction<number> =>
  get_state().then(m =>
  unit_State(m.get(v))

let set_var = (v:string, n:number) : Instruction<Unit> =>
  get_state().then(m =>
  set_state(m.set(v,n))
```

Let us now define a new instruction that, at the same time, increments and returns a variable:

```ts
let incr_var (v:string) : Instruction<number> =>
  get_var(v).then(v_val =>
  set_var(v, v_val + 1).then(_ =>
  unit_State(v_val)))
```

We can now combine these instructions together in order to build a minimal program, embedded in our application:

```ts
let swap_a_b: Instruction<Unit> = get_var("a").then((a_val) =>
  get_var("b").then((b_val) =>
    set_var("b", a_val).then((_) => set_var("a", b_val))
  )
);
```

Of course the example above is a bit contrived, but it should be clear that whenever we need to build an interpreter of sorts, then this would be the way to go. Interpreters come in handy more often than not: a search/filter API, for example, would need to run some custom instruction, a plugin system would do the same, and so on.
