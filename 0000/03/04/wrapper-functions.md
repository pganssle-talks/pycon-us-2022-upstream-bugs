# One-off Workarounds

```python
def f(x, a):
    return x.sum() + a

df = pd.DataFrame([[1, 2], [3, 4]])

# Passing `a` by position doesn't work in pandas >=1.1.0,<1.1.4
# print(df.agg(f, 0, 3))
print(df.agg(f, 0, a=3))
```
<br/>
<br/>
<h3 style="text-align: left">Reasonable if:</h3>

- You only hit the bug in one place.
- The workaround is very simple
- You are indifferent between the bug-triggering and workaround code.

Notes:

The first step is to try to introduce a workaround. In this case, we can work around the bug pretty trivially here by passing the argument by keyword rather than position, so I add a little TODO comment indicating why I chose to write the code this way, and move on.

This sort of thing works just fine when you only hit the bug in one place and the workaround is very simple. It's also very reasonable if you are indifferent between the version that triggers the code and the version that doesn't — in this case I don't have any particular preference for positional arguments, so it's not a big deal to switch this permanently.

S: 1m15s
T: 6:30

--

# Wrapper functions

```python
def dataframe_agg(df, func, axis=0, *args, **kwargs):
    """Wrapper function for DataFrame.agg.

    Passing positional arguments to ``func`` via ``DataFrame.agg`` doesn't work
    in ``pandas >=1.1.0,<1.1.4``. This wrapper function fixes that bug in
    affected versions and works normally otherwise.
    """

    if args:
        def func_with_bound_posargs(arg0, **kwargs):
            return func(arg0, *args, **kwargs)

        func = func_with_bound_posargs

    return df.agg(func, axis=axis, **kwargs)

print(dataframe_agg(df, f, 1, 3))
```
<br/>

- Encapsulates complicated workaround logic.
- Provides an easy target for later removal.

Notes:

If the workaround is a little complicated or you hit it a bunch of places, it may make sense to use a wrapper function. Going back to our pandas example, I've written a little wrapper function that just binds the positional arguments to the function before it's passed to the `agg` function, so that no positional arguments ever have to be passed through `agg`.

I wrap that up in a function called `dataframe_agg` and then I can switch over all my afflicted call sites over to using the wrapper where the bug is fixed. This is nice because it encapsulates the workaround logic, and also because when it's time to actually fix the bug it's a fairly simple search and replace to restore the original code.

S: 1m30s
T: 8:00

--

# Wrapper functions: Opportunistic upgrading

<img
    id="splash"
    src="images/tech-debt-hilarious.png"
    alt="Book cover: I'll Clean Up That Technical Debt Later... And Other Hilarious Jokes You Can Tell Yourself; Special 'TODO Comments' Edition"
    style="max-height:800px"
/>

Notes:

Of course, you can say that you'll eventually go remove all the hacks, but just in case, you can try to minimize the scope of your hack by building in an expiration; if possible, you can detect at runtime whether the hack is needed, and apply it if and only if you do! This is something I'm calling opportunistic upgrading, and it gets more and more useful as the fixes get more and more hacky.

S: 30s
T: 8:30

--

# Opportunistic upgrading
<br/><br/>

```python
def dataframe_agg(df, func, axis=0, *args, **kwargs):
    """Wrapper function for DataFrame.agg.

    Passing positional arguments to ``func`` via ``DataFrame.agg`` doesn't work
    in ``pandas >=1.1.0,<1.1.4``. This wrapper function fixes that bug in
    affected versions and works normally otherwise.
    """
    if args and _has_pandas_bug():
        def func_with_bound_posargs(arg0, **kwargs):
            return func(arg0, *args, **kwargs)

        func = func_with_bound_posargs
    return df.agg(func, axis, *args, **kwargs)
```
<br/>

Hack is only triggered if you otherwise would have triggered the bug!

Notes:

