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

In the [previous post](/2012/11/27/math-envy-and-coffeescripts-foibles/) I presented the basics of operational semantics and showed how derivations trees can be used to differentiate two terms that are syntactically similar. This post develops the closing thoughts further with the introduction of type rules, example tools for automating evaluation and type derivation, and a concrete definition of semantic ambiguity. The primary goal is to establish the best way to detect ambiguous term pairings and then outline what will work for a tool that can be generalized beyond the CoffeeScript subset.

## Type Rules

Type rules are similar in construction to evaluation rules, consisting of a premise and conclusion. As with evaluation rules the premise establishes the preconditions for the conclusion. Again, each rule is tagged with a name for reference but preceded by a _t-_ in this case to distinguish them from inference rules (_e-_).

<div class="center">
  <img src="/assets/images/diagrams/cs-type-rules.png"></img>
</div>

Type rules without a premise like _t-true_ and _t-false_ are taken to be true out of hand. That is, the terms `true` and `false` both have the type `Bool`. The others are more complicated.

<div class="center">
  <img src="/assets/images/diagrams/cs-type-rules-lambda-term.png"></img>
</div>

_t-lambda_ illustrates how to determine the type of a lambda term like `(-> true)`. The premise above the line states that if the subterm `t` has the _concrete_ type `T`, then the conclusion `λt` has the type `X -> T`. Here `X` is a _type variable_ because we don't know whether the lambda will be evaluated with the invocation operator `()` or applied to an argument. `T` will be concrete because it can be determined from the body of the lambda expression. For example, in `(-> true)` the subterm `true` has the type `Bool` so the lambda term has the type `X -> Bool`.

<div class="center">
  <img src="/assets/images/diagrams/cs-type-rules-lambda-invocation.png"></img>
</div>

_t-inv_ shows how to determine the type of lambda invocation like `(-> true)()`. The premise states if the lambda term has the type `X -> T` the term `λt()` has the type `T`. For example, `(-> true)()` evaluates to `true` and has the type `Bool`. It's worth noting that `X` is constrained to be the `Unit` or empty type since no argument is used.

<div class="center">
  <img src="/assets/images/diagrams/cs-type-rules-lambda-application.png"></img>
</div>

_t-app_ is the type rule for lambda applications, eg. `(-> true) false`. The premise says that if the lambda term `λt` on the left has the type `X -> T` the conclusion is that the application will have the result type of the lambda term. Again, the type of an application, like invocation, is only concerned with the type of the *first* lambda's subterm `t` and it ignores the type of the argument that it's applied to.

## Type Rule Stacking

This notation makes it easy to establish the type of a term by stacking the type rules on one another in the same fashion as evaluation rules. Taking a very simple example, some diagrams will illustrate how this works:

```coffeescript
(-> true)
```

<div class="center">
  <img src="/assets/images/diagrams/cs-type-derivation-simple.png"></img>
</div>

This highlights how to derive the type at the bottom from it's subterms. Typing the innermost subterm `true` with _t-true_. That can then be "stacked" by using it to replace the premise of type rule _t-lambda_. The type derivation expands from the subterm to establish each subsequent parent term's type. Another more complex derivation:

```coffeescript
(-> (-> false))() true
```

<div class="center">
  <img style="width: 300px" src="/assets/images/diagrams/cs-type-derivation-complex.png"></img>
</div>

The second subterm of the application is unimportant where the type of the term is concerned and as a result it's wholly ignored. Working on the left term, the tree extends upward until it reaches the atomic value type, `false : Bool`. The complexity of nested lambdas and invocation make for a taller stack of type rules to reach the atomic `false` when compared with the previous example.

## Not Quite There

At this point the type rules can describe the original issue. A derivation tree based on the typing rules highlights that the term is untypable. Taking our canonical example, `(-> true)() -> false`:

<div class="center">
  <img style="width: 300px" src="/assets/images/diagrams/cs-type-derivation-original-issue.png"></img>
</div>

Once the derivation tree reaches the outermost term it breaks. There is no type rule for the application of something with type `Bool` to something with type `X -> Bool` since _t-app_ requires the first term have the type `X -> T` in its premise. It's a type error.

Previously we saw that this this would result in a type error under evaluation by the CoffeeScript interpreter. We also saw that it was easy to construct a term that suffered the same semantic confusion without the type error `(-> (-> true))() -> false`. This issue applies to the type derivation as well.

In addition we saw that it's possible to construct terms, albeit in the boolean example language, that might produce the same value through different evaluation paths. That is, they had different derivation trees in the evaluation relation but the same evaluation result. This issue also applies to type derivations.

