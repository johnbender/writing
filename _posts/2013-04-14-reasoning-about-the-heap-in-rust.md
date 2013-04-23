---
layout: post
title: "Reasoning About the Heap in Rust"
tags:
- rust
- math
status: draft
type: post
published: true
listed: false
---

If you follow programming languages or web technologies closely it's likely that you've heard of [Rust](http://www.rust-lang.org). Rust is one part of a larger effort by [Mozilla Research](http://www.mozilla.org/en-US/research/) to build a new browser engine in [Servo](http://www.mozilla.org/en-US/research/projects/#servo), but its value as a development tool certainly extends well beyond that initial goal. In particular it has received attention for its memory model which, "encourages efficient data structures and safe concurrency patterns, forbidding invalid memory accesses that would otherwise cause segmentation faults" [!!].

In this post we'll take a look at the basics of Hoare logic and an extension Separation logic which aid in reasoning about imperative program behavior and heap state. Then, we'll apply those tools to formalize the semantics of [Rust's memory ownership system](http://static.rust-lang.org/doc/0.6/tutorial.html#ownership).

## Hoare logic

In the late 1960s [Tony Hoare](http://en.wikipedia.org/wiki/C._A._R._Hoare) proposed a formal system for reasoning about programs which would eventually be referred to as [Hoare logic](http://en.wikipedia.org/wiki/Hoare_logic). The central feature of Hoare logic is a triple `{P} C {Q}` where `P` and `Q` are predicate logic assertions, referred to as the pre/post conditions, and `C` is a command (reads: program/program fragment). The idea is that, outside of any guarantee of termination[!!], if `P` is true before `C` and `Q` is true after `C`, the triple proves partial correctness for `P` and `Q`.

A simple example using C with the assertions in comments:

```c++
// { x == n }
x = x + 1
// { x == n + 1 }
```

Here the triple `{P} C {Q}` asserts that if `x` is equal to some `n` before the `x = x + 1` then `x` will be equal to `n + 1` afterward.

In addition to this basic structure it's possible to define axioms for common programming constructs like assignment, branching, while loops, and for loops that allow for more general reasoning and manipulation of assertions. For assignment it takes the form `{P[E/V]} V=E {P}`. That is, substituting `E` for `V` in `P` before the assignment should hold and `P` should hold afterward [!!].

```c++
// P[E/V]
x = x + 1
// P
```

Borrowing `Q` from the earlier example here, `P` is `x == n + 1`. Substituting `x + 1` for `x` in `P` gives `x + 1 == n + 1`, or `x == n`, for the precondition.

```c++
// The result of substituting `x + 1` for `x` in `x == n + 1`:
// { x == n }
x = x + 1
// { x == n + 1 }
```

With the help of this and other axioms, established for each programming environment, Hoare logic allows the wielder to write specifications for programs. For most applications (especially those written by my usual reader) the approach might be heavy handed, but there are many domains where this type of specicification is necessary. In particular it's often important to specify the behavior of a program with regards to memory.

## Separation Logic

[Separation logic](http://en.wikipedia.org/wiki/Separation_logic) is an extension to Hoare logic that provides tools for reasoning about memory use and safety by establishing new assertions for specifying how a program will interact with the heap and stack[!!].

The four assertions that Separation logic adds for describing the heap are:

* `emp` - for the empty heap. It asserts that the heap is empty, and it can be used to extend assertions about a program that don't interact with the heap.
* `x |-> n` - for the singleton heap. It asserts that the heap contains one cell at address `x` with contents `n`.
* `P * Q` - as a replacement for `^` with disjoint heaps. It asserts that, if there is a heap where `P` holds and a *separate* (disjoint) heap where `Q` holds, both `P` and `Q` hold in the conjunction of those two heaps.
* `P -* Q` - as a replacement for implication `=>`. It asserts that there is a heap in which `P` holds then `Q` will hold in the *current heap* extended by the first heap.


There are also some shortcuts for common heap states:

* `x |-> n, o, p` is equivalent to `x |-> n * x + 1 |-> o * x + 2 |-> p`. That is, `x` points to a series of memory cells that can be accessed by using `x` and some pointer arithmetic.

confined to memory asserted in {P}
frame rule

summary
empty heap
singleton heap
separating conjunction
separating implication

## Rust Ownership

summary
variables or gc
`~` variable scoped allocation on the heap
`@` reference scoped allocation on the heap (gc'd)
`&` borrowed pointers if there's time (another post?)


{% highlight rust %}
fn main() {
  let a = ~0;
  // a is destroyed here
}
{% endhighlight %}

{% highlight rust %}
fn main() {
  let a;

  {
    let b = ~1;

    // move ownership of *b to a
    let a = b;

    // !! reference to b here will not compile
  }

  // *b is destroyed here
}
{% endhighlight %}

gc/reference scoped, "managed allocation on the heap/ref counting"

{% highlight rust %}
fn main() {
  let mut x = @X { f: 0 };
  let y = &x.f;

  // !! &x.f now subject to gc
  x = X { f: 1 }
}
{% endhighlight %}

{% highlight rust %}
fn main() {
  let mut x = @X { f: 0 };
  let x1 = x;
  let y = &x1.f;

  // :)
  x = X { f: 1 }
}
{% endhighlight %}

## Formalizing Ownership

variable scoped, "uniquely owned allocation on the heap"


{% highlight rust %}
// { emp }
fn main() {
  // { emp }
  let a = ~0;
  // { a |-> 0 }
  let b = ~1;
  // { a |-> 0 * b |-> 1}
}
// { emp }
{% endhighlight %}

{% highlight rust %}
// { emp }
fn main() {
  let a;
  {
    // { emp }
    let b = ~1;
    // { b |-> 1 }
    let a = b;
    // { a |-> 1 ^ !b } ????

    // !! reference to b here will not compile
  }

  // *b is destroyed here
}
// { emp }
{% endhighlight %}

gc/reference scoped, "managed allocation on the heap/ref counting"

{% highlight rust %}
// { emp }
fn main() {
  // { emp }
  let mut x = @X { f: 0, g: 1 };
  // { x |-> 0, 1 }
  let y = &x.f;
  // { x |-> 0, 1 ^ f |-> 0 }
  x = X { f: 2, g: 3 }
  // { x |-> 2, 3 ^ f |-> 0 }
  // !! &x.f now subject to gc
}
// { emp }
{% endhighlight %}

{% highlight rust %}
fn main() {
  let mut x = @X { f: 0 };
  let x1 = x;
  let y = &x1.f;

  // :)
  x = X { f: 1 }
}
{% endhighlight %}

## Conclusion

### Footnotes

* http://static.rust-lang.org/doc/0.6/tutorial.html#introduction
* When termination can be show, the triple proves total correctness.
* There are actually two forms of the assignment axiom. The second proposed by Floyd is more complex but addresses issues that exist in common imperative languages the first cannot.
* Link to papers on Separation logic https://wiki.mpi-sws.org/star/cpl
