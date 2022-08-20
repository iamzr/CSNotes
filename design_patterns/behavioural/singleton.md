# Design Patterns: Singleton

A singleton is a class designed to only ever have on instance.
The class itself is responsible to ensuring this requirement.

Examples:

The pattern is usually used for shared resources e.g.
- access to the file system
- access to a shared network resource e.g. printer

or for performance reasons when it's expensive to run up an instance so it's better to just have a one-time configuration.
Also it's possible to only create an instance when a request for that thing is recieved - this is also known as lazy loading.

## Structure

```mermaid
classDiagram
class Singleton
Singleton : -_instance 
Singleton : Instance()$ void
```

Single class, private instance, with a public static method that allows you to access that instance.

## Features
Applications that use the singleton class have at anytime, only 0 or 1 instance of the class.

Singleton classes are created without parameters. If you need similar instances depending on some parameters check out the [factory pattern](factory.md)

Lazy instantiation as the default. This means that due to performance reasons singleton instances are not usually created unless something requests them.
Another appoarch is to create an instance at startup and to use that same instance for the life of the application.

In the code
Singleton classes should have a single, private, parameterless constructor.
Because of this subclassing isn't allowed.
To ensure this singleton classes should be marked as *sealed*

The only reference to the singleton should be in a private static field in the singleton class itself.

The rest of the application accesses the singleton via a public static method the classes exposes for this purpose.

## Examples

### A naive implementation

```cs
public sealed class Singleton
{
    private static Singleton _instance;

    public static Singleton Instance
    {
        get
        {
            if(_instance == null)
            {
                _instance = new Singleton()
            }

            return _instance
        }
    }
}
```

Here we can see that if we were to call `Instance()` it would return a non-null instance of the `Singleton` class.

Furthermore, if we were to call `Instance` three times it would only call the constructor once.
This is because the first time `Instance()` was called a constrcutor was created, which means  `_instance` would no longer be null.
Then for the subsequent calls it would not hit the constructor.

However, if we were to call `Instance()` three times in parallel i.e. in three seperate threads. What we find is that there will be three seperate calls to the constructor.
This demonstrates why this is a naive implementation - it is not thread safe.
We would prefer to only call the constructor once, even in a multi-threaded environment.
As we just shown, in a multi-threaded environment, the `if` block above can be reached by multiple thread concurrently, thus resulting in multiple instantiates of `Singleton`.
The impact this would have depends on the application, it could be a minor performace issue or could introduce substantial problems in the application.

### Making it thread-safe

```cs
public sealed class Singleton
{
    private static Singleton _instance = null;
    private static readonly object padlock = new object()

    public static Singleton Instance
    {
        get
        {
            lock (padlock) // this lock is used on every refence to Singleton

            if(_instance == null)
            {
                _instance = new Singleton()
            }

            return _instance
        }
    }
}
```

To get around the lack of thread-safety, we can introduce a `lock`.
We create an instance of an `object` to be used as a lock.
We then check the `lock` object in the getter, this forces all the threads to align. And only allows one thread at a time ensuring you dont get two threads creating an instance at the same time.


This solves the issue of thread-safety, but now every time we call instance we have the overhead of the `lock`, which will negatively affect performance.

Note: for locks always use a private instance variable not a shared, public, static value that could get used elsewhere in the application.
Locks should be one-to-one with their lock statements.

```cs
public sealed class Singleton
{
    private static Singleton _instance = null;
    private static readonly object padlock = new object()

    public static Singleton Instance
    {
        get
        {
            if(_instance == null) // Only get lock if the instance is null
            {
              lock (padlock)
              {
                if (_instance == null)
                {
                _instance = new Singleton()
                }
              }                
            }

            return _instance
        }
    }
}
```

Here we have implemented double-check locking, you can see that we check to see if `_instance` is null twice and inbetween check the `lock`.
This makes sure we only check the `lock` when `_instance` is null, which would only be when the application starts up or when the first request is made to this instance.

You get the same behavior as before, but you get improve performance since you aren't checking your lock as often.

When it comes to multiple threads now, if the `_instance` is null, and there are multiple concurrently calls, the `lock` will be called and so only one thread will be able to enter the block at a time.
Once the first thread has gone through, `_instance` will no longer be null and so will no longer enter the `if` block and simply return `_instance`.

The above approach is good, but has some issues with the ECMA CLI spec which might be a concern. If you're interested about these concerns click [here](www.csharpindepth.com/articles/singleton).

### Leveraging Static Constructors
We can improve performance and keep thread-safety by utilizing static constructors.
In C#, static constructors run only once per application and are called when any static member of a type is referenced.
Make sure you use an explicit static constructor to avoid issues with the C# compiler and beforefieldinit (this is a hint the compiler uses to let static initalizers can be called sooner, this is the default if the typew doesn't have a static constuctor.) Ading

