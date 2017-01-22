---
layout: post
title: Weird thread behaviour when instrumenting Parsec using PIN
date: 2017-01-22
author: "Debugging William"
header-img: "img/post-bg-2016.jpg"
excerpt_separator: <!--more-->
tags:
- Pin
- Parsec

---

# Problem

When using *PIN* to instrument a *parsec-3.0* application(i.e. `canneal`), I encounter `C: Tool (or Pin) caused signal 11 at PC xxx`. 

Some PIN application(like `malloctrace`) does not have this problem, however when the PIN application is thread-related, e.g. we want to signal at every thread's start:

```
int main() 
{
  // PIN init code
  ...
  PIN_AddFiniFunction(Fini, 0);
  
  // Register Analysis routines to be called when a thread begins
  PIN_AddThreadStartFunction(ThreadStart, 0);
  
  // Other instruments
  ...
}

void ThreadStart(THREADID tid, CONTEXT *ctxt, INT32 flags, VOID *v) 
{
  printf("thread:%d\n", tid);
}

VOID Fini(INT32 code, VOID *v)
{
  printf("finish.\n");
}

```

For a normal multi-thread application, instrumenting it leads to output like

```
...

thread:0
thread:1
thread:2
(more threads)

...

finish.

```

**Multiple "thread"s and single "finish".**

But if instrumenting parsec using 

```
pin -t <toolname> -- parsecmgmt -a run -c gcc-pthreads -p canneal -i simsmall -n 8
```

We got:

```
...

thread:0

...

finish.

...
finish.

(more finishes)

```

**Single "thread"s and multiple "finish".**

# A Possible Explanation

I assume `parsecmgmt` dynamically search, load and run (e.g. `execv()`, but I have not read the source code) the binary for us and PIN got lost when the true binary is run. So it only sees one threads(the entry `parsecmgmt`) but see many exits(multi-threads' exits of `canneal`).

# Solution

Don't call the `parsecmgmt` binary. It's a wrapper and does a lot of other things (e.g. untar input file) besides running the true `canneal` for us (which also generates some useless instrumenting information).

Find the true binary in `pkgs/`. For example, if we want to run the `canneal`, the pthread version is `{parsec root}/pkgs/kernels/canneal/obj/amd64-linux.gcc-pthreads/canneal`. 

For the arguments, refer to `.runconf` files. Say we like to run with input `-i simsmall`, open `{parsec root}/pkgs/kernels/canneal/parsec/simsmall.runconf`. And we manually enter the arguments in it: `{$NTHREADS} 4 10000 2000 100000.nets 32`, replacing *NTHREADS* with thread numbers like 8.

The final version of *PIN* command-line is like

```
pin -t <toolname> -- {parsec root}/pkgs/kernels/canneal/obj/amd64-linux.gcc-pthreads/canneal 8 4 10000 2000 100000.nets 32
```

It almost works except *parsec* complains (vector out-of-range error) because it can't find file `100000.nets`. For `canneal`, it's in `{parsec root}/pkgs/kernels/canneal/inputs/input_simsmall.tar`

```
tar -xf {parsec root}/pkgs/kernels/canneal/inputs/input_simsmall.tar
```

Now `100000.nets` is in our current directory. Run the previous command-line again and everything is good. You may want to remove the untarred file to save space.
