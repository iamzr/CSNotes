## The Interface Segregation Principle

The principle:
> An object should only depend on interfaces it requires and should not be enforced to implement any method (or property) it does't require.

Take the following example:
We have the following interface `INumberOperations` which defines operations on complex numbers.

```csharp
interface INumberOperations
{
    Number GetComplexConjugate()
    Number Addition()
}
```

<!-- TODO: Could maybe improve this example -->
We can see that `INumberOperations` has the `GetComplexConjugate()` method. However, if we think about it, calculating the complex conjugate only applies to complex numbers, not real numbers. So why should this be in the interface for all numbers? Why don't we instead have a general interface and a seperate one for complex numbers e.g. `IComplexNumberOperations`.

```csharp
internal interface IComplexNumberOperations
{
    Number GetComplexConjugate();
}

interface INumberOperations: IComplexNumberOperations
{
    Number Addition();
}
```

<!-- TODO: Not sure about this line -->
Breaking down this interface into two allowa us to improve code readability, testability and maintainability 

-----

# Interface segregation principle

The principle:
> Clients should not be forced to depend on methods they do not use

Most developers have a tendency to think about interfaces in terms of their concrete classes that implement those interfaces.
But this isn't actually their purpose, the reason for the interfaces is actually to introduce loose coupling.



Corollary:
Prefer small cohesive interfaces to large, "fat" ones

## What does interface mean?
This applies to any OOP language not just C#.
Some C# examples:
- C# `interface` type/keyboard
- Also refereces to the accessible interface of a `class`
  - A types interace in this context is whatever can be acceessed by client code working with an instance of that type.

## What is a Client?
In this context, the client is the code that is interacting with an instance of the interface - it's calling the code

Not to be confused with client/server achritecture

If you have code that uses an instances of a type
Then your code is the client in this scenario.
Whatever methods you can call on that instance is its interface. 

The ISP states that your code should not depend on methods in that instance that it does not use.

## Problem with large interfaces

Say you have a large interface that has say 50 methods.

Say you want to implement your own version of that interface. 
You would have to implement all of those 50 methods.
Not doing so would mean you're breaking [LSP](lsp.md).
But say you only needs say one or two methods out of the 50?
Then what, should you have to depend on this massive interface?

Violating the ISP results in classes depending on this they dont need.;
This increases coupling and makes it harder to change or swap out indivisual implementsation since the number of clients that might break is much larger.
This is because you'll have clients that depend on the interface but aren't using all parts of it.
Which might be the parts you're changing.

<!-- Need to come up with a good example -->


Large interfaces means more dependencies.
This means:
- more coupling
- more brittle code due to coupling
- Client code that is more difficualt to test
  - SInce fake implementation requires more work
  - SInce the code using larger interfaces tends to be more coupled
- More difficult deployment
  - CHange sto types that implement fat interfaces require more testing of downstream dependencies.
  - Thus making more difficult and risky

## Detecting ISP violations
- Larger interfaces
  - It lacks cohesion 
  - Follow pain-drive development
- If your implementations of an interface include `NotImplementedException` then it means the interface was bigger than your clients needed.
- Code uses a small subset of a larger interface

## Example
Consider the following interface

```cs
public interface IMarketingService
{
    void SendEmail(string email, string subject, string body);

    void SendText(int areaCode, int number, string message)
}
```
This interface lacks cohesion, this is shown in the following example.
Say you wanted to implement this interface for a email marketing
```cs
public class EmailMarketingService : IMarketingService
{
    public void SendEmail(string email, string subject, string body)
    {
        // code to send and email
    }

    public void SendText (int areaCode, int number, string body)
    {
        throw new NotImplementedException();
    }
}
```

Here we can see that the `EmailMarketingService` doesn't fully implement the interface and we have some `NotImplementedException()`.
This violates the ISP.

To solve this instead what we can do is split the interface into two.

```cs
public interface IEmailMarketingService
{
    void SendEmail(string email, string subject, string body);
}

public interface ITextMarketingService
{
    void SendText(int areaCode, int number, string message)
}
```

As you can see this solution is a lot more cohesive.

Although this is just a small example the same thing can be done for much larger interfaces.
Group together methods within a larger interface and then seperate them out into their own interfaces.

## What about legacyu code that's coupled to the orginal interface?
If you have legacy code tha depends on a large interface and you dont want to break your code, there's an easy solution.

C# lets you use those smaller interfaces.
Say you have the `IMarketingService` interface and you seperated it out into `IEmailMarketing` and `ITextMarketing` as we did above.
But you then realise you have code legacy code that still depends on `IMarketingService`, you can simply do the following

```cs
public interface IMarketingService : IEmailMarketingService, ITextMarketingService
{

}
```
What are have done here is the `IMarketingService` now inherits from the two interfaces we made, which means it'll contain all the methods from both of them.
So when that legacy code uses the `IMarketingService` it'll still have the same sigature it had before - i.e. it'll still have all those methods even though the body of `IMarketingService` is empty.

Note: this only works under the assumption you own the large interface, this wouldn't work if you're using an interface from a third-party framework or SDK.
In that case you're better off using the [adapter design patter](../../design_patterns/structural/adapter.md)

## Related Concepts
ISP related to the [LSP](lsp.md).
Larger interfaces are harder to implement and so more likely to only be partially implemented, therefore to not be substitutable for their base type - i.e. violating LSP

ISP heavily relies upon cohesion which relates to the [SRP](srp.md)
Small cohesive interfaces are preferable to large interfaces who's methods aren't all closely related to each other.

When addressing problems with ISP, remember to follow PDD.
YOu shouldn't break up interfaces just to satify the principle.
Instead use the principle to identify the source of some pain you have with the code base.

If you try and pre-emtively use ISP you'll end up with lots of small interfaces that will be hard to group together and understand.
When you would have been better off using a few cohesive interfaces.

## Fixing ISP violation

Approaches

1. Break up large interfaces into smaller ones
Compose fat interfaces from smaller ones for backward compatibility.

2. For larger interfaces you dont control use the [adapter design pattern](../../design_patterns/structural/adapter.md)
Your code should work with the adapter interface that you control and then it in turn will work the the underlying large interface.
In this scenario, only the adapter should know about the large interface.

3. It's easy to follow ISP if you allow the client code to define the interfaces tha code will work with. 
It's impossible to violate the interface segregation principle, which says clients shouldnt be depending on methods they don't use, if the clients are the ones defining the interface to include exactly the methods they're going to use.


## Summary
- Prefer small cohesive interfaces to large expansive ones
- Following ISP helps with SRP and LSP
- Break up large interfaces bu using
  - Interface inheritance
  - The adapter design pattern


  -----

They're thinking about it the wrong way around, it's not the concrete classes that need the interfaces, but it's te clients that need the interface.
The client owns the interface and uses it to tell you what it needs.
There's no reason for a clietn to define a member of an interface if the client doesn't actually need it.

You should favor role interfaces over header interfaces.
What's a header interface?
It's the ones you see that are extracted from a concrete class.
They look like old-fashioned header-files like from c++ code
<!-- TODO: Add a link -->

A role interface?
It only defines a few member based on given role.
An extreme role interface is one that has a single member.
DOing this can be quite helpful in instances when you're trying to resolve violates of LSP.
