- https://hg.sr.ht/~nbuwe/pclu
- https://todo.sr.ht/~nbuwe/pclu

# Introduction

This repository has as its base the latest pclu release fetched from
ftp://ftp.lcs.mit.edu/pub/pclu/CLU/DEBIAN/pclu-debian.tar.gz

It didn't work b/c of what looks like a last minute 32/64-bit mixup.

I fixed it and cleaned it up to use C99, both in the "asm" C sources
and in the compiler output.  I took the liberty of reformatting the
"asm" C sources to use a more contemporary style.  Since the upstream
will probably never release any updates this should not be a problem I
hope and it hopefully makes life quite a bit easier for people used to
more modern C.

This is still a work in progress, but in the current state it should
have a good chance of working on any Unix where Boehm GC works.


# Building Boehm GC

pclu came with its own copy of Boehm GC sources.  It was stock gc-7.2f
with two trivial wrappers added, `clu_alloc_atomic()` and `clu_alloc()`.
These two wrappers are now moved to `code/sysasm/Opt/alloc.c` and we can
remove all gc includes from `pclu_sys.h`.

Unfortunately some C code still *does* peek into the gc internals so a
public installation of gc cannot be used.

Get a copy of libatomic_ops and install it.  Here is how to install
version 7.10.0 in your home directory's `local` subdirectory.
```bash
$ mkdir ~/libatomic_ops
$ cd ~/libatomic_ops
$ wget https://github.com/bdwgc/libatomic_ops/releases/download/v7.10.0/libatomic_ops-7.10.0.tar.gz
$ tar xf libatomic_ops-7.10.0.tar.gz 
$ mkdir ~/local
$ cd libatomic_ops-7.10.0
$ ./configure --prefix=$(echo ~/local)
$ make
$ make install
```

Get a copy of gc and install it.  Here is how to install version 8.2.12 in your home directory's `local` subdirectory.
```bash
$ mkdir ~/gc
$ cd ~/gc
$ wget https://github.com/bdwgc/bdwgc/releases/download/v8.2.12/gc-8.2.12.tar.gz
$ tar xf gc-8.2.12.tar.gz 
$ cd gc-8.2.12/
$ ./configure --enable-static=yes --enable-shared=no --enable-threads=no --prefix=$(echo ~/local)
$ make
$ make install
```

To build pclu you need to set `CLUHOME`.  In this directory do
```bash
$ export CLUHOME=$(pwd)
```

# Bootstrapping the compiler

```bash
# build the libpclu_opt.a library
$ (cd $CLUHOME/code; make -w OPT_FLAGS='-g -O0 -Wall -Wextra')

# build the compiler from the pre-generated sources
$ (cd $CLUHOME/code/cmp; make -w OPT_FLAGS='-g -O0 -Wall -Wextra')
$ cp $CLUHOME/code/cmp/pclu $CLUHOME/exe/

# dump clu libraries: lowlev.lib  misc.lib  useful.lib
# (think precompiled headers)
$ (cd $CLUHOME/lib; make libs)

# dump cmp.lib for the compiler sources
$ (cd $CLUHOME/cmpclu;make lib)
# rebuild the compiler
$ (cd $CLUHOME/cmpclu;make -w OPT_FLAGS='-g -O0 -Wall -Wextra')
# rebuild the compiler again (probably not needed)
$ (cd $CLUHOME/cmpclu;make -w OPT_FLAGS='-g -O0 -Wall -Wextra')
# verify that the compiler produced the same results.  No diff should show.
$ git diff code/cmp
```

The makefiles are currently set up expecting the gc library to be installed in
`$(HOME)/local` as shown above.


# Junk
To avoid touching `-I` flags in the makefiles, at least for the initial
attempts, it might be handy to create a symlink:

    cd code/include && ln -s ../gc/include/gc

and, similarly, for the library:

    cd code && ln -s gc/lib/libgc.a


# Symlinks

TODO: Get rid of `make symlinks`.

# Other Notes

Check `howto.install` but bear in mind that some things might be out of
date or not work.  Makefiles need more updates and clean ups.  Here's
what I do for now:




# Debugger

Clu debugger is in-process and is compiled and linked directly into
the program.  It should now be in the state where it compiles, and the
debug version of hello world can actually run.  Since debugger relies
on nm(1) to get the symbol addresses, it doesn't work with ASLR and
plink uses `-no-pie` to link debug code.

Simple stuff seems to work, but there are quite a few problems.
E.g. while `debug/libinfo.h` has debug info for builtin stuff, it's
not actually available to the program/debugger as in the naive link
nothing refers debug info symbols and so they are not linked.


# Emacs

I fixed `elisp/clu.el` to work with modern emacs and added `font-lock`
support.  I actually like its indentation style more, but Clu code in
the distribution uses `cludent` style.  It would be nice to make the
style selectable, akin to `c-set-style`.


# Random remarks

I've tested this procedure on Ubuntu/amd64 and NetBSD/macppc.

I have not yet tried a more recent version of gc (8.x).  The original
distribution had a rather old gc-7.2f.  It worked for me on Ubuntu, so
I stuck with it, upgrading to gc-7.2g eventually.

On NetBSD you will need to use gmake.  On my 8-STABLE macppc I could
only get gc-7.2g to work and only with `-O1`.

The common idiom to compile Clu to C but don't compile the C output is
to use `-ccopt 0` (bzw. `-ccdbg 0`).  What makes the magic value `0`
magic is ... its length.  See `cc_one` in top2.clu - it won't do
anything if the command length is less than 4.
