#notes

# Basic flask api
# File system
Using mvc pattern

Everything in one file

Testing of basic functions



# Production level API

# Key points
using latest version of sql alchemy, not using old query stuff
testing along the way
application factory

## Structure
Taking inspiration from MVC file pattern.

```
root
| run.py
| configs
    | development_config.py
    | test_config.py
    | production_config.py
| instance
    | config.py
| api
    | __init.py__
    | controllers
    | models
    | services
| tests
    | unit_tests
    | integration_tests
```

## api/__init__.py
## Running the api for dev purposes using run.py
## Building out configs
### Configs
### Instance configs
### testing

## Models with SQLAlchemy
### Testing

## Controllers
setting up with blueprint 
### Testing

## Services
Buisness layer?
## Contains all the business logic and does the actual calculations

## Authentication
Adding users to seperate db and accessing certain restricted endpoints with auth0 or whatever

## Logging

## functional Testing?
Testing whether whole end points work