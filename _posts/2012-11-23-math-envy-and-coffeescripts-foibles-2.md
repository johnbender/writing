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

The type information provides a second way to differentiate syntactically similar terms, and there are cases where both the evalution and type information are necessary to distinguish terms. For example `((x, y) -> x + y)(1, 2)` has a type derivation identical to `((x, y) -> x * y)(1, 2)`, but it clearly evaluates differently [1].

With that, a term can be represented by a triple `(S, E, T)`, where `S` is the syntax string of the term, `E` is the evalutation derivation, and `T` is the type derivation. The triple can be used to determine whether two terms will cause confusion.

One approach would be to first compare the `S` values for two triples and then determinine if the `E` and `T` values match. Terms with "similar" `S` values but different `E` or `T` values might be ambigous and could be flagged for review. Using the [Levenshtein Distance](http://en.wikipedia.org/wiki/Levenshtein_distance) to keep the calculation for similarity simple:

!! insert image of dist func

_lev_ is the Levenshtein distance function and _dist_ is just the ratio of the distance between the two strings and the maximum length of both. Normalizing the distances allows for a baseline with a given grammar that might capture all "very similar" terms regardless of the term length. For `(-> true)() -> false` and `(-> true) () -> false`:

!! insert image of dist invocation

An exact definition of string distance that can be reconciled as a threshold "setting" makes building an automated tool a bit easier.

## Happy Parsing

It's time to build something concrete from the formal notion of semantic ambiguity. An AST for this CoffeeScript subset will provide enough information to produce the `(S, E, T)` triple. I've chosen Haskell along with the Alex and Happy libraries to implement a simple lexer and parser. As you would expect the parser BNF definition looks very similar to the grammar definition presented in the previous post:

```
%token
    white   { Whitespace }
    bool    { Boolean $$ }
    '()'    { Unit       }
    '->'    { Arrow      }
    '('     { LeftParen  }
    ')'     { RightParen }

Expr   : Value                      { $1 }
       | Lambda '()'                { Invoke $1 }
       | Expr white Expr            { Apply $1 $3 }

Lambda : '()' white '->' white Expr { Lambda $5 }
       | '->' white Expr            { Lambda $3 }
       | '(' Lambda ')'             { $2 }

Value  : bool                       { BooleanExpr $1 }
       | Lambda                     { $1 }
```

 You can view the full lexer and parser implementations [here](https://gist.github.com/8d7db37e8a6dc99e1ea3).

There are two differences from the original grammar definition. Lambda terms in parenthesis are just a convenience for readability. More importantly application requires that any term be permitted as the left side. This enables the grammar to reproduce the original issue since `(-> true)() -> false` translates to an invocation applied to a lambda term. The corrected grammar:

!! include image of whole fixed grammar

With the corrected grammar an additional inference rule is required to ensure the left term of an application is fully evaluated befor applying it [!!].

!! include image of e-arg-eval and e-app-eval

Where _e-arg-eval_ ensures that the argument of an application is fully evaluated, _e-app-eval_ ensures that the applicand is fully evaluated before the application takes place. The AST result from the parser is built with Haskell types. Pattern matching can be used with the type and inference rules to produce evaluation and derivation results.

## Matching Rules

To start let's look at a simple evaluator and derivation builder implementation.

```haskell
-- an enumeration of each inference rule
data InfRule = Inv | App | ArgEval | AppEval
```

The `InfRule` Haskell type is a simple enumeration of the tags belonging to each inference rule. _e-inv_ corresponds to `Inv` and so on.

```haskell
-- an intermediate form for performing derivation and evaluation
data RuleMatch = None | RuleMatch InfRule (Maybe Expr) Expr

-- match a rule and provide the relevant sub terms for action
matchRule :: Expr -> RuleMatch
```

Both the evaluator and the derivation builder will opperate based on the inference rule that applys to each term and its subterms. The function `matchRule` takes an expression, `Expr`, and provides three pieces of information in a `RuleMatch` result: the inference rule that applies to the term, an optional term for the premise of an inference rule pulled from the body of the parent term, and a term for the conclusion of the inference rule also pulled from the body of the parent term. There are pattern matching definitions for each rule.

```haskell
matchRule (BooleanExpr _) = None
matchRule (Lambda _)      = None
```

The value terms `true`, `false` and `(-> x)` form the base case of the rule match. That is, whenever another function requests a rule match on the value terms `None` is provided to signal that the term has been fully evaluated

```haskell
-- Rule: e-inv
matchRule (Invoke (Lambda t)) = RuleMatch Inv Nothing t
```

Invocation is simple. It can only be applied to a lambda term and the result of the invocation is the lambda's subterm; eg. `(-> true)()` evaluates to `true`. An invocation on anything else will simply drop through this match and ultimately to the catch all `error` case. For example the CoffeeScript `true()` is invalid. Its abstract representation from the parser is `Invoke (BooleanExpr True)` which clearly won't match here. On a match, the `RuleMatch` result contains the rule tag for invocation `Inv`, nothing for an inference rule premise since there isn't one for _e-inv_ and the subterm `t` for futher derivation in the conclusion.

```haskell
-- Rule: e-app
matchRule (Apply (Lambda t) (BooleanExpr _)) = RuleMatch App Nothing t
matchRule (Apply (Lambda t) (Lambda _))      = RuleMatch App Nothing t
```

Application is also simple. Like invocation it only works with lambda terms, but it carries the addition requirement that the argument be a value term. The grammar shows that the only `v` or value terms are lambdas and boolean values so there's a match for those cases here. When there's a match the rule tag is `App` and the lambda subterm is again provided for possible further inspection/operation.

```haskell
-- Rule: e-arg-eval
matchRule (Apply t i@(Invoke _))  = RuleMatch ArgEval (Just i) t
matchRule (Apply t a@(Apply _ _)) = RuleMatch ArgEval (Just a) t

-- Rule: e-app-eval
matchRule (Apply i@(Invoke _) t)  = RuleMatch AppEval (Just i) t
matchRule (Apply a@(Apply _ _) t) = RuleMatch AppEval (Just a) t
```

_e-arg-eval_ and _e-app-eval_ are more complicated than either _e-inv_ or _e-app_ which makes sense when comparing them as inference rules. Both _e-arg-eval_ and _e-app-eval_ carry a premise.

!! include image of both of inference rules

Both rules require that some evaluation take place on one of the subterms. More importantly the shape of the term remains the same. Neither _e-arg-eval_ or _e-app-eval_ change the shape of the term to which they apply, only the shape of the sub terms. This is in contrast to _e-inv_ and _e-app_ which discard the invocation operator and second term respectively. As a result the `RuleMatch` contains the subterm that needs to be evaluated further and the other subterm that remains stagnant. Note that the _e-arg-eval_ rule is applied first so that the _e-app-eval_ rule can ignore the second subterm under the assumption that it's a value term (ie, not `Invoke` or `Apply`).

```haskell
matchRule t = error $ "No inference rule apply's for: " ++ (show t)
```

Finally, in situations like `true()` or `true (-> true)` where no rule applies, an error is raised.

## Evaluating the Options

The information contained in a `RuleMatch` instance can be used to evaluate or derive a given term. Evaluation is a simple matter of applying the rules recursively.

```haskell
-- perform a single evaluation step
eval :: Expr -> Expr
eval t =
    case (matchRule t) of
      None                             -> t
      (RuleMatch _ Nothing t1)         -> t1
      (RuleMatch ArgEval (Just t1) t2) -> Apply t2 (eval t1)
      (RuleMatch AppEval (Just t1) t2) -> Apply (eval t1) t2
```

`eval` performs a single step of evaluation according the the inference rules. The first case match returns the original term `t` because `None` is the match for fully evaluated value terms like `true`, `false`, and `(-> x)`. The second match handles both the `Inv` and `App` by returning the subterm of the invoked or applied lambda term. That is, the `matchRule` function does a bit of evaluation for these two rules by stripping the applied lambda term so `eval` simply works on the subterm. For example, `(-> true) true` and `(-> true)()` become `true` which is `eval`'d and matches the base case.

```haskell
      (RuleMatch ArgEval (Just t1) t2) -> Apply t2 (eval t1)
      (RuleMatch AppEval (Just t1) t2) -> Apply (eval t1) t2
```

For `ArgEval` and its cousin `AppEval` the subterm that needs further evaluation gets it and then the whole term is reassessed. The order of which evaluation happens first is preserved here by recursion. If the argument in an application needs more than one evaluation step, `eval` will continue to work on it until the result is returned to the original invocation. Subsequently if the applicand needs evaluation it will do the same. For example, in `(-> true) (-> true)()` the second term is evaluated with an `Inv` and then the boolean result is the argument to the first lambda term.

```haskell
-- reduce an expression to a value term
fullEval :: Expr -> Expr
fullEval t =
    case (matchRule t) of
      None -> t
      _    -> fullEval $ eval t
```

`fullEval` simply applies `eval` to `t` until it reaches a value term.

## Automating Evaluation Derivation

The `RuleMatch` instance is primarily geared toward building derivation trees. That's why the structure appears so awkward in use with `eval`.

```haskell
data Derivation a = Empty | Derivation a [Derivation a] (Derivation a)
```

The `Derivation` data type get's instantiated with a rule type. In this case it's instantaited with `InfRule` since we're dealing with evaluation (type derivation will be discussed later). The first `a` is a tag from the `InfRule` enumeration, the list of sub-derivations represents zero or more possible premise requirements, and the final derivation is the conclusion. Taking the derivation tree of a simple example `(-> (-> true)) false` which is parsed to:

```haskell
Apply (Invoke (Lambda (Lambda (BooleanExpr True)))) (BooleanExpr False)
```

In english, the application of an invocation of a lambda with a lambda subterm to a boolean value. The resulting derivation tree in the original notation takes the form:

!! include image derivation tree

The `Derivation` instance has to work from the outside in so it's much harder to read than the notation, but it contains the same information

```haskell
Derivation AppEval [Derivation Inv [] Empty] (Derivation App [] Empty)
```

Working from the outside in, it's clear that the applicand `(-> (-> true)()` needs evaluation before it can be applied to the argument `false` (_e-app-eval_). The premise of _e-app-eval_ requires that the applicand take a step and here that means an invocation (_e-inv_). Finally the result of the invocation `(-> true)` is applied to the `false` (_e-app_) as the "conclusion" of the _e-app-eval_. In reality, _e-app_ is applied to the result of the first derivation tree as it is with the logic notation.

```haskell
-- build a derivation from an expression
derive :: Expr -> Derivation InfRule
derive t =
    case (matchRule t) of
      None                         -> Empty
      (RuleMatch rule Nothing t1)  -> Derivation rule [] (derive t)
      (RuleMatch rule (Just t1) _) -> Derivation rule [derive t1] (derive $ eval t)
```

The `derive` function works in a similar fashion to `eval`. For a value/`None` result from `matchRules` there are no inference rules that apply. For _e-inv_ or _e-app_ derive can recurse and build a derivation from the lambda's subterm. For _e-arg-eval_ or _e-app-eval_ the premise must be further derived and the conclusion is a derivation for the original term `t` with one evaluation step applied. That is, evaluating the subterm `t1` once inside the original term `t`. The use of `eval` to do that in this case is a convenience.

The conversion of the CoffeeScript AST to a derivation tree means that the `S` and `E` of the `(S, E, T)` triple are in place. All that's left is to convert the AST into a type derivation.

## Automating Type Derivation



### footnotes

1. It might be that when a function identifier is the only difference between terms, here `*` and `+`, it's reasonable to ignore ambigous terms. In this case because the total string length for both terms is small it might be that a single character difference is enough to break some arbitrary threshold. I'm leaving this for futher consideration.
2. Assuming it's possible, it's interesting to think abougt what the inverse result means. That is, when two terms are very syntactically different but have identical types/evaluation derivations. This might signal the two terms or the parent language as antithetical to Python's slogan of "one and only one way to do it".
3. The implementation in Haskell forced these issues out into the open. I'm curious if proving progress and preservation would have pointed out the flaws in my approach (this may be obvious one way or another to a better educated reader).
