<!-- index-start -->

## Index

- [An applied example
](#an-applied-example)
<!-- index-end -->

# Error management

Consider the state monad that we discussed in the previous chapter. This monad is based on the assumption that a process will always succeed, and therefore produce both a new state `s`, and a result `a`.

Of course, this sort of luxury is not always part of the case at hand. Processes in real life fail, and therefore return no result. Fortunately, we have already modeled absence of result and presence of exceptions, via `Either`.

Therefore, we could consider combining the `Either` and `State` monads into a single monad, which we will call `Process`:

```ts
type Process<s, e, a> = Fun<s, Either<e, Pair<a, s>>>;
```

This monad could be seen as a state, where the result might be replaced by some arbitrary error `e`. The operators will therefore borrow either the general idea, or outright make us of, the implementations of the operators, of both `State` and `Either`. Let us begin with `unit_Process`, which invokes `unit_Either` to encapsulate the result as needed:

```ts
let unit_Process = <s, e, a>(): Fun<a, Process<s, e, a>> =>
  Fun((x: a) =>
    Fun((state: s) => unit_Either<Pair<a, s>, e>().f(Pair<a, s>(x, state)))
  );
```

Joining processes requires launching a process, then propagating the possible errors (the left hand side of `map_Either`) and then joining the resulting nested `Either` after the nested inner process has been evaluated as well.

```ts
let join_Process = <s, e, a>(): Fun<
  Process<s, e, Process<s, e, a>>,
  Process<s, e, a>
> =>
  Fun<Process<s, e, Process<s, e, a>>, Process<s, e, a>>(
    (p: Process<s, e, Process<s, e, a>>) =>
      p.then(map_Either(id<e>(), apply())).then(join_Either())
  );
```

The final combinator is transforming processes, which simply transforms the resulting `Either`:

```ts
let map_Process = <s, e, a, b>(
  f: Fun<a, b>
): Fun<Process<s, e, a>, Process<s, e, b>> =>
  Fun<Process<s, e, a>, Process<s, e, b>>((p: Process<s, e, a>) =>
    p.then(map_Either(id<e>(), map_Pair(f, id<s>())))
  );
```

## An applied example

Let us now consider an example application of our `Process` monad. Let us define errors as simply strings, and the state as, once again, a memory map:

```ts
type Memory = Immutable.Map<string, number>;
type Error = string;
type Instruction<a> = Process<Memory, Error, a>;
```

The core instructions therefore become reading and writing to memory, but this time, when the variable name is not available, we can gracefully handle the error:

```ts
let get_var = (v: string): Instruction<number> =>
  get_state().then((m) =>
    m.has(v)
      ? unit_Process(m.get(v))
      : inl(`Error: variable ${v} does not exist`)
  );

let set_var = (v: string, n: number): Instruction<Unit> =>
  get_state().then((m) => set_state(m.set(v, n)));
```

We can also define error recovery mechanisms. For example, we could try running an instruction, and if it fails, running another instruction. This is the same mechanism as exception handling:

```ts
let try_catch = <a>(p: Instruction<a>, q: Instruction<a>): Instruction<a> =>
  Fun((m) => p.then(map_Either(q.f(m), id())).f(m));
```

We can now build a simple program which gracefully handles missing variables:

```ts
let swap_a_b = () : Instruction<Unit> =>
  try_catch(get_var("a"), set_var("a", 0).then(get_var("a")).then(a_val =>
  try_catch(get_var("b"), set_var("b", 0).then(get_var("b")).then(b_val =>
  set_var("a", b_val).then(
  set_var("b", a_val))))
```
