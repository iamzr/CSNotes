## The Interface Segregation Principle

The principle:
> An object should only depend on interfaces ir requires and should not be enforced to implement any method (or property) it does't require.

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

