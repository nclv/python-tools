### Testing for Exceptions

> > Let's start simple. You have a function that throws an exception and you want make sure that it happens under right conditions and includes correct message: 

```python
import pytest

def test_my_function():
    with pytest.raises(Exception, match='My Message') as e:
        my_function()
        assert e.type is ValueErrors
```

> > Here we can see simple context manager that Pytest provides for us. It allows us to specify type of exception that should be raised as well as message of said exception. If the exception is not raised in the block, then test fails.

### Filtering Warnings

```python
import warnings
from sqlalchemy.exc import SADeprecationWarning

def test_my_function():
    # SADeprecationWarning is issued by SQLAchemy when deprecated API is used
    warnings.filterwarnings("ignore", category=SADeprecationWarning)
    ...
    assert ...

def test_my_function():
    with warnings.catch_warnings(record=True) as w:
        warnings.simplefilter("ignore")  # ignore annoying warnings
        ...
        assert ...
    # warnings restored
```

> > Here we show 2 approaches - in first one, we straight up ignore all warnings of specified category by inserting a filter at the front of a filter list. This will cause your program to ignore all warnings of this category until it terminates, that might not always be desirable. With second approach, we use context manager that restores all warnings after exiting its scope. We also specify ```record=True```, so that we can inspect list of issued (ignored) warnings if needs be. 

### Testing Standard Output and Standard Error Messages

> > Next, let's look at following scenario: You have a command line tool that has bunch of functions that print messages to standard output but don't return anything.

```python
def test_my_function(capsys):
    my_function()  # function that prints stuff
    captured = capsys.readouterr()  # Capture output
    assert f"Received invalid message ..." in captured.out  # Test stdout
    assert f"Fatal error ..." in captured.err  # Test stderr
```

> > To solve this, Pytest provides fixture called ```capsys```, which - well - captures system output. All you need to use it, is to add it as parameter to your test function. Next, after calling function that is being tested, you capture the outputs in form of tuple - ```(out, err)```, which you can then use in assert statements. 

### Patching Objects

> > Sometimes when testing, you might need to replace objects used in functions under test to provide more predictable dataset or to avoid said function from accessing resources that might be unavailable. mock.patch can help with that: 

```python
from unittest import mock

def test_my_function():
    with mock.patch('module.some_function') as some_function:  # Used as context manager
        my_function()  # function that calls `some_function`

        some_function.assert_called_once()
        some_function.assert_called_with(2, 'x')

@mock.patch('module.func')  # Used as decorator
def test_my_function(some_function):
    module.func(10)  # Calls patched function
    some_function.assert_called_with(10)  # True
```

> > In this first example we can see that it's possible to patch functions and then check how many times and with what arguments they were called. These patches can also be stacked both in form of decorator and context manager. Now, for some more powerful uses:

```python
from unittest import mock

def test_my_function():
    with mock.patch.object(SomeClass, 'method_of_class', return_value=None) as mock_method:
        instance = SomeClass()
        instance.method_of_class('arg')

        mock_method.assert_called_with('arg')  # True

def test_my_function():
    r = Mock()
    r.content = b'{"success": true}'
    with mock.patch('requests.get', return_value=r) as get:  # Avoid doing actual GET request
        some_function()  # Function that calls requests.get
        get.assert_called_once()
```

> > First example in above snippet is pretty straightforward - we replace method of ```SomeClass``` and make it return ```None```. In the second, more practical example, we avoid being dependent on remote API/resource by replacing ```requests.get``` by mock and making it return object that we supply with suitable data.

> > There are many more things that ```mock``` module can do for you and some of them are pretty wild - including side effects, mocking properties, mocking non-existing attributes and much more. If you run into problem when writing tests, then you should definitely checkout [docs](https://docs.python.org/3/library/unittest.mock.html) for this module, because you might very well find solution there. 

### Sharing Fixtures with conftest.py

> > If you write a lot of tests, then at some point you will realize that it would be nice to have all Pytest fixtures in one place, from which you would be able to import them and therefore share across test files. This can be solved with ```conftest.py```.

> > ```conftest.py``` is a file which resides at base of your test directory tree. In this file you can store all test fixtures and these are then automatically discovered by Pytest, so you don't even need to import them.

> > This is also helpful if you need to share data between multiple tests - just create fixture that returns the test data.

> > Another useful features is ability to specify scope of a fixture - this is important when you have fixtures that are very expensive to create, for example connections to database (```session``` scope) and on other end of spectrum are the one that need to be reset after every test case (```function``` scope). Possible values for fixture ```scope``` are: ```function```, ```class```, ```module```, ```package``` and ```session```. 

### Parametrizing Fixtures

> > We already talked about fixtures in above examples, so let's go little deeper. What if you want to create a bit more generic fixtures by parametrizing them?
```python
import pytest, os

@pytest.fixture(scope='function')
def reset_sqlite_db(request):
    path = request.param  # Path to database file
    with open(path, 'w'): pass
    yield None
    os.remove(path)

@pytest.mark.parametrize('reset_sqlite_db', ['/tmp/test_db.sql'], indirect=True)
def test_send_message(reset_sqlite_db):
    ...  # Perform tests that access prepared SQLite database
```
> > Above is an example of fixture that prepares SQLite testing database for each test. This fixture receives path to the database as parameter. This path is passed to the fixture using the ```request``` object, which attribute ```param``` is an iterable of all arguments passed to the fixture, in this case just one - the path. Here, this fixture first creates the database file (and could also populate it - omitted for clarity), then yields execution to the test and after test is finished, the fixture deletes the database file.

> > As for the test itself, we use ```@pytest.mark.parametrize``` with 3 arguments - first of them is name of the fixture, second is a list of argument values for the fixture which will become the ```request.param``` and finally keyword argument ```indirect=True```, which causes the argument values to appear in ```request.param```.

> > Last thing we need to do is add fixture as a argument to test itself and we are done. 

Let’s consider a minimal use case and assume that we have integration tests that require some created rows upon setup. The `PersonRepository` class is responsible for the creation of those rows:

```python
class PersonRepository:

@db_session
def create(self, name, age):
	return Person(name=name, age=age)

@db_session
def get_by_name(self, name):
	return list(select(p for p in Person if p.name == name))
```

We would like to have a common way to deliver rows to tests. Fixtures are here to help:
```python
@pytest.fixture
def person():
	return PersonRepository().create(name='Andrew', age=20)
```
Now, in order to use a fixture within a test, we input its name to the test function’s parameters. Pytest handles the rest:
```python
@pytest.mark.integration

def test_person_repository_gets_the_person_by_name(person):
retrieved_persons = PersonRepository().get_by_name(name=person.name)
assert len((retrieved_persons)) == 1
assert retrieved_persons[0].id == person.id
assert retrieved_persons[0].name == person.name
assert retrieved_persons[0].age == person.age
```

### Skipping Tests

> > There are situations, when it's reasonable to skip some tests, whether it's because of environment (Linux vs Windows), internet connection, availability of resources or other. So, how do we do that?
```python
import pytest, os

@pytest.mark.skipif(os.system("service postgresql status") > 0,
                    reason="PostgreSQL service is not running")
def test_connect_to_database():
    ... # Run function that tries to connect to PostgreSQL database
```
> > This shows very simple example of how you can skip a valid test based on some condition - in this case based on whether the PostgreSQL server is running on the machine or not. There are many more cool features in Pytest related to skipping or anticipating failures and they are very well documented [here](http://doc.pytest.org/en/latest/skipping.html), so I won't go into more detail here as it seems redundant to copy and paste existing content here. 
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE2OTIzMTA4NjFdfQ==
-->