# Lucid ORM

<code>Lucid ORM</code> is Lightpack's highly performant and sleek object relational mapper
to help you with some of the most common tasks when dealing with relational databases.

## Tradeoffs

When it comes to data access patterns, there is a lot of debate about **ORMs** in any programming 
community. Some even consider it an <code>anti-pattern</code>. Some hate it because it may introduce 
possible <code>leaky-abstraction</code> and <code>cost performance</code>. Some even think that an 
ORM makes <b>simple things easy and difficult things dead difficult</b>.

And then there is a whole new concept of thinking about your business objects called <b>Domain Driven Design</b> that promoted thinking in terms of domain models.

In that sense, **Lightpack** has no opinion about whether you should use or reject the
concept of ORM at all. That being said, **Lighpack** does come with a dead simple abstraction 
which you might like though for its simplicity and performance.

## Performance

**Lucid ORM** is fast because of two major aspects in its design:

* It has a very small layer of abstraction.
* It doesn't use inflectors at run-time to make intelligent guesses about table names and primary keys. 

## Models

**Lucid ORM** provides support for **models/entities** classes that represent a single record in your database table. It also provides few more capabilities that often come handy when dealing with relational databases.

* Before/After hooks
* Timestamps support
* Query builder access

Learn more about models [here](/models).

## Relationships

**Lucid ORM** is a simplified `Active Record` pattern implementation that also supports relationships for the most common use-case scenarios.

Following relationships are supported:

* 1:1 (One-to-many)
* 1:N (One-to-many)
* N:N (Many-to-many)

Learn more about relationships [here](/relationships).

## Eager Loading

**Lucid ORM** supports eager loading for associated models to avoid the `N+1` query performance issue. 

Learn more about relationships [here](/eager-loading).