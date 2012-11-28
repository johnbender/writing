---
layout: post
title: Math Envy and CoffeeScript's Foibles, Part 1
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

At Strange Loop 2011 while participating in a [language panel (5:06)](http://www.infoq.com/presentations/Language-Panel) Jeremy Ashkenas was asked, "What is the worst idea that was ever introduced into programming languages that continues to afflict us today?" He responded, "... mathematics envy". I can't completely disagree with Mr. Ashkenas. Math appears to get in the way on occasion [1]. Even so it struck me as an odd response given that much of computing is built on the work of great mathematicians. For an example look no further than the [inner workings](kwingolog.org/archives/2011/08/02/a-closer-look-at-crankshaft-v8s-optimizing-compiler) of V8's optimizing compiler that runs a lot of Jeremy's code.

Fast forward a year and issues with CoffeeScript's flexible syntax start popping up in [blog](http://surana.wordpress.com/2011/02/08/coffeescript-oddities/) [posts](http://ceronman.com/2012/09/17/coffeescript-less-typing-bad-readability/). Interactions between whitespace, operators, comprehensions, and lambda declarations appear to be a source of runtime confusion. To be fair it seems that they are rarely the cause of serious problems, but it left me wondering if they could have been avoided during the creation of the language. That is, could the timely application of mathematics have prevented these problems early in CoffeeScript's genesis?

What follows is the first of two posts aimed at answering that question. This post provides an introduction to operational semantics, a description of one semantic issue in CoffeeScript, and a definition of the operational semantics for a CoffeeScript subset capable of reproducing said issue. The second post will introduce type derivations, define them for the same CoffeeScript subset, and attempt to formalize semantic ambiguity more generally. The background needed to understand the math is covered, but the post generally follows my thought process.

## CoffeeScript Confusion

I've chosen to address the lambda syntax that has been cited by both of the previously linked posts. Specifically the ability in CoffeeScript to omit parenthesis in lambda declarations and how that affects using 0-arity lambdas as arguments. Here's an example borrowed from Manuel Cerón's [post](http://ceronman.com/2012/09/17/coffeescript-less-typing-bad-readability/) on the topic with both the CoffeeScript and the JavaScript output for clarity

```coffeescript
deSomething = () -> true

doSomething () ->  false

doSomething() ->  false
# !!! => true(-> false);
```

The first invocation of `doSomething` applies it to the inline lambda. The second invokes it directly with the `()` operator and then attempts to apply the result `true` to lambda defined with `-> false`. This results in a type error. For clarity here's the equivalent in JavaScript.

```javascript
var doSomething = function(){
  return true
};

doSomething(function(){
  return false;
});

doSomething()(function(){
  return false;
});
// !!! => true(function(){ return false; });
```

It's easy to see where this might cause issues given that the only difference between the two expressions is a single whitespace character. The goal then is to apply some formalism to this part of CoffeeScript. Ideally the formalism will result in an approach, technique, or tool that can point out (or would have pointed out) problems like this to a language designer _during_ the process of language creation.

## Operational Semantics

Operational Semantics is one way [2] to formalize the semantics of a programming language. We'll build a basic understanding of how it works by borrowing an example language from Pierce's book _Types and Programming Languages_ [3].

<div class="center">
  <img style="width: 60%" src="/assets/images/diagrams/bool-grammar.png"></img>
</div>

The grammar definition is made of up of two "meta variables" `t` and `v`. Assigned to those meta variables is a set of possible terms in the language each separated by a `|`. `t` represents all of the ways to construct terms in the language (see the example below). `v` is a subset of `t`, as witnessed by its inclusion in `t` but it is distinct for a reason. `v` is the set of terms that are acceptable as the final result of evaluation.

```haskell
-- simple values
true
false

-- a simple compound term
if true then false else true

-- a more complex compound term, parens for clarity
if false then false else (if true then false else true)
```

`v`'s distinction means that any term like the second and third in the example has to eventually evaluate to either `true` or `false`. Any other result means that something is "stuck". It also means that `true` or `false` themselves cannot evaluate further. What it means for a term to get "stuck" will be covered later.

Also, notice that that the only other construct in the grammar `if t then t else t` has sub terms represented with the meta variable `t`. This captures the ability to use `true`, `false`, or another `if t then t else t` in place of each sub term `t` as in examples two and three.

With the building blocks of the example language in place the next step is to establish a set of rules that will define the way terms are evaluated. That is, what steps should be used to reduce any term to `true` or `false`, and in what order. Superficially this language seems extremely simple, but there are some subtle details about how term evaluation that need to be captured.

<div class="center">
  <img src="/assets/images/diagrams/bool-inference-rules.png"></img>
</div>

These equations are collectively referred to as the _evaluation relation_ and individually as _inference rules_. Each of them plays an important role in the _evaluation strategy_ [!!] of the example language which instructs the reader in how to evaluate any term in the language. All of them are tagged with a name preceded by an "_e-_" for evaluation. The tags will be helpful later when referring to the rules and also for keeping them conceptually distinct from the type rules.

_e-true_ and _e-false_ are fairly simple. They represent the expected evaluation results for the different guard values in an `if t then t else t` term. With `true` you get the first subterm and with `false` you get the second subterm. Also, notice that there are no rules for either `true` or `false` by themselves. This further reinforces that `true` and `false` are values and that there's no way to evaluate them further. Equation 3, _e-if_ is more interesting in its construction and how it captures an important subtlety of the evaluation strategy.

<div class="center">
  <img src="/assets/images/diagrams/bool-inference-rules-guard.png"></img>
</div>

There are two parts to this rule. Above the line is the _premise_ and below is the _conclusion_. The premise establishes a requirement or precondition for applying the conclusion to a given term. For _e-if_ the premise says that if the first subterm `t` can be evaluated to `t'` then the parent term `if t then t else t` should evaluate to `if t' then t else t`. The importance here is that evaluation will focus on the guard term and not the other subterms. A different evaluation strategy might fully evaluate the second or third subterms before evaluating the first subterm.

```haskell
-- parenthesis are supplied for clarity only
-- |     first/guard subterm     |      |       second subterm        |
if (if true then false else false) then (if false then true else false) else false
```

This term could conceivably take two evaluation paths without _e-if_. One alternative strategy would first evaluate the second subterm `if false then true else false` to `false`, then evaluate the guard `if true then false else false` to `false`, and finally the full term to `false`. Obviously that evaluation of the second subterm is unnecessary because the guard term evaluates to `false` and the second subterm is ignored completely.

There is enough information here for someone interested in implementing this language without wondering about how to combine terms or how those terms should be evaluated. Additionally there are interesting properties that can be proved inductively using the inferences rules. For example it's possible to show that there is one and only one way to evaluate each term at each step [4]. The next step then is to turn back to CoffeeScript and begin apply operational semantics to see if anything interesting happens.

## CoffeeScript Grammar

In the interest of keeping this post semi-digestible the grammar will represent a small subset of CoffeeScript necessary to reproduce the aforementioned ambiguity. In the original example, the overhead of assignment and identifiers can be avoided by using lambda expressions directly.

```coffeescript
(-> true) () ->  false
# => true

(-> true)() ->  false
# !!! => true(function(){ return false; });
```

Which of course translates into JavaScript similarly to the original example:

```javascript
(function(){ return true; })(function(){ return false; });
// => true

(function(){ return true; })()(function(){ return false; });
// !!! => true(function(){ return false; });
```

Here the use of atomic boolean values alleviates the need for arguments in the lambda syntax. Again, simplicity in reproducing the issue is preferred for the sake of brevity. Next is a precise definition of terms in the form of a language grammar.

<div class="center">
  <img src="/assets/images/diagrams/cs-grammar.png"></img>
</div>

To reiterate, the left hand side of each `::=` assignment is a meta variable that can be used in other parts of the grammar. In the case of `\t` it was easier to create a meta term than to repeat each possible lambda form in `t`. `v` on the other hand is the set of acceptable final results of evaluation. Finally `t` is complete set of terms and possible compound terms. Notable among them is the invocation and application of lambda terms (`\t`).

Some translated examples should help to clarify:

```coffeescript
# invocation
(-> false)()

# application
(() -> true) true

# both
(-> (() -> true))() (-> true)
```


<div class="center">
  <img src="/assets/images/diagrams/cs-grammar-examples.png"></img>
</div>

The most important part to note is that lambda terms capture the subterm be it `true`, `false` or another lambda.

Looking at the examples it's reasonable to ask whether there's value in providing a grammar that looks a lot like it's own language. First, it maps the two different lambda forms to one form in the grammar, which makes reasoning about evaluation and types easier. Second, differentiating values (`true`, `false`, and `-> t`) from other terms by calling them values is important for knowing when evaluation has finished.

## Inference Rules

With the grammar in place the next step is to define both the inference rules and evaluation strategy. Obviously it will use the call-by-value, left to right strategy [5] employed by CoffeeScript and JavaScript.

<div class="center">
  <img src="/assets/images/diagrams/cs-inference-rules.png"></img>
</div>

The only impact of the evaluation strategy (call-by-value l-to-r) is that arguments to lambda terms must be fully evaluated before application can take place. For example `(-> true) (-> false)()` would first evaluate to `(-> true) false` as a result of the `()` operator.

<div class="center">
  <img src="/assets/images/diagrams/cs-inference-rules-application.png"></img>
</div>

The `v` in _e-app_ means that any argument a lambda term is applied to should be fully evaluated. In other words it should be a term in the meta variable set `v`. Once applied the result is the lambda's unaltered subterm.

<div class="center">
  <img src="/assets/images/diagrams/cs-inference-rules-application-argument.png"></img>
</div>

_e-arg-eval_ stipulates in the premise (above the bar) that if the term a lambda will be applied to can take a step of evaluation it should. _e-app_ informs the reader when lambda application can take place and _e-arg-eval_ informs the reader how to get there. Taken together these three rules define how terms in the CoffeeScript subset get evaluated [6].

## Derivation Trees

The inference rules in an operational semantics definition can be used in a _derivation tree_ to show how a term will evaluate. Rules are "stacked" on one another to show how they work in succession to produce an evaluation result.

```haskell
-- parenthesis are supplied for clarity only
-- |     first/guard subterm     |      |       second subterm        |
if (if true then false else false) then (if false then true else false) else false
```

Again, depending on the way the inference rules are defined this example could evaluate two different ways. Here the derivation tree for the evaluation of this term results in false (bottom right)

<div class="center">
  <img src="/assets/images/diagrams/bool-derivation-tree-example.png"></img>
</div>

Unfortunately the way this "tree" is constructed isn't obvious. First, taking the second subterm and replacing it with a variable prevents the equations from getting too long. We already know that the second sub term isn't important in the final evaluation (see the introductory section on operational semantics) and it's easier to read the equations when they aren't squished

<div class="center">
  <img src="/assets/images/diagrams/bool-derivation-tree-example-simplify.png"></img>
</div>

From among the inference rules _e-true_, _e-false_, and _e-if_ one will apply to simplify the term. The obvious place to start seems to be applying _e-true_ to the first subterm `(if true then false else false)`, but the second subterm `(if false then true else false)` could just as easily have _e-false_ applied to it. Recall that the third rule _e-if_ tells the user/reader which will take precedence. It says that if the guard (first subterm) can be evaluated it should be, leaving us to evaluate the first subterm using _e-true_ as the first part of the derivation tree.

<div class="center">
  <img src="/assets/images/diagrams/bool-derivation-tree-first-rule.png"></img>
</div>

It's easy to see that this looks just like the "raw" form of the _e-true_ rule. The only difference is the replacement of the last two subterms with `false` on the left side of the arrow and the resulting subterm with `false` on the right side of the arrow. It might look a little confusing with the bar resting on top of the _e-true_ rule, but that signifies the applied rule has no premise/precondition. Next, since  _e-if_ forced the application of _e-true_, it makes sense that it figures in to the derivation tree. Importantly _e-if_ has a precondition, one which the application of _e-true_ satisfies.

<div class="center">
  <img src="/assets/images/diagrams/bool-inference-rules-guard.png"></img>
</div>

_e-if_'s precondition requires that the first subterm of an `if t then t else t` evaluate before the other two subterms. In other words it requires that `t -> t'` be replaced by some evaluation.

<div class="center">
  <img src="/assets/images/diagrams/bool-derivation-tree-second-rule.png"></img>
</div>

Here, it's been replaced by `if true then false else false -> false` from the application of _e-true_. The bottom/conclusion of the inference rule can be replaced by the whole term evaluated to replace the first subterm with `false`. It's a "stack" of the two inference rules _e-true_ and _e-if_. All that's left is to slide _e-false_ on to the bottom of the stack.

<div class="center">
  <img src="/assets/images/diagrams/bool-derivation-tree-example.png"></img>
</div>

The final evaluation result is `false`. At this point it may seem odd to call it a derivation "tree", but a more complex grammar could have multiple terms in the premise that require evaluation resulting in a tree like structure.

## Evaluating a Solution

Finally we know enough to apply the operational semantics to our problem. First the least complex term that prevents evaluation.

```coffeescript
(-> true)() -> false
```

Translating this example into the grammar representation yields a form that will work with the inference rules.

<div class="center">
  <img src="/assets/images/diagrams/cs-derivation-trees-stuck.png"></img>
</div>

_e-inv_ is applied because `-> false` can't evaluate any further (it's a value in `v`), but then what? The only rule that has evaluation in it's premise is _e-arg-eval_ and that only allows for the second term `-> false` to be evaluated. It does **not** allow for the consumption of an argument. _e-inv_ and _e-app_ are only good for terms with lambdas as the first subterm which is `true` after the initial application of _e-inv_. It's "stuck".

Here someone will say, "We already knew that because there's a type error when you evaluate the JavaScript!". Consider a slightly more complex example.

```coffeescript
(-> (-> true))() -> false
```

And the derivation tree to match.

<div class="center">
  <img src="/assets/images/diagrams/cs-derivation-trees-not-stuck.png"></img>
</div>

The value returned by the first lambda can be applied to an argument with _e-app_ so the result of evaluation is `true`. Viewing the accompanying term with an additional space will provide contrast.

```coffeescript
(-> (-> true)) () -> false
```

<div class="center">
  <img src="/assets/images/diagrams/cs-derivation-trees-not-stuck-with-space.png"></img>
</div>

Clearly the two terms are syntactically similar but each has a very different meaning. This semantic differentiation makes it possible to start thinking about a concrete notion of semantic ambiguity in a programming language. Informally, if two terms are _very similar_ syntactically but have different derivation trees they are semantically ambiguous.

There are two problems with this definition if the goal is to come up with something useful for actual language implementers. First, "very similar" is nebulous. Luckily computer science is littered with string "distance" algorithms. Second, automatically generating derivation trees for terms is likely to be difficult.

It might be possible to just compare evaluation results instead of the derivation trees but there are problems with that approach. In the example boolean language it's extremely easy to define two terms that evaluate to an identical result but have different derivation trees.

```haskell
-- example one
if (if true then true else true) then false else (if true then false else true)

-- example two
if (if true then false else true) then false else (if true then false else true)
```

The fact that the evaluation path is very different gets lost in a forest of `true`'s and `false`'s. More concretely the string distance between the two terms is at most 5 characters out of 78, but the first example has a derivation tree with two rules (_e-true_, _e-if_) against the second examples stack of three (_e-true_, _e-if_, _e-true_). Again, this is in spite of the fact that the evaluation _result_ is `false` in both cases. You simply can't tell without a lot of effort how different the evaluation is and in a language with side effects the difference could be critical.

## Next Time

In the next post I'll take a look at how type information could replace the derivation trees as the semantic differentiator. Type information is often readily available even in languages like CoffeeScript that don't have type annotations. If finding a difference between two terms is simple enough maybe a tool can be built to automate the process of ferreting out confusing term pairings.

### footnotes

1. An example is the confusion over Monads/Functors due in part to their odd names and relationship with mathematics.
2. [Denotational Semantics](http://en.wikipedia.org/wiki/Denotational_semantics) and [Axiomatic Semantics](http://en.wikipedia.org/wiki/Axiomatic_semantics) are alternate ways to define language semantics.
3. This example language is borrowed almost verbatim from Types and Programming Languages but I've added in my own explanation. I cannot over emphasize how much this book has contributed to my education over the last year or so.
4. This Theorem is referred to as Determinacy of Evaluation. I may go back and do some simple proofs for my own education after this post and a possible follow up.
5. Technically JavaScript uses a strategy known as Call by Sharing, which differs from Call by Value in how deals with objects. More information at http://dmitrysoshnikov.com/ecmascript/chapter-8-evaluation-strategy/ courtesy of [@raganwald](https://twitter.com/raganwald).
6. For my own education I'm _planning_ to post a proof of determinacy in a later post.
