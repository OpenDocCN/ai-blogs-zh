<!--yml
category: 未分类
date: 2024-07-01 18:17:57
-->

# Multi-day debugging : ezyang’s blog

> 来源：[http://blog.ezyang.com/2011/02/multi-day-debugging/](http://blog.ezyang.com/2011/02/multi-day-debugging/)

Most of my hacking cycles right now are going towards debugging the new code generator for GHC. The code generation stage of GHC takes the Spineless Tagless G-machine (STG) intermediate representation (IR) to the C-- high-level assembly representation; the old code generator essentially performed this step in one big bang. The new code generator is many things. It is a more modular, understandable and flexible codebase. It is a client of cutting edge research in higher-order frameworks for control-flow optimization.

It is also frickin’ hard to debug.

I used to get frustrated and give up if I couldn’t figure out what was causing a bug within a few hours of close analysis. Working on GHC has enlightened me about the multi-day debugging: a well-defined bug that persists despite several days of intense analysis. (I’ve only managed this a few times in the past—I’m quite proud that I managed to pull together enough information to resolve [“the bug”](http://bugs.php.net/42362). What is “the bug”? Have you ever been browsing a MediaWiki site and then been mysteriously asked to download a PHP file? Yeah, that’s “the bug”). It has exponentially increased my proficiency with gdb and has been an amazing romp in the theoretics and practice of compiler construction. I’ve felt stupid for not immediately understanding what in retrospect seem perfectly clear and obvious concepts. I’ve felt an amazing rush, not from when the problem is solved (though that certainly gives a good feeling), but when my plan of attack is making progress. I’ve seen my theories evolve from one to another to another, and have learned never to trust any experimental observation at first sight.

While the debugging process is not yet done (though I think I’m close to having a correct—but slow—new code generation pipeline), I thought I’d take out some time to describe the journey.

### Why debugging GHC is easy?

Fascinatingly enough, while the bugs result in extremely strange behavior in compiled programs that takes ages to decipher, once the bad behavior is fully understood, the fix is usually a one-liner. It is this fact that makes debugging GHC frustrating and brilliant at the same time: sometimes code you’re debugging is fundamentally mistaken, and you have to rewrite the whole thing. GHC’s code is fundamentally clear (a testament to those who wrote it), and a bug is usually just a small detail someone forgot to attend to. The solutions are like Mystery Hunt solutions: short, and you know when you’ve found it. Nothing messy like, “What is this actually supposed to do?”

I have the benefit of an existing code generation pipeline which I can use to compare my results with, although doing so is not trivial since the new code generator does go about the compilation in a fundamentally different way, and so sections of code are frequently not comparable.

I also have the benefit of a wondrous test-suite which produces me programs that reproduceably segfault with little fuss, and have been relatively blessed with bugs that show in single-threaded situations. My programs have well defined inputs and outputs, and I have sophisticated mechanisms for inspecting the internal state of the multipass compiler.

### What have I fixed so far?

Warning: Gory technical detail ahead.

#### Build it

My first job was to make the latest new code generation code compile with the latest GHC branch (it had bit-rotted a bit in the interim.) This went mostly smoothly, except for the fact that Norman Ramsey really likes polymorphic local definitions and MonoLocalBinds reared its ugly head in Hoopl and a few other modules.

#### Test 4030

[Test 4030](http://hackage.haskell.org/trac/ghc/ticket/4030) was this “simple” program (simple is in quotes, because as Simon Peyton-Jones put it, “that looks like a hard one to start with... threads, exceptions, etc.”)

```
main = do tid <- block $ forkIO $ let x = x in x
        killThread tid

```

The resulting code segfaulted in stg_BLACKHOLE_info when attempting to dereference “something.”

```
0x822a6e0 <stg_CAF_BLACKHOLE_info>:  jmp    0x822a620 <stg_BLACKHOLE_info>
0x822a620 <stg_BLACKHOLE_info>:      mov    0x4(%esi),%eax
0x822a623 <stg_BLACKHOLE_info+3>:    test   $0x3,%eax
0x822a628 <stg_BLACKHOLE_info+8>:    jne    0x822a663 <stg_BLACKHOLE_info+67>
0x822a62a <stg_BLACKHOLE_info+10>:   mov    (%eax),%ecx -- SEGFAULT!

```

This something ended up being a new stack slot that Simon Marlow introduced when he had [rewritten the blackholing scheme](http://hackage.haskell.org/trac/ghc/ticket/3838). The solution was to port these changes to the new code generator. I ended up manually reviewing every patch within the merge time window to ensure all changes had been ported, and probably squished a few latent bugs in the process. There’s no patch because I ended up folding this change into the merge (since the new blackholing scheme had not existed at the time the new code generator branch was frozen.)

#### Test ffi021

Test ffi021 involved creating a pointer to an imported FFI function, and then dynamically executing it. (I didn’t even know you could do that with the FFI!)

```
type Malloc = CSize -> IO (Ptr ())

foreign import ccall unsafe "&malloc" pmalloc:: FunPtr Malloc
foreign import ccall unsafe "dynamic" callMalloc :: FunPtr Malloc -> Malloc

```

This ended up being a latent bug in the inline statement optimizer (not a bug in the new code generator, but a bug that the new codegen tickled). I got as far as concluding that it was an optimization bug in the native code generator before Simon Marlow identified the bug, and we got [a one-line patch](http://www.mail-archive.com/cvs-ghc@haskell.org/msg24392.html).

```
hunk ./compiler/cmm/CmmOpt.hs 156
-   where infn (CmmCallee fn cconv) = CmmCallee fn cconv
+   where infn (CmmCallee fn cconv) = CmmCallee (inlineExpr u a fn) cconv

```

#### Test 4221

This one took three weeks to solve. The original test code was fairly complex and highly sensitive to code changes. My first theory was that we were attempting to access a variable that had never been spilled to the stack, but after talking to Simon Peyton Jones about how stack spilling worked I got the inkling that this might not actually be the problem, and stopped attempting to understand the Hoopl code that did spilling and went back to analysis. There was another false end with regards to optimization fuel, which I hoped would help pinpoint the point of error but in fact doesn't work yet. (Optimization fuel allows you to incrementally increase the number of optimizations applied, so you can binary search which optimization introduces the bug. Unfortunately, you most of the so-called “optimizations” were actually essential program transformations on the way to machine code.)

The breakthrough came when I realized that the bug persisted when I changed the types in the input program from CDouble to CInt64, but not when I changed the types to CInt32\. This allowed me to identify the erroneous C-- code involving *garbage collection* and reduce the test-case to a very small program which didn’t crash but showed the wrong code (since the program needed to run for a while in order to trigger a stack overflow at precisely the right place):

```
{-# LANGUAGE ForeignFunctionInterface #-}
module Main(main) where

import Foreign.C

foreign import ccall safe "foo" foo :: CLLong -> CLLong
-- Changing to unsafe causes stg_gc_l1 to not be generated
-- Changing to IO causes slight cosmetic changes, but it's still wrong

main = print (foo 0)

```

After a huge misunderstanding regarding the calling convention and futile attempts to find a bug in the stack layout code (I assumed that `slot<foo> + 4` indicated a higher memory location; in fact it indicated a lower memory location than `slot<foo>`), I finally identified the problem to be with the `stg_gc_*` calling convention.

My first patch to fix this changed the callee (the `stg_gc_*` functions) to use the observed calling convention that the new code generator was emitting, since I couldn’t see anything wrong with that code. But there was an anomalous bit: by this theory, all of the calls to GC should have used the wrong calling convention, yet only doubles and 64-bit integers exhibited this behavior. My patch worked, but there was something wrong. This something wrong was in fact the fact that 32-bit x86 has *no* general purpose non-32-bit registers, which was why the code generator was spilling only these types of arguments onto the stack. I learned a little bit more about GHC’s virtual registers, and determined another one line fix.

```
hunk ./compiler/cmm/CmmCallConv.hs 50
-               (_,   GC)               -> getRegsWithNode
+               (_,   GC)               -> allRegs

```

#### Test 2047 (bagels)

This one is in progress. Fixing the GC bug resolved all of the remaining mysterious test suite failures (hooray), and with this I was able to recompile GHC with all of the libraries with the new code generator. This triggered [test 2047](http://hackage.haskell.org/trac/ghc/ticket/2047) to start segfaulting.

It took me a little bit of time to establish that I had not introduced a bug from compiling the stage 2 compiler with the new codegen (which I had done overzealously) and confirm which library code had the bug, but once I had done so I managed to reduce it to the following program (which I had lovingly named “bagels”):

```
import Bagel

main = do
    l <- getContents
    length l `seq` putStr (sort l)

```

with sort defined in the module Bagel as such:

```
module Bagel where
-- a bastardized version of sort that still exhibits the bug
sort :: Ord a => [a] -> [a]
sort = mergeAll . sequences
  where
    sequences (a:xs) = compare a a `seq` []:sequences xs
    sequences _ = []

    mergeAll [x] = x
    mergeAll xs  = mergeAll (mergePairs xs)

    mergePairs (a:b:xs) = merge a b: mergePairs xs
    mergePairs xs       = xs

    merge (a:as') (b:bs') = compare a a `seq` merge as' as'
    merge _ _ = []

```

and run with the following data:

```
$ hexdump master-data
0000000 7755 7755 7755 7755 7755 7755 7755 7755
*
000b040

```

This program has a number of curious properties. The segfault goes away if I:

*   Turn off compacting GC
*   Reduce the size of master-data
*   Turn off optimizations
*   Use the old codegen
*   Put all of the code in one file
*   Remove the seqs from 'sort' (which isn't actually a sort)
*   Remove the seqs from 'main'
*   Make the sort function monomorphic on Char

The current theory is someone (either the new code generator or the compacting GC) is not handling a tag bit properly, but I haven’t quite figured out where yet. This is the only outstanding bug unique to the new code generator.