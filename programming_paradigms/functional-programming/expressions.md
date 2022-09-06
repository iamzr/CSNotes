# Expressions

One easy way we can use expressions is by using operators over their statement based expressions.

Take for example this singleton class:

```cs
public sealed class MySingleton
{
    private static MySingleton _instance;

    private My Singleton
    {
    }

    public static MySingleton Instance
    {
        get {
            if (_instance == null)
            {
                _instance = new MySingleton();
            }

            return _instance;
        }
    }

    // Other code here
}
```

Now as opposed to using the if-statement in the getter, we could us an operator instead.

```cs
public sealed class MySingleton
{
    private static MySingleton _instance;

    private My Singleton
    {
    }

    public static MySingleton Instance
    {
        get {
            return _instance == null ? new MySingleton : _instance;
        }
    }

    // Other code here
}
```

Although this is better since we aren't using a statement, this is a somewhat clumbersome appraoach this there is much better operator we can use here.
The null coalesing operator.

```cs
public sealed class MySingleton
{
    private static MySingleton _instance;

    private My Singleton
    {
    }

    public static MySingleton Instance
    {
        get {
            return _instance ?? (_instance = new MySingleton());
        }
    }

    // Other code here
}

```

This version is much cleaner.
We no longer have all the imperatie noies from the if statemet nor the explict null check.
We now how everythig the original did in a single line and in a compact form.

## Expression bodied members

Expression bodied members take advantage of the fact that that expression return a value.
When a member body contains an expression like our instance expression, we can replace the member body with a lambda expression-like syntax.

```cs
public sealed class MySingleton
{
    private static MySingleton _instance;

    private My Singleton
    {
    }

    public static MySingleton Instance => _instance ?? (_instance = new MySingleton());

    // Other code here
}

```

In doing this, the compiler creates a property with only a getter body.

We can use the same technqiue to create a method body instead.

Review:
Favouring expression in a statement based langauage like C# isn't always easy.
But using operators instead of statements or wrapping common imperative patterns in higher order functions, we can quickly make our code more predictable, maintainable and testable.