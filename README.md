# _giflib_

a library for reading and writing GIF files.

## Overview

_giflib_ provides code for reading GIF files and transforming them into 
RGB bitmaps, and for writing RGB bitmaps as GIF files. GIF is a legacy
format and the _giflib_ maintainers recommend against generating new
images in the format. PNG has a more extensible design with better color
support and better compression.

### Animation and on-line identification

The primary reason to use GIF is support for terminal graphics and color
animations. GIF specifies a terminal protocol in Appendix G for on-line
identification of the GIF Data Stream for display of images in terminals.

Animated PNG (APNG) offers an alternative for animations but has not been
ratified as of 2022 and this is not included in the libpng implementation.
Here are details of the libpng fork supporting Mozilla's APNG variant:

- https://sourceforge.net/projects/libpng-apng/
- https://github.com/glennrp/libpng/issues/267

## Project information

This repository if a fork of _giflib_ adding CMake support and minor
portability changes to support building with Microsoft Visual Studio.

Latest versions of _giflib_ are currently hosted at:
- http://sourceforge.net/projects/giflib/

Please report bugs to the bug tracker on sourceforge:
- https://sourceforge.net/p/giflib/bugs/

The project has a long and confusing history, described in the history
section of this document which was sourced from `history.adoc`.

## Authors

- Gershon Elber, &lt;`gershon`_(at-symbol)_`cs.technion.sc.il`&gt;,
  original giflib code.
- Toshio Kuratomi, &lt;`toshio`_(at-symbol)_`tiki-lounge.com`&gt;,
  uncompressed gif writing code, former maintainer.
- Eric Raymond, &lt;`esr`_(at-symbol)_`snark.thyrsus.com`&gt;,
  current and long time former maintainer of giflib code.

There have been many other contributors; see the attributions in the
version-control history to learn more.

## Building

A CMake project has been added to this project to aid in the use of
_giflib_ in cross platform projects. The code has been tested to build
on Linux, macOS and Windows with GCC, Clang and the Microsoft compiler.

```
cmake -B build
cmake --build build
cmake --install build
```

## Portability

This codebase is C99-conformant. 

The Makefile is required to build the docs and a distribution tarball.
You will need xmlto to build the derived forms of the documentation
from the DocBook-XML sources. If you are going no modify the website,
you will also need to have asciidoc and the ImageMagick `convert`
utility installed.

You can run "make check" after the library and utilities have been
built to see a regression test of the codebase. No output (other than
the test header lines) is good news.

## Release Instructions

1. Check the SourceForge tracker for bugs and patches.
2. Bump the version number in gif_lib.h. Do "make version"
   to confirm that it looks sane when extracted to the Makefile.
3. Version-stamp the top entry in the NEWS file. 
4. If you are changing major versions, sync the XBS-SourceForge-Folder
   attribute in the control file.
5. 'make dist' to make a tarball.
6. Tag the release in the repo.
7. Ship the release tarball.

The last three steps can be done with "make release" if you have shipper
installed.

## History

GIF (Graphics Interchange Format) was originally developed on the
CompuServe timesharing service in the late 1980s. It was described
by a GIF standard issued in 1987 and revised in 1989. A copy of the
GIF89 standard is included in the `doc/` directory.

This code originated as a linkable library for DOS programs, together
with command-line tools for generating and viewing and analyzing GIF
images. The DOS code was written by Gershon Elber using Borland C
under MS-DOS sometime between the issue of GIF87 and mid-1989 (1.0 was
dated 14 June 1989; one portion, getarg.c, was dated 11 Mar 88).

At some time no later than the end of 1989 Eric S. Raymond (aka "ESR")
ported this DOS version to System V Unix. Between 1989 and 1992 ESR
reworked various portions of the API, improving and simplifying 
the code's interface.

ESR's 2.1 version was the first to include the DGifSlurp()/EGifSpew()
function pair for enabling non-sequential operations on GIF images
(also the tools icon2gif, gifovly, and gifcompose; the last was
removed in 5.0).

ESR's Unix port was incorporated into the NCSA Mosaic browser in 1994,
which is how GIF became (with JPEG) one of the two most popular image
formats on the early Web.

Beginning around 1993, patent claims by Unisys over the LZW
compression method used in GIF theatened adverse legal consequences
for users and developers of programs incorporating the format. The
threats became serious in 1999, with Unisys demanding license fees
for any software using the format.

One response to this was the development of PNG in 1995. Another was
that ESR sought a lead developer outside the U.S. to hand the project off 
to, and passed it to Toshio Kuratomi. ESR remembers this as happening
in 1994, but that date could be wrong as some headers imply 3.0 was issued
under  ESR's name in 1996. But other files do date Toshio's first release
to 1994. Toshio shipped 4.0 in December 1998.

Subsequently, the project shipped for some time as "libungif" with
support for compressed GIFs removed to avoid the LZW patent issues.
Compression support was merged back in after the last blocking patent
expired in 2004; this became release 4.0.0. After that merge the
code was again known as giflib.

By 2006, support for PNGs was sufficiently universal that GIF could be
described as a legacy format. Anything you can do with it GIF could
probably be better done with PNG. Nevertheless (and despite efforts
like "Burn All GIFs Day" in November 1999) the GIF format has remained
widely popular.

In April 2012 ESR rejoined the project to do some code cleanups 
and auditing, and Toshio Kuratomi asked him to take back the lead.
ESR released version 4.2 in May 2012.

Version 5.0, released in June 2012, fulfilled almost all the to-do
items from 18 years of backlog. It made the library thread-safe, added
direct support for GIF89 graphics control blocks, and tossed out large
amounts of obsolete utility code.

More recent version of the code (5.1.0 and onwards) have been hardened
by both static analysis and fuzz testing. While these failed to turn
up bugs in normal rendering cases, they did uncover some crash and
corruption bugs that could be tickled by carefully crafted malformed
GIFs.

This code is very old, very stable, and *everywhere* - browsers
game consoles, smartphones, pretty much everything that opens an
HTTP port and does graphics uses it.

The utilities in this source tree were important as GIF production
tools early in the format's history, but have been superseded by
multi-format viewers and editors. Most installable binary packages
shipped as 'giflib' include the library and header file only.
