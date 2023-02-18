# Questions

## _1_

Given a monad `Monad<a>`, which one is the correct implementation of its `bind_Monad` function?

**A**:

```ts
let bind_Monad = <a, b>(m: Monad<a>, f: Fun<a, Monad<b>>): Monad<b> =>
  join_Monad<a>().then(map_Monad<a, Monad<b>>(f)).f(m);
```

**B**:

```ts
let bind_Monad = <a, b>(m: Monad<a>, f: Fun<a, Monad<b>>): Monad<b> =>
  map_Monad<a, Monad<b>>(f).p(m);
```

**C**:

```ts
let bind_Monad = <a, b>(m: Monad<a>, f: Fun<a, Monad<b>>): Monad<b> =>
  map_Monad<a, Monad<b>>(f).then(join_Monad<b>()).f(m);
```

**D**:

```ts
let bind_Monad = <a, b>(m: Monad<a>, f: Fun<a, Monad<b>>): Monad<b> =>
  map_Monad<a, Monad<b>>(f).then(join_Monad<b>());
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

Given the monad `Either<a, b>` and the function

```ts
let mistery_Function = (v1: number, v2: string): Either<number, string> => {...}
```

complete the missing types denoted with \_\_\_ in the following code:

```ts
let foo = (v: number): Either<___> =>
  bind_Either(
    mistery_Function(v),
    Fun((x1: ___) =>
      bind_Either(
        mistery_Function(x1 * 2),
        Fun((x2: ___) => unit_Either<___>().f(x2 / 3))
      )
    )
  );
```

**A**: All the other answers are wrong

**B**:

```ts
let foo = (v: number): Either<number, string> =>
  bind_Either(
    mistery_Function(v),
    Fun((x1: string) =>
      bind_Either(
        mistery_Function(x1, "##"),
        Fun((x2: string) => unit_Either<string, number>().f(x2))
      )
    )
  );
```

**C**:

```ts
let foo = (v: number): Either<number, string> =>
  bind_Either(
    mistery_Function(v),
    Fun((x1: Either<number, string>) =>
      bind_Either(
        mistery_Function(x1 + "##"),
        Fun((x2: Either<number, string>) =>
          unit_Either<string, number>().f(x2 + "??")
        )
      )
    )
  );
```

**D**:

```ts
let foo = (v: number): Either<a, b> =>
  bind_Either(
    mistery_Function(v),
    Fun((x1: Either<a, b>) =>
      bind_Either(
        mistery_Function(x1 + "##"),
        Fun((x2: Either<a, b>) => unit_Either<a, b>().f(x2 + "??"))
      )
    )
  );
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

Consider the monad `Option<a>` seen in class, complete the missing types (denoted with \_\_\_) in the implementation of its `map_Option` function:

```ts
type Option<a> = {
  kind: "none"
} | {
  kind: "some"
  value: a
}

let map_Option = <___>(f: Fun<___>): Fun<___>
```

**A**:

```ts
let map_Option = <a, b>(f: Fun<a, b>): Fun<a, Option<b>> => {...}
```

**B**:

```ts
let map_Option = <a, b>(f: Fun<a, b>): Option<a> => {...}
```

**C**:

```ts
let map_Option = <a, b>(f: Fun<a, b>): Option<b> => {...}
```

**D**:

```ts
let map_Option = <a, b>(f: Fun<a, b>): Fun<Option<a>, Option<b>> => {...}
```

<details><summary>Show Answer</summary>
<p>

```
D
```

</p>
</details>
<br>
<hr>

# Practical Exercises

## _4_

Implement the bind operator for the `List` monoid:

```ts
let bind = function<a, b>(k: Fun<a, List<b>>): Fun<List<a>,List<b>> {...}
```

<hr>

## _5_

Define the `Set` functor. A set is a data structure containing non-repeated elements of type `a`. Implement the `map_Set` function to give it a functorial structure:

```ts
let map_Set = <a, b>(f: Fun<a, b>): Fun<Set<a>, Set<b>>
```

Extend the `Set` functor with `unit` and `join` to give it a monoidal structure

```ts
let unit_Set = <a>(): Fun<a, Set<a>>
let join_set = <a>(): Fun<Set<Set<a>>, Set<a>>
```

Extend the `Set` monoid with `bind` to give it a monadic structure

```ts
let bind = <a, b>(k: Fun<a, Set<b>>): Fun<Set<a>,Set<b>>
```

<hr>

## _6_ (hard)

Create a `DoublyLinkedList<a>` functor. A doubly linked list is a data structure with two cases:

1. An empty list.
2. An element preceeded by a doubly linked list and a followed by another doubly linked list.

Implement the `map_DoublyLinkedList` function for a Binary Tree.

```ts
let map_DoublyLinkedList = <a, b>(_: Fun<a, b>) :
Fun<DoublyLinkedList<a>, DoublyLinkedList<b>>
```

Extend this functor with `join` and `unit` and give it a monoidal structure.

```ts
let unit_DoublyLinkedList = <a>() : Fun<a, DoublyLinkedList<a>>
let join_DoublyLinkedList = <a>():
  Fun<DoublyLinkedList<DoublyLinkedList<a>>,
      DoublyLinkedList<a>>
```

Define `bind` for the BinaryTree to make it a monad

```ts
let bind_DoublyLinkedList = <a, b>(k: Fun<a,DoublyLinkedList<b>>) :
  Fun<DoublyLinkedList<a>,DoublyLinkedList<b>>
```
