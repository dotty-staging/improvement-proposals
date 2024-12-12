---
layout: sip
permalink: /sips/:title.html
stage: design
status: draft
title: SIP-68 - Soft Modifier for `enum` `case`
---

**By: Katarzyna Marek<sup>1</sup> and Hamza Remmal<sup>2</sup> and Kacper Korban<sup>2</sup>**

<sup>1</sup>VirtusLab<br>
<sup>2</sup>EPFL

## History

| Date          | Version            |
|---------------|--------------------|
| Dec 4th 2024  | Initial Draft      |

## Summary

<!-- A summary of the proposed changes. This should be no longer than 3 paragraphs.
It is intended to serve in two ways:

- For a first-time reader, a high-level overview of what they should expect to
  see in the proposal.
- For returning readers, a quick reminder of what the proposal is about. -->

TODO

## Motivation

<!-- A high-level overview of the proposal with:

- An explanation of the problems or limitations that it aims to solve,
- A presentation of one or more use cases as running examples, with code showing
  how they would be addressed *using the status quo* (without the feature), and
  why that is not good enough.

This section should clearly express the scope of the proposal. It should make it
clear what are the goals of the proposal, and what is out of the scope of the
proposal. -->

TODO

## Proposed solution

### High-level overview

<!-- A high-level overview of the proposed changes, and how they allow to better
solve the running examples. This section should be example-heavy, and not dive
into corner cases.

Example:

~~~ scala
// This is an @main method
@main def foo(x: Int): Unit =
  println(x)
~~~ -->

TODO

### Specification

We propose to allow the use of the soft `infix` modifier with `enum case`.

This is achieved by allowing parsing soft modifiers in front of `enum case`.

The updated [documentation page](https://dotty.epfl.ch/docs/reference/soft-modifier.html) should state (the
change is **bolted**):
> A soft modifier is treated as potential modifier of a definition if it is
> followed by a hard modifier or a keyword combination starting a definition
> (def, val, var, type, given, class, trait, object, enum, **enum case**, case
> class, case object). Between the two words there may be a sequence of newline
> tokens and soft modifiers.

The parser grammar will remain unchanged, since `infix` is a `LocalModifier` and
`LocalModifier`s are allowed before `EnumCase`

An additional check will then be added to restrict soft modifiers for `enum
case` to `infix` only.

This specification change will also leave the possibility for supporting other
soft modifiers for `enum case` in the future. (e.g. experimental soft modifier
`erased`)

### Compatibility

This proposal extends the spece of compiling Scala programs by those that use
`infix` in front of `enum case`. Those previously failing programs are by
default binary compatible. With a correct implementation, this proposal also
shouldn't introduce any new restrictions for already accepted programs.

## Related work/Relevant links

Links:
- PR with the draft implementation: allow soft modifier before `enum` `case`
  [#22072](https://github.com/scala/scala3/pull/22072)
- Linked issue: Should `infix` work on `enum` `case`?
  [#22010](https://github.com/scala/scala3/issues/22010)
- Specification docs page for [Soft
  Keywords](https://dotty.epfl.ch/docs/reference/soft-modifier.html)
- Similar (closed) issue on `enum`: Sould `infix` work on `enum`?
  [#18933](https://github.com/scala/scala3/issues/18933)
- PR reverting back to following the spec: Align implementation with spec of
  soft modifiers [#15961](https://github.com/scala/scala3/pull/15961)

## FAQ
