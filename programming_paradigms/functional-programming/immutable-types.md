# Immutable types

The easiest way to tame side effects in C# is to force immutatibility with custom types.

When are types are immutable we train ourselves to stop introducing side effects in places where they may intially bw convenitent but may cause problems down the line.
Languages like F# enforce immutability by default; but we have to do a bit of work to make this happen in C#


### Example

Take for example the following code:

```cs
public class PriceRange
{
    public Price StartPirce { get; set; }
    public Price endPrice { get; set; }

    public bool PriceInRange(Price price)
    {
        return startPrice.CompareTo(price) <= 0 && endPrice.CompareTo(price) >=0
    }
}
```

One why to force immutatbilty is to convert the `startPrice` and `endPrice` property setters to private and add a constructor to set those values:

```cs
public class PriceRange
{
    public Price StartPirce { get; private set; }
    public Price endPrice { get; private set; }

    public PriceRange(Price start, Price end)
    {
        startPrice = start;
        endPrice = end;
    }

    public bool PriceInRange(Price price)
    {
        return startPrice.CompareTo(price) <= 0 && endPrice.CompareTo(price) >=0
    }
}
```

Althouhg private setters are a step in the right direction.
There is still a problem. Having the private setters means that changes from outside that class cannot occur but this doesnt prevent changes from inside the class from occuring.

To make the class truely immutable, we need to replace the auto-implemented properties 
<!-- TODO: what are auto-implemented properties -->
with explict properites backed by read only fields.


```cs
public class PriceRange
{
    // backing fields 
    private readonly Price _startPrice;
    private readonly Price _endPrice;

    public Price startPirce { get { return _startPrice; } }
    public Price endPrice { get { return _endPrice; } }

    public PriceRange(Price start, Price end)
    {
        _startPrice = start;
        _endPrice = end;
    }

    public bool PriceInRange(Price price)
    {
        return startPrice.CompareTo(price) <= 0 && endPrice.CompareTo(price) >=0
    }
}
```

The `readonly` operator n the backing fields means we can set the fields value only as part of the declaration or in the constructor.
Trying to set the value anywhere else will result in a compile error.

We have not successful got an immutatle class.

Now it's pretty tedious having to implement these backing fields ourselves.
C# 6 came out with a feature called Getter-only auto-properties that define the backing fields are read only for us.

We can see this in action below:

```cs
public class PriceRange
{
    public Price startPrice { get; } 
    public Price endPrice { get; } 
    
    public PriceRange(Price start, Price end)
    {
        _startPrice = start;
        _endPrice = end;
    }

    public bool PriceInRange(Price price)
    {
        return startPrice.CompareTo(price) <= 0 && endPrice.CompareTo(price) >=0
    }
}
```

Defining types as immutable may seem forign at first, but they're usually worth the effort.

You'll typically find they're much simplier to because you're not having to manage state changes and their behavior is easier to reason about due to the fact they dont change.

Immutability is especially important for expression because expression should be evaluted for the result and not an effect whenever possible.

### Favour expressions

Statemetns are used to cause some effect, such as updating a database or writing a log.

Expressions  are executed for their result.
Althought C# is a statement-based language it does have rich support for expressions.
This is typically observable as the difference between void functions and functions that return a value.

To effectively write code in a functional style, we must look to favou expressions where possible.


Review:
Enforcing immuntability is good because it actively guides us awa from writing code that gives us side effects.
It also, forces us too think about the seperation between data and behaviour.
