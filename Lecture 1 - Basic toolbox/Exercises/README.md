## _1_

Select the **False** sentence about a referentially-transparent function

**A**: he result of the function might change depending on the point of the program it is invoked from

**B**: Its result only depends on the input arguments

**C**: Its result does not depend on the point of the program it is invoked from

**D**: Its calls can always be replaced by its result in any part of the program

<details><summary>Show Answer</summary>
<p>

```
A
```

</p>
</details>
<br>
<hr>

## _2_

Given the following functions:

```ts
let incr = Func<number, number>((x) => x + 1);
let convert = Func<number, string>((x) => String(x));
```

Which of the following expressions is equivalent to the result of their composition

```ts
incr.then(convert);
```

**A**:

```ts
Func<number, string>((x) => String(x));
```

**B**:

```ts
Func<number, string>((x) => String(x + 1));
```

**C**: The complier throws a type error

**D**:

```ts
String(x + 1);
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

Given the following functions:

```ts
let convert = Func<number, string>((x) => String(x));
let square = Func<number, number>((x) => x * x);
```

Which expression is equivalent to their composition?

```ts
convert.then(square);
```

**A**:

```ts
Func<number, string>((x) => x * x);
```

**B**: That composition is invalid

**C**:

```ts
String(x * x);
```

**D**:

```ts
x * x;
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

## _4_

Given the following functions:

```ts
let incr = Func<number, number>((x) => x + 1);
let double = Func<number, number>((x) => x * 2);
```

Which of the following expressions is equivalent to `incr.then(double)`?

**A**:

```ts
Func<number, number>((x) => (x + 1) * 2);
```

**B**: That composition is invalid

**C**:

```ts
(x + 1) \* 2
```

**D**:

```ts
(x) => (x + 1) * 2;
```

<details><summary>Show Answer</summary>
<p>

```
A
```

</p>
</details>
<br>
<hr>

## _5_

Given the following functions:

```ts
let incr = Func<number, number>((x) => x + 1);
let square = Func<number, number>((x) => x * x);
```

Which of the following expressions is equivalent to `incr.then(square).f(4)`?

**A**:

```ts
(x) => (x + 1) * (x + 1);
```

**B**: 25

**C**:

```ts
Fun<number, number>((x) => 25);
```

**D**:

```ts
Fun<number, number>((x) => (x + 1) * (x + 1));
```

<details><summary>Show Answer</summary>
<p>

```
B
```

</p>
</details>
<br>
