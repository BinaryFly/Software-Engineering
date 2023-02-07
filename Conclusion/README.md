# Conclusion

In the course of the previous chapters we have explored some advanced design patterns, which help us when building complex programs which enjoy some powerful guarantees in terms of correctness and the ability to combine sub-programs into bigger programs without encountering unexpected side effects all of a sudden.

We started by discussing the importance of type safety and referential transparency, the two cornerstones for all implementations in subsequent chapters.

**Type safety** basically means that the shape of data and computations must be known at compile time, and that the compiler must be in state to verify whether or not the implementation matches these shapes and structures.

**Referential transparency** means that we do use mutation very carefully, and never leaking outside the local scope of a function. This guarantees that the same function, given the same parameters, will always work the same way, and that there will be no sharing of mutable state which might cause dramatic consequences such as a variable being used in two unrelated, logically incompatible, processes at the same time.

We then defined the first design pattern, **functors**, which encapsulate all generic data containers, processes, and more, by specifying how they must be able to transform when a mapping for their generic argument is known. The transformation must _preserve the structure_ of the functor, meaning that composition of functors and identity all map along specific patterns.

We then moved on to an apparently simple concept, that of **monoids**, which seems to pop up in a lot of places: strings, numbers, lists, and even more. By extending monoids from being defined on functors instead of concrete types, we achieved a definition which is far more general and applicable to even more contexts.

This generalization of monoids built on functors is called **monads**. The fact that monads are, at their core, monoids, and the fact that monoids tend to naturally arise everywhere, suggested that monads might be used to manipulate a lot of concepts, potentially very unrelated from each other.

We then explored some of these concepts that can be represented in the form of monads.

We began with **Option**, which allows us to manage errors and missing information in a type safe manner.

We then extended error management to also contain an error payload, thereby giving rise to **Either**.

Having shown some data structures and how they can be modeled with monads, we moved on to modeling processes. The ability to model processes within type safe applications yields great expressive power to developers, since processes are usually seen as something that is only modeled implicitly with code, and not explicitly with well crafted structures that enjoy a well defined interface.

We started with **State**, a simple model of processes with state and incapable of gracefully handling failure. We used `State` to implement a small domain specific language.

Combining **State and Either** gave us the ability to model stateful processes which explicitly and gracefully handle failure, within the boundaries of type safety.

By further adding Continuations, we defined **Coroutine**: a stateful process which handles failure gracefully, but which can also suspend in order to actively cooperate
