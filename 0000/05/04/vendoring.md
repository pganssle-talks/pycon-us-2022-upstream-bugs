# Vendoring

What is vendoring? <!-- .element: class="not-centered" style="width: 95%; font-weight: bold" -->

<div style="height: 600px" data-fragment-index="0" class="disappearing-fragment fade-out"/>
<img
    id="splash"
    src="images/webster-vendoring.png"
    class = "disappearing-fragment fragment"
    alt="Webster's dictionary fails to define vendoring"
    style="height: 600px"
    data-fragment-index="0"
/>

<img
    id="splash"
    src="images/webster-vendoring-coincidence.png"
    class = "nospace-fragment disappearing-fragment fragment"
    alt="The same Webster's dictionary entry, with an arrow pointing to the suggested word 'censoring' from some text. The text is red Comic sans and reads, 'Coincidence? Learn the truth at https/dictionary-lies.horse'"
    style="height: 600px"
    data-fragment-index="1"
/>

```
$ tree setuptools/_vendor/
setuptools/_vendor/
├── __init__.py
├── ordered_set.py
├── packaging
│   ├── __about__.py
│   ├── _compat.py
│   ├── __init__.py
│   ├── markers.py
│   ├── py.typed
│   ├── requirements.py
│   ├── specifiers.py
│   ├── _structures.py
│   ├── tags.py
│   ├── _typing.py
│   ├── utils.py
│   └── version.py
├── pyparsing.py
└── vendored.txt

1 directory, 16 files
```
<!-- .element class="fragment nospace-fragment" data-fragment-index="2" -->

<span class="fragment nospace-fragment" data-fragment-index="2">
<b>vendoring</b>, <em>n.</em>, including a copy of one or more dependencies in a project's source code.
</span>

Notes:

Webster's dictionary defines vendoring as, "The word you have entered isn't in the dictionary", but that's because the fat cats at the dictionary don't _want_ you to know that vendoring is actually the practice of including your dependencies directly in your project's source tree.

S: 30s
T: 19:30

--

# How to vendor a package
<br/>

1. Copy the source code into your project tree somewhere (e.g. under `myproject._vendored`).
2. Update references: `squalene` → `myproject._vendored.squalene`
3. Apply any patches to your local copy. <!-- .element class="fragment" data-fragment-index="0" -->

<br/>

## Advantages <!-- .element class="fragment" data-fragment-index="1" -->

<br/>

- No chance that your hack will break if the dependency is upgraded.<!-- .element class="fragment" data-fragment-index="1" -->
- Scoped to your package only — no modifying of globals.<!-- .element class="fragment" data-fragment-index="1" -->
- Allows two packages to use otherwise incompatible versions of a shared dependency.<!-- .element class="fragment" data-fragment-index="1" -->

Notes:

The way it works is that you copy the source code into your source code directory somewhere, usually under a submodule called `_vendored` or something like that, then you change all the references to the package in your code to refer to the version of the dependency that lives in your project. So if you vendor a package called `squalene`, you would change all your import statements to import `myproject._vendored.squalene` instead.

If you are vendoring a copy for the purposes of making local fixes rather than just vendoring an upcoming version of the code with your fixes in it, you then would apply whatever patches you need to your local copy.

The advantages of this are that it's fairly self-contained. Your dependency version is pinned and won't break out from under you, and the fixes are scoped to your package only. This is also useful for breaking dependency resolution deadlocks — if you need version 2 of a package and another of your dependencies needs version 1, you can vendor version 2 while you wait for your dependency to be made compatible with the latest version.

S: 1m45s
T: 21:15

--

<!-- .slide: class="not-centered" -->
# Cautions

