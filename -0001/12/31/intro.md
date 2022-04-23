<h1 style="font-size: 3em">What to Do When the Bug is in Someone Else's Code</h1>
<br/>
<br/>
<br/>
<span style="font-size: 2.5em">
Paul Ganssle
</span>
<br/>
<br/>
<img src="images/pganssle-logos.svg" height="40px" alt="@pganssle">
<br/>
<br/>
<span style="font-size: 1em;"><em>This talk on Github:
<a href="https://github.com/pganssle-talks/pycon-us-2022-upstream-bugs">pganssle-talks/pycon-us-2022-upstream-bugs</a></em>
</span><br/>
<a rel="license" href="https://creativecommons.org/publicdomain/zero/1.0/">
    <img src="external-images/logos/cc-zero.svg" height="45px">
</a>
<br/>

Notes:

My name is Paul Ganssle, I'm a software engineer at Google and also a contributor to many open source projects. Among other things, I'm a core developer of Python and a maintainer of dateutil and setuptools.

As someone who primarily develops libraries, I'm obviously a fan of shared code, but I also recognize that there are risks to taking on dependencies, third party or not. One of those risks is the topic of today's talk: when something you depend on has a bug or any other incompatibility, it's not as easy to fix as it would be to fix a bug in your own code.

T: 1m

--

<h1><span class="emoji">⚠️</span> Warning <span class="emoji">⚠️</span></h1>

<img id="splash"
     src="images/nails-scaled.jpg"
     alt="A nail partially driven into a piece of wood"
     style="max-height: 300px; margin-bottom:0px; margin-top:0px"
     /><br/><img id="splash"
     src="images/hammerlike-scaled.jpg"
     alt="A series of decreasingly hammer-like objects: A hammer, an axe, a bottle of maple syrup and a can of butane fuel."
     style="max-height: 550px; margin-top: 0px; margin-bottom: 0px"
     />


Notes:

Before we get started, though, I want to give you a warning.

This talk deals with a number of strategies for handling bugs in your dependencies, but it's organized as a series of strategic retreats away from what I would consider the "right thing to do".

While I've used each of these strategies in the past, they're mostly a collection of "least bad" options, and each strategy is worse than the last.

We will of course start with the right tool for the job, but it'll get increasingly hacky and dangerous as we move on.

S: 30s
T: 1:30
