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

Declarative programming languages are often characterized by how they express computation in terms of a desired outcome. This is in contrast to imperative languages which are seen, vis-a-vis declarative languages, as expressing computation with a series of steps to reach an outcome. The advantage of the declarative approach is clear, the program author states his or her intentions and the rest is left to the runtime [1]. In practice many languages that have earned the declarative label require much of the "how" to be included out of necessity.

In this post we'll look at two examples where the declarative approach has proven valuable: the data definition subset of SQL (DDL) and configuration management (Puppet?). In particular, we'll examine where each falls down as the desired system state evolves over time and how that might be remedied.

## SQL

SQL is the language most often cited when an example of a declarative language is called for. The DDL subset provides the tools to establish a database schema by creating tables, keys, constraints, indexes, and more. For example creating a table `foo`:

```sql
CREATE TABLE foo (
    bar integer,
    baz text
);
```

Though you could argue that removing the `CREATE` would make it more "declarative", this accurately captures the desired state of the database. Namely, it should contain a table identified by `foo` with two columns, one and integer and the other text [2].

Unfortunately database schemas rarely remain this simple, an issue largely addressed by SQL's declarative nature. More importantly, schemas must be "evolved" towards that complexity over time to support new instances as they are created and old ones as they are updated.

```sql
CREATE TABLE foo (
    bar integer,
    baz text
);

-- ... many months and DDL statements later ...

ALTER TABLE foo ADD COLUMN bak text;

-- ... many months and DDL statements later ...

ALTER TABLE foo DROP COLUMN bak text;
```

Since it's unreasonable to drop the table and recreate it each time there's a change to its structure, table alterations are made. Unfortunately this diverges from the declarative approach significantly. The maintainers of the schema are no longer defining the desired table structure and trusting that the runtime will make the necessary changes. They're defining the necessary changes manually so the runtime can construct the desired schema. Possibly worse, making an alteration to a complex set of DDL statements requires comprehension of all alterations that came "before" leaving a lot of room for error.

So the question is, how do you preserve the declarative nature of DDL SQL statements while maintaining the idempotent properties of a schema definition?

## Configuration Management

Configuration management has seen an explosion in popularity as applications are more frequently being hosted on virtualized hardware. The disposability of virtual servers has driven a need for tools that can quickly and consistently provision a new machine to serve a specific role. Encouragingly, many of the tools along with their accompanying EDSLs and custom languages have taken a declarative approach to system state.

TODO find an example resource that can have sub-statements removed / or needs extra statements for alteration


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

Another issue highlighted here is that removing something from a system requires declaring its absence. An outside observer might wonder why anyone should be troubled with declaring all the things that should *not* be present on a system!

Additionally, for both SQL and these configuration management tools, it's unclear how these statements should be handled over time in a code base. Clearly once all existing systems have been updated it's safe to remove them, but neither runtime provides facilities for that type of accounting leaving the user open to mistakes.

## Explicit Change

In these situations information about changes to declarations should be tracked by the runtime and reflected in system state. That is, the language and its accompanying runtime would benefit from a semantic definition of syntactic change by using it in conjunction with source history to determine system state.

```sql
CREATE TABLE foo (
    bar integer,
    baz text
);

-- ...

ALTER TABLE foo ADD COLUMN bak text;
```

The intent is to add a column to the table `foo`. Ideally the `CREATE TABLE` definition would simply expand to include the new column and databases built with the old defintion would update to match the new one.

```diff
@@ -1,4 +1,5 @@
 CREATE TABLE foo (
   bar integer,
-  baz text
+  baz text,
+  bak text
 );
```

In this example the difference between the two declarations could be reconciled by simply comparing it with the existing table. The need for access to source history arises from more sweeping changes to declarations like outright removal.

```diff
@@ -1,5 +1 @@
-CREATE TABLE foo (
-  bar integer,
-  baz text,
-  bak text
-);
```

The absence of this table declaration could be handled by the runtime in a number of ways without access to source history. If the system requires that DDL statements only exist in a single canonical source file the runtime can confidently reconcile the absence of the table from the schema defintion and remove the table. Alternatively, the runtime could be fed all the DDL statements it needs to construct the schema properly and then reconcile that with its internal state thereby avoiding the "single source" requirement of the first option.

The issue with both of these solutions is that the table removal is implicit. For example, in the second case if a source file isn't included in the list to be fed to the runtime a database table might be dropped inadvertantly. In contrast the diff constitutes an explicit declaration of the table's *absence* from the schema defition.

Possibly the largest advantage of the change based approach is the prospect of rolling back schema changes using source control. In the previous example dropping a table may be difficult to reconcile [3], but things like indexes and constraints can be added and removed safetly by simply reverting the change using soure control.

TODO sql example

## The Semantics of Change

Language semantics are traditionally defined in terms of evaluation. Here the idea is that the addition and removal of syntax carries meaning in addition to the syntax itself.

```diff
@@ -1,4 +1,5 @@
 CREATE TABLE foo (
   bar integer,
-  baz text
+  baz text,
+  bak text
 );
```
```sql
ALTER TABLE foo ADD COLUMN bak text;
```

The diff is semantically equivalent to the SQL statement when considered in the light of semantic changes and this achieves two very important goals. It promotes the declarative nature of SQL DDL and suggests that if `CREATE TABLE` declarations can support that same features as the `ALTER TABLE` declarations it might be possible to remove the latter all together.

```diff
@@ -1,3 +1 @@
-package { "apache2":
-  ensure => "installed"
-}
```
```puppet
package { "apache2":
  ensure => "absent"
}
```

Again the removal of the package declaration signals the intent to remove the package itself from target systems. A side benefit is that the `ensure => "absen"` and `action :remove` are suddenly unnecessary.

## Considerations

For both SQL and configuration management there is still the question of what should be done about artifacts associated with certain declarations. In SQL the effects of a `CREATE TRIGGER` on the data are not easily unwound without a comprehensive history of the data over time, and that makes the semantics of its absence hard to define. For configuration management even the simple installation of a package might affect system startup, create temporary files, and otherwise alter system state in ways that can't be undone by simply removing it. This is to say nothing of more complex declarative abstractions for setting up web applications, databases, message queues, etc.

In spite of these outstanding questions there are many types of declarations that have straightforward meanings when changed, and they do little to diminish the value in incorporating those meanings into declarative runtimes.

## Conclusion

Declarative languages offer a lot of power to users but it is often extremely difficult to remain purely declarative in the face of change. By incorporating the semantics of change into these environments the burden on the user can be reduced while maintaining expresive power and safety guarantees.

### footnotes

1. "It seems like a sufficiently advanced compiler...", famous last words from an anonymous PhD who dropped out shortly thereafter.
2. It's worth noting that most (all?) implementations will fail when confronted with a `CREATE TABLE` statement for an existing table. The example can easily be augmented with `IF NOT EXISTS` to get the desired effect.
3. It might be achieved with access to backups.
