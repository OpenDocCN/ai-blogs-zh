<!--yml
category: 未分类
date: 2024-07-01 18:18:16
-->

# Principles of FFI API design : ezyang’s blog

> 来源：[http://blog.ezyang.com/2010/06/principles-of-ffi-api-design/](http://blog.ezyang.com/2010/06/principles-of-ffi-api-design/)

This is part three of [a six part tutorial series on c2hs](http://blog.ezyang.com/2010/06/the-haskell-preprocessor-hierarchy/). Today, we take a step back from the nitty gritty details of FFI bindings and talk about more general design principles for your library.

On the one hand, writing an FFI binding can little more than churning out the glue code to let you write “C in Haskell,” the API of your library left at the whimsy of the original library author. On the other hand, you can aspire to make your interface indistinguishable from what someone might have written in pure Haskell, introducing modest innovation of your own to encode informally documented invariants from the C code into the type system.

*Overall design.* Larger bindings benefit from being divided into two layers: a low-level binding and a higher-level user-friendly binding. There is a large amount of code necessary to make C functions available to call by Haskell, and the obvious thing to do with it is to stow it away in its own namespace, frequently with `Internal` somewhere in the name.

*Low-level design.* In the low-level binding, you should organize your foreign imports the way the C header files were organized. Keep the names similar. While it won't be possible to have identical C function names and Haskell function names—C functions are allowed to begin with capital letters and Haskell functions are not (there is an opposite situation for types and data constructors)—you can still adopt a consistent transformation. c2hs, by default, converts `Foo_BarBaz` in C to `fooBarBaz`; that is, words after underscores are capitalized, the first letter is uncapitalized, and underscores are removed.

There is room to improve upon the original API, however. The rule of thumb is, if you can make a non-invasive/local change that improves safety or usability, do it. These include:

*   Marshalling vanilla C values (e.g. `int`, `float` and even `char*`, if it's a null terminated string) into their natural Haskell forms (`Int`, `Float` and `String`). Care should be taken, as the native Haskell types lose precision to their C counterparts, and thus should ensure that the application doesn't need to squeeze out every last higher bit (e.g. with a bit field). 80% of the time, a lossy transformation is acceptable,
*   Converting `int` into `Bool` from some naming convention (perhaps boolean values are prefixed with `f` for `flag`),
*   Putting `malloc`'d pointers into memory management with foreign pointers. This advice is worth repeating: Haskell has memory management, and it would be criminal not to use it *as soon as possible*. Plus, you don't have to write an explicit Haskell deallocation function.
*   Converting functions that initialize some memory space (`set_struct_default`) into pure versions (`structDefault`) with `unsafePerformIO`, `alloca` and `peek`. Note that you should do this in conjunction with the appropriate `Storable` instance to marshal the C struct into a persistent Haskell datatype.
*   Marshalling more complex C values (arrays, mostly) into Haskell lists, assuming that bounds information is consistent and available locally.

We will discuss these techniques in more detail in the coming posts, since this is precisely where c2hs is used the most.

It's useful to know when not to simplify: certain types of libraries may have highly efficient in-memory representations for large structures; marshalling them to and from Haskell is wasteful. Poorly written C code may also you hand you arrays for which you cannot find easily their length; deferring their marshalling to the higher-level interface may be a better idea. Decide which structures you are explicitly *not* going to marshal and enforce this across the board. My preference is to marshal flat structs that contain no pointers, and leave everything else be.

*High-level design.* While there are certainly exotic computational strcutures like arrows, applicative functors and comonads which can be useful in certain domains, we will restrict our discussion to the go-to tools of the Haskell programmer: pure functions and monads.

*   *Pure functions.* Transforming the mutable substrate that C is built upon into the more precious stuff of pure functions and persistent data structures is a tricky task, rife with `unsafePerformIO`. In particular, just because a C function published purpose *seems* to not involve mutating anything, it may perform some shared state change or rebalance the input datastructure or `printf` on failure, and you have to account for it. Unless your documentation is *extremely* good, you will need to do source diving to manually verify invariants.

    There will be some number of functions that are referentially transparent; these are a precious commodity and can be transformed into pure functions with ease. From there, you will need to make decisions about how a library can and cannot be used. A set of internal state transformation functions may not be amenable to a pure treatment, but perhaps a function that orchestrates them together leaks no shared state. Data structures that were intended to be mutated can be translated into immutable Haskell versions, or frozen by your API, which exposes no method of mutating them (well, no method not prefixed by `unsafe`) to the end user.

*   *Monads.* First an obvious choice: are you going to chuck all of your functions into the IO monad, or give the user a more restrictive monad stack that performs IO under the hood, but only lets the user perform operations relevant to your library. (This is not difficult to do: you `newtype` your monad stack and simply fail to export the constructor and omit the `MonadIO` instance.)

    ```
    newtype MyMonad a = MyMonad { unMyMonad :: ReaderT Env IO a }
      deriving (MonadReader Env, Monad, Functor)

    ```

    You will be frequently passing around hidden state in the form of pointers. These should be wrapped in newtypes and not exposed to the end-user. Sometimes, these will be pointers to pointers, in the case of libraries that have parameters `**ppFoo`, which take your pointer and rewrite it to point somewhere else, subsuming the original object.

    ```
    newtype OpaqueStruct = OpaqueStruct { unOpaqueStruct :: ForeignPtr (Ptr CStruct) }

    ```

    Shared state means that thread safety also comes into play. Haskell is an incredibly thread friendly language, and it's easy, as a user of a library, to assume that any given library is thread-safe. This is an admirable goal for any library writer to strive for, but one made much harder by your dependence on a C-based library. Fortunately, Haskell provides primitives that make thread safety much easier, in particular MVar, TVar and TMVar; simply store your pointer in this shared variable and don't let anyone else have the pointer. Extra care is necessary for complex pointer graphs; be sure that if you have an MVar representing a lock for some shared state, there isn't a pointer squirreled away elsewhere that some other C code will just use. And of course, if you have persistent structures, maintaining consistency is trivial.

    ```
    withMVar (unOpaqueStruct o) $ \o_ ->
      withForeignPtr o_ $ \p ->
        -- peek ’n poke the piggy, erm, pointer

    ```

    One particularly good technique for preventing end-users from smuggling pointers out of your beautiful thread-safe sections is the application of a rank-2 type, akin to the `ST` monad. The basic premise is that you write a function with the type `(forall s. m s a) -> a`. The `forall` constraint on the argument to this function requires the result `a` to not contain `s` anywhere in its type (for the more technically inclined, the `forall` is a statement that I should be able to place any `s` in the statement and have it be valid. If some specific `s'` was in `a`, it would be only valid if I set my `s` to that `s'`, and no other `s`). Thus, you simply add a phantom type variable `s` to any datatype you don't want smuggled out of your monad, and the type system will do the rest. Monadic regions builds on this basic concept, giving it *composability* (region polymorphism).

    ```
    newtype LockedMonad i a = LockedMonad { unLockedMonad :: ReaderT Env IO a }
      deriving (MonadReader Env, Monad, Functor)
    runLockedMonad :: (forall i. LockedMonad i a) -> IO a
    runLockedMonad m = runReaderT (unLockedMonad m) =<< newEnv
    data LockedData i a = LockedData a

    ```

We will not be discussing these ideas as part of c2hs; use of the preprocessor is mostly distinct from this part of the design process. However, it is quite an interesting topic in its own right!

*Next time.* [First steps in c2hs](http://blog.ezyang.com/2010/06/first-steps-in-c2hs/)