### Caching/Memoization with ```lru_cache```

```python
import functools
import time

# caching up to 12 different results
@functools.lru_cache(maxsize=12)
def slow_func(x):
    time.sleep(2)  # Simulate long computation
    return x

slow_func(1)  # ... waiting for 2 sec before getting result
slow_func(1)  # already cached - result returned instantaneously!

slow_func(3)  # ... waiting for 2 sec before getting result
```
Anoather example :
```python
from functools import lru_cache
import requests

@lru_cache(maxsize=32)
def get_with_cache(url):
    try:
        r = requests.get(url)
        return r.text
    except:
        return "Not Found"


for url in ["https://google.com/",
            "https://martinheinz.dev/",
            "https://reddit.com/",
            "https://google.com/",
            "https://dev.to/martinheinz",
            "https://google.com/"]:
    get_with_cache(url)

print(get_with_cache.cache_info())
# CacheInfo(hits=2, misses=4, maxsize=32, currsize=4)
```
> > In this example we are doing GET requests that are being cached (up to 32 cached results). You can also see that we can inspect cache info of our function using ```cache_info``` method. The decorator also provides a ```clear_cache``` method for invalidating cached results. I want to also point out, that this should not be used with a functions that have side-effects or ones that create mutable objects with each call. 

### Use Local Variables

> > local variable in function (fastest), class-level attribute (e.g. ```self.name``` - slower) and global for example imported function like time.time (slowest).

So putting your code into a function is faster than putting your whole code into one file.
```python
#  Example #1
class FastClass:

    def do_stuff(self):
        temp = self.value  # this speeds up lookup in loop
        for i in range(10000):
            ...  # Do something with `temp` here

#  Example #2
import random

def fast_function():
    r = random.random
    for i in range(10000):
        print(r())  # calling `r()` here, is faster than global random.random()
```

### Don't Access Attributes

> > Another thing that might slow down your programs is dot operator (.) which is used when accessing object attributes. This operator triggers dictionary lookup using ```__getattribute__```, which creates extra overhead in your code.

```python
#  Slow:
import re

def slow_func():
    for i in range(10000):
        re.findall(regex, line)  # Slow!

#  Fast:
from re import findall

def fast_func():
    for i in range(10000):
        findall(regex, line)  # Faster!
```

### Use Generators

> >  Generators are not inherently faster as they were made to allow for lazy computation, which saves memory rather than time. However, the saved memory can be cause for your program to actually run faster. How? Well, if you have large dataset and you don't use generators (iterators), then the data might overflow CPUs L1 cache, which will slow down lookup of values in memory significantly.
When it comes to performance, it's very import that CPU can save all the data it's working on, as close as possible, which is in the cache.
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5OTk1MzE5MTksLTUyMzg1ODE1OF19
-->