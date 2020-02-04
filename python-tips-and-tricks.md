From [Awesome blog for tips and tricks](https://martinheinz.dev/)

### Sanitizing String Input

> Problem of sanitizing user input applies to almost every program you might write. Often it's enough to convert characters to lower or upper-case, sometimes you can use Regex to do the work, but for complex cases, there might be a better way:
```python
user_input = "This\nstring has\tsome whitespaces...\r\n"

character_map = {
    ord('\n') : ' ',
    ord('\t') : ' ',
    ord('\r') : None
}
user_input.translate(character_map)  # This string has some whitespaces... "
```
> In this example you can see that whitespace characters "\n" and "\t" have been replaced by single space and "\r" has been removed completely. This is a simple example, but we could take it much further and generate big remapping tables using ```unicodedata``` package and its ```combining()``` function to generate and map which we could use to remove all accents from string.

### Find Close Matches of a Word/String
> Now, for little more obscure feature of Python standard library. If you ever find yourself is situation where you need to find words similar to some input string using something like Levenshtein distance, then Python and ```difflib``` have your back.
```python
import difflib
difflib.get_close_matches('appel', ['ape', 'apple', 'peach', 'puppy'], n=2)
# returns ['apple', 'ape']
```
> ```difflib.get_close_matches``` finds the best "good enough" matches. Here, first argument is being matched against second one. We can also supply optional argument ```n``` which specifies maximum number of matches to be returned. Another available keyword argument ```cutoff``` (defaults to 0.6) can be set to change threshold for score of matched strings. 

### Prompting User for a Password at Runtime
> Lots of commandline tools or scripts require username and password to operate. So, if you happen to write such program you might find ```getpass``` module useful:
```python
import getpass

user = getpass.getuser()
password = getpass.getpass()
# Do Stuff...
```
> This very simple package allows you to prompt user for password as well as get their username, by extracting current users login name. Be aware though, that not every system supports hiding of passwords. Python will try to warn you about that, so just read warnings in command line. 

### Taking Slice of an Iterator
> If you try to take slice of an Iterator, you will get a ```TypeError```, stating that generator object is not subscriptable, but there is a easy solution to that:
```python
import itertools

s = itertools.islice(range(50), 10, 20)  # <itertools.islice object at 0x7f70fab88138>
for val in s:
    ...
```
> Using ```itertools.islice``` we can create a ```islice``` object which is an iterator that produces desired items. It's important to note though, that this consumes all generator items up until the start of slice and also all the items in our ```islice``` object.

### Naming a Slice Using ```slice``` Function
> Using lots of hardcoded index values can quickly become maintenance and readability mess. One option would be to use constants for all index values, but we can do better:
```python
#              ID    First Name     Last Name
line_record = "2        John         Smith"

ID = slice(0, 8)
FIRST_NAME = slice(9, 21)
LAST_NAME = slice(22, 27)

name = f"{line_record[FIRST_NAME].strip()} {line_record[LAST_NAME].strip()}"
# name == "John Smith"
```
> In this example we can see that we can avoid mysterious indices, by first naming them using ```slice``` function and then using them when slicing out part of string. You can also get more information about the slice object using its attributes ```.start```, ```.stop``` and ```.step```.

### Skipping Begining of Iterable
> Sometimes you have to work with files which you know that start with variable number of unwanted lines such as comments. ```itertools``` again provides easy solution to that:
```python
string_from_file = """
// Author: ...
// License: ...
//
// Date: ...

Actual content...
"""

import itertools

for line in itertools.dropwhile(lambda line: line.startswith("//"), string_from_file.split("\n")):
    print(line)
```

> This code snippet produces only lines after initial comment section. This approach can be useful in case we only want to discard items (lines in this instance) at the beginning of the iterable and don't know how many of them there are.

### Find the Most Frequently Occurring Items in a Iterable
> Finding the most common items in list is pretty common task, which you could do using ```for``` cycle and dictionary (map), but that would be a waste of time as there is ```Counter``` class in ```collections``` module:
```python
from collections import Counter

cheese = ["gouda", "brie", "feta", "cream cheese", "feta", "cheddar",
          "parmesan", "parmesan", "cheddar", "mozzarella", "cheddar", "gouda",
          "parmesan", "camembert", "emmental", "camembert", "parmesan"]

cheese_count = Counter(cheese)
print(cheese_count.most_common(3))
# Prints: [('parmesan', 4), ('cheddar', 3), ('gouda', 2)]
```
> Under the hood, ```Counter``` is just a dictionary that maps items to number of occurrences, therefore you can use it as normal ```dict```:
```python
print(cheese_count["mozzarella"])
# Prints: 1

cheese_count["mozzarella"] += 1

print(cheese_count["mozzarella"])
# Prints: 2
```
> Besides that you can also use ```update(more_words)``` method to easily add more elements to counter. Another cool feature of ```Counter``` is that you can use mathematical operations (addition and subtraction) to combine and subtract instances of ```Counter```. 

### Functions with only Keyword Arguments (kwargs)
> It can be helpful to create function that only takes keyword arguments to provide (force) more clarity when using such function:
```python
def test(*, a, b):
    pass

test("value for a", "value for b")  # TypeError: test() takes 0 positional arguments...
test(a="value", b="value 2")  # Works...
```

> As you can see this can be easily solved by placing single ```*``` argument before keyword arguments. There can obviously be positional arguments if we place them before the ```*``` argument.

### Creating Object That Supports with Statements
> We all know how to, for example open file or maybe acquire locks using with statement, but can we actually implement our own? Yes, we can implement context-manager protocol using ```__enter__``` and ```__exit__``` methods:
```python
class Connection:
    def __init__(self):
        ...

    def __enter__(self):
        # Initialize connection...

    def __exit__(self, type, value, traceback):
        # Close connection...

with Connection() as c:
    # __enter__() executes
    ...
    # conn.__exit__() executes
```

> This is the most common way to implement context management in Python, but there is easier way to do it:
```python
from contextlib import contextmanager

@contextmanager
def tag(name):
    print(f"<{name}>")
    yield
    print(f"</{name}>")

with tag("h1"):
    print("This is Title.")
```

> The snippet above implements the content management protocol using ```contextmanager``` manager decorator. The first part of the ```tag``` function (before ```yield```) is executed when entering the ```with``` block, then the block is executed and finally rest of the ```tag``` function is executed.

### Comparison Operators the Easy Way
> It can be pretty annoying to implement all the comparison operators for one class, considering there are quite a few of them - ```__lt__``` , ```__le__``` , ```__gt__``` , or ```__ge__```. But what if there was an easier way to do it? ```functools.total_ordering``` to the rescue:
```python
from functools import total_ordering

@total_ordering
class Number:
    def __init__(self, value):
        self.value = value

    def __lt__(self, other):
        return self.value < other.value

    def __eq__(self, other):
        return self.value == other.value

print(Number(20) > Number(3))
print(Number(1) < Number(5))
print(Number(15) >= Number(15))
print(Number(10) <= Number(2))
```
> How does this actually work? ```total_ordering``` decorator is used to simplify the process of implementing ordering of instances for our class. It's only needed to define ```__lt__``` and ```__eq__```, which is the minimum needed for mapping of remaining operations and that's the job of decorator - it fills the gaps for us.

### Saving Memory with __slots__
> If you ever wrote a program that was creating really big number of instances of some class, you might have noticed that your program suddenly needed a lot of memory. That is because Python uses dictionaries to represent attributes of instances of classes, which makes it fast but not very memory efficient, which is usually not a problem. However, if it becomes a problem for your program you might try using ```__slots__```:
```python
class Person:
    __slots__ = ["first_name", "last_name", "phone"]
    def __init__(self, first_name, last_name, phone):
        self.first_name = first_name
        self.last_name = last_name
        self.phone = phone
```
> What happens here is that when we define ```__slots__``` attribute, Python uses small fixed-size array for the attributes instead of dictionary, which greatly reduces memory needed for each instance. There are also some downsides to using ```__slots__``` - we can't declare any new attributes and we are restricted to using ones on ```__slots__```. Also classes with ```__slots__``` can't use multiple inheritance.

### Defining Multiple Constructors in a Class
> One feature that is very common in programming languages, but not in Python, is function overloading. Even though you can't overload normal functions, you can still (kinda) overload constructors using class methods:
```python
import datetime

class Date:
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day

    @classmethod
    def today(cls):
        t = datetime.datetime.now()
        return cls(t.year, t.month, t.day)

d = Date.today()
print(f"{d.day}/{d.month}/{d.year}")
# 14/9/2019
```
> You might be inclined to put all the logic of alternate constructors into ```__init__``` and solve it using ```*args```, ```**kwargs``` and bunch of ```if``` statements instead of using class methods. That could work, but it can become hard to read and hard to maintain. I would therefore recommend to put very little logic into ```__init__``` and perform all the operations in separate methods/constructors. This way you will get code that is clean and clear both for the maintainer and user of the class. 

### Limiting CPU and Memory Usage
> If instead of optimizing your program memory or CPU usage, you want to just straight up limit it to some hard number, then Python has a library for that too:
```python
import signal
import resource
import os

# To Limit CPU time
def time_exceeded(signo, frame):
    print("CPU exceeded...")
    raise SystemExit(1)

def set_max_runtime(seconds):
    # Install the signal handler and set a resource limit
    soft, hard = resource.getrlimit(resource.RLIMIT_CPU)
    resource.setrlimit(resource.RLIMIT_CPU, (seconds, hard))
    signal.signal(signal.SIGXCPU, time_exceeded)

# To limit memory usage
def set_max_memory(size):
    soft, hard = resource.getrlimit(resource.RLIMIT_AS)
    resource.setrlimit(resource.RLIMIT_AS, (size, hard))
```
> > Here we can see both options to set maximum CPU runtime as well as maximum memory used limit. For CPU limit we first get soft and hard limit for that specific resource (```RLIMIT_CPU```) and then set it using number of seconds specified by argument and previously retrieved hard limit. Finally, we register signal that causes system exit if CPU time is exceeded. As for the memory, we again retrieve soft and hard limit and set it using ```setrlimit``` with size argument and retrieved hard limit. 

### Debugging Program Crashes in Shell
> > If you are one of the people who refuse to use IDE and are coding in Vim or Emacs, then you probably got into a situation where having debugger like in IDE would be useful. And you know what? You have one - just run your program with ```python3.8 -i``` - the ```-i``` launches interactive shell as soon as your program terminates and from there you can explore all variables and call functions. Neat, but how about actual debugger (```pdb```)? Let's use following program (```script.py```):
```python
def func():
    return 0 / 0

func()
```
> > And run script with ```python3.8 -i script.py```
```sh
# Script crashes...
Traceback (most recent call last):
  File "script.py", line 4, in <module>
    func()
  File "script.py", line 2, in func
    return 0 / 0
ZeroDivisionError: division by zero
>>> import pdb
>>> pdb.pm()  # Post-mortem debugger
> script.py(2)func()
-> return 0 / 0
(Pdb)
```
> > We see where we crashed, now let's set a breakpoint:
```python
def func():
    breakpoint()  # import pdb; pdb.set_trace()
    return 0 / 0

func()
```
> > Now run it again:
```sh
script.py(3)func()
-> return 0 / 0
(Pdb)  # we start here
(Pdb) step
ZeroDivisionError: division by zero
> script.py(3)func()
-> return 0 / 0
(Pdb)
```
> > Most of the time print statements and tracebacks are enough for debugging, but sometimes, you need to start poking around to get sense of what's happening inside your program. In these cases you can set breakpoint(s) and when you run the program, the execution will stop on the line of breakpoint and you can examine your program, e.g. list function args, evaluate expression, list variables or just step through as shown above. ```pdb``` is fully featured python shell so you can execute literary anything, but you will need some of the debugger commands which you can find [here](https://docs.python.org/3/library/pdb.html#debugger-commands). 

### Controlling What Can Be Imported and What Not
> > Some languages have very obvious mechanism for exporting members (variables, methods, interfaces) such as Golang, where only members starting with upper-case letter are exported. In Python on the other hand, everything is exported, unless we use ```__all__```:
```python
def foo():
    pass

def bar():
    pass

__all__ = ["bar"]
```
> > Based on code snippet above, we know that only ```bar``` function will be exported. Also, we can leave ```__all__``` empty and nothing will be exported casing ```AttributeError``` when importing from this module.

### Working with IP Addresses
> > If you have to do some networking in Python you might find ```ipaddress``` module very useful. One use-case would be generating list of ip addresses from CIDR (Classless Inter-Domain Routing):
```python
import ipaddress
net = ipaddress.ip_network('74.125.227.0/29')  # Works for IPv6 too
# IPv4Network('74.125.227.0/29')

for addr in net:
    print(addr)

# 74.125.227.0
# 74.125.227.1
# 74.125.227.2
# 74.125.227.3
# ...
```
> > Another nice feature is a network membership check of IP address:
```python
ip = ipaddress.ip_address("74.125.227.3")

ip in net
# True

ip = ipaddress.ip_address("74.125.227.12")
ip in net
# False
```
> > There are plenty more interesting features that I will not go over as you can find those [here](https://docs.python.org/3/howto/ipaddress.html). Be aware though, that there is only a limited interoperability between ```ipaddress``` module and other network-related modules. For example, you can't use instances of ```IPv4Network``` as address strings - they need to be converted using ```str``` first. 
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjA1NjE5NTUwOV19
-->