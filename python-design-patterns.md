From [Python Design Patterns](https://www.toptal.com/python/python-design-patterns).

_Factory_ is a structural Python design pattern aimed at creating new objects, hiding the instantiation logic from the user. But creation of objects in Python is dynamic by design, so additions like Factory are not necessary.

## Design patterns
### Program to an interface not an implementation
We don’t bother with the nature of the object, we don’t have to care what the object is; we just want to know if it’s able to do what we need (we are only interested in the interface of the object).

```py
try:
    bird.quack()
except AttributeError:
    self.lol()
```

Did we define an interface for our bird? No! Did we program to the interface instead of the implementation? Yes!

### Favor object composition over inheritance
Instead of doing this:

```py
class User(DbObject):
    pass

```

We can do something like this:

```py
class User:
    _persist_methods = ['get', 'save', 'delete']

    def __init__(self, persister):
        self._persister = persister

    def __getattr__(self, attribute):
        if attribute in self._persist_methods:
            return getattr(self._persister, attribute)

```

The advantages are obvious. We can restrict what methods of the wrapped class to expose. We can inject the persister instance in runtime! For example, today it’s a relational database, but tomorrow it could be whatever.
## Behavioral patterns
Behavioural Patterns involve communication between objects, how objects interact and fulfil a given task. According to GOF principles, there are a total of 11 behavioral patterns in Python: _Chain of responsibility, Command, Interpreter, Iterator, Mediator, Memento, Observer, State, Strategy, Template, Visitor._
### Iterator
### Chain of Responsibility
### Command
## Creational Patterns
### Singleton
### Dependency Injection
## Structural Patterns
### Facade
### Adapter
### Decorator
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTg4OTYzNTU0NiwzODcwOTg1MzcsLTQxMz
g5MTYyN119
-->