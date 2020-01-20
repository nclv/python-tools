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

The command pattern is handy in situations when, for some reason, we need to start by preparing what will be executed and then to execute it when needed. The advantage is that encapsulating actions in such a way enables Python developers to add additional functionalities related to the executed actions, such as undo/redo, or keeping a history of actions and the like.

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
The Singleton pattern is used when we want to guarantee that only one instance of a given class exists during runtime. Do we really need this pattern in Python? It’s easier to simply create one instance intentionally and then use it instead of implementing the Singleton pattern.

But should you want to implement it, here is some good news: In Python, we can alter the instantiation process (along with virtually anything else).

```py
class Logger(object):
    def __new__(cls, *args, **kwargs):
        if not hasattr(cls, '_logger'):
            cls._logger = super(Logger, cls
                    ).__new__(cls, *args, **kwargs)
        return cls._logger
```

In this example, _Logger_ is a Singleton.

These are the alternatives to using a Singleton in Python:

-   Use a module.
-   Create one instance somewhere at the top-level of your application, perhaps in the config file.
-   Pass the instance to every object that needs it. That’s a dependency injection and it’s a powerful and easily mastered mechanism.

### Dependency Injection

It deals with the question of when (or even better: where) the object is created. It’s created outside. Better to say that the objects are not created at all where we use them, so the dependency is not created where it is consumed. The consumer code receives the externally created object and uses it.

_Don’t get things to drink from the fridge yourself, state a need instead. Tell your parents that you need something to drink with lunch._

Let’s think about a simple example of dependency injection:

```py
class Command:

    def __init__(self, authenticate=None, authorize=None):
        self.authenticate = authenticate or self._not_authenticated
        self.authorize = authorize or self._not_autorized

    def execute(self, user, action):
        self.authenticate(user)
        self.authorize(user, action)
        return action()

if in_sudo_mode:
    command = Command(always_authenticated, always_authorized)
else:
    command = Command(config.authenticate, config.authorize)
command.execute(current_user, delete_user_action)

```

We inject the _authenticator_ and _authorizer_ methods in the Command class. All the Command class needs is to execute them successfully without bothering with the implementation details. This way, we may use the Command class with whatever authentication and authorization mechanisms we decide to use in runtime.

We have shown how to inject dependencies through the constructor, but we can easily inject them by setting directly the object properties, unlocking even more potential:

```py
command = Command()

if in_sudo_mode:
    command.authenticate = always_authenticated
    command.authorize = always_authorized
else:
    command.authenticate = config.authenticate
    command.authorize = config.authorize
command.execute(current_user, delete_user_action)

```
The dependency injection technique allows for very flexible and easy unit-testing. Imagine an architecture where you can change data storing on-the-fly. Mocking a database becomes a trivial task.
## Structural Patterns
### Facade
Imagine you have a system with a considerable number of objects. Every object is offering a rich set of API methods. You can do a lot of things with this system, but how about simplifying the interface? Why not add an interface object exposing a well thought-out subset of all API methods? A _Facade!_

Python Facade design pattern example:

```py
class Car(object):

    def __init__(self):
        self._tyres = [Tyre('front_left'),
                             Tyre('front_right'),
                             Tyre('rear_left'),
                             Tyre('rear_right'), ]
        self._tank = Tank(70)

    def tyres_pressure(self):
        return [tyre.pressure for tyre in self._tyres]

    def fuel_level(self):
        return self._tank.level
```

The `Car` class is a _Facade_, and that’s all.

### Adapter
If _Facades_ are used for interface simplification, _Adapters_ are all about altering the interface.

Let’s say you have a working method for logging information to a given destination. Your method expects the destination to have a `write()` method (as every file object has, for example).

```py
def log(message, destination):
    destination.write('[{}] - {}'.format(datetime.now(), message))

```

I would say it is a well written method with dependency injection, which allows for great extensibility. Say you want to log to some UDP socket instead to a file,you know how to open this UDP socket but the only problem is that the `socket` object has no `write()` method. You need an _**Adapter**_!

```py
import socket

class SocketWriter(object):

    def __init__(self, ip, port):
        self._socket = socket.socket(socket.AF_INET,
                                     socket.SOCK_DGRAM)
        self._ip = ip
        self._port = port

    def write(self, message):
        self._socket.send(message, (self._ip, self._port))

def log(message, destination):
    destination.write('[{}] - {}'.format(datetime.now(), message))

upd_logger = SocketWriter('1.2.3.4', '9999')
log('Something happened', udp_destination)

```

But why do I find _adapter_ so important? Well, when it’s effectively combined with dependency injection, it gives us huge flexibility. Why alter our well-tested code to support new interfaces when we can just implement an adapter that will translate the new interface to the well known one?

You should also check out and master _bridge_ and _proxy_ design patterns, due to their similarity to _adapter_.

### Decorator
The _decorator_ pattern is about introducing additional functionality and in particular, doing it without using inheritance.

```py
def execute(user, action):
    self.authenticate(user)
    self.authorize(user, action)
    return action()

```

What is not so good here is that the `execute` function does much more than executing something. We are not following the single responsibility principle to the letter.

It would be good to simply write just following:

```py
def execute(action):
    return action()

```

We can implement any authorization and authentication functionality in another place, in a _decorator_, like so:

```py
def execute(action, *args, **kwargs):
    return action()

def autheticated_only(method):
    def decorated(*args, **kwargs):
        if check_authenticated(kwargs['user']):
            return method(*args, **kwargs)
        else:
            raise UnauthenticatedError
    return decorated

def authorized_only(method):
    def decorated(*args, **kwargs):
        if check_authorized(kwargs['user'], kwargs['action']):
            return method(*args, **kwargs)
        else:
            raise UnauthorizeddError
    return decorated

execute = authenticated_only(execute)
execute = authorized_only(execute)

```

Now the `execute()` method is:

-   Simple to read
-   Does only one thing (at least when looking at the code)
-   Is decorated with authentication
-   Is decorated with authorization

We write the same using Python’s integrated decorator syntax:

```py
def autheticated_only(method):
    def decorated(*args, **kwargs):
        if check_authenticated(kwargs['user']):
            return method(*args, **kwargs )
        else:
            raise UnauthenticatedError
    return decorated


def authorized_only(method):
    def decorated(*args, **kwargs):
        if check_authorized(kwargs['user'], kwargs['action']):
            return method(*args, **kwargs)
        else:
            raise UnauthorizedError
    return decorated


@authorized_only
@authenticated_only
def execute(action, *args, **kwargs):
    return action()

```

It is important to note that you are _not limited to functions_ as decorators. A decorator may involve entire classes. The only requirement is that they must be _callables_. But we have no problem with that; we just need to define the `__call__(self)` method.
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE0NjYwNjAyOCwzODcwOTg1MzcsLTQxMz
g5MTYyN119
-->