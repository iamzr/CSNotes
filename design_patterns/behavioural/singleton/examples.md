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