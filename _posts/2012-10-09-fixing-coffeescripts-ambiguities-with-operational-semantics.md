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
```

## Operational Semantics

borrowing from pierce heavily

explanation as formalism for evaluation

if then else example?

show basic type information applied, including type rules

## CoffeeScript Grammar

You could consider this is an "issue" with the grammar/parser and not the evaluation result, ie. maybe it shouldn't allow both lambda forms as method/function arguments, but because both forms are syntactically valid in the current parser implementation it's only an issue when evaluation takes place. Moreover it's only an *obvious* issue when the return type of `doSomething` doesn't allow for invocation. In an ideal world the author would arrive at that conclusion during language creation, so it may be interesting to use the formalism of operational semantics to help us "boil down" the way these terms can be evaluated.

**IMPORTANT** Application of types will show that in the second case the type collapses to (a -> b) which is red flag

lets take this from the top, as though we were designing the language. Being a JavaScript fan, and likely a functional programming fan you, Jeremy Ashkenas, might decide to begin with the fundamental building blocks of computation as conceived by Alonzo Church: Lambdas. And just to get things rolling, ie avoid church numerals and booleans, you throw in
some constants in the form of true and false.

So in place of `dosomething` and in the interest of avoiding assignment semantics
we'll use lamdas directly to represent the syntactic ambiguity.

```coffeescript
(-> true) () ->  false

# vs

(-> true)() ->  false
```

```
\t ::=
  () -> t
  -> t

t ::=
  true
  false
  (\t)()
  (\t) t

v ::=
  \t
  true
  false
```

explanation of the grammar, t == terms, v == values (results of evaluation)

Note that so far it's not at all obvious there might be an issue. Defining lambdas with a `-> t` actually feels really nice. In addition preparing for lambdas with arguments by allowing for a tuple preceding the arrow is a great idea.

```
(\t)() --> t

(\t) t' --> t
```

```
true : Bool

false : Bool

    t : T_2
----------------
\t : T_1 --> T_2

t : T_1 --> T_2
---------------
 (\t)() : T_2

t : T_1 --> T_2
---------------
 (\t) t' : T_2
```

layout a small subset of the grammar to address the optional parens

subset contains closure syntax (arrow), it's parens with body, and
separately function invocation

OR

look at object as param ambiguities (contact friend from conf)

## Evaluation Relation

walk through the thought process of establishing evaluation rules

pinpoint the spot at which you must choose to support both

prefer the space after the function because it's a more common use case

show proof that there's only one way to evaluate each term inductively, "appendix" again
