# POPL'26 Miri 论文阅读笔记

Paper: [Miri: Practical Undefined Behavior Detection for Rust](https://plf.inf.ethz.ch/research/popl26-miri.html)

## Detection Abilities

* Uninitialized memory
* Temporal and spatial memory safety violations
  * out-of-bounds
  * use-after-free accesses
* Memory leaks
* Data races
  * uses a pseudo-random number generator with a fixed seed to resolve
  non-deterministic choices, and it supports a mode where the program is
  executed with many different seeds concurrently. In this mode, Miri finds
  this data race for around 90% of the executions.
* Weak memory behaviors
  * limitation: relaxed accesses do not have to be observed in a consistent
  order by all threads, so non-deterministic bugs may need many different seeds
  to be found.
* Rust type invariants
  * like invalid bit pattern in bool
* Pointer provenance
  * provenance: every pointer has exactly one allocation that it may access,
  and using that pointer to do a read or write outside the bounds of that
  allocation is UB, even if that access happens to be inbounds of another
  allocation. This extra information attached to a pointer is called provenance
* Aliasing violations
  * violating the aliasing requirements of Rust’s reference types

## Architecture Overview

### Based On The Compiler

Reusing the entire rustc frontend and middle-end: the core interpreter is
shared with the Rust compiler’s compile-time function evaluation infrastructure

Omitted aspects for paper brevity:
* dynamically-sized types (i.e., slices and trait objects)
* improved error messages

MIR is a convenient choice:
* all control flow has been compiled to a simple control-flow graph
* most complicated Rust operations have been desugared into simple assignments and function calls
* pattern matching has been compiled into plain conditional jumps, saving us
the effort of having to entirely re-implement Rust’s non-trivial (and evolving)
pattern matching semantics
* still fully typed, giving Miri all the information it needs to detect
Undefined Behavior, including enforcing Rust type invariants
* turn off all MIR optimizations as those may remove UB

### Value and Operand

Operand:
* carry the type: the same mathematical integer may look very different in
memory depending on the type it is stored at
* two categories:
  * scalars
  * non-scalars: i.e. raw list of bytes; the only operations on them are
  loading them from somewhere in memory, and storing them to somewhere else in
  memory; no need more structures: for all other operations where more
  structure is needed, operands need to be converted to scalars. Pointer is an 
  address and provenance (known allocation ID or wildcard).
* copying operands does not always exactly preserve all the bytes. The padding
between fields should not be preserved on a copy, and pointer provenance is not
always perfectly preserved either. Miri post-processes the result of a copy to
erase any data that should not have been preserved

### Memory Model

Memory:
* means “everything a pointer can point to”
* does not just contain allocations that consist of bytes; in Rust, that also includes functions
* follows a typical block-based model in the style of [CompCert](https://inria.hal.science/hal-00703441)
* allocation example:
  ```
  # the first allocation (with ID 0) at address 16 is 4 bytes large, with the
  # first two bytes being initialized to 0 and the last two bytes being uninitialized
  0 ↦→ (16, DataAlloc({ data : [Init(0, ⊥), Init(0, ⊥), Uninit, Uninit] }))

  # the second allocation (with ID 1) at address 32 holds a pointer to the first allocation.
  # As a scalar, that pointer would be written Ptr({ addr : 16, prov : Known(0) })
  # (Miri is parametric in the pointer size and endianess; here we use the by far most common
  # 64-bit little-endian configuration, i.e., with the least significant byte first.)
  1 ↦→ (32, DataAlloc({ data : [Init(16, Known(0)), Init(0, Known(0)), . . . , Init(0, Known(0))] }))
  ```
* Exposed provenance and integer-pointer casts:
  * Miri implements a variant of the [PNVI] model that has been proposed for C,
  specifically a mix of PNVI-escaped (also known as PNVI-address-exposed /
  PNVI-ae) and PNVI-wildcard
  * every time a pointer is cast to an integer, the allocation it is derived
  from (as determined by its provenance) is marked as exposed by adding it to
  the exposed set
  * when an integer is cast to a pointer, we do not try to make a best guess
  about which allocation this pointer should be associated with—we just give it
  a Wildcard provenance
  * when a pointer with that provenance is used to access memory, we check
  whether some exposed allocation exists at the pointer’s address, and reject
  the memory access if not
  * This restriction to only permit accesses to exposed allocations makes the
  model different from PNVI-wildcard.
  * PNVI-ae and the variant PNVI-ae-udi adopted by C standard reject using the
  same pointer to access different exposed allocations, but this is too limited
  for Rust code and it's unclear how to combine PNVI-ae-udi with Rust's Stacked
  Borrows and Tree Borrows models; udi mechanism is not compatible with models
  where more than one provenance could be valid to use for any given memory
  access, and it matters which provenance is used
  * so cast integers to pointers are not considered fully supported by Miri,
  and we warn the user about this when such a cast happens
  * The wildcard mechanism means we might miss some bugs in such programs, but
  Miri is still useful for finding other bugs, and will not report false bugs.

[PNVI]: https://dl.acm.org/doi/10.1145/3290380

### Stack and Threads

Unwinding is explicit in MIR; leave a special marker on the stack frame of
the closure invoked by catch_unwind; unwind_payloads stack

Miri itself is a sequential program, but we can interpret concurrent programs
by tracking a separate stack for each thread, and all the associated per-thread
state. Execution randomly switches between threads to simulate an interleaving
concurrent execution.

### Soundness and Completeness

sound: all behaviors produced by Miri can really occur, and when Miri reports
UB the program really has UB

complete: all really observable behaviors can be found by Miri, and when a
given execution has UB, Miri will report an error

**Miri ensures soundness and only promise completeness for deterministic Rust
programs.**

* Need to handle discrepancy with regular program and compiler.
* To close the gaps in Rust’s specification, we model the current de-facto
Undefined Behavior: by inspecting how relevant language constructs get
compiled, and through extensive discussions with the Rust compiler team, we can
capture all the assumptions about the program that current versions of the
compiler make. These assumptions may change in the future, in which case we
will have to adjust Miri, but for today’s compilers, to the best of our
knowledge, Miri models all UB that can affect a program’s correctness.

## Data Race Detection

Miri implements that memory model by incorporating the C++ data race detector
by Lidbury and Donaldson, which is based on [FastTrack]. But adjustment is
needed, because the memory model in C++11/17/20 changed.

[FastTrack]: https://doi.org/10.1145/1839676.1839699

<details>

<summary>Events can be truly concurrent without being ordered either way, which
vector clocks represent by using a pointwise order—crucially, this order is not total.
</summary>

翻译：
事件可以真正地并发，而无需以任何方式排序，向量时钟通过使用逐点排序来表示这一点——至关重要的是，这种排序不是全序的。

理解：
1. **弱内存模型背景**：
   在计算机系统中，弱内存模型允许某些内存操作的顺序与程序员预期的顺序不同。这主要是为了提高并行处理的效率，但可能会导致一些难以预测的并发行为。


2. **真正并发**：
   - “真正并发”指的是多个事件（如线程中的操作）在时间上是同时发生的，而不是通过时间片轮转等方式模拟的并发。
   - 在弱内存模型中，这些并发事件可能不会按照程序员期望的顺序执行。

3. **向量时钟（Vector Clocks）**：
   - 向量时钟是一种用于检测分布式系统中事件因果关系的工具。
   - 每个进程维护一个向量，向量中的每个元素对应一个进程的逻辑时钟值。
   - 当一个事件发生时，该事件所属进程的向量时钟值会增加，同时其他进程的时钟值保持不变。

4. **逐点排序（Pointwise Order）**：
   - 向量时钟通过逐点比较来确定事件之间的顺序。
   - 如果事件 A 的向量时钟在每个维度上都小于或等于事件 B 的向量时钟，则认为 A 在 B 之前发生（A → B）。
   - 如果 A 和 B 的向量时钟在某些维度上相等，而在其他维度上不相等，则无法确定它们之间的顺序，这种情况下它们是并发的。

5. **非全序（Not Total Order）**：
   - 全序意味着对于任意两个事件，都可以明确地确定它们之间的顺序（即 A < B 或 B < A）。
   - 在弱内存模型中，由于事件可以真正并发，向量时钟无法为所有事件提供全序关系。也就是说，某些事件之间可能没有明确的先后顺序。

总结：
这句话强调了在弱内存模型中，事件可以真正并发，而向量时钟通过逐点排序来表示这种并发关系。这种排序不是全序的，意味着某些事件之间可能没有明确的先后顺序，这是弱内存模型的一个重要特性。

</details>

Detect basic non-atomic data races:
* On each read, we check which thread has most recently written to this
location. We look up that thread in our current clock to ensure that this write
occurred in our past. If it did not, this is a data race. We also update the
read clock, setting the clock’s value for the current thread to the latest
timestamp.
* On each write, we also check the most recent write as above. Furthermore, we
check that the entire read vector clock is in our past, thus ensuring that all
reads have been properly synchronized. If either condition is violated, this is
a data race. Finally, we update the write clock to our current clock and reset
the read clock to zero.

Model synchronization:
* synchronization occurs each time a happens-before relationship is established
between two threads
* lock-based synchronization as an example: When thread A releases the lock,
its current vector clock is stored with the lock. When thread B later acquires
that lock, it merges the lock’s clock into its current clock (by applying a
pointwise maximum)
* use FastTrack and "POPL'17 Dynamic race detection for C++11"
  [ref](https://doi.org/10.1145/3009837.3009857) algorithm

Novel detection on mixed atomic and non-atomic accesses:
* DataRaceALoc tracks, for each thread, when that thread performed its most
recent atomic read and write on this location
* sync clock is used for release-acquire synchronization: on a release write,
this gets reset to the writing thread’s current clock; and on an acquire read,
it gets merged into the reading thread’s current clock
* the field size is used to detect mixed-size atomic accesses: On the first
atomic access to some location, the size field is initialized to the size of
this access. Note that for multi-byte accesses, there is a separate DataRaceLoc
and with it a DataRaceALoc for each byte! Each byte now “knows” the size of the
atomic access it has been part of. Since atomic accesses must be aligned, this
uniquely identifies the exact memory range affected by the access (we can
simply round the address of this byte down to the previous multiple of size to
compute the first byte of the access). Later, if an atomic access happens-after
all prior atomic accesses (which can be checked using the read and write
clocks), we also freely reset the size, since there is no restriction for
different sizes when there are no concurrent (unsynchronized) accesses. The
interesting case is what happens for later non- synchronized accesses: an
atomic read access checks whether the size of the new access matches the
previously recorded size; if not, the size is reset to ⊥. This indicates that
the location may still be read by arbitrary atomic accesses, but no more writes
are possible. We also check whether the current thread’s clock is after the
write clock; if not, then there was an atomic write to this location that
conflicts with this differently-sized read, so we report UB. Finally, for
atomic writes, if the size field is ⊥ or does not match the size of the current
access, we also report UB.

Handle C++20 weak memory rules:
* coherence-ordered-before is introduced (很复杂)
* SC (sequential consistent) fences are strengthened
* release sequences are weakened
* limitations: Miri misses some bugs, but it will not report false bugs
  * have to solve the out-of-thin-air issue by ruling out cycles in the union
  of the program-order and reads-from relations
  * cannot generate executions that contain a cycle in the program-order and
  modification-order relations

## Practicality

More efficient representation of common cases:

| Improved        | Before                 | After                                             |
|-----------------|------------------------|---------------------------------------------------|
| Byte            | 257 values; provenance | bitvector; sparse provenance list                 |
| DataRaceLoc     | redundanct metadata    | DedupRangeMap: compact & dynamically split ranges |
| Operand         | large buffer copy      | indirect operands by delaying loading             |
| VClock          | hashmap overhead       | vector & reusable ID                              |
| local variables | many temporaries       | less Alloc stores                                 |

Rust programs need to call external functions:
* system libraries written in C: IO, synchronization primitives, and other
common OS facilities such as environment variables and thread-local state
* CPU-vendor-specific intrinsics: SSE and AVX families of intrinsics on x86
(Intel/AMD) CPUs

Miri is a fully cross-target interpreter: independent of the host OS. A user
can be on a Windows computer and invoke Miri with
`--target x86_64-unknown-linux-gnu`, and Miri will build and run the code as-if
it was running on Linux.

OS target capabilities:
* Linux: the best-supported target
* all POSIX targets are reasonably well supported: actively tested on macOS,
FreeBSD, and Illumos/Solaris
* Windows is supported, but lacks some of the operations whose equivalent Linux
functions are supported

OS APIs capabilities:
* Files and file descriptor/handle manipulation:
  * basic file operations like open, read, write, and close
  * POSIX pipes and streaming sockets
  * epoll and eventfd on Linux
  * network sockets are not supported yet
* File system manipulation:
  * POSIX listing directories and renaming and removing files
* Concurrency APIs:
  * spawning threads
  * pthread synchronization primitives
  * futex syscalls on Linux/MacOS/Windows/FreeBSD
  * interact with data race detector
* Thread-local state: 
  * TLS is supported to make `thread_local!` work
  * Dealing with TLS destructors are the difficulty
* Timekeeping:
  * current system time
  * thread sleep (and thread scheduling),
  * timeout and a general mechanism of “blocking a thread with a timeout”, and
  an associated “unblock callback”
  * sources of time (system clock and strictly monotone clock). Implement
* Intel vendor intrinsics:
  * SSE/AVX, cryptographic intrinsics through `std::arch`
  * and 100+ Intel-specific intrinsics
* The Rust standard library:
  * full implementation of the process environment to make env var work
  * obtain strong random numbers from OS to mitigate HashDoS attacks
