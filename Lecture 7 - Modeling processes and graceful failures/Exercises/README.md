# Questions

## _1_

Given the `Process` monad that allows to have stateful computations with errors, complete the missing types (denoted with \_\_\_) in its implementation

```ts
type Process<___> = ___

let unit_Process = <___>(): Fun<___,___> => {...}
```

**A**:

```ts
type Process<s, e, a> = Fun<s, Either<e, Pair<a, s>>>

let unit_Process = <s, e, a>(): Fun<a, Either<e, Pair<a, s>>> => {...}
```

**B**:

```ts
type Process<s, e, a> = Fun<s, Either<e, Pair<a, s>>>

let unit_Process = <s, e, a>(): Fun<a, Process<s, e, a>> => {...}
```

**C**:

```ts
type Process<s, e, a> = Either<e, Pair<a, s>>

let unit_Process = <s, e, a>(): Fun<a, Either<e, Pair<a, s>>> => {...}
```

**D**: None of the definitions above are correct.

<details><summary>Show Answer</summary>
<p>

```
B
```

</p>
</details>
<br>
<hr>

## _2_

Given the `Process` monad that allows to have stateful computations with errors, complete the missing types (denoted with \_\_\_) in the implementation of its `join_Process` function

```ts
let join_Process = <___>(): Fun<___,___> => {...}
```

**A**:

```ts
let join_Process = <a>(): Fun<Process<Process<a>>,Process<a>> => {...}
```

**B**:

```ts
let join_Process = <s, a>(): Fun<Process<s, Process<s, a>>,Process<s, a>> => {...}
```

**C**:

```ts
let join_Process = <s, e, a>(): Fun<Process<s, e, Process<s, e, a>>,Process<s, e, a>> => {...}
```

**D**:

```ts
let join_Process = <s, e, a, b>(f: Fun<a, b>): Fun<Process<s, e, a>,Process<s, e, b>> => {...}
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

Given the following type and function definitions using the `Process` monad:

```ts
type Memory = Immutable.Map<string, number>
type Statement<a> = Process<Memory, string, a>

export let getVar = (_var: string): Statement<number> => {...}
```

what is the type of the expression `getVar("x")`?

**A**: `Either<string,Pair<number,Memory>>`

**B**: `Fun<Memory, Either<string,Pair<number, Memory>>>`

**C**: `Fun<number, Either<string,Pair<number, Memory>>>`

**D**: `Pair<number, Memory>`

<details><summary>Show Answer</summary>
<p>

```
B
```

</p>
</details>
<br>
<hr>

# Practical Exercises

## _4_

Extend the imperative language seen in Unit 6 with the `State` monad to support different kind of values for variables instead of just numbers:

```ts
type Value =
  | {
      kind: "number";
      value: number;
    }
  | {
      kind: "string";
      value: string;
    }
  | {
      kind: "bool";
      value: boolean;
    };
```

In this new implementation

```ts
type Memory = Immutable.Map<string, Value>;
type Instruction<a> = Process<Memory, string, a>;
```

Use the `Process` monad to implement the conditional statement for the language with the following new semantics:

The condition can now be `Instruction<Value>` and both booleans and numbers can be used. If a number is used, then the condition is true if the number is positive, otherwise it is considered false. If anything else other than a number or a boolean is provided, then the function returns a type error.

```ts
let ifThenElse = (condition: Instruction<Value>, _then: Instruction<Unit>, _else: Instruction<Unit>):
  Instruction<Unit> => {...}
```

<hr>

## _5_

Adapt the loop instruction of the imperative language built with `State` to now use the `Process` monad. The `condition` of the loop now has the same semantics explained in the previous question.

```ts
let _while = (condition: Instruction<Value>, body: Instruction<Unit>): Instruction<Unit> => {...}
```
