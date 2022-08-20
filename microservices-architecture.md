# Principles of a Microservices Architecture

- Independent Deployment
    - Each microservice should be *independently deployable*
    - The system can be depolyed as a whole but it doesn't have to be
    - You should be able to add and remove microservices from the system as it is running
    - If a microservice has any dependencies, they should be deployed with that microservice
- Modularity
    - Each microservice is a *unique* piece of code, running it's *seperate process*
    - Each microservice is a part of the "solution" which serves the business goals
    - It is responsible for providing a single "functionality" to the system
- Communication
    - When required, microservices should communicate with the "outside world" e.g. other microservices, using well-defined, *standard* lighwight mechanism e.g. HTTP/REST with Json
- Size
 
