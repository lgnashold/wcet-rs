# Tock WCET Analysis Tooling

This repository contains code that uses a modified version of the Haybale symbolic execution
engine to find longest paths (in LLVM IR) through Rust code.

The code currently in this repository is rough research code, and has only been tested on a fork
of Tock that modifies the build system to generate the necessary LLVM IR and MIR files. This Tock
fork also contains some modifications to the Tock kernel necessary to bound symbolic execution. In general,
the changes to the Tock kernel should not modify the behavior, and instead should merely pass additional
information to the compiler (such as communicating loop bounds via assert statements). The Tock fork
is contained in a git submodule of this repository, as Tock uses a non-Cargo based build system and is not
published as a crate.

This code also uses a fork of Haybale that adds Rust-specific symbolic execution support,
allowing this tool to effectively handle code with dynamic dispatch of trait method calls.
This fork also makes some assumptions that are true for Tock, but not true for all Rust code:

- all linking is static, such that all implementations of a trait are available at compile time

- virtualizers in Tock do not call their own instances of a trait (e.g. if `<MuxAlarm as Alarm>::fired()`
  contains a call to `<dyn Alarm>::fired`, we assume that that concrete method being called is any
  implementation of the `Alarm::fired()` trait method except `<MuxAlarm as Alarm>::fired()`.
  Currently, this is broadly enforced for *all* trait object methods, as it was easiest to implement that
  way and I do not believe that any trait object methods in Tock include recursive calls to themselves.
  However, this can, and probably should, be updated to only work this way for virtualizers specifically,
  to reduce the scope of assumptions required.

This tool works for a set of system calls and interrupt handlers on certain Tock boards, but cannot yet
find longest paths for all system calls and interrupt handlers. Deriving actual execution times from
these longest paths requires additional tooling, such as verilator or a mechanism for converting LLVM IR
to platform-specific assembly, and using published instruction timing for that assembly. Currently, this
is future work.
