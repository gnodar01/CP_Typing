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
  * all instances of UserId form a type
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

---

# Python Typing
Type hints through animations

---

# Gradual Typing

<br>

No runtime implications of types
* hence type *annotations*

Types never required
* hence type *hints*

No "overhaul" necessary

Type as you go, when desired, only to aid readability and understanding

---

# Gradual Typing
*Consistent with...* relationship

Type `T1` is *consistent with* type `T2` IFF `T1` is a subtype of `T2`
(but not the other way around)

The type `Any` is *consistent with* every type
(but `Any` is not a subtype of `Any`)

Every type is *consistent with* `Any`
(but every type is not a subtype of `Any`)

<br>

Therefore, `Any` is the set of all values, and the set of all functions on those values
* Both top and bottom of type hierarchy

---

# Duck Typing

```python
def mult_add(x, y, b):
	return x * y + b
```

Type of `x`, `y`, and `b` not important, so long as:
* `x` supports multiplication with `y`, via `__mul__`
* result of `x*y` supports addition with `b`, via `__add__`

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

```python
from typing import List

def words_to_ascii(words: List[str]) -> List[List[int]]:
    return [
        [ord(c) for c in word] for word in words
    ]
```

---

# Typing Module

<br>

`from typing import Dict, Set, Tuple, Generator, Iterable, Type, Any`

---

# What can be typed

<br>

Pretty much everything...
* function arguments
* function return types
* variable definitions

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
    user_id = UserId(123
```

`UserId` is still a subtype of `int`, but now has additional behaviour

---

# Progressive Typing

<br>

Add types when desired, to help yourself understand the code

Organic growth

Focused manual efforts

Static and dynamic type inference

Gradually increasing strictness requirements for new code

---

# Tools

Type checkers
* [mypy](http://mypy-lang.org/)
    * [mypy playground](https://mypy-play.net/)
* [pytype](https://google.github.io/pytype/) by Google
* [pyre-check](https://pyre-check.org/) by Facebook
* [pyright](https://github.com/microsoft/pyright) used in vscode
* [pylint](https://pylint.pycqa.org/en/latest/) - static code analysis

Stub files for type definitions
* [typeshed repo](https://github.com/python/typeshed)

---

# Resources

* [PEP-483](https://peps.python.org/pep-0483/): The Theory of Type Hints
* [PEP-484](https://peps.python.org/pep-0484/): Type Hints
* [PEP-544](https://peps.python.org/pep-0544/): Protocols: Structural subtyping (static duck typing)
* [Other relevant PEPs](https://docs.python.org/3/library/typing.html#relevant-peps-1)
* [Dropbox - our journey to type checking 4 million lines of Python](https://dropbox.tech/application/our-journey-to-type-checking-4-million-lines-of-python)
* [Gradual Typing](https://wphomes.soic.indiana.edu/jsiek/what-is-gradual-typing/)



