@title[Introduction]

## Python common errors #2: Range, Iterable and Iterator

by Yury Krasouski

+++
@title[Introduction:Interview question]

What is the most frequently asked question on `Python` interview for beginners?

+++
@title[Sequence]

What do you know about `xrange` in `Python2` or `range` in `Python3`?

+++
@title[Sequence:Game]

Let's play a game.

```python
from types import GeneratorType
from collections import Iterable, Iterator, Sequence

try:
    r = xrange(10)
except NameError:
    r = range(10)

print(isinstance(r, list))
print(isinstance(r, tuple))
print(isinstance(r, Sequence))
print(isinstance(r, Iterator))
print(isinstance(r, Iterable))
print(isinstance(r, GeneratorType))
```

---
@title[Sequence:Definition]

How do you understand `Sequence` in `Python`?

Is following type a `Sequence`?
 - `set`
 - `str`
 - `list`
 - `dict`
 - `tuple`

+++
@title[Sequence:Validation]

Here is the output for `Sequence` checks.

```python
>>> from collections import Sequence
>>> issubclass(set, Sequence)
False
>>> issubclass(str, Sequence)
True
>>> issubclass(list, Sequence)
True
>>> issubclass(dict, Sequence)
False
>>> issubclass(tuple, Sequence)
True
```

---
@title[Iterable:Difference]

What is the difference between `Iterable` and `Iterator`?

```python
>>> from collections import Iterable, Iterator
>>> issubclass(Iterator, Iterable)
?
>>> issubclass(Iterable, Iterator)
?
```

+++
@title[Iterable:Error]

Please, try to find an error (or errors).

```python
from collections import Iterable

def planify(items, skip=(str, bytes)):
    for x in items:
        if isinstance(x, Iterable) and not isinstance(x, skip):
            for y in planify(x):  # yield from planify(x)
                yield y
        else:
            yield x

items = ('abc', 3, [8, ('x', 'y'), [97]])
print(list(planify(items)))
# ['abc', 3, 8, 'x', 'y', 97]
```

+++
@title[Iterable:Custom]

Let's define very simple class.

```python
class Words(object):
    def __init__(self, sentence):
        self._words = sentence.split()
    def __getitem__(self, index):
        return self._words[index]

sentence = "Minsk Python Meetup"
for word in Words(sentence):
    print(word)

# Minsk
# Python
# Meetup
```

+++
@title[Iterable:Usage with `planify`]

What is your expectation about the output?

```python
from planify import planify
from words import Words

w = Words("python is awesome")
items = ('abc', 3, ['x', 'y'], w)
print(list(planify(items)))
# ['abc', 3, 'x', 'y', ???]
```

+++
@title[Iterable:Output]

Do you expect such behavior?

```python
>>> list(planify(items))
['abc', 3, 'x', 'y', <__main__.Words object at 0x7fa1fa23fb10>]
```
Or a little bit different?

```python
>>> list(planify(items))
['abc', 3, 'x', 'y', 'python', 'is', 'awesome']
```

Is `Words` subclass of `Iterable`?

```python
>>> print(issubclass(Words, Iterable))
```

---
@title[Explanation:Iterable sources]

`.../python3.5/_collections_abc.py:191`

```python
class Iterable(metaclass=ABCMeta):
    __slots__ = ()

    @abstractmethod
    def __iter__(self):
        while False:
            yield None

    @classmethod
    def __subclasshook__(cls, C):
        if cls is Iterable:
            if any("__iter__" in B.__dict__ for B in C.__mro__):
                return True
        return NotImplemented
```
> Why `__slots__` is here?

+++
@title[Explanation:Iterator sources]

`.../python3.5/_collections_abc.py:208`

```python
class Iterator(Iterable):
    __slots__ = ()

    @abstractmethod
    def __next__(self):
        raise StopIteration

    def __iter__(self):
        return self

    @classmethod
    def __subclasshook__(cls, C):
        if cls is Iterator:
            if (any("__next__" in B.__dict__ for B in C.__mro__) and
                any("__iter__" in B.__dict__ for B in C.__mro__)):
                return True
        return NotImplemented
```

+++
@title[Explanation:Iter returns self]

`__iter__` of `Iterator` should return `self`.

