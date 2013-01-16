---
layout: post
title: System F in CoffeeScript
tags:
- coffeescript
- math
status: draft
type: post
published: true
listed: false
vote: "CHANGE"
meta:
  _edit_last: "1"
---

System F is a typed variant of the Lambda Calculus. It "mixes" types and terms relatively freely compared with the average statically typed programming language. System F and it's variants are used frequently in the study of typed computation and an extended form, System FC [1], plays an important role in GHC's compilation process. In this post I'll attempt to translate the grammar for the types and terms of System F into a small subset of CoffeeScript.

## Lambda Calculus

The Lambda Calculus is relatively easy to capture with CoffeeScript, and a quick review of will be helpful in explaining System F. Representing the Lambda Calculus only requires a small subset of CoffeeScript's total synatx. Let's define both grammars side by side.

!! insert image of lambda calculus and coffeescript

In both cases `x` represents a variable and `t t` represents application of one term to another. The only real difference is the lambda abstraction syntax.

!! insert image of abstraction examples

In the Lambda Calculus, abstraction involves preceding or wrapping a term `t` with `λx.t` which captures or defines the variable `x` inside the term `t`. In the example `t` is `λx.y`, using the abstraction `λy.t` creates a new term `λy.λx.y` (otherwise known as the K-combinator). Note that the `x` in `λx.t` could be any variable like `y` in the example.

In CoffeeScript the same initial term is `(x) -> y` and applying the CoffeeScript version of abstraction yields `(y) -> (x) -> y`. Viewing the <a href="http://coffeescript.org/#try:(y)%20-%3E%20(x)%20-%3E%20y">translation</a> to JavaScript is informative. More importantly, both behave the same way when applied to a single argument.

!! insert image of application to identity

The application of a term that takes two arguments (`y` and `x`) to a single term/argument results in a lambda abstraction that can be applied to another term. The <a href="http://coffeescript.org/#try:console.log%20((y)%20-%3E%20(x)%20-%3E%20y)%20((x)%20-%3E%20y)">translation</a> to JavaScript shows that a function object is returned ready for application.

## System F

### Footnotes

1. Some [interesting papers](http://research.microsoft.com/en-us/um/people/simonpj/papers/ext-f/) to read from Microsoft Research this particular extension of System F.
