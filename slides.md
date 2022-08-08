# Typed CellProfiler

Incremental type hints in CellProfiler

---

# Types

a set of values, and a set of functions that can be applied to those values

Type Definition:
* Specify full set of allowable values
  * `Bool: True | False`
* Specify functions which can be used with variables of the type
  * `Sized`: all objects that have a `__len__` method
    * `[1, 2, 3]`
    * `"abc"`
* Class definition
  ```python
  class UserId(int):
      ...
  ```
  * all instances of `UserId` form a type
* More complex, composite types
  * `String`: a subset of `List` such that all elements are of type `Char`
  * `Number`: a union of `Integer`, `Float`, and `Complex`

<style>
ul { font-size: 15px; }
</style>

---

# Subtypes

<br>

Every Type is a Subtype of itself

The set of values becomes smaller in the process of subtyping

The set of functions becomes larger in the process of subtyping

---

# Subtypes

<br>

```
a: Float
b: Integer
```

`a = b`

* Okay
* Every possible value of `b` is also in the set of values of `a`
* Every function from `a` is also in the set of functions of `b`
* `b` (`Integer`) is a subtype of `a` (`Float`)

`b = a`

* Not okay (unless you're purposely upcasting)
* There are some values of `a` not in the set of values of `b`
* There are functions `b` has that `a` doesn't (e.g. bitwise shifts)

---

# Subtypes

<br>

`List[Int]` is **NOT** a subtype of `List[Float]`
* set of values of `List[Int]` are contained in the set of values of `List[Float]`
* set of functions of `List[Float]` are **NOT** contained in the set of functions of `List[Int]`
  * e.g. `append_float()`
* *invariance* relationship

---

# Types vs Classes

<br>

`Class` is a dynamic, runtime concept
* classes are object factories
* returned by `type(obj)` builtin

Types are static, and used by static type checkers outside of runtime

Classes define types, but static types not to be confused with runtime classes
* `int` is a class and type
* `UserId` is a class and type
* `Union[str, int]` is a type, but not a class

---

# Python Typing

Brief history

Jukka Lehtosalo - 2010, PhD research in unifying dynamic and statically typed languages
* Gradual growth from untyped prototype to statically typed product
* Converted research to mypy - an optional static type checker for Python
* Introduced at PyCon 2013, gaining interest of Guido van Rossum
* Joined Dropbox, which implemented mypy on a 4 million+ line codebase

Introduced in 2015: Python 3.5, PEP-484
* Type hinting (type annotation)
* Typing module

Incorporates Gradual Typing
* Formal work by Jeremy Siek and Walid Taha in 2006
* Allows parts of a program to be dynamically typed and other parts to be statically typed

---

# Gradual Typing

<br>

No runtime implications of types*
* hence type *annotations*

Types never required
* hence type *hints*

Dynamic **and** static typing
* annotate only part of a program

No "overhaul" necessary

Type as you go, when desired, only to aid readability and understanding

---

# Gradual Typing
*Consistent with...* relationship

Type `T1` is *consistent with* type `T2` IFF `T1` is a subtype of `T2`
(but not the other way around)

Type `Any` is *consistent with* every type
(but `Any` is not a subtype of `Any`)

Every type is *consistent with* `Any`
(but every type is not a subtype of `Any`)

<br>

Therefore, `Any` is the set of all values, and the set of all functions on those values
* Both top and bottom of type hierarchy
* Dynamic type
* Everything is `Any` unless explicitly annotated otherwise

---

# Python Typing

Building blocks

`Any` is consistent with every type, and vice versa

`Union[t1, t2, ...]`: all types that are subtypes of `t1`, `t2`, ...
* `Union[int, str]` is a subtype of `Union[int, float, str]`
    * *covariance* relationship
* If `ti` is already a `Union`, it is flattened
* Order doesn't matter
* `Union[t1] == t1`

`Optional[t1] == Union[t1, None]`

`Tuple[t1, t2, ..., tn]`: tuple where first element is `t1`, second `t2`, ...
* `Tuple[u1, u2, ..., um]` is a subtype IFF `n == m` and `u1` is a subtype of `t1`, `u2`, of `t2`, ...
* variadic homogeneous tuple: `Tuple[t1, ...]` (literal ellipse)

---

# Python Typing

Building blocks


`Callable[[t1, t2, ..., tn], tr]`: function with positional args `t1` etc., and return type `tr`
* argument list unchecked with `Callable[..., tr]` (literal ellipse)
* `Callable[[], int]` is a subtype of `Callable[[], float]`
    * *covariance* relationship in return type
* `Callable[[float], None]` is a subtype of `Callable[[int], None]`
    * *contravariance* relationship in args
    * counterintuitive, but true

---

# Python Typing

Building blocks

`Intersection[t1, t2, ...]`: Types that are a subtype of each of `t1`, etc.
* Order doesn't matter
* Nested intersections flattened
* Intersection of few types superset of intersection of more types
  * e.g. `Intersection[int, str]` superset of `Intersection[int, float, str]`
* `Intersection[t1]` is just `t1`
* `Intersection[t1, t2, Any] == Intersection[t1, t2]`

---

# Python Typing

Generic types

Generic type constructor
* Like a function for types
* Takes a type, and "returns" a new type

---

# Python Typing

Generic types

```python
def add(x, y):
    return x + y

add(1,2) == 3
add('1', '2') == '12'
add(1.1, 2.2) == 3.3
```

`add` has 2 parameters, that may be of type `int`, `float`, or `str`, so long as both are the *same* type

---

# Python Typing

Generic types

```python
T = TypeVar('T', int, float, str)

def add(x: T, y: T) -> T:
    return x + y

add(1,2) # ok
add('1', '2') # ok
add(1.1, 2.2) # ok
add(1, 2.2) # ok
add(1, '2') # not ok - fails type check
```

For `add(1, 2.2)`, `1` is of type `int`, which is a subtype of `float`, so the generic `T` is calculated to be `float`

For `add(1, '2')`, the only type that the two args have in common is `object` which is not listed in our constraints of `int, float, str`

---

# Motivation

<br>

Python is dynamically typed on purpose
* Great for notebooks, scripts, small scale applications
* Increasingly bad at scale
* Great for single developer projects
* Bad for collaborative projects

---

# Motivation

<br>

A type checker will find many subtle (and not so subtle) bugs
* e.g. forgetting to handle a `None` return value

---

# Motivation

<br>

Increased readability
* New contributors more easily able to understand what a method does, and how to reuse it
* Refactoring is easier

---

# Motivation

<br>

Type checking much faster than testing (for correct types)

Type checkers built into popular IDEs
* Code completion
* Error highlighting
* Go to definition

---

# Simple Types

<br>

Start with atomic types: `str`, `int`, `bool`, `float`, `None`, ...

`def is_valid_path(path: str) -> bool:`


---

# Typing Module

<br>

`from typing import Any, Union, Callable, TypeVar, Generic, Optional`

`from typing import Dict, Set, Tuple, Generator, Iterable, Type, Mapping, Sequence`


---

# Function Annotation

```python
from typing import List

def reverse(words: List[str], inplace: bool=False) -> List[str]:
	if inplace:
		words.reverse()
        return words
	else:
        return words[::-1]
```

---

# Void Functions, Optional Arguments

```python
from typing import Optional

def write_config(
        key,
        value,
        config_file_path: Optional[str]=None
    ) -> None:
	...

```

---

# Variable Annotation

```python
class User:
	id: int
	username: str
	
	def register(self, username: str) -> None:
		self.id = self.generate_new_id()
		self.username = username

def get_user_info(user: User):
    user_id: int = user.id
    ...
```

---

# Type Defintions and Aliases

```python
from typing import List, Tuple, NewType

Protocol   = NewType('Protocol', str)  # type definition
Domain     = NewType('Domain', str)
Port       = NewType('Port', int)
Host       = Tuple[Protocol, Domain, Port]  # type alias

UserId     = int | str
Creds      = Tuple[UserId, str]

Connection = Tuple[Host, Creds]

def connect(connections: List[Connection]):
    pass

connect([
    (
        (Protocol("http://"), Domain("hostname.com"), Port(8080)),
        ("123", "password123")
    ),
    (
        (Protocol("ssh://"), Domain("myserver@192.168.1.100:host"), Port(22)),
        (456, "admin")
    )
])

```

---

# Why Define Types?

```python
from typing import NewType
UserId = NewType(‘UserId’, int) # user ids are integers

# elsewhere 
def generate_dummy_user():
	user_name = “user1”
	user_id = UserId(123)
```

---

# Later...

```python
from typing import NewType
UserId = NewType(‘UserId’, str) # <- decide to change user ids to strings

# elsewhere 
def generate_dummy_user():
	user_name = “user1”
	user_id = UserId(123) # <- type checker will tell you to change this
```

We can change `UserId` to be `str | int`, or we can change all the places that define suer is to now be strings
(and the type checker will tell us all the places to do that)

---

# Alternatively, Later...

```python
class UserId:
	def __init__(self, id: int):
		self.id = id
	def encode_id(self) -> str:
		...

# elsewhere
def generate_dummy_user():
    user_name = "user1"
    user_id = UserId(123)
```

`UserId` is still a subtype of `int`, but now has additional behaviour

---

# Defining Types

```python
from typing import NewType

UserId = NewType('UserId', int)

combinedId = UserId(123) + UserId(456) # resulting type int
combinedId = UserId( UserId(123) + UserId(456) ) # resulting type UserId
```

---

# Type Alias vs Type Definition

<br>

`Alias = Original` is an equivilance relationship

`Derived = NewType('Derived', Original)` specifies that `Derived` is a subtype of `Original`
* `variable: Derived = ...` means `variable` cannot be `int`, it must be `Derived`
* `variable: int = ...` means `variable` can be `int` or `Derived`

---

# Generics

Generic function

```python
from typing import Sequence

T = TypeVar('T')

def first(l: Sequence[T]) -> T:
    return l[0]
```

---

# Generics

User-defined generic types

```python
from typing import Generic, TypeVar, Optional

T = TypeVar('T')
S = TypeVar('S', str, int)

class DataStore(Generic[T, S]):

    def put(self, data: T) -> S:
        ...
    def get(self, id: S) -> T:
        ...

ds: DataStore[str, int] = DataStore()
```

---

# Duck Typing

```python
def mult_add(x, y, b):
    return x * y + b
```

Type of `x`, `y`, and `b` not important, so long as:
* `x` supports multiplication with `y`, via `__mul__`
* result of `x * y` supports addition with `b`, via `__add__`

```python
class Prop:
    def __init__(self, s_val):
        self.s_val = s_val
    def __mul__(self, prop_other):
        return Prop( self.s_val * len(prop_other.s_val) )
    def __add__(self, prop_other):
        return Prop( self.s_val + prop_other.s_val )
    def __repr__(self):
        return self.s_val

mult_add( Prop("foo"), Prop("bar"), Prop("baz") )
> foofoofoobaz

```

---

# Static Duck Typing

```python
from typing import Protocol

class MULT_ADD(Protocol):
    def __mul__(self, other):
        ...
    def __add__(self, other):
        ...

class Prop:
    def __init__(self, s_val):
        self.s_val = s_val
    def __mul__(self, prop_other):
        return Prop( self.s_val * len(prop_other.s_val) )
    def __add__(self, prop_other):
        return Prop( self.s_val + prop_other )
    def __repr__(self):
        return self.s_val
        
def mult_add(x: MULT_ADD, y: MULT_ADD, b: MULT_ADD):
    return x * y + b
    
mult_add(Prop("foo"), Prop("bar"), Prop("baz")) # okay

mult_add(2, 3.0, 2.0+2.0j) # also okay

```

---

# Progressive Typing

<br>

Initial cleanup
* We already have type errors
  * Mostly related to `None` and `Unknown`
  * Some ignorable: untyped external libraries (e.g. wxPython)
* Should be the only "painful" part (for me)

Add types when desired, to help yourself understand the code
* Goal is not eventually reaching 100% type coverage

Gradually increasing strictness requirements for new code

Eventually add type checking to CI

---

# Typed CP

Untyped

<video src="/screen_recordings/pre_typing.mov" controls></video>

---

# Typed CP

Typed

<video src="/screen_recordings/post_typing.mov" controls></video>

---

# Tools

Type checkers
* [mypy](http://mypy-lang.org/)
    * [mypy playground](https://mypy-play.net/)
* [pytype](https://google.github.io/pytype/) by Google
    * Type annotations generation
* [pyre-check](https://pyre-check.org/) by Facebook
* [pyright](https://github.com/microsoft/pyright) used in vscode
* [pylint](https://pylint.pycqa.org/en/latest/) - static code analysis

Stub files for type definitions
* [typeshed repo](https://github.com/python/typeshed)

---

# Resources

* [PEP-3107](https://peps.python.org/pep-3107/): Function Annotations
* [PEP-483](https://peps.python.org/pep-0483/): The Theory of Type Hints
* [PEP-484](https://peps.python.org/pep-0484/): Type Hints
* [PEP-544](https://peps.python.org/pep-0544/): Protocols: Structural subtyping (static duck typing)
* [Other relevant PEPs](https://docs.python.org/3/library/typing.html#relevant-peps-1)
* [Dropbox - our journey to type checking 4 million lines of Python](https://dropbox.tech/application/our-journey-to-type-checking-4-million-lines-of-python)
* [Gradual Typing](https://wphomes.soic.indiana.edu/jsiek/what-is-gradual-typing/)
* [How to enable Python type checking in VSCode](https://www.emmanuelgautier.com/blog/enable-vscode-python-type-checking)


