## The Dependency Inversion Principle

The principle:
> 1. Higher-level modules should not depend on low-level modules. Both should depend on abstractions.
> 2. Abstractions should not depend on details. Details should depend on abstractions.

Take the following example:
```csharp
internal class Cog
{
    int CogSize;
}

internal class Engine
{
    Cog cog;
}

class Train
{
    Engine engine;
}
```

This is an example that doesn't follow the principle. This is because the high-level class `Train` is tightly-coupled to `Engine`, and `Engine` is tightly-coupled to `Cog`. You can say that this a tightly coupled system, where the lower-level components are being used directly by the component above it - which violates point 1 above.

To explain this further, say we used the property `CogSize` in `Engine`. If we changed `Cog` so that it would have the properties `CogRadius` and `TeethSize` and removed `CogSize`. This would mean we'd have to refactor `Engine` to account for this. Hence showing how tightly-coupled it is.

We can get around this using interfaces between components instead of using the objects themselves.

```csharp
internal interface ICog
{
    int CogSize;
}

internal class Cog: ICog
{

}

internal interface IEngine
{

}

internal class Engine: IEngine
{
    ICog cog;
}

class Train
{
    IEngine engine
}
```
<!-- TODO: This whole explanation needs improving. Consider talking about abstractions to explain the interfaces part. -->
By introducing the interfaces there is no longer a dependency between the components i.e. they've been de-coupled.
We can see this because, we know if we make changes to `Cog` it wouldn't affact `Engine` because `Engine` knows that `cog` will always be of the type `ICog`. So now there isn't a dependency between the entities e.g. `Engine` and `Cog` instead it's between an entity `Engine` and an abstraction `ICog`.