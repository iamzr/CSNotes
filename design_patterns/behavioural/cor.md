## Chain of Responsibility Design Pattern

## Characteristics of the Chain of Responsibility Pattern

- Sender - invokes something in the handler
- Handler - Runs through a chain of recievers
- Reciever - Handles the given command

e.g.
sender would be a call to Logger.Log()
Handler Logger has a chain of ILoggers 
Reciever - console logger, file logger, database logger

THe handler makes sure that each reciever gets the request passed to it.
The reciever decides whether it should process that request or not.
Even though the request might not be processed by that particular reciever,
the request will still be passed along to the next receiver in line.

A key feature is that the sender doesn't need to know about the concrete implementation.
This means we can extend our chain of loggers to handle how we want to log our messages.

The patern allows us to decouple our applications making them more maintainable and extensible.


## When to use it
One of the instances where the chain of responsibility pattern can be used is to address the following code smell: when you have a lot of if-else statements.
This pattern allows you to change the if-elseif-else idiom follow a more object oriented approach.

To refactor this if else statement, we put each one in its own handler. 
It's responible for validating that condition and then calling the next handler.
If one of the handler reaches a point where it fails then we either return a value or throw an exception.
