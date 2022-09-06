---
title: "Odin programming language review"
draft: false
---

# A review of the Odin programming language

Written by Dale Weiler

* [Twitter](https://twitter.com/actualGraphite)
* [GitHub](https://github.com/graphitemaster)

Last updated Tuesday, September 6th, 2022

## What is Odin
For those unfamiliar, Odin is a systems programming language that is more conservative in its design than other newer programming languages such as Rust, Zig, and Carbon. The design ideology around Odin is to provide some greatly needed quality of life improvements over the lingua-franca of systems languages: C, while still staying as simple as C. This makes it a really attractive language for those who are unhappy with the increased complexity that other systems languages like to encourage.

## Experience
I started writing Odin professionally approximately a year ago. In that time I've written about 50k lines of Odin. This article serves to share my experience with the language and to provide others with an authentic case study from the perspective of a single developer. This article's intended audience is that of someone who already knows Odin or at least knows of its existence. This will be of very little use if you're just learning Odin as I delve into more of the inner workings of the Odin language and more specifically the Odin compiler. This is written from my perspective. I'm a systems engineer with approximately 10 years of systems architecture and design experience. That context is important because the use case, development process and methodology I use is specific to systems design. Odin is not only a systems programming language, it can also be used to write web applications (via WASM) and is more general purpose than one may think, in these contexts this article will be of no value as I have neither the experience or understanding to comment on the application of Odin those domains.

## Conflict of interest
I think it's important to be as transparent as possible while providing this case study. There are a few conflicts of interest that you, the reader should be aware of.

### Financial interest
Odin is the programming language of choice at the company I work for: [JangaFX](https://jangafx.com/). We use it to develop [EmberGen](https://jangafx.com/software/embergen/) and other upcoming products. I'm financially motivated to develop in Odin. This isn't a personal hobby of mine, this is a serious commercial application of the language.

### Relational interest
The author of Odin is a coworker and a personal friend who I've known for three years. In my experience the communities around certain programming languages can be quite critical and toxic to those who work on them. If you're looking for a rant where I'm critical of my friend because of complaints I have with their language you can stop reading now.

## Learning
Learning Odin is very easy. As someone with a predominantly C and C++ background prior to using Odin, it took me about a week to become productive in it. It should be noted in this time I had direct access to the author of the language to answer questions I had, so your milage may vary.

Every new language has growing pains for beginners and Odin is no different. Early on I discovered five things in particular that proved to be difficult to get used to.

> It's important to understand that I'm a senior software developer. I'm not a beginner programmer, so my definition of what it means to be a "beginner" using Odin is that of someone who already has a decade plus experience in systems programming using systems programming languages.

### Slices
Odin, unlike C and C++ has a builtin slice type. As someone who has a strong propensity towards pointer arithmetic, slices weren't easy for me to become comfortable with. You may find this a bit surprising since most people find pointers and pointer arithmetic confusing. The reality is that slices require a different approach to problem solving. The reason for this is because:

* Slices cannot go "backwards". You cannot have a negative index into a slice and you cannot decrement or subtract a slice.
* Slices always store a length while a lot of pointer-focused code tends to either have an explicit associated length (which is similar to a slice), or have an **implied length** (something I noticed I relied a lot on). An implied length is one where the length is known through some other means other than being explicitly associated. This can be encoded in the function signature like in the case of `glVertex3fv` where you know the pointer to the `float*` parameter will be of three floats, it can be a special terminator value, like `\0` in a string (null-terminated string), or a `NULL` pointer at the end of an array of pointers, etc. These ideas, while more dangerous, are simply not expressible with slices.

It became very apparent while using Odin that I had a [functional fixedness](https://en.wikipedia.org/wiki/Problem_solving#Functional_fixedness) to solving problems relying on this behavior that I had to overcome to become comfortable with slices. This took me several months.

### Hidden reference semantics
Odin has pointers like any other systems language, but it also has hidden reference semantics for values. In two very specific circumstances, primarily the `switch x in y` and `for x in y` statements you can prefix the `y` operand with the address-of operator `&` to get reference semantics on `x`. That is `x` will look and behave like a regular variable, but assignments to it will propagate to the memory address of that variable as if `x` was an implicitly dereferenced pointer. This is similar to references in C++. This is the only place in Odin where references exist, every other place you want these semantics you need to be explicit with pointers. I found this a bit confusing and surprising because it's a very unique and specific situation that does not apply generally in other areas of the language. This is a stark contrast to C++ where reference semantics can be used anywhere thanks to reference types.

### Hidden dereference semantics
In various parts of the language, pointers to things can be implicitly dereferenced for you. This makes the code nicer to read and easier to refactor. This hidden dereferencing is idiomatic of Odin code but it can be difficult for a beginner. To give some examples.

* `len(a)` and `cap(a)` will implicitly dereference for pointer to slice (i.e `^[]T`) and pointer to dynamic array (i.e `^[dynamic]T`).
* `foo[key]` will implicitly dereference for pointer to map (i.e `^map[Key]Value`).
* `foo[index]` will implicitly dereference for pointer to slice (i.e `^[]T`) and pointer to dynamic array (i.e `^[dynamic]T`).
* `a.b` will implicitly dereference if `a` is a pointer, that is `a.b` is sugar for `a^.b` when `a` is pointer type.
* `switch v in u` will implicitly deference if `u` is a pointer of `union` type.
* `for x in y` will implicitly dereference `y` if `y` is a pointer to `map` (i.e `^map[Key]Value`), slice (i.e `^[]T`), or dynamic array (i.e `^[dynamic]T`).
* `a->b()` will implicitly dereference `a` if a is a pointer, but continue to pass `a` as the first argument to `b` as a pointer.

### Hidden memory allocations
Almost nothing in Odin will perform a hidden memory allocation on you. This is a huge improvement over C++ where almost everything can perform a hidden memory allocation. In comparison to C however, two specific situations can incur a hidden memory allocation in Odin. When constructing a dynamic array or map literal. These must be paired with a call to `delete`. This is not immediately obvious to a beginner and Odin has a compiler flag `-no-dynamic-literals` which can disable these, but doing so also prevents you from using them.

### Memory allocation confusion
Odin has two idiomatic ways to allocate memory. The `make` and `new` procedures. When destroying something created with `make` you call `delete`. When destroying something created with `new` you call `free`. This is a bit confusing if you come from C++ where `new` is paired with `delete`. I found myself making this mistake endlessly due to muscle memory. I suspect if I ever go back to writing C++ I'll find myself making the opposite mistake now.

One should be cognizant of the muscle memory one develops while working in programming languages and how it can become a barrier to learning others and how it can introduce subtle mistakes as well.

## Quality of life
The immediate benefit to using Odin over C and C++ is that it provides a bunch of obvious needed quality of life improvements that benefit the type of work I was already doing as an systems developer.

* Slices

    As mentioned before, took awhile for me to get used to them but they're nicer to work with than pointer arithmetic.

* Builtin dynamic array and hash map types

    No need for writing your own. The implementations of these are quite good. There is no builtin hash set type but Odin supports zero-sized structure types unlike C and C++ so you can just make the value-type of a map an empty structure type and Odin will not allocate any memory for the values.

* Bit sets

    It's quite common in systems programming to want to create bit flags to pack various state into a single integer. In Odin you can create a regular enum without resorting to assigning them all values of `1 << n` and instead just instantiate a `bit_set[Enum]` and get set operations instead.

* SOA (Structure of Arrays)

    Can turn any array of structures into a structure of arrays by giving it the `#soa` attribute. This changes the ordering of the data in memory by having the compiler generate a structure in the alternative memory layout. Pointers to fields still work through the use of fat pointers instead. This can be great for performance critical code where you want to maximize cache utilization.

* Multiple return values

    Not to be confused with a tuple type, this works more similar to Lua.

* Half float

    While not native to CPUs yet, a lot of what people are doing is GPU and AI work where half-float reigns supreme. Having a native half-float type in the language makes interop so much easier. The core library implements math kernels for `f16` as well which saves a tremendous amount of work.

* Array programming

    You often see operator overloaded types in C++ to implement vector types. This is completely unnecessary as Odin already supports all the usual arithmetic operations on array types by applying them component-wise. This is quite nice to use and auto-vectorizes in my experience too.

* Matrix types

    Native matrix types exist up to 4x4 for any of the floating point types, including half-float. This obviates the need for operator overloading completely for game developers.

* Discriminated unions

    Cannot tell you how frustrating it is that C++'s idea of a sum type is a library provided `std::variant`. A union of a single type is an optional type called a `Maybe`.

* `defer`

    Declarative control flow that obviates the need for RAII.

* `or_return` and `or_else`.

    These are hard to explain. Check out [The Value Propagation Experiment](https://www.gingerbill.org/article/2021/09/06/value-propagation-experiment-part-2/)

* Foreign function interface

    Interfacing with C libraries is really nice. While it isn't as nice as Zig which can parse C headers directly, Odin's FFI allows linking against C libraries directly, while letting you rename symbol names through `@(link_prefix)` and specify libraries directly in the source file requiring no changes to the build system, because;

* The compiler is the toolchain

    There's no need for a build system, nor to explicitly call the linker. The compiler does it all.

* Vendor libraries

    The Odin compiler comes with a bunch of vetted vendor libraries out of box for all sorts of things you may need. I even [tweeted](https://twitter.com/actualGraphite/status/1493582583459983361) about how Odin might be the best language for graphics and game developers because everything you'd need comes included.

There are just some of my personal favorite quality of life improvements, many more exist than what I've listed above and you should check out the language if you haven't already to see the others.

What was less obvious and I did not expect while writing Odin was how miserable I actually was writing C++ prior. I needed the perspective of using something else to see just how terrible C++ was on my personal quality of life. I also [tweeted](https://twitter.com/actualGraphite/status/1499287414392737798) about this.

## Stability
When I first started writing Odin I found the compiler would crash often. After personally debugging the compiler it became evident that the threaded type checker had various data races which only seemed to occur on my 32-thread CPU. These are now fixed and I can confidently say that the Odin compiler has been relatively stable in my year using it. It should be noted that sometimes I've run into internal compiler errors or LLVM code generation errors when using more rare and exotic constructs in the language that don't see much usage or testing, but every time I've encountered them they've been easy to work around or fixed pretty much immediately when an issue was opened on the Odin GitHub.

## Correctness
The danger of using new languages is not having all the years of bug fixes and improvements applied to them. The Odin compiler has been quite good at being correct with code correctness from a compilation stand point. During my year using it I had only run into a single logical miscompilation bug caused by a `defer` statement which wasn't being executed when the procedure returned. Thankfully this bug was fixed the same day I report it and I didn't lose much time tracking it down.

### The bug that almost killed me
If that was the the only correctness issue I ran into this wouldn't necessitate having an entire header in this review for correctness though. There was one other correctness issue within the standard library that almost killed me. To give some context to the issue, we have a reactor system (a multi-threaded event loop) I had written for performing blocking IO operations and asset decompression / deserialization on without stalling the main thread which we were experiencing strange memory corruption issues on. I suspected this was caused by your standard data-race by a misuse of synchronization primitives. There was some red-herring robustness issues with locks I noticed early on that shouldn't've mattered but because I couldn't figure out what the issue was I rewrote the event loop model a few times, reducing some lock contention and improving robustness in process and the issue went away for me on Linux completely. This suggested that there was a data-race in the reactor which I later confirmed was the case but didn't know at the time. What was more concerning was that heap corruption was still happening on Windows. This is where I went down the wrong road of assuming that the reactor had data races and I spent the next two weeks replacing all the locking primitives with ones in libc (made possible because Odin has a full projection of the C standard library in `core:c/libc`) and painstakingly testing every possible code path in `helgrind` (since it can track libc primitives). It reported nothing. I even hand drew a time-line graph with order-before and order-after relationship of all locks and I concluded that the reactor was correct. I ignored this issue at this point since it was consuming up all my time and making me lose sleep at night, I focused my efforts on other important matters, after all the issue wasn't happening for me on Linux at all, it was exclusively a Windows problem.

I'm the only person at JangaFX as of current who develops and mains Linux so this wasn't a sound strategy because the week after a coworker starts hitting the problem on Windows leading us to look into it again. This time with a refreshed mind and the confidence it wasn't an issue in my code I suspected the Odin standard library. The way synchronization primitives are implemented on Linux are through the use of the `futex` system call and we already knew that Linux worked. I had assumed that Windows primitives would've used `WaitOnAddress` and `WakeOnAddress` which is similar to `futex` but to my surprise Odin implements the `Mutex` primitive with [Slim Reader/Writer Locks (SRW)](https://docs.microsoft.com/en-us/windows/win32/sync/slim-reader-writer--srw--locks) on Windows.

These are impossible to get wrong and looked perfectly correct, here's the full implementation.
```odin
_Mutex :: struct {
	srwlock: win32.SRWLOCK,
}
_mutex_lock :: proc(m: ^Mutex) {
	win32.AcquireSRWLockExclusive(&m.impl.srwlock)
}
_mutex_unlock :: proc(m: ^Mutex) {
	win32.ReleaseSRWLockExclusive(&m.impl.srwlock)
}
_mutex_try_lock :: proc(m: ^Mutex) -> bool {
	return bool(win32.TryAcquireSRWLockExclusive(&m.impl.srwlock))
}
```

I thought long and hard about how this could possibly fail. Looking back at the reactor code the only way memory corruption could occur is if `mutex_try_lock` succeeded without actually locking the mutex.

Looking at the Windows package which declares the `TryAcquireSRWLockExclusive` procedure we saw the following.
```odin
TryAcquireSRWLockExclusive :: proc(SRWLock: ^SRWLOCK) -> BOOL
```

This looked perfectly reasonably, until a corworker looked at the [MSDN page](https://docs.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-tryacquiresrwlockexclusive) for it.
```c
BOOLEAN TryAcquireSRWLockExclusive(
  [in, out] PSRWLOCK SRWLock
);
```

At this point you should be asking yourself if there is any difference between `BOOL` and `BOOLEAN`. To save you the effort here's how Windows defines them.
```c
typedef int BOOL;
typedef BYTE BOOLEAN;
```

Reading the result of a function which returns a single byte as a four byte integer casted to `bool` in Odin was incorrectly evaluating to `true` because those bytes are largely undefined and can contain anything and anything but a value of 0 is `true`. The lock was never acquired, even though it said it was! These three extra characters on the prototype, three extra bytes (in both cases) ended up being the issue. The relief felt after finding this was immense. I wouldn't wish such pain on even my worst enemy. Thankfully for others, they shouldn't need to run into this as it was promptly [fixed](https://github.com/odin-lang/Odin/commit/7fe36de069520a0567d02b351cfd0514d83aa0c6).

Do not let this horror story keep you from trying out Odin, new languages and early adopters always bare the most pain. That is a genuinely easy mistake anyone could've made. I was surprised to learn that `BOOL` and `BOOLEAN` are different in representation too, what a horrible choice made by Microsoft to introduce such confusion into function boundaries too.

## Performance
While being a systems programming language, Odin's performance characteristics are slightly behind that of C and C++. There's three reasons for this I discovered while profiling compiled code.

1. The compiler is not as aggressive at optimization as C or C++ (and it cannot be)
    * It lacks strict-aliasing optimizations as they're incompatible with the language. This is similar to MSVC which doesn't have them either in C or C++ as Windows depends on this behavior, as does the Linux kernel which disables them in gcc with `-fno-strict-aliasing`. The performance characteristics of this in general vary depending on how your code is written. Systems programmers appear to be divided on the benefit of these.

    * It lacks poison-value based optimizations. Overflowing a signed integer (in C or C++) or operating on a null pointer, etc. The type of values these produce are "undefined" in those languages, when depending on them, or operating on them, these values are considered "poisoned". Compilers of most systems languages optimize on the assumption you do not operate on this "undefined behavior". Odin does not have undefined behavior and so these optimizations cannot be applied. In my experience these optimizations rarely provide any significant performance improvement but some sources I've read cite ~3% improvements in some cases.

    * Odin lacks global value numbering optimizations, only allowing very basic lexically equivalent common subexpressions to be eliminated and similar local value number optimizations. There are allegedly areas in the language where this would break things but I haven't been able to validate that so this may just be an overly conservative decision of the compiler or a bug in LLVM (which Odin uses for it's code generation), being avoided.

    * No devirtualization. Since Odin doesn't support virtual functions, implementing them manually with function pointers as you would in C, (which in Odin is a variable with procedure type) means in contexts where all information is available, the compiler will not directly call the procedure like it would in C++. On x86_64 this is rarely a problem however since RIP-relative addressing for an indirect function call has very little to no meaningful overhead compared to a direct call which behaves the same way, but it does prevent inlining and on other architectures it can have some overhead. This can have a profound effect on performance for procedures that are called very often, like a comparator in a sort implementation. This is one of the reasons `std::sort` in C++ is often faster than `qsort` in C, even when both use the exact same sorting algorithm. This wouldn't normally be a complaint if it weren't for the fact that clang and gcc can devirtualize even regular function pointers in C with various heuristics as part of it's devirtuallization efforts. Nothing in the Odin language would prevent similar optimizations, just the current Odin compiler doesn't currently.


2. No way to inline generics. Odin supports compile-time parametric polymorphism, however there's no way to refer to a procedure or constant at compile-time unless passed literally. This means you often end up with runtime indirect procedure calls which can be seen in the core library. In C++ you can refer directly to a static method or constant with `T::method` or `T::value`, meaning you can refer to constants non-literally through just a `typename T`. This is not possible in Odin, which is the primary limitation for generic inlining. A structure just cannot embed a constant of any type. The standard library choses to use non-constant procedures in generic code such as `sort_by`.
    ```odin
    sort_by :: proc(slice: []$T, cmp: proc(lhs, rhs: T) -> bool) {
        // ...
    }
    ```
    When it could use a constant procedure type like this, which would inline but prevents some procedures from being provided.
    ```odin
    sort_by :: proc(slice: []$T, $cmp: proc(lhs, rhs: T) -> bool) {
        // ...
    }
    ```
    Or in my opinion, a constrained type like this which would work for anything and still inline would be best.
    ```odin
    sort_by :: proc(slice: []$T, cmp: $P)
      where intrinsics.type_is_proc(P) &&
            intrinsics.type_proc_return_type(P, 0) == bool
    {
        // ...
    }
    ```
    But for some reason choses not to.

3. Runtimes bounds checking. Odin defaults to bounds checking all slice indexing. There's obviously a cost associated with this. This can be disabled on a per-scope or per-procedure basis with `#no_bounds_check`, or globally with a compiler flag. For release builds or for performance critical code you can work around it. It's just something to be aware of.

## Debugging
This is the one area of the current Odin compiler that requires serious work and is an unfortunate let-down in my experience. When I first started using Odin, executables could be debugged with the usual tools that I was familiar with as a systems programmer. I could inspect variables and step through code in `gdb`, which many would say is the bare minimum. What was more valuable to me though was that I could use `valgrind` to find memory leaks, `helgrind` to find data races and threading errors, `massif` to find pathological memory usage issues, even `cachegrind` & `callgrind` to find performance issues. They all worked really well. These tools are invaluable to me as a developer and I had used them to great success to improve EmberGen.

> If it isn't obvious by now, I do all development on Linux because I find the ecosystem around systems development superior to that of Windows which lacks all the tools I've mentioned thus far, there isn't even equivalent tools in that space. I also use `dtrace`, `strace` and `ltrace` in my day to day.

The Odin compiler isn't that stable however and over the past year the debugging experience for me personally has become so severely bad that none of the tools above work anymore, not even `gdb`. This is for various reasons, but most notably it's because generating debug information with LLVM (as the Odin compiler does) is pretty much impossible to do correctly. As of writing this, the Odin compiler generates debug information in the DWARF3 format in such a way that the debug symbols crash all the tools when doing a simple backtrace. To make matters worse, the act of generating debug information in LLVM appears to cause code to miscompile entirely and break the application in subtle ways. The mere fact LLVM can even emit broken debug information or miscompile code that uses it is completely absurd. We've not been able to fix this. If anyone with LLVM experience can improve the debugging situation and is looking for a job I believe the Odin author is offering money for this so you should reach out.

The other reason is because Odin is striving to become free-standing in the face of libc. That is there's a push to write the core library in such a way as to not depend on it at all. This is a noble goal, but it means all the existing debug tools which work by replacing libc no longer work or work as well. This means `valgrind` cannot really detect memory issues as well since Odin has it's own internal allocators. `helgrind` cannot work because Odin doesn't use `pthread` primitives. `ltrace` cannot work because that just provides wrappers over every libc function, etc.

This situation should improve overtime since Odin now has an intrinsic for Valgrind client requests, but all the core library needs to be riddled with the appropriate client calls to build up all the infrastructure that has already been built for libc. In some cases it may not actually be possible as `helgrind` assumes a specific pattern of indicating when threading primitives are initialized while Odin relies on zero-initialization of all threading primitives, meaning compiler instrumentation may be required. As of writing this, none of this infrastructure exists though.

> It should be noted that under Windows, none of these issues are a problem. This is because none of these tools exist there, but in the case of Visual Studio's debugger, the generated debug information appears to work correctly, albeit it's still not as good as C or C++.

## Compilation performance
The current Odin compiler compiles everything all the time. It does not currently support caching or pre-compilation of packages. Everything written in Odin can conceptually be thought of as being statically linked. As a result of this, the current Odin compiler is quite slow at compilation in my experience. It's worse than C and worse than carefully build-optimized C++ in some cases. This makes iteration times less than ideal.

## Personal opinion
Odin is a notable improvement over C and I would encourage anyone who wants something better than C but not too far off the deep end like C++, Rust, or Zig to try it. There's still a lot of work to be done in the compiler to address debugging and compilation performance but that can only improve over time.

Odin has various other features which I have not mentioned. There are some features I have not used or have no use for so obviously I cannot be objective about them or provide any meaningful insight into them. There are however features that I do use every day which I do so only because I have to that I do not personally like. So I want to take a moment to share my thoughts on these things.

### Packages
The organization of code in Odin uses a package system where a package is a directory. This eliminates an organizational tool for source files on disk since the directory introduces a new package to the language. This loss of file organization means you cannot organize code on disk with directories anymore and instead must resort to using filename suffixes or prefixes.

### Imports
Related to packages are that of package imports. Odin's import system only allows forming a singular directed acyclic graph of packages, rather than that of a strongly unambiguous graph. What this means is that in Odin, your imports must have no directed cycles and must be topologically orderable. This makes it rather impossible to share code upstream and only allows downstream importing. A [multitree](https://en.wikipedia.org/wiki/Multitree) import system would be easier to work with in my opinion.

### Procedure groups
Odin has explicit procedure overloading through the use of a procedure group. This is effectively a list of procedures that is accessible under a single procedure name. It's basically a hand-written overload resolution-set in C++ parlance. It's a genuinely cool feature except the current implementation is so misbehaved that it fails to correctly select the right procedure or finds the resolution ambiguous when it actually isn't. The procedure group resolution system needs to be completely rewritten in my opinion.

## Final remarks
One year and 50k lines later I don't have anything other to say aside from this. I'm sure you've seen Odin mentioned before on HackerNews, Reddit and maybe even Twitter and your Discord circles. There's a lot of really good articles out there explaining the pros and cons of Odin and how it differs from other languages like Rust and Zig. There's something of a renaissance in systems languages as of recent. What I rarely see and would like to see more of is some personal case studies of actually using the language for projects. A true language begins to show its faults only once you get past the honeymoon phase in my experience and there's few articles written about languages from this perspective. I hope this article gives a more mature and nuanced review of Odin for those looking to maybe try it.