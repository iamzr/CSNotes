# Liskov Substitution Principle

Original Defintion:
> Let \phi(x) be a property provable about objects x of type T.
> Then \phi(y) should be true for objects y of type S where S is a subtype of T.

Alternative definition:
> Subtypes must be substitutable for their base types

Although this may seem intuitive, there are some languages that allow a subtype to modify its supertype behaviour, which is not what we want. This principle is a warning about avoiding this pitfall.

## Basics of OO Design
There's two types of relationships we need to consider here.
- Inheritance - when something IS-A something else
  - e.g. a square is a rectangle
- Property - when something HAS-A something else property
  - e.g. a house has a price property

The LSP states that
> The IS-A relationship is insufficient and should be replaced with IS-SUBSTITUTABLE-FOR

### Example - Rectangle-Square Problem
- A rectangle has hour sides and four right angles
- A square has four equal sides and four right angles
- Per geometry, a square *is a* rectangle

Suppose we have
```cs
public class Rectangle
{
    public virtual int Height {
        get; set;
    }

    public virtual int Width { get; set;}
}
```

and the following class to calculate area

```cs
public class AreaCalculator
{
    public static int CalculateArea(Rectangle r)
    {
        return r.Height * r.Width;
    }
}
```
Now, if we want to implement a square class we can do

```cs
public class Square : Rectangle
{
    private int _height;

    public overridge int Height
    {
        get
        {
            return _height;
        }

        set
        {
            _width = value;
            _height = value;
        }
    }

    // Width similarly implemented
}
```

Since a square is a rectange, `Square` inherits from `Rectange`.
And since, we require the length and width the same, in the setter we set the width and height equal to each other.

#### The problem
The problem is when we have code that is expecting a rectangle but actually gets a square.
Take for example

```cs
Rectangle myRect = new Square { Width: 4, Height: 5};

Assert.Equal(20, AreaCalculator.CalculateArea(myRect))
// False
// The actual result is 25.
```
We can clearly see this is the wrong result, since we should have got 20 instead.

#### What happened?
We know that
- Square has an invariant, it's sides must be equal
  
However, we also know that
- Rectange has an invariant, its sides can change independent of each other

This design breaks rectangle's invariant.
Hence, here we can't substitute a subtype for it's base type. This this violate LSP.

### Possible solutions
We could get rid of square and simply add a flag to rectangle as such:

```cs
public class Recatangle
{
    public int Height { get; set; }
    public int Width {
        get; set;}
    
    public bool IsSquare => Height == Width
}
```

Another option would be to just have a seperate class for squares altogether.
Thus avoiding the LSP violation.
It also provides a clearer representation of a square since it doesn't require a height and width properties

```cs
public class Rectangle
{
    public int Height { get; set; }
    public int Width {
        get; set;}
}

public class Square
{
    public int Side { get; set; }
}
```

## Detecting LSP violations IRL:
### 1. type checking with `is` or `as` in polymorphic code (which means that code should work with any type or subtype)

#### Example
```cs
foreach (var user in users)
{
    if (user is Admin)
    {
        Helper.PrintAdmin(user as Admin);
        break;
    }
    Helpers.PrintUser(user);
}
```
This is a problem because now every time you work some users you'll have to check if it's an `Admin` or not.
If you add more subtypes for example `RestrictedUser` then you'll have to change all the if statements to check for that type too. This obviously violates the OCP because you'd have to modify the code when you wanted to extend it.

#### Solution
To solve this we need to make sure that each of our subtypes is substitutable for the base type.

One solution, would be to have each subtype implement it's own custom functionality for `Print()` so then we'd just have

```cs
foreach (var user in users)
{
    user.Print()
}
```

Another option would be to adjust the `Helper` method to work with any user.

```cs
foreach (var user in users)
{
    Helpers.PrintUser(user)
}
```

Although this might look like moving the problem, doing this means that the `foreach` loop no longer violates LSP.
It would also mean we could have all the time checks in one place instead of everywhere the `Helper` method is used.

### 2. null checks
Another way to detect potential violations are null checks, which are essentially the same as checking the type.

```cs
foreach (var user in users)
{
    if (user == null)
    {
        throw new NullReferenceException()
        break;
    }
    Helpers.PrintUser(user)
}
```
This is a LSP violation because in C# nulls, in general, are not substitutable for actual instances.

One way of avoiding checking for nulls is using the [null object pattern](../../design_patterns/behavioural/null.md).

### 3. NotImplementedException
Indication that only some features of an interface or a base class were implemented. This type cannot be substituted for the interface or base class and hence violated LSP

For example, say you have an interface for a logging service

```cs 
public interface ILoggingService
{
    void SendWarning(string message);

    void SendInfo(string message);
}
```

An actual implementation of this might be a warning service for users.

```cs
public class WarningService : ILoggingService
{
    public void SendWarning(string message)
    {
        // Actually send the user the warning message
    }

    public void SendInfo(string message)
    {
        throw new NotImplementedException();
    }
}
```

This `NotImplementedException()` tells us that the `SendInfo()` method has not been implemented.
As a result it that `WarningService` is not substitutable for `ILoggingService` anywhere it might be used.
Because if somewhere used the `SendInfo()` method, they'll get an error.

## LSP is a subset of polymorphism

Polymorpishm means an IS-A relationship.
And the LSP implies an IS-SUBSTITUTABLE relationship. 
This means that classes obeying LSP also obey polymorphism.
And that if there's code that violates LSP it also violates polymorphism.

More importantly, anywhere you have polymorphism, you almost definitely will want things to be substitutable.


## Fixing LSP Violations

1. Follow the "Tell, Don't Ask" principle
   Instead of asking instances for their type and then conditionally performing different actions.
   Instead encapsulate that logic in the type itself, tell it to perform an action.
   Packaging state and behaviour together is a basic princple of OO design. 

2. YOu can address null referenes by using
   - C# features such as:
     - Nullable reference types
     - Null conditional operators
     - Null coalescing operators
   - Guard clauses to use exceptions to prevent null values from reaching the primary logic of your methods
   - [Null object design pattern](../../design_patterns/behavioural/null.md)

3. Whenever implementing an inferface be sure to fully implement it and follow [ISP](isp.md)

## Tell, Don't Ask

Recall we had the following:
```cs
foreach (var user in users)
{
    if (user is Admin) // ask
    {
        Helper.PrintAdmin(user as Admin); // response
        break;
    }
    Helpers.PrintUser(user); // reponse
}
```

Here have the data i.e the users seperate from the logic i.e. the way in which we print the user.

Instead if we follow the "Tell, Don't Ask" principle, we'd have

```cs
foreach (var user in users)
{
    user.Print() // tell
}
```

where instead each user would implement it's own version of print. This way we can just iterate over the users and not have to worry about different implementations for different types of users.
This gives us a more modular and cohesive design.

## Key takeaways
- Need to ensure that subtypes are substituable for their base types
- Ensure base type invariants are enforce for subtypes
- To find LSP violations look for:
  - type checking
  - null checking
  - `NotImplementedException`