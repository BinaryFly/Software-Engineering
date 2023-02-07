<!-- index-start -->

## Index

- [Coroutines
  ](#coroutines)
- [Suspensions
  ](#suspensions)
- [Animations
](#animations)
<!-- index-end -->

# Introduction

The processes that we have modeled so far are complete, but offer little in the way of interaction with the rest of the application. For example, suppose that we were modeling a process which takes a really long time to compute, for example a machine-learning process which selects the most likely products a user would buy given his past shopping behavior. The only possible action we can perform given a process is to run it, and thus freeze every other operation until the process is done and produces either a result, or an error.

In this chapter, we will define the last extension to our process modeling framework, so that processes become _interruptible_. Interruptible processes do not necessarily run until completion: they may choose to pause themselves, therefore better integrating with interactive applications. During an interruption of such a process, the application will be able to update animations, handle user input, and so on.

## Coroutines

Interruptible processes are an old and powerful concept, known by many names. _soft thread_, _coroutine_, _green thread_, and many others. We will use _coroutine_ in the rest of the chapter, but it is a bit of an arbitrary choice.

The core idea of coroutines is that running the process will result in either of three possible outcomes: a result, an error, or an interruption. We already know how to handle results and errors, but interruptions are a new concept. An interruption gives us an insight in the state of the coroutine so far, but also tells us how we could resume execution in order to move further towards the completion of the process. This "rest of the process" is quite easily represented as a process itself. This leads us to the following definition:

```ts
type Coroutine<s, e, a> = Fun<s, Either<NoRes<s, e, a>, Pair<a, s>>>;
type NoRes<s, e, a> = Either<e, Continuation<s, e, a>>;
type Continuation<s, e, a> = Pair<s, Coroutine<s, e, a>>;
```

Notice that `Coroutine` has the same structure as the `Process` from the previous chapter: it is an instance of either a result (`Pair<a,s>`, containing both the final result and state), or something that is not a result (`NoRes<s,e,a>`). `NoRes<s,e,a>` is then either an error, or the rest of the process (`Continuation`), which in turn is the current state of the process so far, and the coroutine that would perform the rest of the process.

Let us now define the monadic operators. Let us begin with `unit_Co`, which simply encapsulates the proper result, which is the right hand side of the resulting `Either`:

```ts
let unit_Co = <s, e, a>(x: a): Coroutine<s, e, a> =>
  Fun((s) => inr().f({ x: x, y: s }));
```

Joining a nested coroutine is a tad trickier. This complexity stems from the fact that running a nested coroutine might produce a continuation, which itself is a nested coroutine and must, therefore, be joined. We take some more explicit steps in the implementation in order to illustrate this aspect:

```ts
let join_Co = <s,e,a> : Fun<Coroutine<s,e,Coroutine<s,e,a>>,
  Coroutine<s,e,a>> => Fun(p => Fun(s => {
    let res : Either<NoRes<s,e,Coroutine<s,e,a>>,
                     Pair<Coroutine<s,e,a>,s>> = p.f(s)
    if (res.kind == "left") {
      if (res.value.kind == "left") {
        return inl().then(inl()).f(res.value.value)
      } else {
        let rest : Pair<s,Coroutine<s,e,Coroutine<s,e,a>>> = res.value.value
        return inl().then(inr()).then(map_Pair(id<s>(), join_Co()).f(rest))
      }
    } else {
        let final_res : Pair<s,Coroutine<s,e,a>> = res.value
        return final_res.y.f(final_res .x)
    }
  }))
```

The structure preserving transformation that arises from the fact that a coroutine is a functor looks a lot like `join_Co`, in the sense that the continuation must be recursively transformed as well:

```ts
let map_Co = <s, e, a, b>(
  f: Fun<a, b>
): Fun<Coroutine<s, e, a>, Coroutine<s, e, b>> =>
  Fun((p) =>
    Fun((s) => {
      let res: Either<NoRes<s, e, a>, Pair<a, s>> = p.f(s);
      if (res.kind == "left") {
        if (res.value.kind == "left") {
          return inl().then(inl()).f(res.value.value);
        } else {
          let rest: Pair<s, Coroutine<s, e, a>> = res.value.value;
          return inl()
            .then(inr())
            .then(map_Pair(id<s>(), map_Co(f)).f(rest));
        }
      } else {
        let final_res: Pair<s, a> = res.value;
        return map_Pair(id<s>(), f).f(final_res);
      }
    })
  );
```

## Suspensions

Coroutines are built in order to provide a mechanism for the representation of process which might be interrupted. For example, an interactive process or a process that produces intermediate results which would need to be shown to the user to give an insight of how far a long running computation has come, would be ideally modeled as a process with interruptions.

Our coroutine easily supports interruptions. Interruptions are modeled as a way to construct a coroutine which suspends once, and then (after it is resumed) simply returns a result (of no consequence):

```ts
let suspend = <s,e>() : Coroutine<s,e,Unit> =>
  Fun(s => inl().inr().f({ x:unit_Co({}), y:s })
```

Thanks to this utility function, our coroutines will be able to perform computations, suspend them, and even fail if needed.

## Animations

Armed with our powerful library, it is now possible to build some pretty advanced applications with minimal code complexity. This minimal code complexity is of paramount importance. When the architecture of code does not allow sufficient expressive power, then advanced applications can grow in complexity exponentially over a very short amount of time. This makes some features almost impossible to build in terms of costs, and ultimately results in the unpleasantness associated with working with some legacy applications and their jumble of unreadable code.

Our goal is to build animations, which we will be running on the command line. Of course, it should be noted that such animations are not bound the command line in any way. The same code could be use to animate a more complex application based on, for example, an HTML `Canvas`, an _Angular_ or _Reactjs_ application, or even non-web applications.

Let us begin by defining our rendering operations as coroutines where the state is a rendering string buffer (`R`), and errors are strings (`E`). The short names are chosen because they will be repeated _a lot_:

```ts
type R = string;
type E = string;
type Op<a> = Coroutine<R, E, a>;
```

Drawing a string is a process which terminates right away, but with a modified state with the string to draw concatenated:

```ts
let draw_string = (s: string): Op<Unit> =>
  Fun((r) => inr().f({ x: {}, y: r + s }));
```

Some utilities that will come in handy draw some specific characters that we will draw in various contexts:

```ts
let draw_nothing = draw_string("");
let draw_asterisk = draw_string("*");
let draw_space = draw_string(" ");
let draw_newline = draw_string("\n");
```

We can now draw an alternating line, that is a line where the symbols alternate between asterisks and spaces:

```ts
let draw_alt_line = (c: number, n: number): Op<Unit> =>
  c >= n
    ? draw_nothing
    : (c % 2 == 0 ? draw_asterisk : draw_space).then((_) =>
        draw_alt_line(c + 1, n)
      );
```

By using the alternating line renderer, we can draw a checkerboard square:

```ts
let draw_alt_square = (r: number, n: number): Op<Unit> =>
  r >= n
    ? draw_nothing
    : draw_alt_line(r % 2, n + (r % 2)).then((_) =>
        draw_newline.then((_) => draw_alt_square(r + 1, n))
      );
```

It is now time to draw the actual animation. The animation will draw the square once, then suspend, then draw it with an offset of 1, so that the animation is visible by alternating pixels between space and asterisk, suspend again, and finally repeat itself:

```ts
let draw_animation = (): Op<Unit> =>
  draw_alt_square(0, 5).then((_) =>
    suspend<R, E>().then((_) =>
      draw_alt_square(1, 6).then((_) =>
        suspend<R, E>().then((_) => draw_animation())
      )
    )
  );
```

At this point we can perform the actual rendering. The actual rendering is a reusable function (at least in our rendering context) which takes as input a rendering process, invokes it, draws the result, and repeats the whole process (with a delay of some tens of milliseconds), if the process was not done:

```ts
let run_animation = (a: Op<Unit>) => {
  let res = a.run.f("");
  console.clear();
  if (res.kind == "left" && res.value.kind == "left")
    return console.log("error", res.value.value);
  if (res.kind == "left" && res.value.kind == "right")
    return console.log(res.value.value.snd);
  console.log(res.value.value.snd);
  let rest = res.value.value.fst;
  setTimeout(() => run_animation(rest), 250);
};

run_animation(draw_animation());
```

Note how the code has become linear, and is quite readable. Moreover, the code contains very little unnecessary technical details. Most of the words are actually carrying a lot of meaning, related to the problem and its solution, and is not just machinery for the sake of machinery. By striving towards elegance and expressive power, we can let our creative power roam free and soar to heights previously undreamed of. And this is the power, and beauty, of programming.
