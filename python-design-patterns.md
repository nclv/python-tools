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
## Behavioral patterns
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
eyJoaXN0b3J5IjpbMzg3MDk4NTM3LC00MTM4OTE2MjddfQ==
-->