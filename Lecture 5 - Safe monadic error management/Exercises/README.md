# Practical Exercises

## _1_

Build a function that simulates a connection to a Server that might fail. This function takes as input the IP address of the server as a string. The function fails if the IP is invalid or randomly because of network errors. If it succeeds the function returns a data structure representing the content of the server. Use the following data structure to represent a server connection:

```ts
interface ServerConnection {
  ip: string; //ip address
  hello: string; //hello message
}
```

and store a collection of servers in a global variable. Use `Math.random` to simulate a network failure with a probability of 15% (in this case the function returns `none`).

```ts
let connect = (ip: string): Option<ServerConnection> => {...}
```

Build a function that simulates a request to a Server after a successful connection. This function takes as input a `ServerConnection`. The function might fail with a probability of 25% due to server bottlenecks or network errors. If it succeeds it returns the content of the server. Represent the content of the server as

```ts
interface ServerContent {
  ip: string //ip address
  content: string //content message
}

let get = (ip: string): Option<ServerContent> => {...}
```

Build a function that uses the option monad and `bind_Option` to handle the server connection. This function first requests the connection and prints the welcome message from the server. After this it uses the server connection to get the content of the server and prints it.

```ts
let fetch = (ip: string): Option<ServerContent> => {...}
```

<hr>

## _2_

Define an exception type as

```ts
type Exception<a> = Either<string, a>;
```

Use the `Exception` monad to re-implement the functions of Question 1. This time report a message for each separate failure event in the following way:

- In `connect` report a different exception message for a connection failure or an invalid ip. Use two different probabilities for the failures

- In `get` report a different exception message for a connection failure or server unavailability. Use two different probabilities for the failures.

- The function `fetch` should use `bind_Either` in combination with the new versions of `connect` and `get`.

<hr>

## _3_

Adapt the implementation of the `Either` monad to define `bind` as a method `then` of the `Either` data structure. Use this new implementation to refactor what has been done in Question 2.
