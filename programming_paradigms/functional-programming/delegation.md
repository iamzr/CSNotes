# Delegation

 In .Net functions are passed around using something called Delegation.

Delegation is what truely enables us to functional programming in .Net

## What is delegation

Delegates allow us to name funtion signatures
They are the mechanism that allows us to treat functions as types.

e.g. take the build in event handler delegate

Observer the functional notation of the signature for the for the event handler delegation.
```cs 
EvenHandler: (object, EventArgs) -> void 
```

What this means is an event handler is any object that takes in an object along with a `EventsArgs` object and returns `void`.

Here we see the actual delegatoin definition in C#
```cs
public delegate void EventHandler(
    object sender,
    EventArgs e,
)
```

You might notice this is very similar to how we would define an abstract class except here we use the `delegate` keyword.
You can clearly see 
- the return type
- indentifier
- parameter list

.Net is a OO language so cares very little about function signatures outside of the overload resolution.
So how does delegeation bridge the gap between OOP and functional?

Delegations are actually a compiler trick!
Rathter than passing around funcitons the compiler creates a new type that derives from the built-in `MulticastDelegate` class.

MulticastDelegate: an abstract class which represents one or more methods to be invoked

This generated class includes an invoke method which has the same signature as the method represented by the delegate.

Each method associated with the delegate instance is added to an invokation list.
The `invoke` method then interates over this, which, as the name suggests, invokes each of method in the order it was added to the delegate.

In other words,
the compiler is wrapping the method calls into the generated class and abstracting that detail away from us.

MulticastDelegate is special because only the compiler can derive from it.
This ensures proper behavior and so the delegate keyword must be used to create a new delegate type.

<!-- TOD: Add in the history of delegation here if you have time -->

Delegation has envolved over time. We're going to look at the diferent ways we cna use delegations


here is an example of a delegate
```cs
// Basic converter from GBP to various currencies

public static Func<decimal, decimal> GetConverter(string currency)
{
    switch (currency)
    {
        case "USD": return (x) => x * 1.15

        case "BTC": return x => x * 0.000058

        // More cases
    }
}

private static decimal Eval(decimal amount, string currency)
{
    var x = Decimal.Parse(elements[0]);

    return GetConverter(currency)(amount)
}
```

So now I could make use of this in the following way

```cs
Eval(123.34, "USD")
Eval(4325, "BTC")
```

To explan what's happening in this example.
We first define the delegate with using the `Func` notation.
We provide a list of types, where the last type is the return type.
So if it read
```cs
Func<string, decimal, bool, bool, int> 
```
You would know the return type is an `int`.

We then have the function name that takes in a string arguement.

Next we have a switch statement that in each case returns not a `decimal` but a lambda expression.
We would read the lambda operator `=>` as goes to. 
So we know that the input variables are the parameters for the expression on the right hand side.

We dont have to explicitly say what the types of the inputs are, here the input would simply be `x`, this is because the compiler already knows this from the delegate definition where we said we'd pass in a `decimal`.

We then have the `Eval` method that we actually use to evaluate 