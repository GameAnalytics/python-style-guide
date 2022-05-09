# Python at GA
This guide covers code style and best practices, when writing Python at GA.
It is meant as a reference for writing _and_ reviewing Python code.

Although specific rules regarding style and best practices are usually based on sound arguments,
they will often, to some degree, be subjective and arbitrary.
So to avoid too much bikeshedding and pointless discussion, this guide is based on a small set of
other guides that are considered to have some authority in the wider Python community:

1. [PEP8](https://www.python.org/dev/peps/pep-0008/)
2. The book [Clean Code, by Robert C. Martin](https://www.goodreads.com/book/show/3735293-clean-code)
3. [Google's style guide on Python](https://google.github.io/styleguide/pyguide.html)
   
Each chapter takes the form of a list of rules, followed by subsections of clarifications for each rule.

Finally, there are sometimes good reasons for breaking the rules, so use your best judgement.

## Style
1. PEP8 is **law**, never break PEP8. The only exceptions are:
    - lines are allowed to be up to 99 characters.
    - functions with long argument list should prefer [Black style](https://github.com/psf/black/issues/1178)
2. Use a linter like `flake8` to make sure that your code follows PEP8.
3. Consider running `flake8` as a Github CI check.
4. Decide on, and be consistent about, style issues not covered by PEP8, like:
   - Usage of `"` vs `'` for string literals
   - multi-line function definitions / calls
5. Make names descriptive, but concise.
6. Symbols that are internal to a module or class, should start with `_`.
7. Avoid redundant naming, e.g. `models.UserModel`.
8. Use [new-style format strings](https://docs.python.org/3/library/string.html#format-string-syntax),
   don't use old style `%` format strings or string concatenation with `+`.
9. Public functions must be documented with docstrings
10. Keep comments to a minimum, generally only needed when they are necessary for understanding the code.
11. Never commit commented-out code. Just delete it, Git remembers.
12. Remove dead code as early as possible. Don't leave it around *just in case we need it*, Git remembers.
13. Follow the boy scout rule, leave code better than you found it.

### PEP8
[PEP8](https://www.python.org/dev/peps/pep-0008/) forms the basis of this style guide.
Familiarize yourself with the basic rules of PEP8, but don't memorize it.
Instead, use a linter like `flake8` in your editor, to make sure that your code is compliant.

Although PEP8 says to keep lines to 79 columns, this seems to be somewhat outdated today.
The primary motivation for limiting line length is the ability to have 2 editor windows next to each other without wrapping lines.
Since most people use 16:9 monitors today, a max line length of 99 should be fine.

Running a linter in you CI chain (Travis etc.) makes reviews a lot quicker, as the reviewer can immediately see if the code is PEP8 compliant.

PEP8 leaves certain choices open, such as how to break function declarations across multiple lines
and whether to use `'` or `"` for string literals. Decide what to do on a per-repository basis, but be consistent within each repository.

### Names
Keep names as short as possible, while still being descriptive. Avoid abbreviations for anything which is not already an acronym (e.g. HTTP).
Certain abbreviations are well known and widely used within the Python community, e.g. `np` for Numpy, `dt` for datetime. These are okay to use.
Also, resist the temptation to put the type of the variable into the name, e.g. `user_info_dict`, `permission_list`, etc.

Functions should have names that clearly describe what they *do*. If you find yourself struggling to come up with a descriptive name for a function,
it might be that the function is trying to do too much. If so, split the function up. Functions should do only 1 thing.

Example of appropriate naming, for a variable holding the number of days between 2 events:
```python
# Too long
number_of_days_between_events = get_dt_between_events().days

# Not descriptive
dbe = get_dt_between_events().days

# Okay
days_between_events = get_dt_between_events().days
```
Avoid redundant naming.
Redundant naming is when the name of the type or module that a symbol belongs to, is included in the name of the symbol.
For example, if we have `models` module, with a User model inside, redundant naming would be calling the user model `UserModel`.
Instead, it should simply be called `User`. Capitalization indicates that `User` is a class, it's location (inside models),
indicate that it's a model.

In cases where this would cause ambiguity or uncertainty when reading the code, try to avoid importing the symbols directly, for example, if we want the `User` model from the models submodule of the database package:
```python
from database.models import User
# May not be clear what User is, in this context
my_user = User()


from database import models
# Less ambiguous, we can clearly see where User comes from
my_user = models.User()
```
The idea is that with `models.User()` it's easier to understand what `User` is in this context.

### Docstrings, Comments and Dead Code.
When writing docstrings, use one of the existing standards for docstrings, like [Numpy docstrings](https://numpydoc.readthedocs.io/en/latest/format.html)
or whatever your automatic documentation generator expects.

Comments are okay, when you:
1. Copied some strange logic fom StackOverflow, that you don't quite understand (Link to SO page).
2. You just have to do some kind of sick hack as a workaround, for performance, etc.
   ([famous example from Quake 3](https://github.com/id-Software/Quake-III-Arena/blob/dbe4ddb10315479fc00086f08e25d968b4b43c49/code/game/q_math.c#L561))
3. There is just no way to write the code in a clear and understandable form.

It should be obvious that the situations above should be rare and *if* you feel the urge to write a comment,
always consider if the code could be written in a cleaner, more easily understandable form.
Comments also have a tendency to become obsolete or incorrect with time, in which case they are misleading at best.
Example of bad comments:
```python
# Get all frobs for users   <-- Redundant, stating the obvious
user_frobs = get_user_frobs(user_id)

# Request contains user_id, access_level and name   <-- Will become outdated/misleading
request = get_request()
```

Don't ever commit commented-out code. It creates a lot of noise in the code and it tends to just sit around forever.
Likewise, projects have a tendency to accumulate dead code with time. As soon as you identify a piece of code as being dead, delete it.
If you remove a call to a piece of code (function, class, etc.) and that code is not used anywhere else, just remove it.

## Python Best Practices
1. Don't write Python 2, unless you are changing legacy code.
2. Use Python's exceptions for error handling, don't hesitate to write your own exceptions if needed.
3. Never return `None` as an indication of error. Raise an exception instead.
4. Consider using the typing system in Python 3, check types in your CI chain.
5. But don't constrain functions unnecessarily, use abstract types, use duck-typing as an advantage, not a liability.
6. Python has a rich standard library, you are highly encouraged to use it whenever possible.
7. Don't be afraid to use abstractions like list comprehensions, generators, iterators, context managers etc. where appropriate.
8. Don't use mutable types as default arguments.
9. Prefer pure functions over object-oriented programming, but use OOP when appropriate.
10. When writing classes, don't create getters and setters unless necessary, just access the attribute directly.

### Exceptions and Error Handling
Python's error handling is built around exceptions. Use them.

Raise exceptions in your code when encountering an invalid state with no clear way to recover from,
e.g. when a caller tries to use a function in a way that is not well-defined
```python
def get_persons_by_age(age):
   if age < 0:
      raise ValueError('age cannot be negative')
   ...
```

But in general, you should never check the type of arguments passed to the function.
Python's type system is built around the concept of duck-typing. Meaning that as long as an object
*behaves* like it should, i.e. defines the needed interface, it doesn't matter what the
type actually is. Checking the type would then limit the generalizability of the function.

As an example, consider a function that needs to iterate over a collection of objects and do something
for each of them. This function could take any type as argument, as long as the type is an *iterable*
of some sort. Implementing a check to see if the type is a list would be pointless and actually harmful.

If a caller passes an incompatible object as an argument, the function should eventually fail,
often in way where it's not obvious that it was due to an incompatible argument.
That is unfortunately the price we pay for having duck-typing.

When *running* code that might raise exceptions, we should catch exceptions that are expected and
that we know how to handle in a good way
```python
def load_config(conf_path):
   try:
      conf_file = open(conf_path)
   except IOError:
      print('Could not load config, loading default config')
      conf_file = open(DEFAULT_CONFIG_PATH)
   ...
```

You should however, only catch exceptions that you know how to handle.
If an exception would be unrecoverable, or if it would be an indication of a bug in your code, there is no point in catching it.
For this reason, you should only catch specific exceptions. Don't ever do this
```python
try:
   user = get_user(user_id)
except Exception:
   # Oh no, something bad happened
   ...
```
This would catch any exception, even ones caused by bugs in our own code.
Even exceptions that we may not even know existed and which we have no way of recovering from.

This is even worse
```python
try:
   user = get_user(user_id)
except:
   # Oh no, something bad happened
   ...
```
As it will even catch exceptions that are not even errors, like `KeyboardInterrupt` (user presses ctrl+c).

The only possible cases where catching `Exception` is okay, is when we need to log that an exception happened,
but in these cases, the exception should be re-raised immediately
```python
try:
   user = get_user(user_id)
except Exception as e:
   logging.error(e)
   raise
```

A bare `except:` might be okay if we are writing an application and it needs to clean something up before exiting.

Returning `None` to indicate an error is never acceptable, as it is now the callers responsibility to check if an error happened.
They most likely won't do that, so now the error just propagates onwards, away from where it actually happened.
`None` also doesn't tell us or the user anything about what actually went wrong.

Note however that there can be legitimate reasons to return `None` from a function.
If a function would normally return some scalar value, which might not exist and if the non-existence of that value is not an error,
the function may return `None`. The documentation of the function should state that it might return `None`.

Pythons built-in exceptions are often enough, but sometimes it can be useful for us to define our own exceptions.

One example where this could be useful, is when we want to pass low-level errors up to a higher layer of abstraction.
Maybe we are writing a client library for a database and we don't want to expose the library users to database exceptions.
```python
import sqlalchemy.orm.exc as exc
import db
import models

class InvalidUser(Exception):
   pass

def get_user(user_id):
   try:
      user = db.session.query(models.User).filter(models.User.uid == user_id).one()
   except exc.NoResultFound:
      raise InvalidUser(f'user with id {user_id} does not exist')
```

### Types and Generalization
Python is a duck-typed language, meaning that any object can be used anywhere, as long as it *behaves* as it should,
that is, as long as it exposes the right methods. Consider the following function
```python
def find_squanched_frob(frobs):
   for frob in frobs:
      if frob.is_squanched():
         return frob
   return None
```

This function works as long as `frobs` is an iterable, it doesn't matter if it's a list, dict, generator, etc.

One downside of duck-typing is that many errors that could have been prevented, by having a static type system,
will only reveal themselves at runtime. For example, passing a non-iterable to `find_squanched_frobs` by mistake
```python
squanched_frob = find_squanched_frobs(5)   # Will raise TypeError
```

Of course, if the code was well covered with tests, we would find this error before deploying,
but in Python 3, it is also possible to use type hints from the `typing` module.
Note that these types are not checked as part of running the code, neither when parsing or at runtime.
So when using types, it is necessary to run a static type checker like `mypy`.
This can be run in the editor when linting and as part of the CI chain.

When using type hints, use generic types whenever possible, to retain duck-typing.
```python
from typing import Iterable, List

# Bad
def find_squanched_frob(frobs: List[Frob]) -> Frob:
...

# Good
def find_squanched_frob(frobs: Iterable[Frob]) -> Frob:
...
```

Always consider if your code could be made more generic.
For example, maybe there are other things than a `Frob` that can be squanched.
Perhaps `Frob` inherits from the `Squanchable` base class or maybe it implements the `Squanchable` [protocol](https://www.python.org/dev/peps/pep-0544/).
```python
def find_squanched_things(squanchables: Iterable[Squanchable]) -> Squanchable:
...
```

Whether or not to use type hints should be considered on a per-repository basis.
However, you should probably use type hints if:
1. You are writing a library/package that others may import and use in their code.
2. You are writing code that will run in production.
3. You are writing code for which either of the above might be true, at some point in the future.

Finally, there are a few specific rules about typing:
1. Parameters that might legally be `None`, should be typed as `Optional[]`
2. Constants should be typed as `Final[]`
3. Don't use the `Any` type, it defeats the purpose of using types.

### Abstractions and Syntactic Sugar
Python contains a lot of high-level abstractions that are often considered advanced features of the language.
These include things like decorators, context managers, generators, list comprehensions, etc.

Don't be afraid to use these features because you think they might make the code hard to read or too complex.
There is of course a right time and place for all of these, but when used right, they often make the code easier to read and *less* complex.

For example, for building up a list
```python
# Okay
user_emails = []
for user in users:
   user_emails.append(user.email)

# Better
user_emails = [user.email for user in users]
```

The list comprehension immediately expresses the semantics of what is going on, we are building a list.
The for-loop does not, as it can be used for many things, not just building lists.

When code is simpler, it is also often less error prone.
It's impossible to forget closing a file, with a context manager, for example
```python
# Bad
f = open('studios.csv')
studios_data = f.read()
f.close()

# Good
with open('studios.csv') as f:
   studios_data = f.read()
```

Don't be shy about implementing your own context manager, decorator etc. when appropriate.
For example, decorators are very useful for validating inputs.

Finally, Python has a very large and well-documented standard library.
If you are finding yourself writing something generic like a config-file parser or secure hashing algorithm (okay, don't ever do that yourself),
you should take a look in the standard library, there is probably a module for it.

### Mutability and Object-Oriented Programming
An important lesson from functional programming, is that mutable state should be avoided.
It makes programs more complex, harder to reason about and more difficult to change, without introducing bugs.

In general we should use *pure* functions when we are implementing the logic of our codebase.
A pure function is one that doesn't have any side-effects and is deterministic,
meaning it doesn't change any external objects or variables and it always returns the same output, when given the same inputs.

```python
frobs = get_frobs()

# Mutates external state
def squanch_frobs(frobs):
   for frob in frobs:
      frob.squanch()

# Mutates external state and doesn't say which state
def squanch_frobs():
   for frob in frobs:
      frob.squanch()

# Pure - doesn't mutate external state
def squanch_frobs(frobs):
   return [frob.squanch() for frob in frobs]
```

As with most other things in this guide, there are exceptions to the rule. Building a new list is of course more expensive
than mutating the items in the existing list, so when operating with large amounts of data or if performance is very critical,
it is fine to do like this

```python
def squanch_frobs(frobs):
   for frob in frobs:
      frob.squanch()
   return frobs

frobs = get_frobs()
frobs = squanch_frobs(frobs)
```

When we actually *need* to do side-effects, like write a file, send an HTTP request etc. We should make sure that our function does *just* that.
In general, functions should do only one thing
```python
# Bad
def calculate_results_and_write_to_file(studio_data, output_path):
   # Do some computation here
   result_data = []
   for row in studio_data:
      new_row = ...
      result_data.append(new_row)
   
   # Write to file
   with open(output_path, 'w') as f:
      f.write(result_data)


# Good
def calculate_results(studio_data):
   result_data = []
   for row in studio_data:
      new_row = ...
      result_data.append(new_row)

   return result_data


def write_results(result_data, output_path):
   with open(output_path, 'w') as f:
      f.write(result_data)
```

Python is a multi-paradigm language and has support for all the common object-oriented concepts like classes, inheritance, polymorphism, etc.
But there is often no need to use OOP. We can often get by with just a set of pure functions arranged in modules.

That doesn't mean that OOP is not a useful tool. Classes can be incredibly useful for
1. Storing related data together, e.g. a User class with name, email, user_id, etc. Check out [dataclasses](https://docs.python.org/3/library/dataclasses.html)
2. Encapsulating concepts that are inherently stateful, like database connections, UI elements, data structures.
3. Inheritance can be useful for Polymorphism. This works especially well with the typing system.

As an example of `3.`, imagine that you are writing a library that needs to be able to interact
with 2 kinds of data backends, Druid and SQL.
You could then write an abstract DataBackend class, defining the methods that all data
backends should implement.
Then write 2 classes, DruidBackend and SQLBackend, that inherit from this class,
and use DataBackend as a type for any function that needs to be able to interact with both.
A [protocol](https://www.python.org/dev/peps/pep-0544/) may sometimes be a better fit.

The Pythonic way of accessing class attributes in general, is not to use getters and setters.
Instead, the user simply accesses the attribute directly. Any attribute which is considered internal
to the class, should be prefixed with `_`.

In case it is necessary to run code whenever the user sets or gets an attribute,
use the [`@property` decorator](https://docs.python.org/3/library/functions.html#property).
Note that if you start out by using a simple attribute, you can later change it to a `@property` decorated getter method
without changing the class interface at all. The users of the class will be none the wiser.

It is also considered good practice to use the `@classmethod` decorator for factory methods and `@staticmethod` for any static methods.

When writing functions with default arguments, don't use mutable default values like
```python
def squanch_frobs(frobs, excluded_frobs=[]):
...
```

Intuitively, one would think that the list assigned to `excluded_frobs` would be re-initialized as
empty, everytime the function is called. But it is actually constructed once, at import time, so
instead becomes a form of mutable global state. If we defined `squanch_frobs` like this
```python
def squanch_frobs(frobs, excluded_frobs=[]):
   excluded_frobs.append(1)
   print(excluded_frobs)
```
Then called it multiple times, we would get
```console
>>> squanch_frobs(my_frobs)
[1]
>>> squanch_frobs(my_frobs)
[1, 1]
```

When you don't need the argument to be mutable inside the function, you could simply use a non-mutable
alternative, like a tuple instead of a list.

If you do need the argument to be mutable, the Pythonic way of handling it would be to set it to `None`
and then initialize in the function body
```python
def squanch_frobs(frobs, excluded_frobs=None):
   if excluded_frobs is None:
      excluded_frobs = []
   ...
```

## Testing
Proper test coverage is always important, more so in dynamically typed languages like Python, where even trivial type errors
may not be discovered until runtime. In general, if you are writing a library or writing code that will run in production or a *live*
environment, you **must** have proper test coverage.

There are many different test philosophies and methodologies, but one of the most popular testing methods is unit testing.
Good unit tests have the following properties:
1. Small. Test only 1 thing per test.
2. Thorough. Make sure all branches are covered.
3. Independent. Don't rely on the existence or execution order of other tests.
4. Fast. If your test relies on potentially large input data, try make a small, but realistic sample of data for the test.

There are many different testing frameworks for Python. The most popular one appears to be [pytest](https://docs.pytest.org/en/latest/).

When dealing with external entities and services, like an SQL database, or AWS, it is usually a good idea to mock these entities.
Many services even have their own Python libraries, specifically for mocking tests.
For example, the popular boto3 library for interacting with AWS, has a mock version called moto3.

It is the responsibility of the developer implementing a feature to make sure that new and changed code is adequately covered with tests.
Remember to take tests into account when estimating time for a project.

It is highly recommended to integrate services like [codecov](https://codecov.io/) or [coveralls](https://coveralls.io/) into your CI chain.
A per-repository policy, requiring PRs to have positive coverage diff, can also be a good idea.

## Virtual Enviroments, Packaging and Versioning

### Virtual Environments
Python installations are system-wide, by default. The same goes for Python packages, installed with `pip install`.
Since different projects might need different versions of the same package, or even different versions of Python,
it is necessary to separate projects into virtual environments.

Many different virtual env tools exist for Python. Python has a built-in tool called venv, but this tool is very low-level,
so it might be a good idea to install a more high-level tool, like virtualenv.

Somewhat recently, a couple of new tools called [Poetry](https://python-poetry.org/)
and [Pipenv](https://pipenv-fork.readthedocs.io/en/latest/) have appeared.
Both of these tools are intended to replace Pip and offer both virtual environment support and dependency management.

One benefit of using these over pip, is that they promise to make deterministic environments, something that pip unfortunately cannot do.
This means that you can install dependencies in your dev environment, test that everything works, deploy to production and be certain that
the exact same versions of your dependencies will be installed there, even without pinning the version numbers.

Both offer more or less the same features, except that Poetry makes packaging considerably easier, as it needs only the `pyproject.toml` file,
as discussed in the next section.

### Packaging
You should always consider whether your project should be an installable Python package.
If you are writing a library that needs to be imported into other Python projects, your project needs to be a package.
There are other good reasons to package your project. For example, if your project comes with one or more CLI tools,
users could simply install your package, like `pip install flask` and immediately be able to run your CLI in the shell
```console
~$ flask --help
Usage: flask [OPTIONS] COMMAND [ARGS]...

  This shell command acts as general utility script for Flask applications.

  It loads the application configured (through the FLASK_APP environment
  variable) and then provides commands either provided by the application or
  Flask itself.
```

If you are building a service or web-app, that is going to be deployed to some infrastructure,
you typically don't need to make a package out of your code. But it might still be useful for versioning or deployment considerations.

When developing on a package, it's a good idea to install the package as editable.
This tells your package manager that code should be runned or imported directly from the source folder
rather than from installed packages. This way you won't have to re-install the package
everytime you want to test some changes to the code. Root packages are installed as editable by default in Poetry,
here is how to do it in [Pipenv](https://pipenv-fork.readthedocs.io/en/latest/basics.html#editable-dependencies-e-g-e)
and [Pip](https://pip.pypa.io/en/stable/reference/pip_install/#editable-installs).

Unfortunately, packaging in Python has always been rather messy and lacking in standardization.
The recommended recipe has, until recently, been to [use setuptools and write a setup.py file](https://packaging.python.org/tutorials/packaging-projects/).

Recently however, an attempt to standardize packaging has been made, with [PEP 517](https://www.python.org/dev/peps/pep-0517/)
and [PEP 518](python.org/dev/peps/pep-0518/).
This enables package authors to specify what tools are required to build their package, through the `pyproject.toml` file,
without having to implicitly require that the user installs a specific build system, like setuptools.

Python packaging is a very active area of development in the Python community and you are strongly
encouraged to use the latest accepted standards which, at the time of writing, is using `pyproject.toml`.

Poetry is it's own build system, so only needs the `pyproject.toml` file in order to build both wheels and source distributions.
If you don't use Poetry, you are free to use any other build system, but most users will probably want to use setuptools.
[This article](https://snarky.ca/what-the-heck-is-pyproject-toml/) contains an explanation of how to use the `pyproject.toml` file with setuptools.

Once a package has been built into a wheel, this artifact can then be distributed and installed anywhere, by any package manager.

### Versioning
If you are building a package, especially if you are building a library, you need to version the package.
You are free to use any versioning scheme, as long as you are consistent about what the version numbers *mean*.

A popular versioning standard is [semantic versioning](https://semver.org/) or SemVer.
Every version is denoted by a triplet `MAJOR.MINOR.PATCH`.\
Increment `PATCH` when making backwards compatible bug-fixes.\
Increment `MINOR` when adding functionality that is backwards compatible.\
Increment `MAJOR` when adding functionality that is not backwards compatible.

A `MAJOR` version of 0, indicates that the package is still under development and that anything may change at any time.