```python
>>> import squalene
>>> from my project._vendored import squalene as vendored_squalene
>>> squalene.magnitude.Magnitude(1) < squalene.magnitude.Magnitude(2)
True
>>> vendored_squalene.magnitude.Magnitude(1) < vendored_squalene.magnitude.Magnitude(2)
True
>>> squalene.magnitude.Magnitude(1) < vendored_squalene.magnitude.Magnitude(2)
...
TypeError: '<' not supported between instances of 'squalene.magnitude.Magnitude'
           and 'myproject._vendored.squalene.magnitude.Magnitude'
>>> squalene.magnitude.Magnitude is myproject._vendored.squalene.magnitude.Magnitude
False

```
<br/>

Reference to the package's top-level name within the vendored package will still hit the global package:

```python
# Contents of _vendored/squalene/world_destroyer.py
from .magnitude import WORLD_DESTROYING_MAGNITUDE
from squalene.magnitude import Magnitude

def destroy_world(world, start_magnitude=None):
    magnitude = start_magnitude or Magnitude(3)
    while magnitude < WORLD_DESTROYING_MAGNITUDE:
        magnitude.increase(1)
```
<br/>

Solving this may require one of:

- Extensive modifications to the source.
- Import hooks.
- Messing around with `sys.path`.

Notes:

There are two very common issues that arise with vendoring that are worth keeping in mind, and they are closely related.

The first is that when there are two or more copies of the vendored module in your application, you will start to see incompatibilities arise that you might not have expected. For example, most classes only define comparisons between instances of the same class, but since the vendored version of the module re-defines all the classes in the package, the instances provided by the vendored version cannot be compared to instances of the class from the global version.

This comes up in the context of the second problem, which is that references to the package's top level name within the vendored package still hit the global package, but relative references hit the vendored version. So in this example, we have a function that uses mixed references to a vendored class and to the equivalent class in the globally installed module.

In order to solve the immediate compatibility problem, you either have to make extensive modifications to the vendored copy to update all its references or you have to use some dark wizardry to get the namespaces to resolve correctly.

S: 1m30s
T: 22:45

--

# Downsides

- Hard to implement.
- Hard to maintain.
- Has a tendency to be leaky in one way or another (import system wasn't really built with this in mind).
- Doesn't work well for any dependency that is part of the public API.

Notes:

So obviously we're already seeing some downsides here. It's hard to implement and it can be hard to maintain, not least of which because it complicates your build and packaging. It also has a tendency to be leaky in one way or another, since the import system wasn't really designed to support multiple versions of a package in the same runtime.

And, as you can imagine from the problems of cross-version comparisons, this doesn't work well for any dependency that is exposed in your public API. You can't be returning references to your own vendored version of a package if other people are going to be making use of it.

S: 45s
T: 23:30

--

# Real-life examples

- `pip` and `setuptools` vendor all their dependencies to avoid bootstrapping issues (no patching).
    - Manipulates namespace resolution to get name resolution to work.<br/><br/>

- `invoke` vendors all its dependencies (including separate Python 2 and 3 trees for `pyyaml`)
    - No dependencies have been updated in > 5 years <span class="emoji">☹</span><br/><br/>

- This talk!
    - `reveal.js` and `jekyll-revealjs` are vendored into the source.
    - <!-- .element class="fragment" data-fragment-index="0" --> <tt>jekyll-reveal</tt> even carries a patch! <span class="emoji">🤦</span><br/>
    <img src ="images/jekyll-reveal-prs.jpg"
         alt="A screenshot of the jekyll-reveal repository, showing two PRs open since November 2021"
         class="fragment" data-fragment-index="0"
         style="display:block; margin-left: auto; margin-right: auto;"
         />

Notes:

This talk's repo carries at least one patch in `jekyll-revealjs` that I haven't had time to try and upstream. I have also removed some patches that were accepted upstream.

`pip` and `setuptools` both have policies that fixes must be done upstream, but `pip` does do things like only partially vendor `setuptools`. Both use spooky namespace manipulation to get the name resolution to work — and their solutions are not compatible with one another!

S: 45s
T: 24:15
