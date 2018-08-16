---
title: Fast Pipe
---

`->` is a convenient operator that allows you to "flip" your code inside-out. `a(b)` becomes `b->a`. It's a piece of syntax that doesn't have any runtime cost.

Imagine you have the following:

```reason
validateAge(getAge(parseData(person)))
```

This is slightly hard to read, since you need to read the code from the innermost part, to the outer parts. Use fast pipe to streamline it

```reason
person
  ->parseData
  ->getAge
  ->validateAge
```

Basically, `parseData(person)` is transformed into `person->parseData`, and `getAge(person->parseData)` is transformed into `person->parseData->getAge`, etc.

**This works when the function takes more than one argument too**. 

```reason
a(one, two, three)
``` 

is the same as

```reason
one->a(two, three)
```

This works with labeled arguments too.

### Tips & Tricks

Try not to abuse pipes; they're a mean to an end. Newcomers sometimes shape a library's API to take advantage of the pipe. This is rather backward.

Conventionally, we don't turn the innermost layer of function call into a pipe. So the above example would usually be written as:

```reason
parseData(person)
  ->getAge
  ->validateAge
```

## JS Method Chaining

_This section requires understanding of [Bucklescript's binding API](https://bucklescript.github.io/docs/en/function#object-method)_.

JavaScript's APIs are often attached to objects, and often chainable, like so:

```js
const result = [1, 2, 3].map(a => a + 1).filter(a => a % 2 === 0);

asyncRequest().setWaitDuration(4000).send();
```

Assuming we don't need the chaining behavior above, we'd bind to each case this using `bs.send` from the previous section:

```reason
[@bs.send] external map : (array('a), 'a => 'b) => array('b) = "";
[@bs.send] external filter : (array('a), 'a => 'b) => array('b) = "";

type request;
external asyncRequest: unit => request = "";
[@bs.send] external setWaitDuration: (request, int) => request = "";
[@bs.send] external send: request => unit = "";
```

You'd use them like this:

```reason
let result = filter(map([|1, 2, 3|], a => a + 1), a => a mod 2 == 0);

send(setWaitDuration(asyncRequest(), 4000));
```

This looks much worse than the JS counterpart! Now we need to read the actual logic inside-out. We also cannot use the `|>` operator here, since the object comes _first_ in the binding. But `->` works!

```reason
let result = [|1, 2, 3|]
  ->map(a => a + 1)
  ->filter(a => a mod 2 === 0);

asyncRequest()->setWaitDuration(400)->send;
```

## Pipe Into Variants

There are good justifications on why a variant's contructors are not function calls. Though it's a bit inconvenient practically still. For example, it'd be nice to be able to
Variant constructors not being functions are sometimes a bit inconvenient. We've piggy-backed on top of fast pipe to make this work:

```reason
let result = name->preprocess->Some
```

We turn this into:

```reason
let result = Some(preprocess(name))
```
