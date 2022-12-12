# Functional programming in C#

> OO makes code understandable by encapsulating moving parts
> FP makes code understandlbe by minimizing moving parts
> - Micheal Feathers

defintion
> FP is a paradigm which ocncentrates on computing results rather than on performing actions

Three central themes:
1. Tame side effects
A side effect is a consequential and usually detrimental effect as a result of some action

In programming, a side effect is anything that happens to the state of a system as a result of invoking a fucntion

Why are side effects bad?
When a function has side effects we have not not only consider it's main result but also how it affects the rest of the system.
Side effects also make functions more difficult to test since their very nature implies dependencies on other parts of the system.

Functional Purity 

Language purity is the amount of control a language exercises over our ability to induce side effects.

We classify languages as either:
Purely functional e.g. Haskell
- Dont allow functions to have side effects, except in limited cases
- Referential transparaency - ability to replace a functin with it's result in the code without having a noticeable effect on the overall system.
- Pure functions are more predictable because they always behave in the same way regardless of the external state.
- Since pure functoins dont changein the overall state of the system  the code if more suited for parallel or asynchronous execution.

C# is an impure language because it doesn't actively prevent functions from changing system state.
The primary way we can get the behaviour we want is by forcing immutability in our types.
C# doesn't do this by default.

2. Expression based. 

This means that everything produces some result.
This is very different from a statement-based language like C# where many constructs do not produce a direct result but instead executed as a side effect.

The difference between statements and expressions

Statement:
- Define actions
- Executed for their side effect
```cs
string evenOrOdd;

if (value % 2) // this is the if-statement here
{
    evenOrOdd = "even";
}
else
{
    evenOrOdd = "odd";
}
```

Expressions:
- Produce results
- Executed for their results

```cs 
var evenOrOdd = value % 2 == 0 ? "even" : "odd";
```

Here we have used the ternary/conditonal opertator "?" 

Note a few things.
First, the expression version of the code above is much shorter.
This is common for functional programming since expressions are used to comute resuls as opposed to statements that cause side effects.

Second, we dont have an unassigned variabe `evenOrOdd` in the expression-based version of the code.
Nor do we have multiple places where `evenOrOdd` is assigned,
i.e. it's assigned in multiple places in the statement-based example in lines 5 and 9.

Expressions are more natually testable than their statement counterparts.
E.g. if we know a functions result based on it's input we can easily mock it without mocking frameworks or deriving additional types solely to test it's behaviour.

Expression are naturally composable

Since expressions give back a value they can easilt be combined with other statements and expressions by using them in place of a varible or value.

### Example - composibility

Statement-based

Here we have the same statement as before.
```cs
string evenOrOdd;

if (value % 2 == 0)
{
    evenOrOdd = "even";
}
else
{
    evenOrOdd = "odd";
}

var msg = $"{value} is {evenOrOdd}"
```

Not how we have this variable `evenOrOdd` that we probably won't use outside of this context.

Expression-based
```cs
var msg = $"{value} is {(value % 2 == 0) ? "even" : "ddd"}
```

See how much shorter this version is.
Note how we made use of string interpolation and used an expression to simply fill in the hole when needed.
This also gets rid of the need for a superfulous variable `evenOrOdd`.

3. Treat functions as Data

This is argueably the most important theme in functional programming.
The ability to treat functions just like any other data type.
When a language does this we say that functions are first class citizens of that language.

C# makes functions first class citizens through [delegation](../statement-expressions-etc.md).

Treating functions as data is important because it allows for higher-order functions

> higher-order functions: functions that accept other functions and/or return other functions

Higher-order functions allow us to think at different levels of abstract just like we would with an OO mindset.

Higher-order functions give us the ability to compose functionality by substituting behavior according to a strict defintion, a functions signature- as opposed to a complex inheritance structure you might see in OOP.

# LINQ

Over the years C# has started implemented a number of functional features.
One prominant one is LINQ.

LINQ is a combination of:
- Extension methods
- Generics
- Delegation/lambda expressions
- anonymous types
- expression trees

