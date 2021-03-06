---
description: The mighty throw new Error("not implemented")
---

# Limitations

Not everything that can be implemented has been implemented already and there are features that are not yet feasible to implement due to the relevant [post-MVP WebAssembly features](https://webassembly.org/docs/future-features/) still being on their way through specification.

## Closures

One of the most limiting factors at this point is the lack of closures, which are part of [WebAssembly GC](https://github.com/WebAssembly/gc/blob/master/proposals/gc/Overview.md#closures) that is still in the works. Take for example

```typescript
function computeSum(arr: i32[]): i32 {
  var sum = 0
  arr.forEach(value => {
    sum += value; // cannot find "sum"
  })
  return sum
}
```

Quite a bummer, right? Of course there are ways to do it differently. One can, for example, make `sum` a global variable that can be accessed everywhere

```typescript
var sum = 0 // now "sum" in global scope
function computeSum(arr: i32[]): i32 {
  arr.forEach(value => {
    sum += value
  })
  return sum
}
```

or write this snippet differently:

```typescript
let sum = 0
for (let i = 0; i < arr.length; ++i) {
  sum += arr[i] // works
}
```

It's still necessary to figure out how to go forward with these, like whether we should implement something on our own or wait for the specification to land.

## Exceptions

[WebAssembly exception handling](https://github.com/WebAssembly/exception-handling) is not available yet, so the following will currently `abort` the program:

```typescript
function doThrow(): void {
  throw new Error(":(")
}
```

Also means: `try` and `catch` are not supported yet.

## OOP

Another major bummer is that object oriented programming is still missing some of the building blocks to support interfaces \(here: virtual members in general\). One can have classes possibly extending other classes, though, but will hit another limitation sooner or later:

```typescript
class A {
  foo(): void {}
}

class B extends A {
  foo(): void {}
}

var a: A = new B()
a.foo() // calls A#foo, not the overloaded B#foo
```

Again, there are ways to write code that does the right thing.

```typescript
if (a instanceof B) {
  let b = a as B
  b.foo()
}
```

Most of this can be implemented without the respective WebAssembly features being ready, yet is a lot of work, not as efficient as one might expect and will be thrown away again once there is something better, but we are working on it.

