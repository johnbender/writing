---
layout: post
title: "A Better SQL"
tags:
- sql
- math
status: draft
type: post
published: true
listed: false
vote: false
---

The relational model for data is ubiquitous. Consequently so is SQL which is the primary way in which users leverage the relational model, but SQL has its warts. In particular schema maintenance with the DDL subset of the language can be awkward when idempotent migrations are called for. This task is so cumbersome that frequently it's delegated to the application layer. In this, the first of two posts proposing improvements to SQL, I'll lay out an alternate semantics for SQL DDL that embraces schema change through syntactic change and confines DDL to its declarative core.

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

Additionally the user might extend the `create` with an `if exists` which will ensure the proper end state whether the target database is at the initial state without the table `foo` or at the second state with the `bak`less table `foo`. That would be idempotent, and here it's easy to understand the state of the table because the examples is very simple. As the schema definition grows more complex through many `drop`s, `add`s and type casts the final state of the table is less clear:

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

The key insight here is that we can permit schema migrations while retaining an entirely descriptive declarative syntax by appealing to information that's derived from the syntax but not contained in they syntax itself. Namely we can appeal to the differential information available in any sane programming environment via source control tools.

In our toy example the desired table definition included a new column `bak`. In our idealized form of SQL that would look like this:

```sql
create table foo (
  bar int,
  baz text,
  bak text
);
```

The SQL runtime only considers the existing syntax and makes no attempt to reconcile that syntax with it's existing schema. This makes perfect sense in light of the fact that a user is permitted to run small snippets in addition to full schema migration scripts. That is, the RDBMS can't know where this declaration is coming from nor why it's being run. On the other hand the well outfitted user can have a strong understanding of both of those facts, and thus the alteration statements.

But! If the runtime is provided with more information about how a given declaration has changed it can use that in lieu of the alteration statements to reconcile the current schema state.

```diff
@@ -1,4 +1,5 @@
 create table foo (
   bar int,
-  baz text
+  baz text,
+  bak text
 );
```

Just looking at the diff it's very clear what the intention is, to add the column `bak` to the table. All that's left is to assign some semantics to this diff and its parent table statement. A simple pre-processor might map that change to the corresponding alter statement in the output DDL.

##

# # Conclusion

This technique can be extended to other languages that manage system state declaratively like Puppet's DSL or even HTML. Though in the case of Puppet understanding the state mapping is quite complex because system components frequently generate artifacts that are not explicitly declared.
