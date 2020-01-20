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
Behavioral Patterns involve communication between objects, how objects interact and fulfill a given task. According to GOF principles, there are a total of 11 behavioral patterns in Python: _Chain of responsibility, Command, Interpreter, Iterator, Mediator, Memento, Observer, State, Strategy, Template, Visitor._
### Iterators
### Chain of Responsibility
This pattern gives us a way to treat a request using different methods, each one addressing a specific part of the request. You know, one of the best principles for good code is the _Single Responsibility_ principle:

_Every piece of code must do one, and only one, thing._

This principle is deeply integrated in this design pattern.

For example, if we want to filter some content we can implement different filters, each one doing one precise and clearly defined type of filtering. These filters could be used to filter offensive words, ads, unsuitable video content, and so on.

```py
class ContentFilter(object):
    def __init__(self, filters=None):
        self._filters = list()
        if filters is not None:
            self._filters += filters

    def filter(self, content):
        for filter in self._filters:
            content = filter(content)
        return content

filter = ContentFilter([
                offensive_filter,
                ads_filter,
                porno_video_filter])
filtered_content = filter.filter(content)
```
### Command
This is one of the first Python design patterns I implemented as a programmer. That reminds me: _Patterns are not invented, they are discovered_. They exist, we just need to find and put them to use. I discovered this one for an amazing project we implemented many years ago: a special purpose WYSIWYM XML editor. After using this pattern intensively in the code, I read more about it on some sites.

The command pattern is handy in situations when, for some reason, we need to start by preparing what will be executed and then to execute it when needed. The advantage is that encapsulating actions in such a way enables [Python developers](https://www.toptal.com/python) to add additional functionalities related to the executed actions, such as undo/redo, or keeping a history of actions and the like.

Let’s see what a simple and frequently used example looks like:

```py
class RenameFileCommand(object):
    def __init__(self, from_name, to_name):
        self._from = from_name
        self._to = to_name

    def execute(self):
        os.rename(self._from, self._to)

    def undo(self):
        os.rename(self._to, self._from)

class History(object):
    def __init__(self):
        self._commands = list()

    def execute(self, command):
        self._commands.append(command)
        command.execute()

    def undo(self):
        self._commands.pop().undo()

history = History()
history.execute(RenameFileCommand('docs/cv.doc', 'docs/cv-en.doc'))
history.execute(RenameFileCommand('docs/cv1.doc', 'docs/cv-bg.doc'))
history.undo()
history.undo()
```
## Creational Patterns
### Singleton
### Dependency Injection
## Structural Patterns
### Facade
### Adapter
### Decorator
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwNDY0MTkwOTMsMzg3MDk4NTM3LC00MT
M4OTE2MjddfQ==
-->