# Maintaining a Fork

<img
    id="splash"
    src="images/fork-path.jpg"
    alt="A fork in a path in the woods"
    style="max-height: 750px"
/>

Notes:

The last option should be deploying and maintaining a patched version of the library in your production distribution. The difference with vendoring is that this is global; rather than using an unmodified version of the upstream, you put your patched version into your production pipeline. Whether that's some linux-like distribution system, a monorepo, your own mirror of PyPI, etc.

Unfortunately, in my experience, people often take this as their _first_ option, because just patching your local version is relatively easy to do, and the cost only comes later.

S: 1m
T: 25:15

--

# Accomplishing this: distros / monorepos

- Mostly accomplished with `.patch` files.
- These can be managed by `quilt`, see: https://raphaelhertzog.com/go/quilt
<br/>

```diff
From f9c06582c58e01deab10c6fcc081d4d7cb0f1507 Mon Sep 17 00:00:00 2001
From: Barry Warsaw <barry@python.org>
Date: Fri, 18 Nov 2016 17:07:47 -0500
Subject: Set --disable-pip-version-check=True by default.

Patch-Name: disable-pip-version-check.patch
---
 pip/cmdoptions.py | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)

diff --git a/pip/cmdoptions.py b/pip/cmdoptions.py
index f71488c..f75c093 100644
--- a/pip/cmdoptions.py
\+++ b/pip/cmdoptions.py
@@ -525,7 +525,7 @@ disable_pip_version_check = partial(
     "--disable-pip-version-check",
     dest="disable_pip_version_check",
     action="store_true",
-    default=False,
\+    default=True,
     help="Don't periodically check PyPI to determine whether a new version "
          "of pip is available for download. Implied with --no-index.")
```
<br/>
<div class="fragment" data-fragment-index="0">
<ul><li>Can also accomplish this with <tt>sed</tt> or other scripts in simple cases:</li></ul>
</div>

```bash
# Excerpt from an Arch Linux PKGBUILD
prepare() {
  cd $_pkgname-$pkgver

  sed -i 's|../../vendor/http-parser/http_parser.h|/usr/include/http_parser.h|' $_pkgname/parser/cparser.pxd
}
```
<!-- .element class="fragment" data-fragment-index="0" -->

Notes:

So if you want to accomplish this, mostly I've seen this done using patch files, where you store the original code as a tarball and a series of diffs for how it needs to change. This way you have a clean record of everything you've changed, and to the extent that they still cleanly apply, you can also easily re-apply the changes when you upgrade to a new version.

A lot of people use this software called `quilt` for generating and managing those patches. I don't really have time to go into it, but this link does a good job of explaining how it works and the basics of how to use it better than I could in 30 seconds anyway.

In some simple cases — and obviously we want to keep our patches as simple as possible — you can even use something like `sed` to just do a search-and-replace at build time. This seems to be a fairly common idiom in Arch Linux packages, where they have a strong policy against extensive patching, and tend to have much simpler patches.

S: 1m15s
T: 26:30

--

# Downsides

- You are maintaining a fork that upstream doesn't know about.<br/><br/>
- Updating all your patches adds friction to the upgrade process.<br/><br/>
- No guarantees of compatibility.<br/><br/>

Notes:

So with that we're back to talking about how this is a bad idea. While this is method is appealing because it's pretty easy to implement and there are tools available to help you do it, the big problems come when you have to carry these patches over a long period of time.

You end up maintaining a fork that upstream doesn't know about, so they may move in another direction that is incompatible with your patch. Each patch adds some friction to your upgrade process, as you have to reconcile your changes with upstream, and you may find that other people in your organization have come to rely on changes you made that are incompatible with newer versions of the software.

S: 1m
T: 27:30

--

# Real-life Examples

- Nearly every Linux distro, either heavily (e.g. Debian) or lightly (e.g. Arch).<br/><br/>
- `conda` and `conda-forge` packages.<br/><br/>
- Most big companies.<br/><br/>

## <!-- .element class="fragment" data-fragment-index="0" --> Success story

<img
    src="images/attrs-zope.png"
    alt="A Github PR making zope a semi-optional test dependency for attrs"
    class="fragment"
    data-fragment-index="0"
    />

Notes:

I hardly have to mention real life examples, because this sort of thing is pretty commonly done. Various organizations are more or less willing to maintain their own flavor of open source software, and it's very common to have something like this in big companies.

At work I've been trying to keep `attrs` up-to-date, but it carried a few patches that were a bit annoying to fix every time. Nothing major, but I was able to get most of them removed except one where we disabled all `zope`-related tests because we don't use `zope` and packaging `zope` just to test that `attrs` works with `zope` would be ridiculous. I made a PR adding a mechanism for disabling `zope` directly in `attrs`'s test suite and to my surprise Hynek accepted it, so I was able to remove my last `attrs` patch!

S: 1m30s
T: 29:00
