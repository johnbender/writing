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

System F is a variant of the Lambda Calculus. It "mixes" types and terms relatively freely compared with the average statically typed programming language. System F and it's variants are used frequently in the study of typed computation and an extended form, System FC, plays an important role in GHC's compilation process [1]. This post will translate the grammars of the Lambda Calculus, Simply Typed Lambda Calculus, and finally System F into a small subset of CoffeeScript while covering the basics of application and abstraction in each.

## Lambda Calculus

The Lambda Calculus is relatively easy to capture with CoffeeScript, and a quick review of will be helpful in explaining System F. Representing the Lambda Calculus only requires a small subset of CoffeeScript's total synatx. Let's define both grammars side by side.

!! insert image of lambda calculus and coffeescript

In both cases `x` represents a variable and `t t` represents application of one term to another. The only real difference is the lambda abstraction syntax.

!! insert image of abstraction examples

In the Lambda Calculus, abstraction involves preceding or wrapping a term `t` with `λx.t` which captures or defines the variable `x` inside the term `t`. In the example `t` is `λx.y`, using the abstraction `λy.t` creates a new term `λy.λx.y` (otherwise known as the K-combinator). Note that the `x` in `λx.t` could be any variable like `y` in the example.

In CoffeeScript the same initial term is `(x) -> y` and applying the CoffeeScript version of abstraction yields `(y) -> (x) -> y`. Viewing the <a href="http://coffeescript.org/#try:(y)%20-%3E%20(x)%20-%3E%20y">translation</a> to JavaScript is informative. More importantly, both behave the same way when applied to a single argument.

!! insert image of application to identity

The application of a term that takes two arguments (`y` and `x`) to a single term/argument results in a lambda that can be applied to another term. The <a href="http://coffeescript.org/#try:console.log%20((y)%20-%3E%20(x)%20-%3E%20y)%20((x)%20-%3E%20y)">translation</a> to JavaScript shows that a function object is returned ready for application.

## Simply Typed Lambda Calculus

System F expands on the Simply Typed Lambda Calculus which in turn is an expansion on the Lambda Calculus. The grammar of the Simply Typed Lambda Calculus makes two alterations to that of the untyped Lambda Calculus.

It adds a type requirement to the argument of the abstraction. The CoffeeScript equivalent requires a function that compares a type argument to the argument term. To keep things simple, `ifft` will throw an exception when the types don't line up [2].

```coffeescript
# define ifft
```

Additionally both are extended with the notion of constants, `c`.


### Footnotes

1. There are some [interesting papers](http://research.microsoft.com/en-us/um/people/simonpj/papers/ext-f/) to read from Microsoft Research this particular extension of System F.
2. This is a more [intrinsic](http://en.wikipedia.org/wiki/Simply_typed_lambda_calculus#Intrinsic_vs._extrinsic_interpretations) approach to the translation as we aren't attempting to use types to reason about the terms/grammars.
