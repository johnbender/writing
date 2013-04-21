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

In the late 1960s [Tony Hoare](http://en.wikipedia.org/wiki/C._A._R._Hoare) proposed a formal system for reasoning about programs which would eventually be referred to as [Hoare logic](http://en.wikipedia.org/wiki/Hoare_logic). The central feature of Hoare logic is a triple `{P} C {Q}` where `P` and `Q` are formal logic assertions and `C` is a command (reads: program/program fragment). The idea is that, outside of any guarantee of termination, if `P` is true before `C` then `Q` will be true afterward.

A simple example using C:

```c
// { x == n }
x = x + 1
// { x == n + 1 }
```

Here the triple `{P} C {Q}` asserts that if `x` is equal to some `n` before the `x += 1` then `x` will be equal to `n + 1` afterward.

In addition to this basic structure there are axioms for common programming constructs like assignment, branching, while loops, and for loops that allow for more general reasoning and manipulation of these assertions. For assignment it takes the form `{P[E/V]} V=E {P}`. That is, substituting `E` for `V` in `P` before should hold and `P` should hold afterward [!!].

```c
// { x + 1 == n + 1}
x = x + 1
// { x == n + 1 }
```

Looking at the assignment axiom, `P` is `x == n + 1` since that's final assertion. Substituting `x + 1` for `x` in `P` gets us `x + 1 == n + 1`.

summary
while example

## Separation Logic

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
* There are actually two forms of the assignment axiom. The second proposed by Floyd is more complex but addresses issues that exist in common imperative languages the first cannot.
