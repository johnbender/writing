---
layout: post
title: Fixing CoffeeScript
tags:
- javascript
- coffeescript
- math
status: draft
type: post
published: true
meta:
  _edit_last: "1"
---

Ashkenas quote from language panel

it appears to be popular for it's terse syntax

has been criticised as _too_ flexible in some cases

## CoffeeScript Confusion

```coffeescript
deSomething = () -> true

# and

doSomething () ->  false

# vs

doSomething() ->  false
```

```javascript
var doSomething = function(){ return true };

// and

doSomething(function(){ return false; });

// vs

doSomething()(function(){ return false; });

// !!! => true(function(){ return false; });
```

## Operational Semantics

borrowing from pierce heavily

explanation as formalism for evaluation

if then else example?

!! deeper discusion of grammar required. eg, why the ::== and what each line means. important for eval/type rule understanding !!

show basic type information applied, including type rules

## CoffeeScript Grammar

You might consider this a problem with the grammar/parser and not the evaluation result. That is, maybe it shouldn't allow both lambda forms as method/function arguments, but because both forms are valid in the current parser implementation it's only an issue when evaluation takes place. Moreover it's only an *obvious* issue when the return type of `doSomething` doesn't allow for invocation. In an ideal world the author would arrive at these conclusions early during the process of creating the language grammar, and it might be interesting to use the formalism of operational semantics to help "boil down" the way these terms can be evaluated, hopefully revealing the ambiguity.

Creating a subset of CoffeeScript including the grammar, evaluation rules, and some simple types as though we were designing the language should be illustrative. In the interest of keeping this post semi-sane in length the subset will be the minimum necessary to reproduce the aformentioned ambiguity. Returning to the original example, the overhead of assignment and identifiers can be avoided by using lamda expressions directly:

```coffeescript
(-> true) () ->  false
# => true

# vs

(-> true)() ->  false
# !!! => true(function(){ return false; });
```

Which of course translates into JavaScript similarly to the original example:

```javascript
(function(){ return true; })(function(){ return false; });

// vs

(function(){ return true; })()(function(){ return false; });

// !!! => true(function(){ return false; });
```

It's interesting to note that neither case makes it visually obvious there's a problem.

Having identified the subset of CoffeeScript necessary for reproducing the original issue, we can begin with a precise definiton of terms in the form of a language grammar.

<div class="side-by-side">
  <img src="/assets/images/diagrams/cs-grammar-meta-terms.png"></img>
  <img src="/assets/images/diagrams/cs-grammar-terms.png"></img>
</div>

`v`, `\t` and `t` all represent meta terms. In the case of `\t` it was easier to create a meta term in place of repeating each possible lambda form in `t`. `v` on the other hand has special meaning as it did in the operational semantics example. It represents the set of acceptable final results of evaluating expressions in our CoffeeScript subset. Finally `t` is total set of terms and possible compound terms. Notable among them is the invocation and application of lambda terms (`\t`).

Some examples of translated into the grammar, starting with the most basic:

```coffeescript
true
```

![translation]()

```coffeescript
(-> false)()
```

![translation]()

```coffeescript
(() -> true) true
```

![translation]()

```coffeescript
(-> (() -> true))() (-> true)
```

![translation]()

Note that `\t` representations include information about their subterms and that all of the translated expresions are valid CoffeeScript [!!].

## Evaluation Relation

With the grammar in place the next step is to define both the evaluation rules and strategy so that our terms can perform some computation. To keep things simple our language subset will use the call-by-value, left to right strategy [!!] employed by CoffeeScript and numerous other languages (C, Java, Python etc). As you might have guessed by looking at the grammer there are only two evaluation rules, both of which are concerned with evaluation of lambda terms.

```
(\t)() --> t

(\t) t' --> t
```

