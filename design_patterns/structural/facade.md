# C# Design Patterns: Facade

## Problems to solve

Consider the "big ball of mud" class, made famous by Martin Fowler.

```mermaid
classDiagram
class ClassA
ClassA : Method1()
ClassA : Method2()
ClassA : Method3()
ClassA : Method4()
ClassA : Method5()
ClassA : Method6()
ClassA : Method7()
ClassA : Method8()
ClassA : Method9()
ClassA : Method10()
```

Say you have a program that needs to call four methods from `ClassA`:
- `ClassA.Method2`
- `ClassA.Method4`
- `ClassA.Method1`
- `ClassA.Method7`

Now, to actually determine which we want we are likely going to have to examine and read through the internals of class A to figure out which methods we actually want.

This is where a facade comes in.
We can have a facade class that has a refence to `ClassA` and only exposes the methods needed by the program.

```mermaid
classDiagram 
direction LR
class ClassA
ClassA : Method1()
ClassA : Method2()
ClassA : Method3()
ClassA : Method4()
ClassA : Method5()
ClassA : Method6()
ClassA : Method7()
ClassA : Method8()
ClassA : Method9()
ClassA : Method10()
ClassA <|-- FacadeClass

class FacadeClass
FacadeClass : Method1
FacadeClass : Method2
FacadeClass : Method4
FacadeClass : Method7
FacadeClass <|-- Program

class Program
Program : FacadeClass.Method2()
Program : FacadeClass.Method4()
Program : FacadeClass.Method1()
Program : FacadeClass.Method7()
```

This means we couild use moe meaningful method names if we wanted, and our program now talks to the facade without knowing about the big `ClassA`.

## Second scenario


## The Facade Pattern
## Using Facade in practice