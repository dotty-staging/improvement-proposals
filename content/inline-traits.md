---
layout: sip
permalink: /sips/:title.html
stage: implementation
status: waiting-for-implementation
presip-thread: https://contributors.scala-lang.org/t/pre-sip-foo-bar/9999
title: SIP-NN - Inline Traits
---

**By: Author A and Author B and Author C**

## History

| Date          | Version            |
|---------------|--------------------|
| Feb 19th 2022 | Initial Draft      |

## Summary

<!--
A summary of the proposed changes. This should be no longer than 3 paragraphs. It is intended to serve in two ways:

- For a first-time reader, a high-level overview of what they should expect to see in the proposal.
- For returning readers, a quick reminder of what the proposal is about. -->

## Motivation

A high-level overview of the proposal with:

- An explanation of the problems or limitations that it aims to solve,
- A presentation of one or more use cases as running examples, with code showing how they would be addressed *using the status quo* (without the feature), and why that is not good enough.

This section should clearly express the scope of the proposal. It should make it clear what are the goals of the proposal, and what is out of the scope of the proposal.

## Proposed solution

This is the meat of your proposal.

### High-level overview

An inline trait is a trait that will inline its definitions in the class, trait or object that extends it. In the following example, we have the implementation of a field and a method, and we have an abstract method.

```scala
inline trait T:
  val x: Int = 42
  def f: Int = g(x)
  def g(i: Int): Int

class C extends T:
  def g(i: Int): Int = i + 1
```

The implementations of `x` and `f` are inlined in `C`. As they are inlined, they are not needed in the trait anymore. The resulting code after inlining is:

```
trait T:
  val x: Int
  def f: Int
  def g(i: Int): Int

class C extends T:
  val x: Int = 42 // inlined
  def f: Int = g(x) // inlined
  def g(i: Int): Int = i + 1
```


### Specification

<!-- A specification for the proposed changes, as precise as possible. This section should address difficult interactions with other language features, possible error conditions, and corner cases as much as the good behavior.

For example, if the syntax of the language is changed, this section should list the differences in the grammar of the language. If it affects the type system, the section should explain how the feature interacts with it. -->

#### Methods in inline traits

##### Source
```scala
inline trait InlineTrait:
  def f(x1: T1, ..., xn: Tn): R = ...

class C extends InlineTrait
```
##### After typing (pickled)

When typing a class that extends an inline trait, the non-abstract methods of the inline trait are identified and a forwarder is created for each one of them.
The forwarder is a method that calls the implementation of the method in the inline trait.

```scala
inline trait InlineTrait:
  // annotated as inlinable but not inline (only inlined in extending class)
  def f(x1: T1, ..., xn: Tn): R = ...

class C extends InlineTrait:
  override def f(x1: T1, ..., xn: Tn): R = super.f(x1, ..., xn)
```

The purpose of the forwarder is to generate the method signature in the class.
This can be useful to avoid unnecessary virtual calls. For example a call to `c.f(...)` on a `c: C` will be typed as calling `C.f` directly.
If the trait is generic, this can also allow calls to used the non-boxed version of the method.


##### Inlining phase

The call in the forwarder is inlined as if it was a call to an inline method.

```scala
class C extends InlineTrait:
  def f(x1: T1, ..., xn: Tn): R = ...
```

All the implementations in `InlineTrait` are made abstract.
These methods are guaranteed to have an implementation in the subclass.

```scala
inline trait InlineTrait:
  def f(x1: T1, ..., xn: Tn): R
```

#### Abstract methods in inline traits

Inline traits can have abstract methods.
These are handled as regular abstract methods.

```scala
inline trait InlineTrait:
  def g(x1: T1, ..., xn: Tn): R

class C extends InlineTrait:
  def g(x1: T1, ..., xn: Tn): R = ...
```

#### Overridden methods in inline traits

A class can override a method defined in an inline trait.
Nothing needs to be generated in the class in this case.

```scala
inline trait InlineTrait:
  def f(x1: T1, ..., xn: Tn): R = ...

class C extends InlineTrait:
  override def f(x1: T1, ..., xn: Tn): R = ...
```

#### Final methods in inline traits

Inline trait methods can be final.
They are typed the same way as non-final methods, but we need to allow the override of final methods with the forwarder.

```scala
inline trait InlineTrait:
  final def f(x1: T1, ..., xn: Tn): R = ...

class C extends InlineTrait:
  override final def f(x1: T1, ..., xn: Tn): R = super.f(x1, ..., xn)
```

The final modifier must be removed from `InlineTrait.f` after inlining.

#### Implementing or overriding methods in inline traits

An inline trait can implement or override a method from a parent trait.
These are handled as regular methods in inline traits.

```scala
trait Trait:
  def f(x1: T1, ..., xn: Tn): R
  def g(x1: T1, ..., xn: Tn): R = ...

inline trait InlineTrait extends Trait:
  def f(x1: T1, ..., xn: Tn): R = ...
  override def f(x1: T1, ..., xn: Tn): R = ...
```

#### Protected and package private methods in inline traits

An inline trait can implement or override a method from a parent trait.
The forwarders will be generated with the same visibility as the method in the inline trait.

```scala
inline trait InlineTrait:
  protected def f(x1: T1, ..., xn: Tn): R = ...
  private[P] def g(x1: T1, ..., xn: Tn): R = ...
```


#### Private methods in inline traits

Inline trait methods can be private.
Support for these is a bit more complex because of the scope in which private methods are visible.

```scala
inline trait InlineTrait:
  private def f(x1: T1, ..., xn: Tn): R = ...
  def g(x1: T1, ..., xn: Tn): R = ... f(...) ...

class C extends InlineTrait
```

```scala
inline trait InlineTrait:
  protected def packagename$InlineTrait$f(x1: T1, ..., xn: Tn): R = ...
  def g(x1: T1, ..., xn: Tn): R = ... packagename$InlineTrait$f(...) ...

class C extends InlineTrait:
  protected def packagename$InlineTrait$f(x1: T1, ..., xn: Tn): R = super.packagename$InlineTrait$f(x1, ..., xn)
  def g(x1: T1, ..., xn: Tn): R = super.g(x1, ..., xn)
```

```scala
inline trait InlineTrait:
  def g(x1: T1, ..., xn: Tn): R

class C extends InlineTrait:
  private def packagename$InlineTrait$f(x1: T1, ..., xn: Tn): R = ...
  def g(x1: T1, ..., xn: Tn): R = ... packagename$InlineTrait$f(...) ...
```



### Compatibility

<!-- A justification of why the proposal will preserve backward binary and TASTy compatibility. Changes are backward binary compatible if the bytecode produced by a newer compiler can link against library bytecode produced by an older compiler. Changes are backward TASTy compatible if the TASTy files produced by older compilers can be read, with equivalent semantics, by the newer compilers.

If it doesn't do so "by construction", this section should present the ideas of how this could be fixed (through deserialization-time patches and/or alternative binary encodings). It is OK to say here that you don't know how binary and TASTy compatibility will be affected at the time of submitting the proposal. However, by the time it is accepted, those issues will need to be resolved.

This section should also argue to what extent backward source compatibility is preserved. In particular, it should show that it doesn't alter the semantics of existing valid programs. -->

### Feature Interactions

<!-- A discussion of how the proposal interacts with other language features. Think about the following questions:

- When envisioning the application of your proposal, what features come to mind as most likely to interact with it?
- Can you imagine scenarios where such interactions might go wrong?
- How would you solve such negative scenarios? Any limitations/checks/restrictions on syntax/semantics to prevent them from happening? Include such solutions in your proposal. -->

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
