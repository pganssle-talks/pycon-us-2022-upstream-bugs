# Monkey Patching

<img
    id="splash"
    src="external-images/monkey-mac.jpg"
    alt="A knitted monkey working at a Mac"
    style="max-height:800px"
/>

Notes:

So, let's get into the next strategy, and this is where we start getting into properly dangerous and hacky territory. This strategy is Monkey Patching.

S: 10s
T: 12:40

--

# Intro to Monkey Patching

```python
import random

flabs = __builtin__.abs  # Store the original method

def six_pack(x):
    """Nothing is truly absolute. Embrace ambiguity."""
    abs_value = flabs(x)
    if random.random() < 0.8:
        return abs_value
    else:
        return -abs_value

__builtin__.abs = six_pack  # Use our new method instead of `abs()`

print([abs(3) for _ in range(10)])
# [3, 3, 3, 3, -3, -3, 3, -3, -3, 3]
```
<br/>
<br/>

Affects anyone using the namespace:

```python
>>> from fractions import Fraction
>>> set(map(hash, [Fraction(110, 3) for _ in range(100)]))
{768614336404564687, 1537228672809129264}
```

Notes:

So what is monkey patching? The idea behind monkey patching is that most modules and classes in Python are mutable and live in a global namespace, so you can actually just dynamically modify code that you want to patch at runtime.

Here's an example where we decide that the `abs` function is a bit stodgy, and so we're going to make it embrace chaos by making it return a negative number 20% of the time. We accomplish this by defining our own version of `abs` and then assigning dunder-builtins dot abs to the function we want to patch in, at which point calling `abs` sometimes returns negative numbers.

This works locally but also globally for anyone using the namespace or module we've patched. Evidently the `hash` function of the `Fraction` class uses `abs` somewhere, so you can see that it's being affected by our patched `abs` function.

S: 1m10s
T: 13:50

--

# How does this help us?
<br/>

```python
from functools import wraps
import pandas as pd


if _has_pandas_bug():
    _df_agg = pd.DataFrame.agg
    @wraps(pd.DataFrame.agg)
    def dataframe_agg(df, func, axis=0, *args, **kwargs):
        if args:
            def bound_func(x, **kwargs):
                return func(x, *args, **kwargs)
            func = bound_func
        return _df_agg(df, func, axis=axis, **kwargs)

    pd.DataFrame.agg = dataframe_agg
```
<br/>
<br/>

- Fixes the issue globally and transparently.
- May fix the issue in *other* code you don't control.
<br/>

Notes:

So, how does this help us? Obviously it's cool that we can break the hash of fraction objects, but how does that help us fix bugs?

Well, looking back at our `pandas` example, we could implement our wrapper instead like this — detect that we are affected by the bug, and if so modify `DataFrame` so that the `agg` method calls our function, rather than changing all the call sites.

This also can fix the issue globally and transparently. Rather than having to change all the code in your application to use wrapper functions, you can just directly fix the method that's causing you problems, and then remove the monkey patch when the bug is fixed. This will also fix the bug for everyone in your current runtime as well, including other code you don't control.

S: 1m
T: 14:50

--

# Why is this a terrible idea?

<img
    id="splash"
    src="external-images/bike-airplane.jpg"
    alt="A bicyclist tethered to a propeller plane."
    style="height:550px"
/>

- Action at a distance.
- No one else is expecting you to do this.
- Often tightly coupled to implementation details.

Notes:

S: 1m
T: 15:50

--

# Scoping the patch correctly

```python
# Contents of pimodule.py
import math

def pi_over_2() -> float:
    return math.pi / 2
```
<br/>

```python
# Contents of pimodule2.py
from math import pi

def pi_over_2() -> float:
    return pi / 2
```
<br/>

```python
import math
import pimodule
import pimodule2

math.pi = 3  # Pi value is too high imo

print(pimodule.pi_over_2())  # 1.5
print(pimodule2.pi_over_2())  # 1.5707963267948966
```
<!-- .element class="disappearing-fragment fade-out fragment" data-fragment-index="0" -->

```python
import math
import pimodule
import pimodule2

math.pi = 3  # Pi value is too high imo
pimodule2.pi = 3

print(pimodule.pi_over_2())  # 1.5
print(pimodule2.pi_over_2())  # 1.5
```
<!-- .element class="nospace-fragment fade-in fragment" data-fragment-index="0" -->

Mind your namespaces!

Notes:

S: 1m40s
T: 17:30

--

# Scope as tightly as possible
<!-- .slide: class="not-centered" -->
<br/>
If you only need the patch to apply to your code, use a context manager:

```python
from contextlib import contextmanager

@contextlib.contextmanager
def bugfix_patch():
    if _needs_patch(): # Don't forget opportunistic upgrades!
        _do_monkey_patch()
        yield
        _undo_monkey_patch()
    else:
        yield


# Use as a context manager
def f():
    unaffected_code()

    with bugfix_patch():
        affected_code()


# Or as a decorator
@bugfix_patch
def affected_function():
    ...
```

Notes:

One tool that might can be useful for keeping the scope of your patches tight is that if you only want them applied while your specific code is running, you can use `contextlib.contextmanager` to easily create a context manager. You write a little function that applies and then removes the monkey patch, then decorate it with `contextlib.contextmanager` and the function will return an object that can either be used as a context manager or a decorator.

S: 30s
T: 18:00

--

# Real-life examples

- `setuptools` extensively patches `distutils` on import 

```python
def patch_all():
    # we can't patch distutils.cmd, alas
    distutils.core.Command = setuptools.Command

    has_issue_12885 = sys.version_info <= (3, 5, 3)

    if has_issue_12885:
        # fix findall bug in distutils (http://bugs.python.org/issue12885)
        distutils.filelist.findall = setuptools.findall

    needs_warehouse = (
        sys.version_info < (2, 7, 13)
        or
        (3, 4) < sys.version_info < (3, 4, 6)
        or
        (3, 5) < sys.version_info <= (3, 5, 3)
    )

    if needs_warehouse:
        warehouse = 'https://upload.pypi.org/legacy/'
        distutils.config.PyPIRCCommand.DEFAULT_REPOSITORY = warehouse
    ...
```

- ...and `pip` invokes the monkey patch even if you don't import `setuptools`!
<br/>
<br/>

_**Take Heed:** This was expedient at the time, but `setuptools` has been working to unravel this for years._

Notes:

S: 1m
T: 19:00
