---
layout: post
title: Fixing CoffeeScript's Ambiguities with Operational Semantics
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

You could consider this a problem with the grammar and parser and not the evaluation result. That is, maybe it shouldn't allow both lambda forms as method/function arguments, but because both forms are valid in the current parser implementation it's only an issue when evaluation takes place. Moreover it's only an *obvious* issue when the return type of `doSomething` doesn't allow for invocation. In an ideal world the author would arrive at these conclusions early during the process of creating the language grammar, and it might be interesting to use the formalism of operational semantics to help "boil down" the way these terms can be evaluated, hopefully revealing the ambiguity.

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

It's interesting to note that neither case makes it particularly obvious there's a problem. Having identified a subset of the language that can reproduce the original issue, a grammar is necessary to get a precise definiton of the terms required to build it.

<div class="center">
<img src="/assets/images/diagrams/cs-grammar.png"></img>
</div>

`v`, `\t` and `t` all represent meta terms. In the case of `\t` it was easier to create a meta term in place of repeating each possible lambda form in `t`. `v` on the other hand has special meaning as it did in the operational semantics example. It represents the set of acceptable final results of evaluating expressions in our CoffeeScript subset. Finally `t` is total set of terms and possible compound terms. Notable among them is the invocation of, and application of a single argument to, lambda terms (`\t`).

Some examples of our CoffeeScript subset translated into the grammar, starting with the most basic:

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

With the grammar in place the next step is to define both the evaluation rules and strategy so that our terms can perform some computation. To keep things simple our language subset will use the call-by-value left to right strategy employed by CoffeeScript and numerous other languages (C, Java, Python etc). As you might have guessed by looking at the grammer there are only two evaluation rules, both of which are concerned with evaluation of lambda terms.

```
(\t)() --> t

(\t) t' --> t
```

The impact of the evaluation strategy (call-by-value l-to-r) for our little language is that the arguments applied to lambda terms in the second evaluation rule must be as "evaluated" as far as possible, before the invocation can take place. For example `(-> true) (-> (-> false))()` would take a step first to `(-> true) (-> false)` as a result of the `()` operator before applying the second lambda term as an argument to the first producing `true`. Ultimately this is unimportant in addressing the particular set of ambiguous expressions we've chosen but is noted here for completeness. Otherwise both evaluation rules are simple. Given the `()` operator or the application of a fully evaluated term, a lambda expression will evaluate to its subterm.

So far it's not obvious there might be an issue. Defining lambdas with a `-> t` is appealing, and preparing for arguments with a null tuple preceding the arrow makes sense. Continuing on, the application of type rules to our language is the last step to having a complete picture of how it works.


# Type Rules

```
true : Bool

false : Bool

    t : T_2
----------------
\t : T_1 --> T_2

\t : T_1 --> T_2
---------------
 (\t)() : T_2

\t : T_1 --> T_2
---------------
 (\t) t' : T_2
```

The type rules are fairly intuitive. Given the statment above the line, the statment below the line is the type. For those type rules without a premise (the statement above the line) like `true : Bool` and `false : Bool`, we take them to be true out of hand. One extremely important note: because neither of the lambda expressions make use of arguments, Top (`Top`) is used to represent the acceptance of any type. So when `t` in `\t` has the type `T_1` it means that `\t` will, under invocation or application, result in an expresion of typ `T_1` regardless of what it's applied to or invoked with. As an example:

```
(-> true) : \top --> Bool

(-> true) (-> false) : Bool
```

When using the second lambda as an "argument", it's ignored establishing the type from the result of the first lambda term (`true`). From this it's clear that the type of the "argument" doesn't matter.

At first, this notation might look arbitrary but it makes it easy to establish the type of a term by "stacking" the type rules on one another. For example the term `(-> true)`:

```
(no premise)
------------
true : Bool
-------------------
\true : \top --> Bool

```

Beginning with the most basic type rule for one of the values `true` (lacking a premise) the stack expands outward from the subterms to establish the original term's type. This stacking is referred to as a derevation tree. Taking the more complex term from earlier `(-> true) (-> (-> false))()`, it's clear why:


```
                       (no premise)
                       ------------
                       false : Bool
                       -----------------
(no premise)           \false : \top --> Bool
------------           ----------------------------------
true : Bool             \\false : \top --> (\top --> Bool)
--------------------   ----------------------------------
\true : \top --> Bool   (\\false)() : \top --> Bool
---------------------------------------------------------
            \true (\\false)() : Bool
```

Since there are two subterms involved in application, each branch extends upward until it reaches the atomic value types `true : Bool` and `false : Bool`.


prefer the space after the function because it's a more common use case

show proof that there's only one way to evaluate each term inductively, "appendix" again


# footnotes

!! Tested at http://coffeescript.org/.
