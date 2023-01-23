---
title: The 5 levels of python enterpriseyness
date: 2023-01-23
public: true
---

# The 5 levels of python enterpriseyness

Most python probably exists as short scripts that run maybe a few dozen times at
most, often only once and then never seen again. Although sometimes we like to
imagine whatever we're working on is going to underpin some critical infrastructure
and have to keep ticking away for years (probably not, but we can imagine). Or maybe
you're a java programmer, or just read too many design pattern books. Sometimes
we have to write "enterprisey" python, bump up that line count, imagine we're real
programmers.

As I've slowly moved from biologist writing awful scripts to writing more
"software", here's what I imagine are the 5 levels of increasing enterpriseyness
in python:


## preamble: example data

Here's the imaginary simple data we're working with, a `sample`. It's just a numpy
array, a barcode, and a boolean flag.

```python
import numpy as np

data = np.random.randn([10, 10])
barcode = "123321123"
is_valid = True
```

## level 1: simple functions

Just a vanilla python function. Nothing notable about it unless you consider default
arguments something special (looking at you golang), perhaps even considered outdated
if you're heavily into modern python.

```python
def save_sample(data, barcode, is_valid=False):
    valid_str = "valid" if is_valid else "invalid"
    save_path = f"/save/dir/{barcode}_{valid_str}.npy"
    np.save(save_path, data)
    return save_path

save_sample(data, barcode, is_valid)
```

## level 2: type hints

Now we've added type hints. They look cool and make us think our code is more robust
or something, but they're ignored by the python interpreter, so basically only documentation
unless somebody other than you is running pyright or mypy on this code. I'm
personally a huge fan of type hints, but they're not at all enforced, nothing is
stopping you from writing lies in the type hints.

```python
def save_sample(data: np.ndarray, barcode: str, is_valid: bool = False) -> str:
    valid_str = "valid" if is_valid else "invalid"
    save_path = f"/save/dir/{barcode}_{valid_str}.npy"
    np.save(save_path, data)
    return save_path
```

## level 3: objects, kind of

Now you're thinking we should keep these separate variables together. This could
be a list, tuple or dictionary.

```python
sample = {"data": data, "barcode": barcode, "is_valid": True}
save_sample(**sample)
```


## level 4
### level 4a: namedtuples

I like [NamedTuples](https://docs.python.org/3/library/typing.html#typing.NamedTuple),
your function now accepts a single `Sample` object. Ok it's less flexible but I
feel it adds clarity. Perhaps slightly confusingly there are also named tuples
in [`collections.namedtuple`](https://docs.python.org/3/library/collections.html#collections.namedtuple)
though these lack type hints.

```python
from typing import NamedTuple

class Sample(NamedTuple):
    data: np.ndarray
    barcode: str
    is_valid: bool = False


def save_sample(sample: Sample) -> str:
    valid_str = "valid" if sample.is_valid else "invalid"
    save_path = f"/save/dir/{sample.barcode}_{valid_str}.npy"
    np.save(save_path, sample.data)
    return save_path


sample = Sample(data, barcode, is_valid)
save_sample(sample)
```

#### level 4b: dataclasses

[Dataclasses](https://docs.python.org/3/library/dataclasses.html) are useful. We've
made `Sample` immutable with `frozen=True`. Is immutability enterprisey? I'm not
sure, but it's a nice feature a lot of the time.

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Sample:
    data: np.ndarray
    barcode: str
    is_valid: bool = False


sample = Sample(data, barcode, is_valid)
save_sample(sample)
```

Dataclasses and NamedTuples can sometimes be used interchangeably by accessing their
fields with the dot-notation though a key difference is dataclasses are actually
*classes* so you can add methods to them.

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Sample:
    data: np.ndarray
    barcode: str
    is_valid: bool = False

    def save(self) -> str:
        valid_str = "valid" if self.is_valid else "invalid"
        save_path = f"/save/dir/{self.barcode}_{valid_str}.npy"
        np.save(save_path, self.data)
        return save_path

sample = Sample(data, barcode, is_valid)
save_sample(sample)
```

## level 5: classes

Classes, methods, object-orientated-programming (OOP) - the pinnacle of enterprise
code. Nobody can doubt you now, this codebase is destined to last at least a decade
and be hated by several people you never meet.

```python
class Sample:
    def __init__(self, data: np.ndarray, barcode: str, is_valid: bool = False):
        self.data = data
        self.barcode = barcode
        self.is_valid = is_valid
        self.save_path = None

    def save(self) -> None:
        valid_str = "valid" if self.is_valid else "invalid"
        save_path = f"/save/dir/{self.barcode}_{valid_str}.npy"
        np.save(save_path, self.data)
        self.save_path = save_path


sample = Sample(data, barcode, is_valid)
sample.save()
```

## level 6+

You can take OOP to an extreme (see the excellent [FizzBuzzEnterpriseEdition](https://github.com/EnterpriseQualityCoding/FizzBuzzEnterpriseEdition)),
this is very much satire and hopefully doesn't reflect the code you'll encounter
in the wild.


A further option is to split the barcode off into a separate type with its own
validators instead of a simple string. There are also more sensible ideas like
using [pydantic](https://docs.pydantic.dev/) much like NamedTuples with type
validation during runtime, you can add logging, writing esoteric Exception
classes (`SampleSaveError`) to catch and handle in specific ways and so on.


## conclusion

Personally I think around level 4 (namedtuples/dataclasses) is a sweet spot for
the type of work I do, which is making small tools for scientific computing.
I think there's more to explore here as "enterprise" code is generally associated
with OOP. I'm not entirely sure why, but it would be interesting to see what
would be considered "enterprisey" code written in a functional style.
