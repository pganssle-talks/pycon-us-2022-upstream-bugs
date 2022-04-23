# Strategy Recap
<style>
.pro-header {
    background-color: #4cc649;
    padding-bottom:5px !important;
}

.con-header {
    background-color: #f93232;
    padding-bottom: 5px !important;
}

div.procon-container {
    display: flex;
    width: 90%;
    margin-left: auto;
    margin-right: auto;
    margin-bottom: 20px;

}

div.pro-or-con-div {
    width: 50%
}
</style>

## Patching upstream <span class="emoji">💖</span>

<div class="procon-container">
<div class="pro-or-con-div">

<h4 class="pro-header">Pros:</h4>
<ul>
<li> You fix the bug for everyone</li>
<li> Nothing to maintain afterwards (for you...)</li>
<li> Improves your relationship with the maintainers of software you use (hopefully)</li>
</ul>
</div>
<div class="pro-or-con-div">
<h4 class="con-header">Cons:</h4>
<ul>
    <li>Delays!</li>
    <li>You have to convince someone to accept your patch (or fix the bug)</li>
</ul>
</div>
</div>

<br/><br/>

## Wrapper functions <span class="emoji">🆗</span>

<div class="procon-container">
<div class="pro-or-con-div">
<h4 class="pro-header">Pros:</h4>
<ul>
    <li>Helps maintain cross-version compatibility</li>
    <li>Easy to remove when the need is done</li>
    <li>Can opportunistically upgrade</li>
    <li>Can roll out immediately</li>
</ul>
</div>
<div class="pro-or-con-div">
<h4 class="con-header">Cons:</h4>
<ul>
    <li>Only works when it's possible to work around the bug.</li>
    <li>Only works for the code you are currently writing.</li>
</ul>
</div>

--

## Monkey patching <span class="emoji">🙈</span>

<div class="procon-container">
<div class="pro-or-con-div">
<h4 class="pro-header">Pros:</h4>
<ul>
    <li>Make targeted global fixes.</li>
    <li>Doesn't complicate packaging or deployment.</li>
</ul>
</div>
<div class="pro-or-con-div">
<h4 class="con-header">Cons:</h4>
<ul>
    <li>Hard to reason about.</li>
    <li>Not likely to be compatible across versions.</li>
    <li>Can cause compatibility problems with other users of the same library.</li>
</ul>
</div>
</div>

## Vendoring <span class="emoji">☣️</span>

<div class="procon-container">
<div class="pro-or-con-div">
<h4 class="pro-header">Pros:</h4>
<ul>
    <li>Can unblock dependency resolution issues.</li>
    <li>Isolates any changes from the wider system.</li>
</ul>
</div>

<div class="pro-or-con-div">
<h4 class="con-header">Cons:</h4>
<ul>
    <li>Complicated to implement right.</li>
    <li>Doesn't work well when the vendored package is part of your public interface.</li>
    <li>Tooling support is very weak.</li>
</ul>
</div>
</div>

## Maintaining a fork <span class="emoji">☢️</span>

<div class="procon-container">
<div class="pro-or-con-div">
<h4 class="pro-header">Pros:</h4>
<ul>
    <li>Relatively easy to implement in some systems.</li>
    <li>Tools exist for this.</li>
</ul>
</div>

<div class="pro-or-con-div">
<h4 class="con-header">Cons:</h4>
<ul>
    <li>Upstream doesn't know about your fork!</li>
    <li>Adds friction with upgrades.</li>
    <li>Compatibility degrades over time.</li>
</ul>
</div>
</div>