Focusing on LINQ to objects, it focuses on the following three

Generics - allow us to safely operate against stronly typed sequences while still providing full visibility into the sequence.
Extension methods - are higher order functions which add functionality such as sorting, filtering or transforming vaues of any type that embrace `IEnumerable<T>`
Delegation/lambda expressions - let us define how extension methods operate against the sequence.

## Example

Lets look at an example for filtering and sorting.

#### Imperative approach

<!-- TODO: what does imperiative mean -->

```cs 
var x = 0;

while (x < arry.Count)
{
    if (arry[x] < 0)
    {
        arry.RemoveAt(x)
    }

    else
    {
        ++x;
    }
}

arry.Sort();
```

THis code gets the job done but it;s not functional.

The reasons why we dont want to write code like this:

1. the while loop
   This indicates there's some mutable state.
   In oredr for the loop to run the condition must be false to run and some side effet much occur inside the loop that changes that condition for the loop to terminate

   Inspecting the condition we see that either `x` must change or `arry.Count` 

   Looking at the code we see both change either an item is removed from `arry` or `x` is incremented.

   Changing `x` isn't too much of a concern since it's probably only used in this small code block.
   However, changing `arry` is more of a concern since it could be used outside of the method where this code is located.
   This could potentially lead to bigger problems in the application. For example, imagine if another thread tried to apply a similar method to the same array!
   So ideally we'd want to avoid this all together.

   We have another side-effecting call wit the `Sort()` method.
   How do we know it's a side-effecting call? It's a void method. If it didn't do anything executing it would be pointless.
   Because it doesn't do anything, we must be invoking it solely for it's side effect.
   The `Sort()` method here sorts the list in place, thereby mutating the list.

   Also, lookingn throughout this code we can see that this is entirely staemented-based.
   If we look a the following example that uses expressions instead we will see that they more clearly show what were doing with the data.

   The overarching problem with an imperative approach like this is that it hides patterns that functional languages have used for decades. 
   But unfortanately for OOP lanauges many of those patterns require higher-order functions to use effectively.
   AS a result, we get similar code many times in our code without realising its the same problem over and over.

   FOrtunately, LINQ addresses all of these concerns.

### Expression-based (LINQ way)

The followin achieves the exact same solution as the code above.

```cs
from x in arry
where x > 0
orderby
select x;
```

How much cleaner is that?

Now people might be put off from using LINQ in this way due to the syntactic sugar covering up what's really going on here.

More importantly many LINQ methods aren't accessible using the contextual keywords.
And those that are often have powerful overloads only available to the method synatx, so lets see that that looks like.

Although the above seem like magic, it's simply generics, extensions methods and delegation working together in functional harmony.

```cs
arry
    .Where(x => x > 0)
    .OrderBy(x => x)
```

Here we use the method synatax, thus getting rid of the syntactic sugar.
This declarative approach has abstracted away much of the plumbing code we saw in the example earlier.
AS a result, we can focus on solving the problem as opposed to parsing code that's only really there to satisfy the compiler.
In other words, we dont know or need to know how the `Where` and `OrderBy` methods iterate over the source sequences.
We just tell the higher order functions how to handle each value of the sequence and wait for the return values.

Using the method syntax it;s clear what's going on. The LINQ methods each operate against an instasnce of `IEnumerable<T>`

What might not be obvious is `arry` here is never actually changed!
Rather than mutating the original list,
the LINQ methods create new sequences where each item in the source sequences is mapped to the resulting sequence according to the lambda expression.
As such we can safely use `arry` elsewhere if necessary.

Futhermore, if the LINQ methods caused side effects it would violate the [command-query segreation architectural pattern] thus invalidating the name LINQ or "language integrated query". 
Not mutating the original list tells use the method chain results in a composed expression,
because each extension method returns an expression that then gets used by the next method in the chain.
Given that each method produces a new sequence, we are free to continue chaining the methods until we get our desired result.

It's true that LINQ inherent immutatbility does result in more allocations that the new sequences are constructed but makes operating against the same source list from multiple threads much less scary.