```python
>>> it = iter(range(10))
>>> next(it)
0
>>> next(it)
1
>>> it2 = iter(it)
>>> next(it2)
?
```

---
@title[Solutions]

### What is the possible solutions for `Words` and an `Iterable`?

+++
@title[Solutions:Inheritance]

The first solution is inheritance from `Iterable`.

```python
class Words(Iterable):
    def __init__(self, sentence):
        self._words = sentence.split()
    def __getitem__(self, index):
        return self._words[index]
    def __iter__(self):  # abstractmethod
        return iter(self._words)

w = Words("python is awesome")
items = ('abc', 3, ['x', 'y'], w)
print(list(planify(items)))
# ['abc', 3, 'x', 'y', 'python', 'is', 'awesome']
```

+++
@title[Solutions:__iter__]

Or just to add `__iter__` method.

```python
from collections import Iterable

class Words(object):
    def __init__(self, sentence):
        self._words = sentence.split()
    def __getitem__(self, index):
        return self._words[index]
    def __iter__(self):
        return iter(self._words)  # iter(self)

w = Words("python is awesome")
items = ('abc', 3, ['x', 'y'], w)
print(list(planify(items)))
# ['abc', 3, 'x', 'y', 'python', 'is', 'awesome']
```
> There are no such hooks for `Sequence`.

+++
@title[Solutions:Question]

### What if we are not able to modify `Words` class?

+++
@title[Solutions:Strings difference]

Notice the difference for `Python2` and `Python3`.

`Python2`

```python
>>> iter("abc")
<iterator object at 0x7f07862b6210>
```

`Python3`

```
>>> iter("abc")
<str_iterator object at 0x7fe4d989b828>
```

+++
@title[Solutions:String tip]

Why this happens?

`Python2`

```python
>>> str.__mro__
(<type 'str'>, <type 'basestring'>, <type 'object'>)
>>> hasattr("foo", "__iter__")
False
>>> isinstance(s, Iterable)
True  # Why?
```

`Python3`

```python
>>> str.__mro__
(<class 'str'>, <class 'object'>)
>>> hasattr("foo", "__iter__")
True
>>> isinstance(s, Iterable)
True
```

+++
@title[Solutions:String virtual subclass]

There is a registry for virtual subclasses.

`Python2`

```python
>>> Iterable._abc_registry
<_weakrefset.WeakSet object at 0x7f496d959d50>
>>> list(Iterable._abc_registry)
[<type 'str'>]
>>> list(Sequence._abc_registry)
[<type 'tuple'>, <type 'buffer'>, <type 'xrange'>, <type 'basestring'>]
```

`Python3`

```python
>>> Iterable._abc_registry
<_weakrefset.WeakSet object at 0x7f4b574783c8>
>>> list(Iterable._abc_registry)
[]
>>> list(Sequence._abc_registry)
[<class 'tuple'>, <class 'memoryview'>, <class 'str'>, <class 'range'>]
```

+++
@title[Solutions:Registration method]

Registration via method `Iterator.register`

```python
from collections import Iterable

from planify import planify
from words import Words

Iterable.register(Words)

w = Words("python is awesome")
items = ('abc', 3, ['x', 'y'], w)
print(list(planify(items)))
# ['abc', 3, 'x', 'y', 'python', 'is', 'awesome']
```

+++
@title[Solutions:Registration decorator]

We can also use decorator but only for `Python3`.

```python
from collections import Iterable

@Iterable.register
class Words(object):
    def __init__(self, sentence):
        self._words = sentence.split()
    def __getitem__(self, index):
        return self._words[index]

w = Words("python is awesome")
items = ('abc', 3, ['x', 'y'], w)
print(list(planify(items)))
# ['abc', 3, 'x', 'y', 'python', 'is', 'awesome']
```
+++
@title[Solutions:Check an `Iterable`]

## How we can check that object is _really_ iterable?

+++
@title[Solutions:Sad but true]

There is no such function like `isiterable`.

Use `iter()` or `for` and catch `TypeError`.

> Sad but true.

---
@title[Thank you]

## Thank you!

Presentation:

`gitpitch.com/krasoffski/pyerrors2`

krasoffski@ (gmail, telegram, github, etc)