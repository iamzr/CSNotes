# Specification Design Pattern C#

##

DDD includes many patterns and practices, the specification pattern is one of them.
The specification pattern helps gain domain knowledge in a certain place and reuse it for various things.

## Indroduction

## What is the specification pattern?

It helps avoid domai knowledge duplication - adhering to the [DRY principle](../../dry.md)

Allows for a declarative approach.

## Basic design
When you have a piece of domain knowledge you can encapsulate it in a single unit, the specification, and then reuse it throughout your code.

## Main use cases
In-memory validation.
    When you need to check some object you have at hand fits some criteria.
    THis is useful when validating input from users or third party systems
Retriving data from the database
Construction to order
    When creating objects that have certain criteria
    You can't care about the content of the objects just that they have certain attributes

They might not look like they have much in common, and so quite often domain knowledge is actually duplicated for each of these cases.
This reduces code quality and techincal debt.

Most common use cases are the first two.

## Example 
Here's an example to explain why we need this pattern.



## Implementing the specificatio pattern

# Refactor towards better encapsulation