```cs
public sealed class Singleton
{
    private static readonly Singleton _instance = new Singleton();

    // Reading this will initialize the _instance
    public static readonly string GREETING = "Hello!";

    // Tell C# compiler not to mark-type as beforefieldinit
    static Singleton()
    {
    }

    public static Singleton Instance
    {
        get
        {
            return _instance;
        }
    }
}
```

The modifications we've made here simple.
We have added a static inializer when `_instance` is decleared.

We have added a `static Singleton` constructor.
However, with this design is we read from any other static field or member of this class, we will also inialize an instance of the `Singleton`.
So we were to read `GREETiNG` it will result in the instantiation of our singleton.

We can further improve upon this design as follows

```cs
public sealed class Singleton
{
    // Reading this will initialize the _instance
    public static readonly string GREETING = "Hello!";

    // Tell C# compiler not to mark-type as beforefieldinit
    static Singleton()
    {
    }

    public static Singleton Instance
    {
        get
        {
            return Nested._instance;
        }
    }

    public class Nested
    {
        // Tell C# compiler not to mark type as beforefieldinit
        static Nested()
        {

        }

        internal static readonly Singleton _instance = new Singleton();
    }
}
```

Here we have implemented a nested class, which has a `readonly Singleton` instance.
This type will only initialize the first time when it's called.
Note there are no other static fields on the `Nested` class and this is how we get around the issue of accidentlly loading it too soon.

We can't mark the `_instance` in `Nested` as `private`, since we want to use it in `Instance()` so the best we can do is mark it as internal.


This solution is better than the last since it's thread-safe, it doesn't use locks which means better performance. But in the case of the nested example it is complex and non-intuitive - it's probably not the first thing you'll jump to when you want to implement a singletons.

### Using `Lazy<T>`

Another way we can create a singleton us to use the `Lazy<T>` type
It provides built-in support for lazy initalization.
To create a `Lazy<T>` type you:
- specify the Type you're using
- specify a means of create the Type

Lets see how this is implemented.

```cs
public sealed class Singleton
{
    // reading this will initialize the instance
    private static readonly Lazy<Singleton> _lazy = new Lazy<Singleton>(() => new Singleton());

    private Singleton()
    {
    }

    public static Singleton Instance
    {
        get
        {
            return _lazy.Value;
        }
    }
}
```

We now have a field `_lazy` of type `Lazy<Singleton>`, this will create a new `Lazy<Singleton>`  at construction. And a lambda function is passed into the `Lazy<T>` constructor contaning the logic required to make a `Singleton` instance.

Only other point to note, we now return `_lazy.Value` in the `Instance` method. This is guarenteed to never be null, as this is a property of `Lazy<T>` and there will only be one instance of it since its coming from a static field.

This is probably the best approach because it has all the thread-safety features we want, as well as being simple and intuitive to understand.

## What problems does the singleton pattern solve?
Some people consider the singleton to be an antipattern for the following reasons:
- Direct static use in code (as opposed passing an interface they implement), leads to tight coupling and difficult to test code.
- Doesn't follow seperation of concerns
- Doesn't follow single responsibility principle - since it's managing it's instance life-time as well as whatever it's actual work is
- If you're going to implement more than one singleton you have to duplicate all the logic to get that behavior which violates the DRY princple
- There are alternative approachs that get the same result as the singleton.

## How is the singleton pattern structured?

## How do you apply the singleton pattern?

## Alternatives and related patterns

One alternative is to use static classes.
In C#, you can declare a class as static and the compiler ensures that the class defintion only contains static members.

Let's compare the differences between the singleton and a static class since they're quite similar and many people get them confused.

| Singleton | Static Class |
|----|---|
| Can implement interfaces (since it is an actual class) | No interfaces (since it's just a collection of static method and functions)|
| Can be passed as an arguement to method. Therefore, can be used in the strategy design pattern and dependency injection | Cannot be passed as arguements to methods|
| Can be assigned to variables | Cannot be assigned |
| Support polymorphism | Purely procedual |
| Can have a state | CAn only access global state|
| Can be serialized | Cannot be serialized |


<!-- TODO: Include part on IoC containers and whatnot -->


## Summary
- A singleton class is deigned to only ever have one instance created
- The singleton pattern makes the class itself responsible for enforcing Singleton behaviour
- It's easy to get the pattern wrong when implementing by hand
- `Lazy<T>` is one of the better ways of applying the pattern
- Singletons are different from static classes
- If your framework uses IOC/DI containers like in .NET applications, the they are usually a better place to manage instance lifetime