The way it works is that you do something that detects whether or not the environment that is currently running is affected by the bug, and skip the workaround if you're not. So, looking back at our original wrapper function, we just have to add this `_has_pandas_bug()` function, and the hack is only executed if the bug otherwise would have been triggered!

S: 45s
T: 9:15

--

# Opportunistic upgrading
<br/>

## By feature detection
```python
import functools

import pandas as pd

@functools.lru_cache(1)  # Need to execute this at most once
def _has_pandas_bug():
    def f(x, a):
        return 1

    try:
        pd.DataFrame([1]).agg(f, 0, 1)
    except TypeError:
        return True

    return False
```
<br/>

## By version checking
<!-- .element class="fragment" data-fragment-index="1" -->

```python
import functools

@functools.lru_cache(1)  # Need to execute this at most once
def _has_pandas_bug():
    from importlib import metadata  # Python 3.8+, backport at importlib_metadata
    from packaging import Version  # PyPI package

    return Version("1.1.0") <= metadata.version("pandas") < Version("1.1.4")
```
<!-- .element class="fragment" data-fragment-index="1" -->

Notes:

Pros for feature detection:
- Doesn't require knowledge of exactly which versions are affected.
- Accurate version may not be available at runtime in all situations.
- The bug may be simple to check for, but difficult to describe in terms of versions and platforms.

Pros for version-based:
- Works even when the bug is hard to detect, like if it's expensive to realize you've triggered the bug: e.g. a memory leak, or something that triggers a segfault.
- Relatively simple to implement.

S: 1m45s
T: 11:00

--

# Opportunistic upgrading at import time
<br/>

```python
if _has_pandas_bug():
    def dataframe_agg(df, func, axis=0, *args, **kwargs):
        """Wrapper function for DataFrame.agg.

        Passing positional arguments to ``func`` via ``DataFrame.agg`` doesn't work
        in ``pandas >=1.1.0,<1.1.4``. This wrapper function fixes that bug in
        affected versions and works normally otherwise.
        """

        if args:
            def func_with_bound_posargs(arg0, **kwargs):
                return func(arg0, *args, **kwargs)

            func = func_with_bound_posargs

        return df.agg(func, axis=axis, **kwargs)
else:
    dataframe_agg = pd.DataFrame.agg

print(dataframe_agg(df, f, 1, 3))
```

Notes:

You can also take this one step further and resolve whether or not you need a wrapper method at all at import time by defining the function conditionally.

This can save you some overhead and really minimize the scope of your function, and I would tend to do it any time that checking for the existence of the bug is cheap and when it is ergonomic to do so.

S: 1m
T: 12:00

--

# Real-life Examples

1. Feature backports
    - `importlib_resources`
    - Most things in the `backports` namespace.

2. `six`: Pretty much all wrapper functions to write code that works with Python 2 and 3.

<img src="images/six-top-10.png"
     alt="An image from pypistats.org showing Most Downloaded PyPI Packages. urllib3 is first with 3.2M/day and six is second with 2.9M/day."
     class = "nospace-fragment disappearing-fragment fragment fade-out"
     data-fragment-index="0"
     style="display:block; margin-left: auto; margin-right: auto;"
/>
<img src="images/six-top-10-now.png"
     alt="An image from pypistats.org showing Most Downloaded PyPI packages. six is now in 9th place"
     class="nospace-fragment fragment fade-in"
     data-fragment-index="0"
     style="display:block; margin-left: auto; margin-right: auto;"
     />

3. [`pytz-deprecation-shim`](https://pytz-deprecation-shim.readthedocs.io/en/latest/)
    - Wrapper classes that mimic `pytz`'s interface
    - Uses `zoneinfo` and `dateutil` under the hood
    - No `pytz` dependency!
    - For helping to migrate off `pytz`.
    <br/>
    <br/>


Notes:

Only mention `six`, leave the other two as bonus content.

S: 30s
T: 12:30
