---
layout: post
title: "A Better SQL"
tags:
- sql
status: published
type: post
published: true
listed: true
vote: https://news.ycombinator.com/item?id=6670048
---

The relational model for data is ubiquitous. That's in part due to SQL's declarative approach to manipulating and exploring data stored as relations. Unfortunately SQL has its warts. In particular schema changes made in the data definition subset of the language (DDL) [1] can be awkward for creating idempotent migrations. Enough so, that the responsibility is frequently delegated to the application layer where more expressive languages can be employed. Here I'll lay out an alternate semantics for SQL DDL that embraces schema change and expands the expressive power of DDL's declarative core.

## A Common Activity

To illustrate how schema changes break the initially declarative semantics of DDL, lets look at an example:

```sql
create table foo (
  bar int,
  baz text
);
```

All tables start this way. The only piece of syntax that might otherwise alert a new user to the fact that this is not an entirely descriptive declarative language is `create`. The definition is very much "what" is desired and not "how" to get it. This breaks down when anything about the table needs to change:

```sql
create table if not exists foo (
  bar int,
  baz text
);

alter table foo add column bak text;
```

All together this will ensure the proper end-state whether the target database is at the initial state without the table `foo` or at the second state where `foo` lacks the column `bak`. In this case it's easy to understand the final state of the table because the example is very simple, but it's acquired an imperative pall with the inclusion of the first `alter`. As the schema definition grows more complex through many `drop`, `add`, and type cast changes, the final state of the table becomes less clear:

```sql
create table if not exists foo (
  bar int,
  baz text
);

alter table foo add column bak text;
alter table foo drop column bak text;
alter table foo add column baks text;
```

It would be better to simply add columns to the original table definition, and then the shape of the resulting table would be immediately clear at a glance.

## Differential Semantics

In our toy example the desired table definition included a new column `bak`. An entirely descriptive update to the table declaration would look like this (Note that the original alter statement is absent):

```sql
create table foo (
  bar int,
  baz text,
  bak text
);
```

Unfortunately the SQL runtime considers the syntax in isolation and makes no attempt to reconcile that with it's internal representation. That makes perfect sense because a user is permitted to run small, ad hoc snippets in addition to full schema migration scripts. That is, the RDBMS can't know where this declaration is coming from nor why it's being run so it's unsafe to assume it should do any reconciliation. In contrast a well outfitted user can provide exactly that information.

```diff
@@ -1,4 +1,5 @@
 create table foo (l;
   bar int,
-  baz text
+  baz text,
+  bak text
 );
```

Looking at the diff, it's clear that the intention is to add the column `bak` to the table. What's required then, is to assign some semantics to this diff. With that established a simple pre-processor could map this differential to the corresponding alter statement in DDL, namely the original alter statement.

The key insight here is that we can permit schema migrations while retaining an entirely descriptive declarative syntax by appealing to the differential information available via source control tools.

## Value Proposition

The basic value proposition is reduced cognitive overhead when maintaining schemas using SQL DDL. In addition, DDL's syntax is reduced by about half because alters and drops [2] can simply go away which should make it easier to learn [3].

This could also be pushed up the stack to migration tools by an enterprising library or framework author. For example, Rails generates and maintains a `db/schema.rb` file that is supposed to represent the state of the schema for the associated migrations. A similar technique could be applied there to divine the appropriate alterations when an change to that file is made in place of using migrations for schema changes.

Finally, by associating meaning with syntactic change the user can more safely understand and execute post commit reverts to schema changes. That is, instead of manually defining the necessary steps to "undo" some previous schema change, the source control system can provide the exact information that is necessary.

## Pitfalls

Obviously, not every migration is just about the schema. Frequently the data has to be altered to conform to the target schema. This is actually an area of active research in the Database Systems community [4].

## Conclusion

For the interested reader, I started work on a [preprocessor](https://github.com/johnbender/sql-delta) implemented in Haskell. Unfortunately since I don't have any plans to pursue this further I haven't been working on it. Also, for comparison I've included two very simple sets of denotational semantics in the footnotes; one to represent the current implementation and one to represent the differential semantics [5]. They highlight the symmetry of this new approach to the language when compared with the current implementation.

This technique can be extended to other languages that manage system state declaratively like configuration management DSLs or even HTML. Though in the case of configuration management, understanding the mapping between syntax and state is quite complex because system components frequently generate artifacts that are not explicitly declared.

Broadly, the idea of differential semantics is to gather more information about intent from readily available sources so that language runtimes (declarative or otherwise) can make more informed decisions about user intent. The results need not be confined to accurate interpretation of the desired system state.

### Footnotes

1. [http://en.wikipedia.org/wiki/Data_definition_language](http://en.wikipedia.org/wiki/Data_definition_language)
2. In our example a drop would be accomplished by removing the table definition completely.
3. The mapping presumes feature parity in the create with the alter statements, but in my study of the standard and Postgres' implementation this appears to be the case.
4. There's a lot of interesting work and tooling around preventing issues resulting from schema migrations: [schema evolution](http://scholar.google.com/scholar?q=prism+schema+evolution&btnG=&hl=en&as_sdt=0%2C5).
5. A denotational semantics for both the current DDL semantics and the proposed semantics. Note that in the proposed section the "differential" semantics eval function is parameterized by the state of the syntax.

<script async class="speakerdeck-embed" data-id="04706730a1090131ed084e04b85186c1" data-ratio="0.772830188679245" src="//speakerdeck.com/assets/embed.js"></script>
