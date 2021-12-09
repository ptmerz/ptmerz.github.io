---
title: "Minimal Mistakes and MathJax"
tags: ["Minimal Mistakes", "Jekyll", "MathJax"]
category: "Blogging platform"
---

After setting up this website with the beautiful
[Minimal Mistakes theme](https://github.com/mmistakes/minimal-mistakes), I noticed that it
wasn't supporting typesetting math equations. Well, it can't be too hard to get this to work,
right? Right...

Note that I am using Minimal Mistakes as a remote theme, and I would really like to keep it
that way if possible. Using a remote theme reduces the bloat of my repository, and means I'm
always getting the latest version. So any solution I'm finding should, if possible, not break
updates!

After a bit of digging, it decided I'd try to set up the
[MathJax JavaScript library](https://www.mathjax.org). Unfortunately, none of the suggestions
([1](https://github.com/mmistakes/minimal-mistakes/issues/735),
[2](https://sort-care.github.io/Latex-on-Blog/)) I found seemed to work properly - the
equations would either not render at all or render with weird typesetting errors. Finally,
I found that Academic Pages, a fork of Minimal Mistakes, has working MathJax support which
is enabled by a few lines which are added to the header of each file. I copied the
last few lines from
[here](https://github.com/academicpages/academicpages.github.io/blob/master/_includes/head/custom.html)
and put them into a newly created `_includes/head/custom.html` in my repo. This is a file
which gets added automatically to the header of each Markdown file when it is converted to
HTML. This file exists in the upstream Minimal Mistakes repo, but it is empty. If Jekyll finds a
file both in the remote theme and in the local repo, it will prefer the local file. So as long
as future versions of Minimal Mistakes will not start using that file, using it locally won't
cause any problems when updating.

Here's how my `_includes/head/custom.html` looks now:
```html
<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
    tex2jax: {
      inlineMath: [ ['$','$'], ["\\(","\\)"] ],
      processEscapes: true
    }
  });
</script>
<script src='https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/latest.js?config=TeX-MML-AM_CHTML' async></script>
```
Note that I left out one line from the Academic Pages header, which reads
```html
<script type="text/x-mathjax-config"> MathJax.Hub.Config({ TeX: { equationNumbers: { autoNumber: "all" } } }); </script>
```
This line would enable equation numbering by default. I don't want equation numbers as a default, so
I left this out. I should also play around a bit to see if different MathJax version would work, but
for now I'm just happy I can typeset nice equations inline, like $a^2 + b^2 = c^2$, and in display
mode

$$ i \hbar \frac{\partial}{\partial t}\Psi(\mathbf{r},t) = \hat H \Psi(\mathbf{r},t) $$