The impact of the evaluation strategy (call-by-value l-to-r) for our little language is that the arguments applied to lambda terms in the second evaluation rule must be as "evaluated" as far as possible, before the invocation can take place. For example `(-> true) (-> (-> false))()` would take a step first to `(-> true) (-> false)` as a result of the `()` operator before applying the second lambda term as an argument to the first producing `true`. Ultimately this is unimportant in addressing the particular set of ambiguous expressions we've chosen but is noted here for completeness. Otherwise both evaluation rules are simple. Given the `()` operator or the application of a fully evaluated term, a lambda expression will evaluate to its subterm.

So far it's not obvious there might be an issue. Defining lambdas with a `-> t` is appealing, and preparing for arguments with a null tuple preceding the arrow makes sense. Continuing on, the application of type rules to our language is the last step to having a complete picture of how it works.


## Type Rules

<div class="center">
  <img src="/assets/images/diagrams/cs-type-rules.png"></img>
</div>

The type rules are fairly intuitive. Given the statment above the line called the *premise*, the statment below the line called the *consequent* or *conclusion* is the type of the term. For those type rules without a premise like `true : Bool` and `false : Bool`, we take them to be true out of hand. That is, the terms `true` and `false` both have the type `Bool`. The others are more complicated.

<div class="center">
  <img src="/assets/images/diagrams/cs-type-rules-lambda-term.png"></img>
</div>

Equation 9 illustrates how to determine the type of a lambda term like `(-> true)`. The premise above the line states that if the subterm `t` has the type `T`, then the conclusion on the bottom, `\t`, has the type `\top \to T`. Translated to our mini example `(-> true)`, because the subterm `true` has the type `Bool`, the lambda term has the type `\top \to Bool`. One extremely important note: because neither of the lambda expressions make use of arguments, Top (`Top`) is used to represent the acceptance of any type. So when `t` in `\t` has the type `T` it means that `\t` will, under invocation or application, result in an expresion of typ `T` regardless of what it's applied to or invoked with. As an example:

<div class="center">
  <img src="/assets/images/diagrams/cs-type-rules-lambda-invocation.png"></img>
</div>

Equation 10 shows how to determine the type of lambda invocation like `(-> true)()`. The premise states if the lambda term has the type `\top \to T` the term `\t()` has the type `T`. This fits with the CoffeeScript `(-> true)()` which obviously evaluates to `true` which has the type `Bool`. Again, `\top` is used to denote the fact that the argument type is unimportant and in this case non existent.

<div class="center">
  <img src="/assets/images/diagrams/cs-type-rules-lambda-application.png"></img>
</div>

Equation 11 is the type rule for lambda applications such as `(-> true) false`. The premise says that if the lambda term `\t` on the left has the type `\top \to T_1` and the term on the right as the type `T_2` the conclusion is that the application of the lambda term will have the type `T_1`. Again, the type of an application, like invocation, is only concerned with the type of the lambda's subterm `t`.

## Type Rule Stacking

This notation might look odd but it makes it easy to establish the type of a term by "stacking" the type rules on one another. For example the term `(-> true)`:

<div class="center">
  <img style="height: 100px" src="/assets/images/diagrams/cs-type-derivation-simple.png"></img>
</div>

Beginning with the most basic type rule for one of the values `true`, which lacks a premise, the stack expands outward from the subterms to establish the original term's type. This stacking is referred to as a derevation tree. Taking the more complex term from earlier `(-> true) (-> (-> false))()`, it's clear why:

<div class="center">
  <img style="height: 200px" src="/assets/images/diagrams/cs-type-derivation-complex.png"></img>
</div>

Since there are two subterms involved in application, each branch extends upward until it reaches the atomic value types `true : Bool` and `false : Bool`.


prefer the space after the function because it's a more common use case

show proof that there's only one way to evaluate each term inductively, "appendix" again


# footnotes

!! Tested at http://coffeescript.org/.
!! Technically JavaScript uses a strategy known as Call by Sharing, which differs from Call by Value in how deals with objects. More information at http://dmitrysoshnikov.com/ecmascript/chapter-8-evaluation-strategy/ courtesy of [@raganwald](https://twitter.com/raganwald).
