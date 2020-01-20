# Installation pour le développement
Ci-dessous la procédure d'installation d'un environnement de développement avec ```pipenv ```.
```sh
# Install dependencies
pipenv install --dev

# Setup pre-commit and pre-push hooks (optional)
pipenv run pre-commit install -t pre-commit
pipenv run pre-commit install -t pre-push
```
Les packages suivants sont disponibles :
```sh
black, isort, flake8, pytest (-s -v), pytest --cov --cov-fail-under=100
```

Le Pipfile contient les packages à installer.
Les paramètres du coverage se trouvent dans ```.coveragerc```, ceux de pre-commit dans ```.pre-commit-config.yaml```.
Pour pytest, flake8 et isort regarder ```setup.cfg```.

Voici maintenant quelques indications :
 - pour exécuter un package ```pipenv run <package>```.
 *Tous les packages sont exécutés avant chaque commit et chaque push si les hooks correspondants sont activés.*
 - pour passer outre les hooks pre-commit et pre-push ```git commit/push --no-verify```.

# Timing and Profiling
Il est utile d'évaluer le temps d'exécution de chaque fonction pour optimiser son programme.
```sh
bash performances.sh perf.prof pyhack.py
```
Maintenant que l'on sait quelle fonction ralentit notre programme, on peut vouloir mesurer sa durée d'exécution, sans s'occuper du reste du code. On peut utiliser pour cela un simple décorateur :
```python
def timeit_wrapper(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()  # Alternatively, you can use time.process_time()
        func_return_val = func(*args, **kwargs)
        end = time.perf_counter()
        print('{0:<10}.{1:<8} : {2:<8}'.format(func.__module__, func.__name__, end - start))
        return func_return_val
    return wrapper
```
Ce décorateur s'utilise comme suit:
```python
@timeit_wrapper
def function(*args):
    ...
print('{0:<10} {1:<8} {2:^8}'.format('module', 'function', 'time'))
```

> > Time package provides time.perf_counter and time.process_time. The difference here is that perf_counter returns absolute value, which includes time when your Python program process is not running, therefore it might be impacted by machine load. On the other hand process_time returns only user time (excluding system time), which is only the time of your process.

See more [timing tips](timing-tips.md). (Memoization, Locals, Imports, Generators)

# Pytest

J'utilise [```pytest```](pytest.md) pour tester mon code.

# [Python Tips and Tricks](python-tips-and-tricks.md)

 * Sanitizing String Input
 * Find Close Matches of a Word/String
 * Prompting User for a Password at Runtime
 * Taking Slice of an Iterator
 * Naming a Slice Using ```slice``` Function
 * Skipping Begining of Iterable
 * Find the Most Frequently Occurring Items in a Iterable
 * Functions with only Keyword Arguments (kwargs)
 * Creating Object That Supports ```with``` Statements
 * Comparison Operators the Easy Way
 * Saving Memory with ```__slots__```
 * Defining Multiple Constructors in a Class
 * Limiting CPU and Memory Usage
 * Debugging Program Crashes in Shell
 * Controlling What Can Be Imported and What Not
 * Working with IP Addresses

# [Python Design Patterns](python-design-patterns.md)
- Design patterns
	- Program to an interface not an implementation
	- Favor object composition over inheritance
- Behavioral patterns
	- Iterator
	- Chain of Responsibility
	- Command
- Creational Patterns
	- Singleton
	- Dependency Injection
- Structural Patten
<!--stackedit_data:
eyJoaXN0b3J5IjpbNzMwNTAzMzQ1XX0=
-->