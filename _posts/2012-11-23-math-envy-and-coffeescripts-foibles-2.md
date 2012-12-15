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

In the [previous post](CoffeeScript/2012/11/27/math-envy-and-coffeescripts-foibles/) I presented the basics of operational semantics and showed how derivations trees could be used to differentiate two terms that were syntactically similar. This post develops the closing thoughts from that post further by introducing type rules, the concept of "progress and preservation", and the first draft of a tool for detecting semantic ambiguity.

! progress and preservation

! haskell parser with ast + type tags

## Type Rules

Type rules are similar in construction to evaluation rules consisting of a premise and conclusion. As with evaluation rules the premise establishes the preconditions for the conclusion. Again, each rule is tagged with a name for reference but preceded by a _t_ in this case to distinguish them from inference rules (_e-*_).

<div class="center">
  <img src="/assets/images/diagrams/cs-type-rules.png"></img>
</div>

For type rules without a premise like _t-true_ and _t-false_ we take them to be true out of hand. That is, the terms `true` and `false` both have the type `Bool`. The others are more complicated.

<div class="center">
  <img src="/assets/images/diagrams/cs-type-rules-lambda-term.png"></img>
</div>

_t-lambda_ illustrates how to determine the type of a lambda term like `(-> true)`. The premise above the line states that if the subterm `t` has the type `T`, then the conclusion `\t` has the type `\top \to T`. Here `T` is a _type variable_. For example, in `(-> true)` the subterm `true` has the type `Bool` so the lambda term has the type `\top \to Bool`.

One likely point of confusion is `\top`, not to be confused with `T`. Neither of the lambda expressions make use of arguments so `\top` is used to represent that lambda terms accept any type under invocation or application. For example, when the subterm `t` in a lambda term has the type `T` it means that the lambda term will result in a term with the type `T` regardless of what it's applied to or invoked with.

<div class="center">
  <img src="/assets/images/diagrams/cs-type-rules-lambda-invocation.png"></img>
</div>

_t-inv_ shows how to determine the type of lambda invocation like `(-> true)()`. The premise states if the lambda term has the type `\top \to T` the term `\t()` has the type `T`. This fits with the CoffeeScript `(-> true)()` which obviously evaluates to `true` which has the type `Bool`. Again, `\top` is used to denote the fact that the argument type is unimportant and in this case non existent.

<div class="center">
  <img src="/assets/images/diagrams/cs-type-rules-lambda-application.png"></img>
</div>

Equation 11 is the type rule for lambda applications such as `(-> true) false`. The premise says that if the lambda term `\t` on the left has the type `\top \to T_1` and the term on the right as the type `T_2` the conclusion is that the application of the lambda term will have the type `T_1`. Again, the type of an application, like invocation, is only concerned with the type of the *first* lambda's subterm `t`.

## Type Rule Stacking

This notation makes it easy to establish the type of a term by stacking the type rules on one another. Taking a very timple example, some diagrams will illustrate how this works:

```coffeescript
(-> true)
```

<div class="center">
  <img src="/assets/images/diagrams/cs-type-derivation-simple.png"></img>
</div>

Equation 13 is the stack, a derivation tree. It explains how the reader can derive the type at the bottom from it's subterms. Starting with equation 14, the subterm `true` is typed by the type rule *true*. That can then be "stacked" by using it to replace the premise of type rule *lambda* in equation 15. The type derivation expands from the subterm to establish each subsequent parent term's type. Make sure to note that type derivations are never really accompanied by the full form of the type rules in equations (just the labels like *lambda* and *true*) but it's helpful here for illustration here. Taking the more complex term from earlier it's clear why they are ommited:

```coffeescript
(-> true) (-> (-> false))()
```

<div class="center">
  <img src="/assets/images/diagrams/cs-type-derivation-complex.png"></img>
</div>

Since there are two subterms involved in application, each branch extends upward until it reaches the atomic value types `true : Bool` and `false : Bool`. The subterm on the left is identical to equation 13. On the right, the nested lambdas and invocation make for a taller stack of type rules to reach the atomic `false`.

## Not Quite There

At this point the CoffeeScript subset has enough definition to describe the original issue in terms of evaluation or typing. That is, using the inference rules it's clear that the evaluation of `(-> true)() -> false` "gets stuck" from the following derivation tree:

<div class="center">
  <img src="/assets/images/diagrams/cs-eval-derivation-original-issue.png"></img>
</div>

After the invocation of the left lambda term the evaluation gets stuck because the right lambda term can't be evaluated any further and no rule exists to handle the application of the term `true` to `(-> false)`. Additionally, a derivation tree based on the typing rules highlight that the term is untypable:

<div class="center">
  <img src="/assets/images/diagrams/cs-type-derivation-original-issue.png"></img>
</div>

Once the derivation tree reaches the outermost term it breaks because once again we have no type rule for the application of `true` to a lambda term like `(-> false)`. It's a type error.

So what's been gained thusfar by applying the formalism of operational semantics and type derivations? It's clear that types, inferred or otherwise, would prevent at least the original case `(-> true)() -> false` as you can see from the type derivation. Unfortunately type derivation or type inference would only alert the end user and not an author in the middle of creating a language and then only if the result of the leftmost lambda term is not a lambda term. That is, if the leftmost lambda term evaluates to a lambda term the the whole thing is well typed. This means type information/derivations alone can't help identify this syntactic ambiguity.

## Detecting Ambiguity

Even if the two terms will *not* result in a type error under evaluation (See the snippet below for an example), they have different types and different derivation trees. Taken together with the fact that the two expresions are "similar" might be enough.

As an aside, you might wonder what value there is in creating type/evaluation derivations since the difference in the final type is clear. In another language two terms with the same final type and very similar syntax might have different type/evaluation derivations. It may even be possible with CoffeeScript and I'm simply not clever enough to think of an example. In any case they both carry useful information that shouldn't be discarded without good reason.

```coffeescript
# fails
(-> true)() -> false

# works fine
(-> (-> true))() -> false

# also works fine
(-> (-> true)) () -> false
```

With at least two concrete ways to identify a semantic difference between terms, the syntax itself can be used as measure of syntactic difference. This provides a sliding scale

Since this particular problem is small the definition can start out simple. Taking the Levenshtein Distance as a function of string size should work for a start. For the syntax/string representation of two terms `a` and `b`:

insert image

*For two terms with inconsistent types, we say they are more or less ambigous based on the calculation Levenshtein Distance/max(Length<sub>1</sub>, Length<sub>2</sub>)*.


### footnotes

!! See the confusion over Monads/Functors due in part to their relationship with mathematics

!! Turing, Church, Algorithms

!! denotational semantics, axiomatic semantics

!! more information about types and programming languages and how much you like it

!! This Theorem is referred to as Determinacy of Evaluation. I may go back and do some simple proofs for my own education after this post and a possible follow up.

!! Tested at http://coffeescript.org/.

!! Technically JavaScript uses a strategy known as Call by Sharing, which differs from Call by Value in how deals with objects. More information at http://dmitrysoshnikov.com/ecmascript/chapter-8-evaluation-strategy/ courtesy of [@raganwald](https://twitter.com/raganwald).
