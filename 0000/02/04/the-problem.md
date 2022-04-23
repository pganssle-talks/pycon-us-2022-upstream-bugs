## A Bug in Someone Else's Code

```python
import pandas as pd

def f(x, a):
    return x.sum() + a

df = pd.DataFrame([1, 2])

print(df.agg(f, 0, 3))
```

<span class="fragment" data-fragment-index="0">
Running this fails with <tt>pandas == 1.1.3</tt>:
</span>

```bash
$ python pandas_example.py
Traceback (most recent call last):
  File ".../pandas/core/frame.py", line 7362, in aggregate
    result, how = self._aggregate(func, axis=axis, *args, **kwargs)
TypeError: _aggregate() got multiple values for argument 'axis'

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "pandas_example.py", line 8, in <module>
    print(df.agg(f, 0, 3))  # Raises TypeError
  File ".../pandas/core/frame.py", line 7368, in aggregate
    raise exc from err
TypeError: DataFrame constructor called with incompatible data and dtype:
           _aggregate() got multiple values for argument 'axis'
```
<!-- .element class="fragment" data-fragment-index="0" -->

Notes:

So, what do we mean by a bug in someone else's code? Here's an example that I recently encountered at work.

In this minimal code, we're expecting to aggregate the DataFrame `df` with the function f. The `.agg` function itself takes two arguments: the function to apply and the axis to apply it on, and then all other positional and keyword arguments are passed along to the function. So the idea is that this will go row by row in our dataframe and call the function, passing it a row and the value 3.

In the latest version of pandas, though, it raises an exception and complains about the `axis` parameter being passed twice. That's strange. Let's check the documentation and make sure we're using it right.

S: 1m 30s
T: 3:00

--

## A Bug in someone else's code

<img
     src="images/pandas-agg-docs.png"
     alt="The documentation for DataFrame.agg, demonstrating that it accepts *args."
     class="disappearing-fragment fragment fade-out"
     style="height:600px"
     data-fragment-index="0"
     />

<img
     src="images/pandas-agg-docs-proof.png"
     alt="The documentation for DataFrame.agg with the *args section boxed in red and the word Proof!!!1one in Comic Sans next to it."
     class="nospace-fragment fragment none"
     style="height:600px"
     data-fragment-index="0"
     />

Notes:

Ah, yes, looks like we are! Proof! This is not my bug, this is a bug in pandas, because, by their very own admission say that you can pass additional positional arguments to this function.

S: 30s
T: 3:30

--

## The Right Thing To Do™

- File an issue upstream<br/>
- Submit a patch to fix the issue upstream <!-- .element class="fragment" data-fragment-index="1" -->
- Wait for release <!-- .element class="fragment" data-fragment-index="2" -->
- Update your version <!-- .element class="fragment" data-fragment-index="3" -->

<img
    src="images/pandas-agg-issue.png"
    alt="Pandas issue #36948: Dataframe.agg no longer accepts positional arguments as of v1.1.0"
    class="disappearing-fragment fragment fade-out"
    data-fragment-index="1"
    />
<img
    src="images/pandas-agg-pr.png"
    alt="Pandas PR #36950: Allow positional arguments in DataFrame.agg"
    class="nospace-fragment disappearing-fragment fragment fade-in"
    data-fragment-index="1"
    />
<img
    src="images/pandas-whatsnew-114.png"
    alt="What's new in 1.1.4: Changelog including the DataFrame.agg change for 1.1.4, with no specified release date."
    class="nospace-fragment fragment fade-in"
    data-fragment-index="2" />

Notes:

What should we do now? Well, we should do the right thing, we should at least start with the right thing.

First we file an issue containing our minimal reproducer and the expected results.

You can also submit a patch to fix the issue upstream. In this case it was pretty straightforward so I submitted a PR and within a day or so it was actually merged. So far this is looking like an open source success story.

Then we just have to wait for a pandas release that includes the fix and update my version and the problem is solved!

S: 45s
T: 4:15

--

## What can go wrong?

- Production deadlines
- Long upstream release cycles
- Long deployment cycles in-house

<img src="images/demo-friday.png"
    alt = "A child at a tablet looking tired with a thought bubble that says, 'But the demo is on Friday!"
    class = "disappearing-fragment fragment fade-out"
    id = "splash"
    style="max-width: 800px"
    data-fragment-index="0"
    />
<img
    src="images/python-annual-release-cycle.png"
    alt="PEP 602: Annual Release Cycle for Python"
    class = "nospace-fragment fragment fade-in"
    data-fragment-index="0"
    />

Notes:

So what could go wrong?

Well, for one thing we could have production deadlines. Maybe this bugfix is critical and needs to be rolled out right away and we don't even have time for the patch to land.

There's also the possibility of long upstream release cycles. I'll note that the release cycle for Python was just *shortened* to 1 year, from 18 months. If you are waiting on a new feature introduced in Python, you may not want to wait a year for it to roll out.

And then there are also delays rolling out the changes in your production environment. Depending on how your production infrastructure is set up, it can take weeks or months to get new versions deployed. This is not at all uncommon in my experience; it can take a long time to do testing, QA and other release processes.

So, if you have a problem and you need it fixed right away, what do you do?

S: 1m
T: 5:15
