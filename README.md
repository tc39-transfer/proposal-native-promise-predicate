# [Promise.isPromise]()

ECMAScript proposal and spec for `Promise.isPromise`.

## Status

[The TC39 Process](https://tc39.es/process-document/)

**Stage**: 2

**Champions**:
- Mathieu Hofman ([@mhofman](https://github.com/mhofman)) (Agoric)

## Motivation

It is currently impossible to check whether an object is a native promise without side effects. The usual pattern is to test whether `Promise.resolve(val) === val`, but in the negative case it creates a new promise resolved from `val`. Besides the unnecessary allocation, it may also trigger user-code related to the [thenable assimilation mechanism](https://promisesaplus.com/#the-then-method).

Code that wants to be defensive, for example avoid synchronous re-entrancy when handling unknown values in async logic, can currently only do so by paying the cost of a tick (`Promise.resolve().then(() => val)`), even when the value is a safe native promise.

Native promises have their internal state safely adopted when doing `await val`, and if https://github.com/tc39/ecma262/pull/3689 is accepted, for other promise resolution paths.

`instanceof Promise` is not appropriate because that checks whether `Promise.prototype` is in the prototype chain of value, not whether the value passes the the `IsPromise` checks the spec performs internally.

## Proposal

A new `Promise.isPromise` predicate that exposes the result of the [`IsPromise`](https://tc39.es/ecma262/#sec-ispromise) abstract operation.

## Precedent

[`Error.isError`](https://tc39.es/ecma262/multipage/fundamental-objects.html#sec-error.iserror) does a similar check for `Error` objects that have magic `stack` behaviour. [`Array.isArray`](https://tc39.es/ecma262/multipage/indexed-collections.html#sec-array.isarray) does a similar check for `Array` objects that have magic `length` behaviour.

## Membrane transparency

Like `Error`, `Promise`s are also objects which are better "passed by copy" than proxied through membranes. Furthermore, it is already possible to asynchronously detect whether an object is a native promise or not through `Promise.resolve()`.

A `Promise.isPromise` predicate would actually let membranes detect promise values which should be recreated on the other side.