In both cases useful information is lost when the derivation is discarded in favor of the final value or type. The advantage with the type information is obviously that no evaluation is required to determine if two terms are "different" in some way other than their syntax. The disadvantage is that not all languages make determining type information easy.

Ultimately the type information provides a second way to differentiate syntactically similar terms. Indeed there are cases where both the evaluation and type information are necessary to distinguish terms. For example `((x, y) -> x + y)(1, 1)` has a type derivation identical to `((x, y) -> x * y)(1, 2)` and the same evaluation result, but it clearly takes a different evaluation path [1].

_Note: The next five sections cover an implementation of a lexer, parser, evaluator and mechanisms for type/evaluation derivation. If you'd rather just read about how the generated evaluation and type derivations are used to find confusing term pairings you can skip to Detecting Ambiguity_

## Happy Parsing

It's time to build something concrete from the formal notion of evaluation and types. An AST for this CoffeeScript subset will provide enough information to perform evaluation and establish derivation trees for both evaluation and types. I've chosen Haskell along with the [Alex](http://www.haskell.org/alex/) and [Happy](http://www.haskell.org/happy/) tools to implement a simple lexer and parser. As you would expect the parser grammar definition looks very similar to the grammar definition presented in the previous post:

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

There are two differences from the original grammar definition. Lambda terms in parenthesis are just a convenience for readability. More importantly a correction must be made to application of two terms, allowing for any term as the left side (the _applicand_) [2]. This enables the grammar to reproduce the original issue, since `(-> true)() -> false` translates to an invocation applied to a lambda term. The corrected grammar:

<div class="center">
  <img style="width: 200px;" src="/assets/images/diagrams/cs-grammar-corrected.png"></img>
</div>

Also, a correction and an addition must be made to the inference rules presented in the previous post. This will ensure that any term type is permitted as the left half of an application, and that it is fully evaluated before applying it.

<div class="center">
  <img src="/assets/images/diagrams/cs-inference-rules-corrected.png"></img>
</div>

Where _e-arg-eval_ ensures that the argument of an application is fully evaluated, _e-app-eval_ ensures that the applicand is fully evaluated before the application takes place.

## Matching Rules

The abstract representation produced by the parser is a simple tree structure built with Haskell types. Pattern matching can be used with the type and inference rules to produce evaluation and derivation results. To start let's look at a simple evaluator and derivation builder.

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

Both the evaluator and the derivation builder will operate based on the inference rules that apply to each term and its subterms. The function `matchRule` takes an expression, `Expr`, and provides three pieces of information in a `RuleMatch` result: the inference rule that applies to the term, an optional term for the premise of an inference rule pulled from the body of the parent term, and a term for the conclusion of the inference rule also pulled from the body of the parent term. There are pattern matching definitions for each rule.

```haskell
matchRule (BooleanExpr _) = None
matchRule (Lambda _)      = None
```

The value terms `true`, `false` and `(-> x)` are the base case of `matchRule`. That is, whenever another function requests a rule match on the value terms `None` is provided to signal that the term has been fully evaluated.

```haskell
-- Rule: e-inv
matchRule (Invoke (Lambda t)) = RuleMatch Inv Nothing t
```

Invocation can only be applied to a lambda term and the result of the invocation is the lambda's subterm, eg. `(-> true)()` evaluates to `true`. An invocation on anything else will simply drop through this match and ultimately to the catch all `error` case. For example the CoffeeScript `true()` is invalid. Its abstract representation from the parser is `Invoke (BooleanExpr True)` which clearly won't match here. On a match, the `RuleMatch` result contains the rule tag for invocation `Inv`, nothing for an inference rule premise since there isn't one for _e-inv_ and the subterm `t` for further derivation in the conclusion.

```haskell
-- Rule: e-app
matchRule (Apply (Lambda t) (BooleanExpr _)) = RuleMatch App Nothing t
matchRule (Apply (Lambda t) (Lambda _))      = RuleMatch App Nothing t
```

Like invocation _e-app_ only works with lambda terms, but it carries the addition requirement that the argument be a value term. The grammar shows that the only `v` (value) terms are lambdas and boolean values so there's a match for those cases here. When there's a match the rule tag is `App` and the lambda subterm is again provided for possible further inspection/operation.

```haskell
-- Rule: e-arg-eval
matchRule (Apply t i@(Invoke _))  = RuleMatch ArgEval (Just i) t
matchRule (Apply t a@(Apply _ _)) = RuleMatch ArgEval (Just a) t

-- Rule: e-app-eval
matchRule (Apply i@(Invoke _) t)  = RuleMatch AppEval (Just i) t
matchRule (Apply a@(Apply _ _) t) = RuleMatch AppEval (Just a) t
```

_e-arg-eval_ and _e-app-eval_ are more complicated than either _e-inv_ or _e-app_ which makes sense when comparing them as inference rules. Both _e-arg-eval_ and _e-app-eval_ carry a premise.

<div class="center">
  <img src="/assets/images/diagrams/cs-inference-rules-app-argeval.png"></img>
</div>

Both rules require that some evaluation take place on one of the subterms. More importantly the shape of the term remains the same. Neither _e-arg-eval_ or _e-app-eval_ change the shape of the term to which they apply, only the shape of the sub terms. This is in contrast to _e-inv_ and _e-app_ which discard the invocation operator and second term respectively. As a result the `RuleMatch` contains the subterm that needs to be evaluated further and the other subterm that remains stagnant. Note that in the function definition the _e-arg-eval_ rule is matched first so that the _e-app-eval_ rule can ignore the second subterm under the assumption that it's a value term (not `Invoke` or `Apply`).

```haskell
matchRule t = error $ "No inference rule applies for: " ++ (show t)
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

`eval` performs a single step of evaluation according the the inference rules. The first case match returns the original term `t` because `None` is the match for fully evaluated value terms like `true`, `false`, and `(-> x)`. The second match handles both the `Inv` and `App` by returning the subterm of the invoked or applied lambda term. The `matchRule` function does a bit of evaluation for these two rules by stripping the applied lambda term. For example, `(-> true) true` and `(-> true)()` become `true`.

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
data Derivation a = Empty | Derivation InfRule Derivation Derivation Expr
```

The `Derivation` data type is comprised of a tag from the `InfRule` enumeration, one _possible_ derivation as a premise, the final derivation as the conclusion, and the expression representing the state of evaluation at a given moment. Taking the derivation tree of a simple example `(-> (-> true))() false` which is parsed to:

```haskell
Apply (Invoke (Lambda (Lambda (BooleanExpr True)))) (BooleanExpr False)
```

In english, the application of an invocation of a lambda with a lambda subterm to a boolean value. The resulting tree in the original notation takes the form:

<div class="center">
  <img src="/assets/images/diagrams/cs-derivation-trees-ast-example.png"></img>
</div>


The `Derivation` instance has to work from the outside in so it's much harder to read than the notation, but it contains the same information

```haskell
Derivation AppEval
  -- |       premise        | |   e-inv value            |
  (Derivation Inv Empty Empty (Lambda (BooleanExpr True)))

  -- |      conclusion      | |   e-app value   |
  (Derivation App Empty Empty (BooleanExpr True))

  -- |                e-app-eval value                  |
  (Apply (Lambda (BooleanExpr True)) (BooleanExpr False))
```

It's clear that the applicand `(-> (-> true)()` needs evaluation using _e-app-eval_ before it can be applied to the argument `false`. The premise of _e-app-eval_ requires that the applicand take a step and here that means an invocation with _e-inv_. Finally the result of the invocation `(-> true)` is applied to the `false` with _e-app_ as the "conclusion" of the _e-app-eval_. In reality, _e-app_ is applied to the result of the first derivation tree as it is with the logic notation.

```haskell
-- build a derivation from an expression
derive :: Expr -> Derivation
derive t =
 case (matchRule t) of
  None                         -> Empty
  (RuleMatch rule Nothing t1)  -> Derivation rule Empty (derive t1) (evald)
  (RuleMatch rule (Just t1) _) -> Derivation rule (derive t1) (derive $ evald) (evald)
 where evald = eval t

```

The `derive` function works in a similar fashion to `eval`. For a value/`None` result from `matchRules` there are no inference rules that apply. For _e-inv_ or _e-app_, `derive` can recurse and build a derivation from the lambda's subterm. For _e-arg-eval_ or _e-app-eval_ the premise must be further derived and the conclusion is a derivation for the original term `t` with one evaluation step applied. That is, evaluating the subterm `t1` once inside the original term `t`. The use of `eval` to do that may look funny but it's just a convenience.

## Automating Type Derivation

Deriving the type for a term in the CoffeeScript subset is slightly less complex than deriving the evaluation. Again, a type rule is matched to each valid AST construction.

```haskell
data RuleMatch  = None | RuleMatch TypeRule Expr

-- match a rule and provide the relevant sub terms for action
matchRule :: Expr -> RuleMatch
```

The `RuleMatch` definition for types requires one less `Expr`. The `derive` and `fixType` definitions for types only require the first subterms in each expression. This is in contrast to `eval` which required both the conclusion and premise terms.

```haskell
-- t-true & t-false
matchRule b@(BooleanExpr True)    = RuleMatch TrueType b
matchRule b@(BooleanExpr False)   = RuleMatch FalseType b

-- t-lambda
matchRule (Lambda t)              = RuleMatch LambdaType t

-- t-inv
matchRule (Invoke (Lambda t))     = RuleMatch Inv t

-- t-apply
matchRule (Apply t@(Lambda _) _)  = RuleMatch App t
matchRule (Apply t@(Invoke _) _)  = RuleMatch App t
matchRule (Apply t@(Apply _ _) _) = RuleMatch App t
```

The `Apply` matches capture only valid applicands and let the rest fall through to the error case. It's also worth noting that each of the `Apply` matches discard the argument term because it's unnecessary to the type of the expression. This fits with the definition of the type rules.

Fixing the type of a given expression is a simple recursive effort on applications. The `Type` data type captures both the `Bool` result, and the recursive `Arrow` type. For example `(-> (-> true))` has the type `Arrow (Arrow Bool)`.

```haskell
-- the two possible types for a given expression
data Type = Bool | Arrow Type

-- determines the type of a given expression
data Derivation = Empty | Derivation TypeRule Derivation Type

fixType :: Expr -> Type
fixType t =
    case (matchRule t) of
      (RuleMatch TrueType _)    -> Bool
      (RuleMatch FalseType _)   -> Bool
      (RuleMatch LambdaType t1) -> Arrow $ fixType t1
      (RuleMatch Inv t1)        -> fixType t1
      (RuleMatch App _)         -> fixType $ eval t
```

The type of an invocation is determined by the lambda's subterm, so `matchRule` provides that as `t1` here for further type information. The type of an application is dependent on the type of it's first argument, so we cheat a bit here and use the single step `eval` to get at the result of the application.

```haskell
derive :: Expr -> Derivation
derive t =
    case (matchRule t) of
      (RuleMatch TrueType _)    -> Derivation TrueType Empty $ fixType t
      (RuleMatch FalseType _ )  -> Derivation FalseType Empty $ fixType t
      (RuleMatch rule t1)       -> Derivation rule (derive t1) $ fixType t
```

The type rules are much easier to apply, they simply descend into the terms to build up the type, providing the fixed type at each step as the conclusion. Taking the same example from the evaluation rules earlier `(-> (-> true))() false` which is parsed to:

```haskell
Apply (Invoke (Lambda (Lambda (BooleanExpr True)))) (BooleanExpr False)
```

The type derivation using logic notation looks like:

<div class="center">
  <img style="width: 300px" src="/assets/images/diagrams/cs-type-derivation-ast-example.png"></img>
</div>

The `Derivation` instance corresponding the the logic notation is again much larger but captures the same information (formatting added after the fact):

```haskell
Derivation App
  (Derivation Inv
    (Derivation LambdaType
      (Derivation TrueType Nothing Bool)
    ) (Arrow Bool) -- Lambda type is X -> <subterm type>
  ) (Arrow Bool) -- Inv type is it's subterm's type
) Bool -- App type T in the applicand's X -> T
```

As noted in the comments each step in the derivation resolves the type at that step based on the inference rules.

## Detecting Ambiguity

For this CoffeeScript subset we've seen that it's possible to build an understanding of evaluation and typing that provides more information than the evaluation result or the fixed type of a individually. Capturing that extra information a term can be represented by a triple `(S, E, T)`, where `S` is the syntax string of the term, `E` is the evaluation derivation, and `T` is the type derivation. This triple can be used to determine whether two terms will cause confusion.

One approach would be to first compare the `S` values for two triples and then determine if the `E` and `T` values match. Terms with "similar" `S` values but different `E` or `T` values might be ambiguous and could be flagged for review. Using the [Levenshtein Distance](http://en.wikipedia.org/wiki/Levenshtein_distance) to keep the calculation for similarity simple:

<div class="center">
  <img src="/assets/images/diagrams/cs-dist.png"></img>
</div>

_dist_ is the Levenshtein distance function and _dist_ is just the ratio of the distance between the two strings and the maximum length of both. This is sometimes referred to as the Levenshtein Ratio. Providing a ratio allows for a baseline with a given grammar that might capture all "very similar" terms regardless of the term length. For `(-> true)() -> false` and `(-> true) () -> false`:

<div class="center">
  <img style="width: 300px" src="/assets/images/diagrams/cs-dist-example.png"></img>
</div>

An exact value for string distance that can be reconciled as a threshold "setting" makes building an automated tool a bit easier. That is, if two terms are deemed "close enough" by virtue of their _dist_ value being below a predetermined threshold and they have different information in either `E` or `T` then they might be flagged [3].

## Fuzzy Search

With a more complete understanding of semantic ambiguity and some tooling to provide the derivation info the next step is to analyze terms and detect ambiguous pairings. That is, we can now define a system that will automate the exploration of the "term space" (all term combinations), and run a check against existing known terms for ambiguous pairs for each generated term.

Storing the triple of known terms for comparison would be fairly easy with the text search capabilities available in most modern databases. You could even [implement](http://www.artfulsoftware.com/infotree/qrytip.php?id=552) the Levenshtein Distance function and ratio and use it to check an new term against known terms. It may be that a purpose built data structure for the storage and retrieval based on a text search algorithm would perform better, but a good all purpose RDBMS would suffice for a first pass.

More interesting is the generation of terms for a non-trivial language. It's seems likely that a _term generator_ would have to start with atomic types and successively wrap them in terms defined with subterms. That part can likely be performed with nothing more than knowledge of the grammar, but there are two issues.

First, the complexity of many programming languages makes re-examining the same terms an enormous waste of time. Tracking the explored terms and "resuming" the exploration process would have a lot of value. Second, generating the derivations to store and compare along with the syntax is an involved effort. Again, It's easy to tag a piece of syntax with the result of execution or typing but information is lost.

## Quick and Dirty

To avoid the extra effort required of the language creator in generating the evaluation and type derivations, a less complicated representation of a term might still be effective. For example the tuple `(S, A)`, where `S` remains the syntax of the term and `A` is the AST representation. Using the lambda term example, and haskell parser detailed earlier:

```haskell
( "(-> true)() -> false",
  Apply (Invoke (Lambda (BooleanExpr True))) (Lambda (BooleanExpr False)) )

( "(-> true) () -> false",
  Apply (Lambda (BooleanExpr True)) (Lambda (BooleanExpr False)) )
```

It's obvious that the abstract representations capture the issue at hand even if there is some information lost [4]. Best of all the AST for a term would be available regardless of the host language and serialization is the only extra requirement. With a term generator that works with a (E)BNF, a way to generate the AST for a term (presumably through the language parser), and a database equipped with the ability to find like terms it seems entirely possible to alert the language creator of complex or convoluted pairings.

## Further Work

First I have to apologize for not building out a tool for generating terms or a schema for term storage. I wanted to do the automated evaluation and type derivations to get a feel for the effort involved and the result was an exceptionally long post. If I find the time to return to this I'd like to build out the term generator and couple it with a simple database. I think that going through the process of building a BNF parser would be a lot of fun by itself.

In the course of these two posts we've seen what it looks like to formalize both the evaluation and type semantics of a simple programming language. We've also come to a relatively satisfying formalization of semantic ambiguity that could be used in conjunction with a common language definition form (BNF, EBNF) to alert a language designer of potential issues [5] [6].

### footnotes

1. It might be that when a function identifier is the only difference between terms, here `*` and `+`, it's reasonable to ignore ambiguous terms. In this case because the total string length for both terms is small it might be that a single character difference is enough to break some arbitrary threshold. I'm leaving this for further consideration.
2. The implementation in Haskell forced these issues out into the open. I'm curious if proving progress and preservation would have pointed out the flaws in my approach (this may be obvious one way or another to a better educated reader).
3. Assuming it's possible, it's interesting to think about what the inverse result means. That is, when two terms are very syntactically different but have identical types/evaluation derivations. This might signal the two terms or the parent language as antithetical to Python's slogan of "one and only one way to do it".
4. For example, the AST doesn't capture the type of lambda form that was used. This may be useful information even if this particular example doesn't require it.
5. Though it would be infinitely more satisfying if we could build a tool based on the ideas and arrive at that same conclusions about this CoffeeScript subset and a few other BNF friendly languages.
6. It's worth pointing out that the CoffeeScript issue with lambdas and invocation [has been/was known to Jeremy](http://news.ycombinator.com/item?id=4849151). It was simply a choice in favor of flexibility. I like to think that the hypothetical tool presented here would be useful in cases where ambiguous term pairings are less obvious and for people who may want less flexibility.
