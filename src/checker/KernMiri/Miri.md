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
