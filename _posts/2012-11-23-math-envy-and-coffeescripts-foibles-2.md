---
layout: post
title: Math Envy and CoffeeScript's Foibles, Part 2
tags:
- coffeescript
- javascript
- math
status: draft
type: post
published: true
listed: false
meta:
  _edit_last: "1"
---

In the [previous post](CoffeeScript/2012/11/27/math-envy-and-coffeescripts-foibles/) I presented the basics of operational semantics and showed how derivations trees could be used to differentiate two terms that were syntactically similar. This post develops the closing thoughts from that post further with the introduction of type rules, a concrete definition of semantic ambiguity, and the first draft of a tool for detecting semantic ambiguity.

## Type Rules

Type rules are similar in construction to evaluation rules consisting of a premise and conclusion. As with evaluation rules the premise establishes the preconditions for the conclusion. Again, each rule is tagged with a name for reference but preceded by a _t_ in this case to distinguish them from inference rules (_e-*_).

<div class="center">
  <img src="/assets/images/diagrams/cs-type-rules.png"></img>
</div>

For type rules without a premise like _t-true_ and _t-false_ we take them to be true out of hand. That is, the terms `true` and `false` both have the type `Bool`. The others are more complicated.

<div class="center">
  <img src="/assets/images/diagrams/cs-type-rules-lambda-term.png"></img>
</div>

_t-lambda_ illustrates how to determine the type of a lambda term like `(-> true)`. The premise above the line states that if the subterm `t` has the _concrete_ type `T`, then the conclusion `λt` has the type `X -> T`. Here `X` is a _type variable_ because we don't know whether the lambda will be evaluated with the invocation operator `()` or applied to an argument. `T` is concrete because the type can be determined from the body of the lambda expression. For example, in `(-> true)` the subterm `true` has the type `Bool` so the lambda term has the type `X -> Bool`.

<div class="center">
  <img src="/assets/images/diagrams/cs-type-rules-lambda-invocation.png"></img>
</div>

_t-inv_ shows how to determine the type of lambda invocation like `(-> true)()`. The premise states if the lambda term has the type `X -> T` the term `λt()` has the type `T`. This fits with the CoffeeScript `(-> true)()` which obviously evaluates to `true` and has the type `Bool`. It's worth noting that `X` is constrained to be the `Unit` or empty type since no argument is used.

<div class="center">
  <img src="/assets/images/diagrams/cs-type-rules-lambda-application.png"></img>
</div>

_t-app_ is the type rule for lambda applications, eg. `(-> true) false`. The premise says that if the lambda term `λt` on the left has the type `X -> T` and the term on the right as second type `T` the conclusion is that the application will have the result type of the lambda term. Again, the type of an application, like invocation, is only concerned with the type of the *first* lambda's subterm `t` and it ignores the argument that it consumes.

## Type Rule Stacking

Just like derivation trees with evaluation rules this notation makes it easy to establish the type of a term by stacking the type rules on one another. Taking a very simple example, some diagrams will illustrate how this works:

```coffeescript
(-> true)
```

<div class="center">
  <img src="/assets/images/diagrams/cs-type-derivation-simple.png"></img>
</div>

This higlights how to derive the type at the bottom from it's subterms. Typing the innermost subterm `true` with _t-true_. That can then be "stacked" by using it to replace the premise of type rule _t-lambda_. The type derivation expands from the subterm to establish each subsequent parent term's type. Another more complex derivation:

```coffeescript
(-> true) (-> (-> false))()
```

<div class="center">
  <img src="/assets/images/diagrams/cs-type-derivation-complex.png"></img>
</div>

Since there are two subterms involved in application, each branch extends upward until it reaches the atomic value types `true : Bool` and `false : Bool`. The subterm on the left is identical to equation 13. On the right, the nested lambdas and invocation make for a taller stack of type rules to reach the atomic `false`.

## Not Quite There

At this point the type rules can describe the original issue. A derivation tree based on the typing rules highlights that the term is untypable. Taking our canonical example, `(-> true)() -> false`:

<div class="center">
  <img src="/assets/images/diagrams/cs-type-derivation-original-issue.png"></img>
</div>

Once the derivation tree reaches the outermost term it breaks. There is no type rule for the application of `true` to a lambda term like `(-> false)`. It's a type error.

Previously we saw that this this would result in a type error under evaluation by the CoffeeScript interpreter. We also saw that it was easy to construct a term that suffered the same semantic confusion without the type error `(-> (-> true))() -> false`. This issue applies to the type derivation as well.

In addition we saw that it's possible to construct terms, albeit in the boolean example language, that might produce the same value through wildly different evaluation paths. That is, they had different derivation trees in the evaluation relation but the same final result. This issue also applies to type derivations.

In both cases useful information is lost when the derivation is discarded in favor of the final value or type. The advantage with the type information is obviously that no evaluation is required to determine if two terms are "different" in some way other than their syntax. The disadvantage is that not all languages make determining type information easy.

## Detecting Ambiguity

The type information provides a second way to differentiate syntactically similar terms, and there are cases in other languages where both are necessary to determine the meaning of a term. For example `(x, y) -> x + y` has a type that is identical to `(x, y) -> x * y`, `Int -> Int -> Int` (with Curry style type signatures), but it clearly evaluates differently [1].

With that, a term can be represented by a triple `(S, E, T)`, where `S` is the syntax string of the term, `E` is the evalutation derivation, and `T` is the type derivation. The triple can be used to determine whether two terms will cause confusion.

One approach would be to first compare the `S` values for two triples and then determinine if the `E` and `T` values match. Terms with "similar" `S` values but different `E` or `T` values might be ambigous and could be flagged for review. Using the [Levenshtein Distance](http://en.wikipedia.org/wiki/Levenshtein_distance) to keep the calculation for similarity simple:

!! insert image

_dist_ is just the ratio of the distance between the two strings with respect to the maximum length of both. Normalizing the distances allows for a baseline with a given grammar that might be considered "very similar" regardless of the term length. For `(-> true)() -> false` and `(-> true) () -> false`:

!! insert image

An exact definition of string distance that can be reconciled as a threshold "setting" makes building an automated tool a bit easier.

## Happy Parsing



### footnotes

1. It might be that when a function identifier is the only difference between terms, in this case `*` and `+`, it's reasonable to ignore ambigous terms. In this case because the total string length for both terms is small it might be that a single character difference is enough to break some arbitrary threshold. I'm leaving this for futher consideration.
2. Assuming it's possible, it's interesting to think abougt what the inverse result means. That is, when two terms are very syntactically different but have identical types/evaluation derivations. This might signal the two terms or the parent language as antithetical to Python's slogan of "one and only one way to do it".
