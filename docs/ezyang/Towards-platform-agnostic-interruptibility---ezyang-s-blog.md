<!--yml
category: 未分类
date: 2024-07-01 18:18:09
-->

# Towards platform-agnostic interruptibility : ezyang’s blog

> 来源：[http://blog.ezyang.com/2010/09/towards-platform-agnostic-interruptibility/](http://blog.ezyang.com/2010/09/towards-platform-agnostic-interruptibility/)

Last post, I talked about some of the notable difficulties in [emulating pthread_cancel on Windows](http://blog.ezyang.com/2010/09/pthread-cancel-on-window/). Today, I want to talk about what a platform agnostic compiler like GHC actually ought to do. Recall our three design goals:

1.  GHC would like to be able to put blocking IO calls on a worker thread but cancel them later; it can currently do this on Linux but not on Windows,
2.  Users would like to write interrupt friendly C libraries and have them integrate seamlessly with Haskell’s exception mechanism, and
3.  We’d like to have the golden touch of the IO world, instantly turning blocking IO code into nice, well-behaved, non-blocking, interruptible code.

I am going to discuss these three situations, described concisely as blocking system calls, cooperative libraries and blocking libraries. I propose that, due to the lack of a cross-platform interruption mechanism, the correct interruptibility interface is to permit user defined handlers for asynchronous exceptions.

* * *

*Interruptible blocking system calls.* In the past, GHC has had [some bugs](http://hackage.haskell.org/trac/ghc/ticket/2363) in which a foreign call to a blocking IO system call caused Windows to stop being interruptible. This is a long standing difference in asynchronous IO philosophy between POSIX and Windows: POSIX believes in functions that look blocking but can be interrupted by signals, while Windows believes in callbacks. Thus, calls that seem harmless enough actually break interruptibility and have to be rewritten manually into a form amenable to both the POSIX and Windows models of asynchrony.

While it is theoretically and practically possible to manually convert every blocking call (which, by the way, works perfectly fine on Linux, because you can just send it a signal) into the asynchronous version, but this is very annoying and subverts the idea that we can simply ship blocking calls onto another thread to pretend that they’re nonblocking.

Since Windows Vista, we can interrupt blocking IO calls using a handy new function [CancelSynchronousIo](http://msdn.microsoft.com/en-us/library/aa363794(VS.85).aspx). Notice that cancelling IO is not the same as cancelling a thread: in particular, the synchronous operation merely returns with a failure with the last error set to `ERROR_OPERATION_ABORTED`, so the system call needs to have been performed directly by GHC (which can then notice the aborted operation and propagate the interrupt further) or occur in C code that can handle this error condition. Unfortunately, this function does not exist on earlier versions of Windows.

> *Aside.* Is there anything we can do for pre-Vista Windows? Not obviously: the under-the-hood changes that were made to Windows Vista were partially to make a function like `CancelSynchronousIo` possible. If we were to enforce extremely strong invariants on when we call `TerminateThread`; that is, we’d have to manually vet every function we consider for termination, and at that point, you might as well rewrite it asynchronous style.

* * *

*Interruptible cooperative libraries.* This is the situation where we have a C library that we have a high level of control over: it may be our own library or we may be writing a intermediate C layer between GHC and an expressive, asynchronous underlying library. What we would like to do is have GHC seamlessly convert its asynchronous exceptions into some for that our C can notice and act gracefully upon.

As you may have realized by now, there are *a lot* of ways to achieve this:

*   Signals. POSIX only, signals can be temporarily blocked with `sigprocmask` or `pthread_sigmask` and a signal handler can be installed with `sigaction` to cleanup and possible exit the thread or long jump.
*   Pthread cancellation. POSIX only, cancellation can be temporarily blocked with `pthread_setcanceltype` and a cancellation handler can be installed with `pthread_cleanup_push`.
*   Select/poll loop. Cancellation requests are sent over a socket which is being polled, handler can choose to ignore them.
*   Event objects. Windows only, threads can receive cancellation requests from the handle from `OpenEvent` but choose to ignore them.
*   IO cancellation. Windows Vista only, as described above.
*   Completion queue. Windows only, similar to select/poll loop.

It doesn’t make much sense to try to implement all of these mechanisms natively. Therefore, my proposal: have GHC call a user-defined function in a different thread upon receipt of an asynchronous function and let the user figure out what to do. In many ways, this is not much of a decision at all: in particular, we’ve asked the programmer to figure it out for themselves. Libraries that only worked with POSIX will still only work with POSIX. However, this is still an advance from the current state, which is that asynchronous exceptions *necessarily* behave differently for Haskell and FFI code.

* * *

*Interruptible blocking libraries.* Because blocking IO is much easier to program than non-blocking IO, blocking interfaces tend to be more prevalent and better tested. (A friend of mine who spent the summer working on Chromium had no end of complaints about the [bugginess](https://bugzilla.mozilla.org/show_bug.cgi?id=542832) of the non-blocking interface to NSS.) It might be practical to rewrite a few system calls into asynchronous style, but when you have a blob of existing C code that you want to interface with, the maintenance cost of such a rewrite quickly becomes untenable. What is one to do?

Alas, there is no magic bullet: if the library never had any consideration for interruptibility, forcibly terminating it is more than likely to leave your program in a corrupted state. For those who would like to play it fast and loose, however, the user-defined function approach would still let you call `TerminateThread` if you *really* wanted to.

* * *

In conclusion, I propose that the interruptibility patch be extended beyond just a simple `interruptible` keyword to allow a user-defined asynchronous exception handler, compiled against the RTS, as well as providing a few built-in handlers which provide sensible default behaviors (both platform specific and non-platform specific, though I expect the latter to give much weaker guarantees).