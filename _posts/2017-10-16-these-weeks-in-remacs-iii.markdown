--- 
layout: post
title: "These Weeks in Remacs III"
tags:
 - emacs
---

Time for another Remacs update: lots of contributions, a wide range of
features, and even a logo!

## Contributing

Since the [last update](/blog/2017/07/15/these-weeks-in-remacs-ii/),
we've seen contributions from lots of new people. We've added
@brotzeit and @shanavas786, bringing us to seven wonderful people who
can approve your PRs.

Speaking of PRs, we've merged an amazing 64 pull requests since the
last update!

If you're looking for a good feature for your first contribution,
@brotzeit has been regularly adding
[new suggestions under the 'good first issue' label](https://github.com/Wilfred/remacs/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22).

## Features

Many Emacs features have now been ported to Rust, with new Rust APIs
for accessing elisp datastructures.

Here's an overview of the features that have landed.

**Arithmetic**:
[arithmetic](https://github.com/Wilfred/remacs/pull/311/files),
[floating point](https://github.com/Wilfred/remacs/pull/234/files), 
[random number generation](https://github.com/Wilfred/remacs/pull/235/files) (using
a Rust RNG!), and
[comparisons](https://github.com/Wilfred/remacs/pull/226/files).

**Symbols**: [symbol properties](https://github.com/Wilfred/remacs/pull/235/files), 
[interning](https://github.com/Wilfred/remacs/pull/255),
[obarrays](https://github.com/Wilfred/remacs/pull/267)
[unbinding](https://github.com/Wilfred/remacs/pull/308/files),
[keywords](https://github.com/Wilfred/remacs/pull/334/files) and
[indirect symbols](https://github.com/Wilfred/remacs/pull/357/files).

**Checksums**: [MD5sum](https://github.com/Wilfred/remacs/pull/260/files)
(using a Rust MD5 crate!).

**Windows**: [liveness check](https://github.com/Wilfred/remacs/pull/258/files),
[type check](https://github.com/Wilfred/remacs/pull/240/files),
[overlays and minibuffer](https://github.com/Wilfred/remacs/pull/298/files),
[minibuffer check](https://github.com/Wilfred/remacs/pull/319/files)
[positions](https://github.com/Wilfred/remacs/pull/317/files) and
[margins](https://github.com/Wilfred/remacs/pull/351/files).

**Processes**: [accessing](https://github.com/Wilfred/remacs/pull/256/files),
[type check](https://github.com/Wilfred/remacs/pull/265/files),
[data structures](https://github.com/Wilfred/remacs/pull/325/files) and
[names](https://github.com/Wilfred/remacs/pull/341).

**Buffers**: [for the current thread](https://github.com/Wilfred/remacs/pull/253/files),
[accessing](https://github.com/Wilfred/remacs/pull/242/files),
[file names](https://github.com/Wilfred/remacs/pull/279/files),
[size](https://github.com/Wilfred/remacs/pull/281/files) and
[modification](https://github.com/Wilfred/remacs/pull/288/files).

**Point**: [bobp](https://github.com/Wilfred/remacs/pull/291/files),
[bolp, eolp](https://github.com/Wilfred/remacs/pull/293/files),
[markers](https://github.com/Wilfred/remacs/pull/297/files),
[point-min, point-max](https://github.com/Wilfred/remacs/pull/302/files)
[forward-point](https://github.com/Wilfred/remacs/pull/321/files) and
[goto-char](https://github.com/Wilfred/remacs/pull/352/files).

**Hash tables**: [copying](https://github.com/Wilfred/remacs/pull/251/files) and [accessing](https://github.com/Wilfred/remacs/pull/349).

**Characters**:
[multibyte conversions](https://github.com/Wilfred/remacs/pull/236/files),
[character tables](https://github.com/Wilfred/remacs/pull/277/files),
[category tables](https://github.com/Wilfred/remacs/pull/282/files)

**Fonts**: [type checks](https://github.com/Wilfred/remacs/pull/248).

**Miscellaneous**:
[prefix arguments](https://github.com/Wilfred/remacs/pull/252/files) and
[identity](https://github.com/Wilfred/remacs/pull/329/files).

We're also periodically pulling GNU Emacs features into Remacs, so all
the features available GNU Emacs trunk are included in Remacs.

## Idiomatic Rust in Remacs

Remacs has gradually developed a set of conventions for elisp data
types. For each type Foo, we define a `LispObject::as_foo`,
`LispObject::as_foo_or_error` and a `FooRef` when you know your elisp
datatype is actually a Foo.

For example, here's how `overlay-start` was implemented in C:

{% highlight c %}
DEFUN ("overlay-start", Foverlay_start, Soverlay_start, 1, 1, 0,
       doc: /* Return the position at which OVERLAY starts.  */)
  (Lisp_Object overlay)
{
  CHECK_OVERLAY (overlay);

  return (Fmarker_position (OVERLAY_START (overlay)));
}
{% endhighlight %}

The C codebase makes heavy use of macros for checking types
(`CHECK_OVERLAY`) and for accessing struct attributes
(`OVERLAY_START`).

Here's the Rust equivalent:

{% highlight rust %}
/// Return the position at which OVERLAY starts.
#[lisp_fn]
fn overlay_start(overlay: LispObject) -> LispObject {
    let marker = overlay.as_overlay_or_error().start();
    marker_position(marker)
}
{% endhighlight %}

We use procedural macros to simplify defining an elisp primitive
function, and type checking is much more explicit.

(This example is
from [PR #298](https://github.com/Wilfred/remacs/pull/298/files).)

Other exciting Rusty features
include
[variadic macros to replace `call1`, `call2` in C with just `call!` in Rust](https://github.com/Wilfred/remacs/pull/266/files),
and
[the ability to mock extern C functions so we can write unit tests](https://github.com/Wilfred/remacs/pull/304).

## Hash Maps

We're not always able to leverage the Rust libraries
available. @DavidDeSimone showed some amazing Rust-fu [exploring
using Rust's FnvHashMap inside Remacs](https://github.com/Wilfred/remacs/pull/286).

Sadly, we weren't able to use the Rust hash map implementation. The C
layer assumes that it can mutate hash table keys in place, and
`unexec` does not play nicely with `mmap`. See the PR for the full details.

## Logo

Finally, we're discussing a logo for Remacs. We've had some great submissions:

<img src="/assets/remacs1.png">
<img src="/assets/remacs2.png">
<img src="/assets/remacs3.png">
<img src="/assets/remacs4.png">

You can join the logo discussion at [PR #360](https://github.com/Wilfred/remacs/pull/360).

As always, if you fancy writing some Rust in support of the world's
lispiest text editor, you can [join us on GitHub](https://github.com/Wilfred/remacs)!
