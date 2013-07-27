---
layout: post
title: System State and Declarative Languages
tags:
- sql
- devops
status: published
type: post
published: true
listed: false
---

Declartive programming languages are often characterized by how they express computation in terms of a desired outcome. This is in contrast to imperative languages which are seen, vis-a-vis declarative languages, as expressing computation with series of steps to reach an outcome. The advantage of the declarative approach is clear, the program author states his or her intentions and the rest is left to the runtime [1]. In practice many languages that have earned the declarative label require much of the "how" to be included out of necessity.

In this post we'll look at two examples where the declarative approach has proven valuable: the data definition subset of SQL (DDL) and configuration management (Puppet?). In particular, we'll examine where each falls down as the desired system state evolves over time and how that might be remedied.

## SQL

SQL is the language most often cited when an example of a declarative language is called for. DDL provides the tools to establish a database schema by creating tables, keys, constraints, indexes, and more. For example creating a table `foo`:

```sql
CREATE TABLE foo (
    bar integer,
    baz text
);
```

Though you could argue that removing the `CREATE` would make it more "declarative", this accurately captures the desired state of the database. Namely that it should contain a table identified by `foo` with two columns, one and integer and the other text [2].

Unfortunately database schemas rarely remain this simple, but this is an issue that SQL's declarative nature goes a long way toward mitigating. More importantly, schemas must be "evolved" towards that complexity over time as requirements change. New instances are created, old ones are updated, and the schema must work in both cases.

```sql
CREATE TABLE foo (
    bar integer,
    baz text
);

-- ... many months and ddl statements later ...

ALTER TABLE foo ADD COLUMN bak text;

-- ... many months and ddl statements later ...

ALTER TABLE foo DROP COLUMN bak text;
```

Since it's unreasonable to drop the table and recreate it each time there's a change to its structure, table alterations are made. Unfortunately this diverges from the declarative approach significantly. The maintainers of the schema are no longer defining the desired table structure and trusting that the runtime will make the necessary changes. They're defining the necessary changes manually so the runtime can construct the desired schema. Possibly worse, making an alteration to a complex set of DDL statements requires comprehension of all alterations that came "before" leaving a lot of room for error.

So the question is, how do you preserve the declarative nature of DDL SQL statements while maintaining the idempotent properties of a schema definition?

## Configuration Management

Configuration managment has seen an explosion in popularity as applications are more frequently being hosted on virtualized hardware. The disposability of virtual servers has driven a need for tools that can quickly and consistently provision a new machine to serve a specific role. Encouragingly, many of the tools along with their accompanying EDSLs and custom languages have taken a declartive approach to system state.

TODO find an example resource that can have substatements removed / or needs extra statements for alteration


```ruby
package "apache2"
```

```puppet
package { "apache2": ensure => "installed" }
```

(Chef's | Puppet's) `package` statement is a declaration in the same sense that `CREATE TABLE` is, it defines a package that should be installed on the system and (Chef | Puppet) handles the rest. Though it's clearly less painful in this instance, it must address the same problem that the SQL does. When the maintainer wishes to remove that package from the system he or she must account for new systems being built from scratch and old systems that have already been built. As a result additional syntax is necessary to clarify the intent.

```ruby
package "apache2" do
  action :remove
end
```

```puppet
package { "apache2": ensure => "absent" }
```

The example here seems particularly odd because removing package requires declaring its absence from the system. Someone new might wonder why anyone should be troubled with declaring all the things that should *not* be present on a system!

Additionally, for both SQL and these configuration management tools, it's unclear how these statements should be handled over time in a code base. Clearly once all existing systems have been updated it's safe to remove them, but neither runtime provides facilities for that type of accounting leaving the user open to mistakes.

## Semantics of Change

semantics of change in syntax

### footnotes

1. "It seems like a sufficiently advanced runtime...", famous last words from an anonymous PhD dropout.
2. It's worth noting that most (all?) implementations will fail when confronted with a `CREATE TABLE` statement for an existing table. The example can easily be augmented with `IF NOT EXISTS` to get the desired effect.
