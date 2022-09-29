# How LINQ works?

Two major interfaces when using LINQ `IEnuumerable` and `IQueryable`
They have two seperate sets of extension method, which despite looking exactly the same work completely differently.

Take for example the where method.

For an `IEnumerable` object we'd see the following
```cs
new [] { 1, 2, 3, 4, 5 }.Where(x => x == 5)
```
whereas with an IQueryable, we'd do the following
```cs
session.Query<Ticket>()
    .Where(x => x.StartTime <= 2000>)
```

However, if we now compare the signatures of the two wear methods
IEnumerable
```cs
public static IEnumerable<TSource> Where<TSource>(
    this IEnumerable<TSource> source,
    Func<TSource, bool> predicate);
```

IQueryable
```cs
public static IQuerable<TSource> Where<TSource>(
    this IEnumerable<TSource> source,
    Expression<TSource, bool> predicate);
```

Differences to know.
The most obvious between the two is that they work on different interfaces.
However, the key point to note is the predicate for the `IQueryable` is not a [delegate](programming_paradigms/functional-programming/delegation.md).
Its's an expression, this is possible due to a C# language feature known as lambda expressions.
They can be compiled to a delegate or an expression depending on the circumstances.

So, if we were to write out the statements using delegates and expressions.
The IEnumerable one would be
```cs
Func<int, bool> func = x => x ==5;
```
The IQueryable one would be
```cs
Expression<Func<int, bool>> expression = x => x == 1;
```

How does the compiler choose which version of `Where` to use?
Well for an array it is simple enough, it would just use the `IEnumerable` one since an array inherits from the `IEnumerable` interface.

However for `session.Query<Ticket>` it's a bit more complicated because it implements both `IEnumerable` and `IQueryable`.
So how does the comiler magically know to use the `IQueryable` version of `Where` as opposed to the `IEnumerable` version?

The trick here is that `IQueryable` inherits from `IEnumerable`
i.e.
```cs
IQueryable : IEnumerable
```
and extension methods in C# are implemented in such a way that when there are multiple suitable overloads it'll pick the one that works with the most specific type.
In our example, `IQueryable` is the more specific interface because it inherits from `IEnumerable`, thus the compile will choose the `IQueryable` version of `Where`

This is usually how link providers work in other [ORM](dictionary.md#ORM) s too.
SO in the case of EFcore you could instead have wrote something like
```cs
dbContext.Tickets.Where(...)
```
and everything were talking about here would still stand.

To summarise the `Where` methods work with different types, one a delegate and the other an expression.
This allows for a unified approach for querying data from memery and db. 

In the case of databases LINQ providers parse the expression and build up a corresponding SQL query.
Expressions allow them to do so because each expression represents a tree of statemtns which can be examined at runtime.
The LINQ provider just runs through this tree and converts it to SQL.
The only issue with this is that only a small subset of operations can actually be translated to SQL.

There is a link between expressions and delegates in C#

Expressions can be converted to delegates but delegates can't be converted back to expressions. 

```mermaid
flowchart LR
expression --> delegate
delegate -- x --> expression
```