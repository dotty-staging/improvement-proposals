---
layout: sip
permalink: /sips/:title.html
stage: implementation
status: waiting-for-implementation
presip-thread: https://contributors.scala-lang.org/t/pre-sip-foo-bar/9999
title: SIP-NN - Natural order right-associativity of extension methods
---

**By: Nicolas Stucki**

## History

| Date          | Version            |
|---------------|--------------------|
| Dec 6th 2022  | Initial Draft      |

## Summary

The current design of right-associativity of extension methods leads to many unnecessary limitations, unintuitive behavior, and complexity.
The idea is to introduce improved semantics for right-associativity of extension methods.
At its core, the only change is to not use the current special rule for right-associativity of extension methods, then everything else follows trivially.

The rule in question swaps the receiver and the argument from right-associativity of extension method definitions. This leads into a few complications reordering
type parameters and contextual parameters as well. The original idea was to make the parameter order align with the argument order (when used as an infix operator).

```scala
extension (x: X) def ::(y: Y): Z = ...
// currently is equivalent to
extension (y: Y) def ::(x: X): Z = ...
// idea avoid the special rule
```

```scala
def example1(x: X, y: Y): Z = x :: y // becomes a call to ::(y)(x)
```

## Motivation

Some mistakes where made when designing right associativity of extension methods. This leads to many unnecessary limitations, unintuitive bevavour, and complexity.
The original idea was to make the parameter order align with the argument order (when used as an infix operator).
This is to make it look like we do not have [SLS §6.12.3](https://www.scala-lang.org/files/archive/spec/2.13/06-expressions.html#infix-operations).

### Problem 1
This works in `example1` but break as soon as we try to call the extension method as an non-infix method.

```scala
def example2(x: X, y: Y): Z = x.::(y) // error, now we should use y.::(x)
def example3(x: X, y: Y): Z = ::(x)(y) // error, now we should use ::(y)(x)
```
In these two example we now have an inversion of the arguments that we should not have.

### Problem 2




A high-level overview of the proposal with:

- An explanation of the problems or limitations that it aims to solve,
- A presentation of one or more use cases as running examples, with code showing how they would be addressed *using the status quo* (without the feature), and why that is not good enough.

This section should clearly express the scope of the proposal. It should make it clear what are the goals of the proposal, and what is out of the scope of the proposal.

## Proposed solution

This is the meat of your proposal.

### High-level overview

A high-level overview of the proposed changes, and how they allow to better solve the running examples. This section should be example-heavy, and not dive into corner cases.

Example:

~~~ scala
// This is an @main method
@main def foo(x: Int): Unit =
  println(x)
~~~

### Specification

A specification for the proposed changes, as precise as possible. This section should address difficult interactions with other language features, possible error conditions, and corner cases as much as the good behavior.

For example, if the syntax of the language is changed, this section should list the differences in the grammar of the language. If it affects the type system, the section should explain how the feature interacts with it.

### Compatibility

A justification of why the proposal will preserve backward binary and TASTy compatibility. Changes are backward binary compatible if the bytecode produced by a newer compiler can link against library bytecode produced by an older compiler. Changes are backward TASTy compatible if the TASTy files produced by older compilers can be read, with equivalent semantics, by the newer compilers.

If it doesn't do so "by construction", this section should present the ideas of how this could be fixed (through deserialization-time patches and/or alternative binary encodings). It is OK to say here that you don't know how binary and TASTy compatibility will be affected at the time of submitting the proposal. However, by the time it is accepted, those issues will need to be resolved.

This section should also argue to what extent backward source compatibility is preserved. In particular, it should show that it doesn't alter the semantics of existing valid programs.

### Other concerns

If you think of anything else that is worth discussing about the proposal, this is where it should go. Examples include interoperability concerns, cross-platform concerns, implementation challenges.

### Open questions

If some design aspects are not settled yet, this section can present the open questions, with possible alternatives. By the time the proposal is accepted, all the open questions will have to be resolved.

## Alternatives

This section should present alternative proposals that were considered. It should evaluate the pros and cons of each alternative, and contrast them to the main proposal above.

Having alternatives is not a strict requirement for a proposal, but having at least one with carefully exposed pros and cons gives much more weight to the proposal as a whole.

## Related work

This section should list prior work related to the proposal, notably:

- A link to the Pre-SIP discussion that led to this proposal,
- Any other previous proposal (accepted or rejected) covering something similar as the current proposal,
- Whether the proposal is similar to something already existing in other languages,
- If there is already a proof-of-concept implementation, a link to it will be welcome here.

## FAQ

This section will probably initially be empty. As discussions on the proposal progress, it is likely that some questions will come repeatedly. They should be listed here, with appropriate answers.
