# citeproc

[![BSD2 license](https://img.shields.io/badge/license-BSD2-blue.svg)](LICENSE)
[![CI
tests](https://github.com/jgm/citeproc/workflows/CI%20tests/badge.svg)](https://github.com/jgm/citeproc/actions)

<!--
[![Hackage](https://img.shields.io/hackage/v/citeproc.svg)](https://hackage.haskell.org/package/citeproc)
[![Stackage Lts](http://stackage.org/package/citeproc/badge/lts)](http://stackage.org/lts/package/citeproc)
[![Stackage Nightly](http://stackage.org/package/citeproc/badge/nightly)](http://stackage.org/nightly/package/citeproc)
-->

This library generates citations and bibliography formatted
according to a [CSL] style.  Currently version 1.0.2 of the CSL
spec is targeted.

This library is a successor to pandoc-citeproc, which was a fork
of Andrea Rossato's citeproc-hs.  I always found it difficult to
fix bugs in pandoc-citeproc and decided that implementing
citeproc from scratch would give me a better basis for
understanding.  This library has a number of other advantages
over pandoc-citproc:

- it is much faster (as a rough benchmark, running the CSL
  test suite takes less than 4 seconds with this library,
  compared to 12 seconds with pandoc-citeproc)

- it interprets CSL more faithfully, passing more of the CSL
  tests

- it has fewer dependencies (in particular, it does not depend
  on pandoc)

- it is more flexible, not being tied to pandoc's types.

Unlike pandoc-citeproc, this library does not provide an
executable.  It will be used in pandoc itself to provide
integrated citation support and bibliography format conversion
(so the pandoc-citeproc filter will no longer be necessary).

At this point, the library still fails some of the tests from the
CSL test suite (105/845).  However, most of the failures are on
minor corner cases.  And because the library already passes
many more CSL tests than pandoc-citeproc did, it seemed worth
publishing an early version even before all these bugs are
ironed out.

[CSL]: https://docs.citationstyles.org/en/stable/specification.html

## How to use it

The main point of entry is the function `citeproc` from the
module `Citeproc`.  This takes as arguments:

- a `CiteprocOptions` structure (which currently just allows you
  to set whether citations are hyperlinked to the bibliography)

- a `Style`, which you will want to produce by parsing a CSL
  style file using `parseStyle` from `Citeproc.Style`.

- Optionally a `Lang`, which allows you to override a default locale,

- a list of `Reference`s, which you can produce from a CSL JSON
  bibliography using aeson's `decode`,

- a list of `Citation`s (each of which may have multiple
  `CitationItems`).

It yields a `Result`, which includes a list of formatted
citations and a formatted bibliography, as well any warnings
produced in evaluating the style.

The types are parameterized on a `CiteprocOutput` instance `a`,
which represents formatted content in your bibliographic
fields (e.g. the title).  If you want a classic CSL processor,
you can use `CslJson Text`.  But you can also use another type,
such as a pandoc `Inlines`.  All you need to do is define
an instance of `CiteprocOutput` for your type.

The signature of `parseStyle` may not be self-evident:
the first argument is a function that takes a URL and
retrieves the text from that URL.  This is used to fetch
the "indendent parent" of a dependent style.  You can supply
whatever function you like: it can search your local file
system or fetch the content via HTTP.  If you're not using
dependent styles, you can get by with `const Nothing`.

