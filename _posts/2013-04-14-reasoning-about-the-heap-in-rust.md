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

If you follow programming languages or web technologies closely it's likely that you've heard of [Rust](http://www.rust-lang.org). Rust is one part of a larger effort by [Mozilla Research](http://www.mozilla.org/en-US/research/) to build a new browser engine in [Servo](http://www.mozilla.org/en-US/research/projects/#servo), but its value as a development tool certainly extends beyond that initial goal. In particular it has received attention for its memory model which, "encourages efficient data structures and safe concurrency patterns, forbidding invalid memory accesses that would otherwise cause segmentation faults" [!!].

In this post we'll take a look at the basics of Hoare logic and an extension Separation logic which aid in reasoning about imperative program behavior and memory state. Then, we'll apply those tools to formalize part of [Rust's memory ownership system](http://static.rust-lang.org/doc/0.6/tutorial.html#ownership).

## Hoare logic

In the late 1960s [Tony Hoare](http://en.wikipedia.org/wiki/C._A._R._Hoare) proposed a formal system for reasoning about programs which would eventually be referred to as [Hoare logic](http://en.wikipedia.org/wiki/Hoare_logic). The central feature of Hoare logic is a triple `{P} C {Q}` where `P` and `Q` are predicate logic assertions, referred to as the pre/post conditions, and `C` is a command (reads: program/program fragment). The idea is that, outside of any guarantee of termination[!!], if `P` is true before `C` and `Q` is true after `C`, the triple proves partial correctness for `P` and `Q`.

A simple example using C with the assertions in comments:

```c++
// { x == n }
x = x + 1
// { x == n + 1 }
```

Here the triple `{P} C {Q}` asserts that if `x` is equal to some `n` before `x = x + 1` then `x` will be equal to `n + 1` afterward.

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

[Separation logic](http://en.wikipedia.org/wiki/Separation_logic) is an extension to Hoare logic that provides tools for specifying memory use and safety with new assertions for how a program will interact with the heap and stack[!!].

The four assertions that Separation logic adds for describing the heap are:

* `emp` - for the empty heap. It asserts that the heap is empty, and it can be used to extend assertions about programs that don't interact with the heap.
* `x |-> n` - for the singleton heap. It asserts that there is a heap that contains one cell at address `x` with contents `n`.
* `P * Q` - as a replacement for `∧` with disjoint heaps. It asserts that, if there is a heap where `P` holds and a *separate* (disjoint) heap where `Q` holds, both `P` and `Q` hold in the conjunction of those two heaps.
* `P -* Q` - as a replacement for implication, `=>`. It asserts that there is a heap in which `P` holds then `Q` will hold in the *current heap* extended by the first heap.

There are also some shortcuts for common heap states that are built on top of these four assertions:

* `x |-> n, o, p` is equivalent to `x |-> n * x + 1 |-> o * x + 2 |-> p`. That is, `x` points to a series of memory cells that can be accessed by using `x` and pointer arithmetic.
* `x -> n` is a basic pointer assertion. It is equivelant to `x |-> n * true`, that suggests there is a heap where `n` is the value at `x` which is a part of a larger heap without any properties of interest.

Again we'll turn to C to demonstrate how these assertions fit with common programs.

```c++
// { emp }
int *ptr = malloc(sizeof(int));
*ptr = 5;
// { ptr |-> 5 }
```

The first assertions states that the heap is empty (`emp`). After the `malloc` call and assignment there exists a singleton heap with a single cell containing the value `5` that is pointed to by `ptr`.


```c++
// { emp }
int *ptr = malloc(sizeof(int));
*ptr = 5;
// { ptr |-> 5 }
int *ptr2 = malloc(sizeof(int));
*ptr2 = 5;
// { (ptr |-> 5) * (ptr2 |-> 5) }
```

Adding `ptr2` means the addition of another singleton heap and the connective `*`. It's possible to write this as `{ (ptr -> 5) ∧ (ptr2 -> 5) }`, but this assertion provides no information about how the heap is arranged. It simply says that there are two pointers to the value 5 somewhere in a heap. They might be pointing to the same memory location. By using the singleton heap pointer `|->` and the connective `*`, the new assertion makes clear that the two pointers are *not* pointing to the same memory location.

!! Implication example

```c++
// { emp }
int *arry = calloc(3, sizeof(int));
*arry = 1;
*(arry + 1) = 2;
*(arry + 2) = 3;
// { arry |-> 1,2,3 }
```

Here the comma separated list of values following the singleton pointer in `{ arry |-> 1,2,3 }` denotes contiguous memory. It's simply a shorthand notation for the pointer arithmetic.

```c++
// { arry |-> 1 * (arry + 1) |-> 2 * (arry + 2) |-> 3 }
```

As you can see this maps directly to the C fragment.


## Rust Ownership

Rust provides two new type modifiers for dealing with pointers and memory managment. Both have very specific semantics that are checked at *compile time* to help prevent memory leaks.

* `~` - provides a lexically scoped allocation on the heap. That is, when the newly assigned pointer variable goes out of scope the memory is freed.
* `@` - provides a garbage collected allocation on the heap. In Rust each [task](http://static.rust-lang.org/doc/tutorial-tasks.html) has it's own garbage collector responsible for handling this type of heap allocation.

!! `&` borrowed pointers if there's time (another post?)

I few simple examples borrowed in part from Rust's tutorials will illustrate when the memory for each of these type modifiers is freed.

{% highlight rust %}
fn main() {
    {
        let a = ~0;
    } // a is out of scope and *a is freed
}
{% endhighlight %}

With the tilde, the memory at `*a` on the heap is freed when the variable to which it is assigned goes out of scope. Since `a` is declared inside an explicit block it goes out of scope at the end of the block and the associated memory is freed.

{% highlight rust %}
fn main() {
    let a;

    {
        let b = ~1;

        // move ownership of *b to a
        a = b;

        io::println(fmt!("%?", *a)); // 1
        io::println(fmt!("%?", *b)); // error: use of moved value: `b`
    }
} // *a is destroyed here
{% endhighlight %}

When the ownership of memory is transfered between variables the compiler prevents further reference to the original owner. In this example `a` is the new owner and the compiler will prevent any further reference to `b`.

Alternately, the `@` type modifier can be used to request that the runtime manage the allocated memory on a per-task basis. This presents some interesting issues when creating pointers to memory allocated as part of a record.

{% highlight rust %}
struct X { f: int, g: int }

fn main() {
    let mut x = @X { f: 0, g: 1 };
    let y = &x.f;

    x = @X { f: 2, g: 3 };

    // original *x is now subject to gc

    io::println(fmt!("%?", *x)); // => {f: 2, g: 3}
    io::println(fmt!("%?", *y)); // => 0
}
{% endhighlight %}

Note that `x` is declared as a mutable variable (Rust's default is immutability). When a pointer is made to the field `f` of the record stored at `*x` and `x` is then reassigned, the memory at `*x` would be subject to garbage collection. Luckily Rust does a bit of work for you and inserts a lexically scoped reference to the original record to prevent deallocation by the garbage collector.

{% highlight rust %}
struct X { f: int, g: int }

fn main() {
    let mut x = @X { f: 0, g: 1 };
    let x1 = x;
    let y = &x.f;

    x = @X { f: 2, g: 3 };

    io::println(fmt!("%?", *x)); // => {f: 2, g: 3}
    io::println(fmt!("%?", *y)); // => 0
}
{% endhighlight %}

You might also wonder how Rust handles references to lexically scoped record fields. In this case the compiler raises an error and (rather nicely) highlights the discrepancy in scoping expectations.

{% highlight rust %}
struct X { f: int, g: int }

fn main() {
    let a;

    {
        let mut b = ~X { f: 1, g: 2 };

        // move ownership of *b to a
        a = &b.f;
    }

    // error: illegal borrow: borrowed value does not live long enough
    io::println(fmt!("%?", *a));
}
{% endhighlight %}

Here, `a` is assigned the memory location of the field `f` of `b`, but the scope of `a` is larger than the scope of `b` which means that `*b` will be freed long before `*a`. Both this example and the previous one touched on the notion of borrowed pointers which deserves a separate treatment.

## Formalizing Ownership

Rust's memory management facilities exist mostly at compile time to prevent the user from shooting themselves in the foot, but it's still worth applying Separation logic to get a feel for what's happening to the heap.

{% highlight rust %}
fn main() {
  // { emp }
  {
    // { emp }
    let a = ~0;
    // { a |-> 0 }
    let b = ~1;
    // { a |-> 0 * b |-> 1}
  }
  // { emp }
}
{% endhighlight %}

{% highlight rust %}
fn main() {
  let a;

  {
    // { emp }
    let b = ~1;
    // { b |-> 1 }
    let a = b;
    // { a |-> 1 }
  }
}
{% endhighlight %}

gc/reference scoped, "managed allocation on the heap/ref counting"

{% highlight rust %}
struct X { f: int, g: int }

fn main() {
  // { emp }
  let mut x = @X { f: 0, g: 1 };
  // { x |-> 0, 1 }
  let y = &x.f;
  // { x |-> 0, 1 ∧ y |-> 0 }
  x = X { f: 2, g: 3 }
  // { x |-> 2, 3 * f |-> 0 }
  // !! this assertion fails to describe the heap
  // !! y + 1, originaly x + 1, is a leak
  // !! &x.f now subject to gc
}
{% endhighlight %}

{% highlight rust %}
struct X { f: int, g: int }

fn main() {
  // { emp }
  let mut x = @X { f: 0, g: 1 };
  // { x |-> 0, 1 }
  let x1 = x;
  // { x1 |-> 0,1 ∧ x |-> 0, 1 }
  let y = &x1.f;
  // { x1 |-> 0,1 ∧ x |-> 0, 1 ∧ y |-> 0 }
  x = X { f: 2, g: 3 }
  // { (x1 |-> 0,1 ∧ y |-> 0) * x |-> 2, 3 }
}
{% endhighlight %}

## Conclusion

### Footnotes

* http://static.rust-lang.org/doc/0.6/tutorial.html#introduction
* When termination can be show, the triple proves total correctness.
* There are actually two forms of the assignment axiom. The second proposed by Floyd is more complex but addresses issues that exist in common imperative languages the first cannot.
* Link to papers on Separation logic https://wiki.mpi-sws.org/star/cpl
