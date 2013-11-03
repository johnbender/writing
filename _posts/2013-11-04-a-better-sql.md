---
layout: post
title: "A Better SQL, Part 1"
tags:
- sql
- math
status: draft
type: post
published: true
listed: false
vote: false
---

The relational model for data is ubiquitous. Consequently so is SQL which is the primary way in which users leverage the relational model, but SQL has its warts. In particular schema maintenance with the DDL subset of the language can be awkward when idempotent migrations are called for. This task is so cumbersome that frequently it's delegated to the application layer. In this, the first of two posts proposing improvements to SQL, I'll lay out an alternate semantics for SQL DDL that embraces schema change through syntactic change and expands the expressive power of DDL's declarative core.

## A Common Activity

To illustrate how schema changes break the initially declarative model that DDL provides lets look at what will be our running example:

```sql
create table foo (
  bar int,
  baz text
);
```

All tables start this way. The only piece of syntax that might otherwise alert the user to the fact that this is not an entirely declarative language is `create`. The definition is very much "what" is desired and not "how" to get it. This breaks down when anything about the table needs to change:

```sql
create table foo (
  bar int,
  baz text
);

alter table foo add column bak text;
```

Additionally the user might extend the `create` with an `if exists` which will ensure the proper end state whether the target database is at the initial state without the table `foo` or at the second state where `foo` lacks the column `bak`. With that the script is idempotent, and here it's easy to understand the final state of the table because the examples is very simple. As the schema definition grows more complex through many `drop`s, `add`s and type casts the final state of the table is less clear:

```sql
create table foo (
  bar int,
  baz text
);

alter table foo add column bak text;
alter table foo drop column bak text;
alter table foo add column baks text;
```

Clearly it would be better to simply add columns to the original table definition, and then the table's schema definition would be clear immediately just by glancing at it.

## Differential Semantics

The key insight here is that we can permit schema migrations while retaining an entirely descriptive declarative syntax by appealing to the differential information available in any sane programming environment via source control tools.

In our toy example the desired table definition included a new column `bak`. In our idealized form of SQL that would look like this:

```sql
create table foo (
  bar int,
  baz text,
  bak text
);
```

The SQL runtime only considers the existing syntax and makes no attempt to reconcile that syntax with it's existing schema. This makes perfect sense in light of the fact that a user is permitted to run small snippets in addition to full schema migration scripts. That is, the RDBMS can't know where this declaration is coming from nor why it's being run. On the other hand the well outfitted user can have a strong understanding of both of those facts

If an augmented runtime is provided with more information about how a given declaration has changed it can use that in lieu of the alteration statements to reconcile the current schema state.

```diff
@@ -1,4 +1,5 @@
 create table foo (
   bar int,
-  baz text
+  baz text,
+  bak text
 );
```

Just looking at the diff it's very clear what the intention is: to add the column `bak` to the table. All that's left is to assign some semantics to this diff and its parent table statement. A simple pre-processor might map that change to the corresponding alter statement in DDL, namely the first alter statement from the second example.

## Value Proposition

The value proposition for the simple case is reduced cognitive overhead in maintaining schemas using SQL DDL. In addition, DDL's syntax is reduced by about half because alters and drops can simply go away (this obviously assumes feature parity in the create with the alter statements).

This can also be pushed up the stack to migration tools. For example, Rails generates and maintains a `db/schema.rb` file that is supposed to represent the state of the schema for the associated migrations. A similar technique could be applied there to divine the appropriate alterations when an change to that file is made in place of using migrations for schema changes.

Finally, by associating meaning with syntactic change the user can more safely understand and execute post commit reverts to schema changes. That is, instead of manually defining the necessary steps to "undo" some previous schema change, the source control system can provide the exact information that is necessary given a semantics that's powerful enough to represent the differential.

## Pitfalls

Clearly, not every migration is just about the schema. Frequently the data has to be altered to conform to the target schema. This is actually an area of active research in the Database Systems community [1].

## Conclusion

For the interested reader, I started working on a [preprocessor](https://github.com/johnbender/sql-delta) implemented in Haskell. Unfortunately since I don't have any plans to pursue this further as a research topic I haven't been working on it. Also, for comparison I've included two very simple denotational semantics in the footnotes. They highlight the symmetry of this new approach to the language when compared with the current implementation.

This technique can be extended to other languages that manage system state declaratively like configuration management DSLs or even HTML. Though in the case of configuration management, understanding the mapping between syntax and state is quite complex because system components frequently generate artifacts that are not explicitly declared.

Broadly, the idea of differential semantics is to gather more information about intent from readily available sources so that language runtimes can make informed decisions about user intent. The results need not be confined to accurate interpretation of the desired system state.

In the next post we'll look at how a type system applied to SQL might provide some useful safety properties.

### Footnotes

1. There's a lot of interesting work and tooling around preventing issues resulting from schema migrations: [schema evolution](http://scholar.google.com/scholar?q=prism+schema+evolution&btnG=&hl=en&as_sdt=0%2C5).
2. A denotational semantics for both the current DDL semantics and the proposed semantics. Note that the differential semantics eval function is parameterized by the state of the syntax.

<p  style=" margin: 12px auto 6px auto; font-family: Helvetica,Arial,Sans-serif; font-style: normal; font-variant: normal; font-weight: normal; font-size: 14px; line-height: normal; font-size-adjust: none; font-stretch: normal; -x-system-font: none; display: block;">   <a title="View SQL DDL Differential Semantics on Scribd" href="http://www.scribd.com/doc/181098166/SQL-DDL-Differential-Semantics"  style="text-decoration: underline;" >SQL DDL Differential Semantics</a></p><iframe class="scribd_iframe_embed" src="//www.scribd.com/embeds/181098166/content?start_page=1&view_mode=scroll&show_recommendations=true" data-auto-height="false" data-aspect-ratio="undefined" scrolling="no" id="doc_64435" width="100%" height="600" frameborder="0"></iframe>
