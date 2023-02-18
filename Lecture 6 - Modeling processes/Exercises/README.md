# Questions

## _1_

Complete the missing types (denoted with \_\_\_) in the implementation of the `State` monad (modelling the control flow of a sequence of statements) and its `unit_State` function

```ts
type State<___> = ___

let unit_State = <___>(): Fun<___,___> => {...}
```

**A**:

```ts
type Statement<s, a> = Fun<s, Pair<a, s>>

let unit_State = <s, a>(): Fun<State<s, State<s, a>>, State<s, a>> => {...}
```

**B**:

```ts
type Statement<s, a> = Pair<a, s>

let unit_State = <s, a>(): Fun<a, Pair<a, s>> => {...}
```

**C**:

```ts
type Statement<s, a> = Fun<s, Pair<a, s>>

let unit_State = <s, a>(): Fun<State<s, a>, State<s, b>> => {...}
```

**D**:

```ts
type Statement<s, a> = Fun<s, Pair<a, s>>

let unit_State = <s, a>(): Fun<a, State<s, a>> => {...}
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

## _2_

Complete the missing types (denoted with \_\_\_) in the implementation of the `State` monad (modelling the control flow of a stateful application) and its `join_State` function

```ts
type State<___> = ___

let join_State = <___>(): Fun<State<___>,State<___>> => {...}
```

**A**:

```ts
type State<s, a> = Fun<s, Pair<a, s>>

let join_State = <a>(): Fun<State<State<a>>,State<a>> => {...}
```

**B**:

```ts
type State<s, a> = Pair<a, s>

let join_State = <a>(): Fun<State<s, State<s, a>>,State<s, a>> => {...}
```

**C**:

```ts
type State<s, a> = Fun<s, Pair<a, s>>

let join_State = <s, a>(): Fun<State<s, State<s, a>>,State<s, a>> => {...}
```

**D**:

```ts
type State<s, a> = Fun<s, Pair<s, s>>

let join_State = <s, a>(): Fun<State<s, State<s, a>>,State<s, a>> => {...}
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

## _3_

Given the following functions using the `State` monad

```ts
type Memory = Immutable.Map<string, number>;
type Instruction<a> = State<Memory, a>;

let get_State = <s>(): State<s, s> => {
  return Fun<s, Pair<s, s>>((state: s) => Pair(state, state));
};

let getVar = (_var: string): Statement<number> => {
  return bind_State(
    get_State(),
    Fun((m: Memory) => {
      let m1 = m.get(_var);
      return unit_State<Memory, number>().f(m1);
    })
  );
};
```

what is the type of the expression `getVar("x")`?

**A**:

```ts
Fun<Memory, Pair<Memory, number>>;
```

**B**:

```ts
Fun<Memory, Pair<Unit, Memory>>;
```

**C**:

```ts
Fun<Memory, Pair<number, Memory>>;
```

**D**:

```ts
Fun<number, Pair<number, Memory>>;
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

Extend the imperative language defined in class with the `State` monad to include the evaluation of a code block:

```ts
let seq = (current: Instruction<Unit>, next: Instruction<Unit>): Instruction<Unit> => {...}
```

A code block always terminates with a special statement that does nothing. This is also used to define empty blocks

```ts
let skip = (): Instruction<Unit> => unit_State<Memory, Unit>().f({});
```

<hr>

## _5_

Extend the imperative language seen in class with the `State` monad to include a conditional statement:

```ts
let ifThenElse = (condition: Instruction<boolean>, _then: Instruction<Unit>,
  _else: Instruction<Unit>): Instruction<Unit> => {...}
```

This statement uses `bind_State` to evaluate the condition. If the condition evaluates to true it runs `_then` otherwise it runs `_else`. Note that `_then` and `_else` may be code blocks (see Question 1).

<hr>

## _6_

Extend the imperative language defined with the `State` monad to include a loop instruction.

```ts
let _while = (condition: Instruction<boolean>, body: Instruction<Unit>):
  Instruction<Unit> => {...}
```

This function uses `bind_State` to evaluate the condition. If the result is true then `body` is evaluated and then recursively the whole `_while` is re-evaluated again. If the condition is false the statement does nothing (you can use `skip` in this case).

<hr>

## _7_ (hard)

Adapt the implementation of the `State` monad to include a method then in the data structure representing `State` such that it becomes:

```ts
interface State<s, a> {
  run: Fun<s, Pair<a, s>>;
  then: (f: Fun<a, State<s, b>>) => State<s, b>; //this is bind
}
```
