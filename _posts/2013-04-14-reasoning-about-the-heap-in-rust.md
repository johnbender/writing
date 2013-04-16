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

If you follow programming languages or web technologies closely it's likely that you've heard of [Rust](http://www.rust-lang.org). Rust is one part of a larger effort by [Mozilla Research](http://www.mozilla.org/en-US/research/) to build a new browser engine in [Servo](http://www.mozilla.org/en-US/research/projects/#servo), but its value as a development tool certainly seems to extend well beyond that initial goal. In particular it has received attention for its memory model which, "encourages efficient data structures and safe concurrency patterns, forbidding invalid memory accesses that would otherwise cause segmentation faults" [!!].

In this post we'll take a look at the basics of Hoare logic and an extension Separation logic which aid in reasoning about imperative program behavior and heap state. Then, we'll apply those tools to formalize the semantics of [Rust's memory ownership system](http://static.rust-lang.org/doc/0.6/tutorial.html#ownership).

## Hoare logic

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
