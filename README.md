
**THIS IS A WORK-IN-PROGRESS AS I READ THE WEBASSEMBLY SPECIFICATION**

# SOME CONCERNS ABOUT WEBASSEMBLY SECURITY AND DETERMINISM


Below are some concerns about WebAssembly security, especially in mission-critical situations and when the code may be adversarial.



## 1. SPEC-LEVEL


### Validation

> Wasm Spec Section 3.4: Modules are valid when all components they contain are valid.

> Wasm Spec Section 4.5.4: Instantiation: if module is not valid, then Fail.

> Wasm Spec Section 7.2.2: An implementation can defer validation of individual functions until they are first invoked. ... invalid functions result in a trap. ... the function must be validated before execution of its body begins.

The third quote breaks earlier requirements.


### IEEE 754-2008 Floating-point arithmetic

Floating-point operations may require special care for the following reasons.

(1) IEEE 754-2008 operations need not be associative, so modern architectures may allow out-of-order execution of floating-point instructions, which can round to different results. Some hardware has a bios switch to prevent out-of-order execution.

(2) `NaN` can to have an arbitrary non-zero significand.

(3) IEEE 754-2008 may be inefficient, so some hardware offer optional short-cuts. Some hardware does not even support IEEE754-2008.

A possible solution is to choose conformant hardware which executes instructions in-order, and to canonicalize `NaN`s. If that is not possible, it may be necessary to implement floating-point operations using integers.


### Non-deterministic behavior.

Some opcodes, such as `memory.grow`, are non-deterministic.



## 2. EMBEDDING-LEVEL

The Wasm spec does _not_ define an embedding environment, API, or ABI for Wasm.


### Not Following Spec Semantics

For speed optimizations, embeddings deviate from the spec's execution semantics. For example, many implementations use separate stacks for operands, control labels, and function call-frames. Proofs are needed that these modified semantics are equivalent to the spec semantics.


### Resource Exhaustion

The Wasm spec does not define a limit on code size or most runtime objects. So system resources can be exhausted. Resource availability may depend on the embedding environment and on the system.


### Breaking out of VM sandbox

JIT spraying is writing arbitrary instructions to memory, then somehow redirecting execution to those instructions. The common solution is for the operating system to mark pages as writable xor executable ("W^X"). This is relevant to any VM which generates and executes binary code. For example, Firefox 46 (2016) implements a W^X policy.


### Asynchronous execution

Many Javascript implementations of Wasm allow modules to be instantiated and executed asynchronously, which may not be desired.


### JITs

A modern JIT can include an interpreter, a basic compiler, and an optimizing compilers. For example, Firefox starts with a basic single-pass compiler from Wasm to native assembly, which allows execution to begin, while, in the background, the Ion compiler performs an optimized compilation.


Some JITs use profilers/tracers which heuristically trigger compilation of "hot" code. Compilation may involve an intermediate representation (IR) with optimization passes. These heuristics and optimizations may introduce bugs, or invariants which make it easy to introduce bugs. It may be difficult to prove correctness of something so complicated.


### Host Functions

In the specification sections 4.2.6 and 4.4.7, host functions are allowed to behave non-deterministically, with few constraints.






## 3. COMPILER-LEVEL


### Compiler bugs.

The translations, optimization passes, or register allocation algorithms may not preserve execution semantics. There are many subtleties in all of these areas.


### Undefined behavior.

A language with undefined behavior may be compiled to unwanted behavior in Wasm. Wasm only guarantees type-safety.


### Compiler run-time.

Some code may take especially long to compile, which could be used in a Denial-of-Service attack.




## 4. HARDWARE-LEVEL


### Bottlenecks, Denial of Service

If arbitrary code is to be executed, an attacker can exploit bottlenecks in hardware, causing slow execution. For example, an attacker can write a function with thousands or millions of local variables (using the shorthand vector notation in binary wasm) and call the function recursively, which will quickly exhaust cache or RAM.


### Hardware Bugs and Backdoors

Modern hardware is complicated (micro-ops, instruction queues, dependency chains, pipelining, etc, all in hardware). It is rumored that modern CPUs are too comlicated to be fully tested. Bugs have been found.

http://danluu.com/cpu-bugs/

There may also be backdoors.

http://danluu.com/cpu-backdoors/

Hardware documentation may be unclear and incomplete.

https://news.ycombinator.com/item?id=17037862


### Cosmic Rays

Cosmic rays may cause errors in electronics. The problem is worse with smaller transistors. ECC may be useful for RAM.





## 5. CONCLUSIONS

It may be wise to have redundancy and variety in both hardware and software. And to use simple toolchains which can be audited. And to somehow penalize every Denial-of-Service vulnerability.


